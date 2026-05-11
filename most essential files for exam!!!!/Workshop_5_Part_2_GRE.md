# Workshop 5 — GRE Tunneling: Part 2 — The GRE Tunnel

## What Is a GRE Tunnel?

At the end of Part 1, we proved that WEST and EAST can reach each other's
serial interfaces through the ISP — but PC-A cannot reach PC-C because the
ISP has no idea the `172.16.x.x` networks exist.

GRE (Generic Routing Encapsulation) solves this by wrapping private packets
inside public ones. When WEST wants to send a `172.16.1.3 → 172.16.2.3`
packet, GRE wraps it inside a new packet with source `10.1.1.1` and
destination `10.2.2.1` — addresses the ISP *does* know. The ISP delivers
the outer packet normally. EAST unwraps it and finds the original inside.

```
Original packet (private, ISP can't route this):
┌─────────────────────────────────────────────┐
│ Src: 172.16.1.3  →  Dst: 172.16.2.3        │
│ [data]                                      │
└─────────────────────────────────────────────┘

After GRE encapsulation (ISP sees only the outer header):
┌─────────────────────────────────────────────────────────────────┐
│ Outer IP Header                │ GRE Header │ Original Packet   │
│ Src: 10.1.1.1 → Dst: 10.2.2.1 │            │ (unchanged inside)│
└─────────────────────────────────────────────────────────────────┘
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^                ^^^^^^^^^^^^^^^^^^
  ISP reads this to forward       │              ISP never sees this
  the packet to 10.2.2.1         added by
                                  GRE
```

To make this work, each router needs a **Tunnel0** interface — a virtual
interface that acts as the entry/exit point of the tunnel. You configure
three things on it:

- **IP address** — the tunnel's own address (from the `172.16.12.0/30` subnet)
- **Tunnel source** — which local physical interface to send the outer packet from
- **Tunnel destination** — the remote router's physical IP address

---

## Tunnel Addressing

From the addressing table in Part 1:

| Device | Interface | IP Address  | Subnet Mask     |
|--------|-----------|-------------|-----------------|
| WEST   | Tunnel0   | 172.16.12.1 | 255.255.255.252 |
| EAST   | Tunnel0   | 172.16.12.2 | 255.255.255.252 |

And the mapping between tunnel endpoints and physical interfaces:

```
WEST                                                    EAST
Tunnel0: 172.16.12.1                    Tunnel0: 172.16.12.2
    │                                       │
    ▼                                       ▼
Source: S0/0/0 (10.1.1.1)      Source: S0/0/1 (10.2.2.1)
    │                                       │
    └──────── ISP routes this ──────────────┘
Destination: 10.2.2.1            Destination: 10.1.1.1
```

Notice the symmetry: WEST's tunnel destination is EAST's serial IP, and
EAST's tunnel destination is WEST's serial IP.

---

## Step 1 — Configure the GRE Tunnel

### WEST Router

```
enable
configure terminal
!
interface Tunnel0
 ip address 172.16.12.1 255.255.255.252
 tunnel source Serial0/0/0
 tunnel destination 10.2.2.1
!
end
```

> **tunnel source Serial0/0/0** — tells the router to use S0/0/0's IP
> (10.1.1.1) as the outer packet's source address. You can also write
> `tunnel source 10.1.1.1` — both forms work. Using the interface name
> is preferred because if the IP ever changes, the tunnel updates
> automatically.
>
> **tunnel destination 10.2.2.1** — the outer packet's destination.
> This is EAST's serial interface — a public address the ISP can route to.
> For the destination, you must use the IP address (not the interface name,
> since it's on a remote router).

### EAST Router

```
enable
configure terminal
!
interface Tunnel0
 ip address 172.16.12.2 255.255.255.252
 tunnel source Serial0/0/1
 tunnel destination 10.1.1.1
!
end
```

> Mirror image: EAST's tunnel source is its own serial, and the destination
> is WEST's serial IP.

No `no shutdown` is needed — unlike physical interfaces (which default
to `administratively down`), tunnel interfaces are virtual. They have no
physical link to be "down." A tunnel comes up automatically once its
source interface is up and its destination is reachable.

---

## Step 2 — Verify the Tunnel Is Up

### Check interface status

On **WEST**:

```
WEST# show ip interface brief
```

Expected output:

```
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  administratively down down
GigabitEthernet0/1     172.16.1.1      YES manual up                    up
Serial0/0/0            10.1.1.1        YES manual up                    up
Serial0/0/1            unassigned      YES unset  administratively down down
Tunnel0                172.16.12.1     YES manual up                    up
```

> The key line is **Tunnel0** showing `up` / `up`. If it shows `up` / `down`,
> the tunnel destination is unreachable — go back to Part 1 and verify
> default routes and serial connectivity.

On **EAST**:

```
EAST# show ip interface brief
```

Expected output:

```
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  administratively down down
GigabitEthernet0/1     172.16.2.1      YES manual up                    up
Serial0/0/0            unassigned      YES unset  administratively down down
Serial0/0/1            10.2.2.1        YES manual up                    up
Tunnel0                172.16.12.2     YES manual up                    up
```

### Check tunnel details

On **WEST**:

```
WEST# show interfaces tunnel 0
```

Key lines to look for in the output:

```
Tunnel0 is up, line protocol is up (connected)
  ...
  Tunnel source 10.1.1.1 (Serial0/0/0), destination 10.2.2.1
  Tunnel protocol/transport GRE/IP
```

This confirms three things:

| Field                  | Value          | Meaning                              |
|------------------------|----------------|--------------------------------------|
| Tunnel source          | 10.1.1.1       | Using WEST's serial IP               |
| Tunnel destination     | 10.2.2.1       | Pointing at EAST's serial IP         |
| Tunnel protocol        | GRE/IP         | Standard GRE encapsulation over IPv4 |

Run the same command on **EAST**:

```
EAST# show interfaces tunnel 0
```

Key lines to look for:

```
Tunnel0 is up, line protocol is up (connected)
  ...
  Tunnel source 10.2.2.1 (Serial0/0/1), destination 10.1.1.1
  Tunnel protocol/transport GRE/IP
```

Notice the source and destination are reversed compared to WEST — this
is the other end of the same tunnel.

---

## Step 3 — Test the Tunnel

### Ping across the tunnel

From **WEST**:

```
WEST# ping 172.16.12.2
```

Expected:

```
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.12.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/3/5 ms
```

> If your first attempt shows `.!!!!` (one timeout then four replies),
> that is normal — the first packet may be lost while the router resolves
> the tunnel path. Ping again and you should see `!!!!!` (5/5).

This ping travels *through* the GRE tunnel, not directly over the serial
link. The router encapsulates it, the ISP forwards the outer packet, and
EAST decapsulates it.

### Traceroute across the tunnel

From **WEST**:

```
WEST# traceroute 172.16.12.2
```

Expected:

```
Type escape sequence to abort.
Tracing the route to 172.16.12.2

  1   172.16.12.2     4 msec    2 msec    3 msec
```

> **Only one hop** — even though the packet physically passes through the
> ISP router. From the tunnel's perspective, WEST and EAST are directly
> connected. The ISP is invisible. This is the whole point of tunneling:
> the private network sees a direct link, while the real path underneath
> may cross multiple routers.

### What still does NOT work

From **PC-A**:

```
C:\> ping 172.16.2.3
```

Expected: **Request timed out.** The tunnel is up, but the routers don't
yet know that `172.16.2.0/24` is reachable *through* the tunnel. To see
why, check WEST's routing table:

```
WEST# show ip route
```

You will see something like:

```
     10.0.0.0/30 is subnetted, 2 subnets
C       10.1.1.0 is directly connected, Serial0/0/0
     172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.16.1.0/24 is directly connected, GigabitEthernet0/1
C       172.16.12.0/30 is directly connected, Tunnel0
S*   0.0.0.0/0 [1/0] via 10.1.1.2
```

WEST knows about its own LAN (`172.16.1.0/24`) and the tunnel network
(`172.16.12.0/30`) — but there is **no entry for `172.16.2.0/24`**.
WEST has no idea that EAST's LAN exists on the other side of the tunnel.
We have a pipe, but no routing protocol running inside it to advertise
the LANs.

**Next: Part 3 — OSPF Over the Tunnel** will make the routers exchange
LAN routes through the tunnel, completing the full path from PC-A to PC-C.

---

## Summary — What We Have Now

```
              172.16.12.0/30 (tunnel network)
  172.16.1.0/24  ┌──────┐ ═══════════════ ┌──────┐  172.16.2.0/24
  ┌───────┐      │ WEST │   GRE Tunnel    │ EAST │      ┌───────┐
  │ PC-A  ├──────│      ├─────ISP─────────│      ├──────│ PC-C  │
  └───────┘      └──────┘   (unaware)     └──────┘      └───────┘

  ✓ Tunnel0 is up on both sides
  ✓ Routers can ping each other through the tunnel (172.16.12.x)
  ✓ ISP sees only outer 10.x.x.x packets — private addresses are hidden
  ✗ PC-A still cannot reach PC-C — no routing inside the tunnel yet
```
