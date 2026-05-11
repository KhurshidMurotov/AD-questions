# Workshop 5 — GRE Tunneling: Part 1 — Build and Connect

## What Are We Building?

Two offices — WEST and EAST — need their private networks to communicate.
The problem: they are connected through an ISP that knows nothing about their
internal addresses. The solution: a GRE tunnel — a virtual pipe through the
ISP that lets WEST and EAST talk directly, as if the ISP wasn't there.

Before we can build the tunnel (Part 2), we need the physical foundation:
devices, cables, IP addresses, and basic routing to the ISP.

---

## Topology

Refer to the topology diagram provided with this workshop. Here is a text
reference you can use alongside it:

```
                              ┌───────────┐
                              │    ISP    │
                              │  Router   │
                              └─────┬─────┘
                          S0/0/0 ───┴─── S0/0/1 (DCE)
                         10.1.1.2       10.2.2.2
                              │           │
                              │           │
                 S0/0/0 (DCE) │           │ S0/0/1
                    10.1.1.1  │           │  10.2.2.1
                        ┌─────┴──┐    ┌──┴─────┐
                        │  WEST  │════│  EAST  │
                        │ Router │    │ Router │
                        └───┬────┘    └────┬───┘
                      G0/1  │              │  G0/1
                  172.16.1.1│              │172.16.2.1
                            │              │
                    F0/5 ┌──┴──┐        ┌──┴──┐ F0/5
                    ─────│ S1  │        │ S3  │─────
                         └──┬──┘        └──┬──┘
                    F0/6    │              │    F0/18
                            │              │
                        ┌───┴───┐      ┌───┴───┐
                        │ PC-A  │      │ PC-C  │
                        │.1.3   │      │.2.3   │
                        └───────┘      └───────┘

              ── Serial link       ══ GRE Tunnel (built in Part 2)
```

The two serial links through the ISP are the **real, physical** path.
The GRE tunnel (double line) is **logical** — it doesn't exist yet.
We build it in Part 2.

---

## Addressing Table

| Device | Interface     | IP Address   | Subnet Mask     | Default Gateway |
|--------|---------------|--------------|-----------------|-----------------|
| WEST   | G0/1          | 172.16.1.1   | 255.255.255.0   | —               |
| WEST   | S0/0/0 (DCE)  | 10.1.1.1     | 255.255.255.252 | —               |
| WEST   | Tunnel0       | 172.16.12.1  | 255.255.255.252 | —               |
| ISP    | S0/0/0        | 10.1.1.2     | 255.255.255.252 | —               |
| ISP    | S0/0/1 (DCE)  | 10.2.2.2     | 255.255.255.252 | —               |
| EAST   | G0/1          | 172.16.2.1   | 255.255.255.0   | —               |
| EAST   | S0/0/1        | 10.2.2.1     | 255.255.255.252 | —               |
| EAST   | Tunnel0       | 172.16.12.2  | 255.255.255.252 | —               |
| PC-A   | NIC           | 172.16.1.3   | 255.255.255.0   | 172.16.1.1      |
| PC-C   | NIC           | 172.16.2.3   | 255.255.255.0   | 172.16.2.1      |

Notice three separate IP networks:

- `10.1.1.0/30` — the link between WEST and ISP
- `10.2.2.0/30` — the link between ISP and EAST
- `172.16.x.x` — all private addresses (LANs and the future tunnel)

The ISP only knows about the `10.x.x.x` addresses. It has no idea
`172.16.x.x` even exists. That's the whole point of the tunnel.

---

## Step 1 — Build the Topology in Packet Tracer

Open a new blank Packet Tracer project and place the following devices:

| Device | Model         | Notes                      |
|--------|---------------|----------------------------|
| WEST   | Router 1941   | Has GigabitEthernet and Serial |
| ISP    | Router 1941   |                            |
| EAST   | Router 1941   |                            |
| S1     | Switch 2960   |                            |
| S3     | Switch 2960   |                            |
| PC-A   | PC            |                            |
| PC-C   | PC            |                            |

### Cabling

| From             | To              | Cable Type          |
|------------------|-----------------|---------------------|
| WEST S0/0/0      | ISP S0/0/0      | Serial DCE          |
| ISP S0/0/1       | EAST S0/0/1     | Serial DCE          |
| WEST G0/1        | S1 F0/5         | Straight-through    |
| S1 F0/6          | PC-A NIC        | Straight-through    |
| EAST G0/1        | S3 F0/5         | Straight-through    |
| S3 F0/18         | PC-C NIC        | Straight-through    |

> **DCE side** means that end provides the clock signal. In Packet Tracer
> the DCE end is whichever side you plug the "DCE" connector into.
> According to our topology: **WEST S0/0/0** is DCE and **ISP S0/0/1** is DCE.
> These are the interfaces where we set the clock rate.

---

## Step 2 — Configure IP Addresses

Every interface needs its IP address from the addressing table, and every
serial DCE interface needs a clock rate. Work through each device below.

### WEST Router

```
enable
configure terminal
hostname WEST
!
interface GigabitEthernet0/1
 ip address 172.16.1.1 255.255.255.0
 no shutdown
!
interface Serial0/0/0
 ip address 10.1.1.1 255.255.255.252
 clock rate 128000
 no shutdown
!
end
```

> **Why clock rate?** Serial links need a clocking signal to synchronize
> data. In real life, the telecom provider's equipment (CSU/DSU) provides
> this. In our lab, we simulate it by setting `clock rate` on the DCE side.

### ISP Router

```
enable
configure terminal
hostname ISP
!
interface Serial0/0/0
 ip address 10.1.1.2 255.255.255.252
 no shutdown
!
interface Serial0/0/1
 ip address 10.2.2.2 255.255.255.252
 clock rate 128000
 no shutdown
!
end
```

### EAST Router

```
enable
configure terminal
hostname EAST
!
interface GigabitEthernet0/1
 ip address 172.16.2.1 255.255.255.0
 no shutdown
!
interface Serial0/0/1
 ip address 10.2.2.1 255.255.255.252
 no shutdown
!
end
```

### PC-A

Open PC-A → Desktop → IP Configuration:

```
IP Address:      172.16.1.3
Subnet Mask:     255.255.255.0
Default Gateway: 172.16.1.1
```

### PC-C

Open PC-C → Desktop → IP Configuration:

```
IP Address:      172.16.2.3
Subnet Mask:     255.255.255.0
Default Gateway: 172.16.2.1
```

---

## Step 3 — Configure Default Static Routes

Right now, WEST knows about its own two networks (`172.16.1.0` and
`10.1.1.0`) but has no idea how to reach anything beyond the ISP.
Same for EAST. We fix this with a **default route** — a rule that says
"if you don't know where to send it, forward it to the ISP."

### WEST Router

```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 10.1.1.2
end
```

> This says: for any destination (`0.0.0.0 0.0.0.0`) I don't have a
> specific route for, send it to `10.1.1.2` (the ISP's nearest interface).

### EAST Router

```
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 10.2.2.2
end
```

> Same logic: send unknown traffic to `10.2.2.2` (the ISP's nearest interface).

The ISP does **not** need a default route. It is directly connected to
both `10.1.1.0/30` and `10.2.2.0/30`, so it already knows how to reach
both WEST's and EAST's serial interfaces.

---

## Step 4 — Verify Basic Connectivity

Now test that the physical network works before we build the tunnel.

### Test 1 — PCs to their gateways

From **PC-A** (Desktop → Command Prompt):

```
C:\> ping 172.16.1.1
```

Expected output:

```
Pinging 172.16.1.1 with 32 bytes of data:

Reply from 172.16.1.1: bytes=32 time<1ms TTL=255
Reply from 172.16.1.1: bytes=32 time<1ms TTL=255
Reply from 172.16.1.1: bytes=32 time<1ms TTL=255
Reply from 172.16.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 172.16.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

From **PC-C**:

```
C:\> ping 172.16.2.1
```

Same pattern — 4 replies, 0% loss.

### Test 2 — Router serial interfaces across the ISP

From **WEST**:

```
WEST# ping 10.1.1.2
```

Expected:

```
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```

```
WEST# ping 10.2.2.1
```

Expected: same — 5/5 success. This proves WEST can reach EAST's serial
interface through the ISP using the default route.

From **EAST**:

```
EAST# ping 10.2.2.2
EAST# ping 10.1.1.1
```

Both should return 5/5 success.

### Test 3 — What does NOT work yet

From **PC-A**:

```
C:\> ping 172.16.2.3
```

Expected: **Request timed out.** This fails because the ISP has no route
to `172.16.2.0` — it doesn't know that network exists. This is exactly
the problem the GRE tunnel will solve in Part 2.

---

## Summary — What We Have So Far

```
   172.16.1.0/24          10.1.1.0/30      10.2.2.0/30        172.16.2.0/24
  ┌───────┐  ┌──────┐  ┌───────────┐  ┌──────┐  ┌───────┐
  │ PC-A  ├──│ WEST ├──│    ISP    ├──│ EAST ├──│ PC-C  │
  └───────┘  └──────┘  └───────────┘  └──────┘  └───────┘
     ✓ works    ✓ works    ✓ works    ✓ works    ✓ works
              ←──────── routers can ping each other ────────→
  ✗ PC-A cannot reach PC-C — ISP doesn't know about 172.16.x.x
```

Everything physical is in place. The routers can reach each other over
the ISP. But the private LANs are still isolated.

**Next: Part 2 — The GRE Tunnel** will create the virtual pipe that
bridges the two private networks through the ISP.
