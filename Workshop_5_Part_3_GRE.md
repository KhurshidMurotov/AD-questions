# Workshop 5 — GRE Tunneling: Part 3 — OSPF Over the Tunnel

## Why Do We Need a Routing Protocol?

At the end of Part 2, we proved the GRE tunnel works — WEST and EAST can
ping each other's tunnel addresses (`172.16.12.x`). But PC-A still cannot
reach PC-C. Here is WEST's routing table right now:

```
     10.0.0.0/30 is subnetted, 2 subnets
C       10.1.1.0 is directly connected, Serial0/0/0
     172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.16.1.0/24 is directly connected, GigabitEthernet0/1
C       172.16.12.0/30 is directly connected, Tunnel0
S*   0.0.0.0/0 [1/0] via 10.1.1.2
```

WEST knows about its own LAN and the tunnel — but has **no entry for
`172.16.2.0/24`** (EAST's LAN). EAST has the same problem in reverse.

We could add static routes, but that doesn't scale. Instead, we run
OSPF *inside the tunnel*. Both routers advertise their LAN and tunnel
networks into OSPF area 0. OSPF exchanges this information through
the Tunnel0 interface, and each router learns about the other's LAN
automatically.

The key insight: OSPF network statements include the **tunnel network**
(`172.16.12.0/30`) and the **LAN network** — but NOT the serial
networks (`10.x.x.x`). The serial links are the ISP's domain. The ISP
does not participate in OSPF at all.

---

## What OSPF Will Advertise

```
WEST advertises into OSPF:              EAST advertises into OSPF:
  172.16.1.0/24  (its LAN)               172.16.2.0/24  (its LAN)
  172.16.12.0/30 (tunnel)                 172.16.12.0/30 (tunnel)
        │                                       │
        └──── exchanged through Tunnel0 ────────┘

After OSPF converges:
  WEST learns: 172.16.2.0/24 via Tunnel0    ← new!
  EAST learns: 172.16.1.0/24 via Tunnel0    ← new!
```

---

## Step 1 — Configure OSPF

### WEST Router

```
enable
configure terminal
!
router ospf 1
 network 172.16.1.0 0.0.0.255 area 0
 network 172.16.12.0 0.0.0.3 area 0
!
end
```

> **router ospf 1** — starts OSPF process number 1. The `1` is just a
> local label — it does not need to be the same number on both routers.
> WEST could use `1` and EAST could use `99` and they would still form
> a neighbor relationship. We use `1` on both simply for neatness.
>
> **network 172.16.1.0 0.0.0.255 area 0** — tells OSPF: "any interface
> whose IP falls in `172.16.1.0/24`, enable OSPF on it and advertise
> that network." This matches G0/1 (172.16.1.1).
>
> **network 172.16.12.0 0.0.0.3 area 0** — matches Tunnel0 (172.16.12.1).
> The `0.0.0.3` is the **wildcard mask** for a /30 subnet. OSPF uses
> wildcard masks instead of subnet masks — they are the inverse. To
> convert: subtract each octet of the subnet mask from 255.
> Subnet mask `255.255.255.252` → wildcard `0.0.0.3` (255-252=3).
> Similarly, `255.255.255.0` → wildcard `0.0.0.255` (255-0=255).
> This network statement is what enables OSPF to run *through* the
> tunnel and find the EAST router.
>
> **Why not `network 10.1.1.0 0.0.0.3 area 0`?** Because the serial link
> to the ISP is not part of our OSPF domain. The ISP doesn't run OSPF.
> Including it would cause OSPF to send hello packets out S0/0/0 for
> nothing, and would advertise the ISP link to EAST, which is unnecessary.

### EAST Router

```
enable
configure terminal
!
router ospf 1
 network 172.16.2.0 0.0.0.255 area 0
 network 172.16.12.0 0.0.0.3 area 0
!
end
```

> Same logic: advertise the LAN (`172.16.2.0/24` via G0/1) and the
> tunnel (`172.16.12.0/30` via Tunnel0). No serial networks.

After entering these commands, you may see a console message on whichever
router's CLI you are watching (it can appear on both):

```
%OSPF-5-ADJCHG: Process 1, Nbr 172.16.12.2 on Tunnel0 from LOADING to FULL
```

This means WEST and EAST have formed an OSPF neighbor relationship
through the tunnel and are exchanging routes. If you don't see this
message within 30 seconds on either router, verify that both routers
have the tunnel network statement and that Tunnel0 is up/up on both
sides.

---

## Step 2 — Verify OSPF Routing

### Check the routing table on WEST

```
WEST# show ip route
```

Expected output:

```
     10.0.0.0/30 is subnetted, 2 subnets
C       10.1.1.0 is directly connected, Serial0/0/0
     172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
C       172.16.1.0/24 is directly connected, GigabitEthernet0/1
O       172.16.2.0/24 [110/1001] via 172.16.12.2, 00:00:30, Tunnel0
C       172.16.12.0/30 is directly connected, Tunnel0
S*   0.0.0.0/0 [1/0] via 10.1.1.2
```

The new line is:

```
O       172.16.2.0/24 [110/1001] via 172.16.12.2, 00:00:30, Tunnel0
```

| Part               | Meaning                                                |
|--------------------|--------------------------------------------------------|
| `O`                | Learned via OSPF                                       |
| `172.16.2.0/24`    | EAST's LAN — the network we couldn't reach before      |
| `[110/1001]`       | Administrative distance 110 (OSPF), metric 1001        |
| `via 172.16.12.2`  | Next hop is EAST's tunnel IP                           |
| `Tunnel0`          | Traffic exits through the tunnel interface              |

> The high metric (1001) is because GRE tunnels have a default bandwidth
> of 100 Kbps in Cisco IOS. The OSPF cost is calculated from this low
> bandwidth. In a real network you would tune this, but for our lab it
> doesn't matter — there's only one path.

### Check the routing table on EAST

```
EAST# show ip route
```

Expected output:

```
     10.0.0.0/30 is subnetted, 2 subnets
C       10.2.2.0 is directly connected, Serial0/0/1
     172.16.0.0/16 is variably subnetted, 3 subnets, 2 masks
O       172.16.1.0/24 [110/1001] via 172.16.12.1, 00:00:45, Tunnel0
C       172.16.2.0/24 is directly connected, GigabitEthernet0/1
C       172.16.12.0/30 is directly connected, Tunnel0
S*   0.0.0.0/0 [1/0] via 10.2.2.2
```

The mirror: EAST now has an `O` route to `172.16.1.0/24` via
`172.16.12.1` through Tunnel0.

### Verify OSPF neighbors

```
WEST# show ip ospf neighbor
```

Expected:

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
172.16.12.2       0   FULL/  -        00:00:30    172.16.12.2     Tunnel0
```

> The Neighbor ID is EAST's router ID. By default, Cisco routers pick the
> highest IP address on any active interface as their router ID. EAST's
> active interfaces are G0/1 (172.16.2.1), S0/0/1 (10.2.2.1), and
> Tunnel0 (172.16.12.2) — so the router ID is `172.16.12.2`.
> State `FULL` means the routers have fully exchanged their databases.

---

## Step 3 — Test End-to-End Connectivity

### PC-A to PC-C

From **PC-A** (Desktop → Command Prompt):

```
C:\> ping 172.16.2.3
```

Expected:

```
Pinging 172.16.2.3 with 32 bytes of data:

Reply from 172.16.2.3: bytes=32 time=5ms TTL=126
Reply from 172.16.2.3: bytes=32 time=4ms TTL=126
Reply from 172.16.2.3: bytes=32 time=4ms TTL=126
Reply from 172.16.2.3: bytes=32 time=3ms TTL=126

Ping statistics for 172.16.2.3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

> If the first ping times out but the rest succeed, that is normal —
> ARP resolution on the first packet. Ping again for a clean 4/4.
>
> **TTL=126** — the packet starts at TTL=128 (Windows default) and
> crosses two Layer 3 hops: WEST router and EAST router. 128 - 2 = 126.
> "But what about the ISP — isn't that a third hop?" No. The ISP
> decrements the TTL of the **outer** GRE packet, not the inner one.
> The ISP never sees or touches the original packet inside. Only WEST
> (at tunnel entry) and EAST (at tunnel exit) decrement the inner TTL.
> This is another way to confirm the tunnel is working correctly.

### PC-C to PC-A

From **PC-C**:

```
C:\> ping 172.16.1.3
```

Same result — 4/4 success, TTL=126.

### Traceroute from PC-A to PC-C

```
C:\> tracert 172.16.2.3
```

Expected:

```
Tracing route to 172.16.2.3 over a maximum of 30 hops:

  1     1 ms      0 ms      0 ms    172.16.1.1
  2     5 ms      4 ms      4 ms    172.16.12.2
  3     5 ms      4 ms      3 ms    172.16.2.3
```

The path tells the full story:

| Hop | IP           | What it is                              |
|-----|--------------|-----------------------------------------|
| 1   | 172.16.1.1   | WEST router (PC-A's default gateway)    |
| 2   | 172.16.12.2  | EAST's tunnel endpoint                  |
| 3   | 172.16.2.3   | PC-C (destination)                      |

> Notice the ISP (`10.1.1.2` / `10.2.2.2`) does **not** appear in the
> traceroute. From the private network's perspective, traffic goes
> directly from WEST into the tunnel and out at EAST. The ISP is
> completely invisible — it just carries the outer GRE packets without
> knowing what's inside.

---

## The Complete Picture

```
   PC-A                                                     PC-C
172.16.1.3                                              172.16.2.3
     │                                                       │
     │ Step 1: PC-A sends to default gateway                 │
     ▼                                                       │
┌──────────┐                                          ┌──────────┐
│   WEST   │                                          │   EAST   │
│          │  Step 2: WEST looks up 172.16.2.3        │          │
│ OSPF says│  in routing table → OSPF says go         │ OSPF says│
│ 172.16.2 │  via 172.16.12.2 (Tunnel0)              │ 172.16.1 │
│ via Tun0 │                                          │ via Tun0 │
│          │  Step 3: GRE wraps packet:               │          │
│          │  outer: 10.1.1.1 → 10.2.2.1              │          │
│          │  inner: 172.16.1.3 → 172.16.2.3          │          │
└────┬─────┘                                          └────┬─────┘
     │                                                     ▲
     │  Step 4: ISP routes outer packet                    │
     │  (sees only 10.x.x.x addresses)                    │
     ▼                                                     │
┌──────────┐                                               │
│   ISP    │───────────────────────────────────────────────┘
│          │  Step 5: EAST receives, strips GRE header,
└──────────┘  finds original packet, forwards to PC-C
```

---

## Research Questions (Answered)

The original workshop asks two portfolio questions. Here are the answers:

**1. What is a tunnel used for?**

A tunnel creates a virtual point-to-point link between two networks
across an intermediate network (like the Internet) that doesn't need
to know about the private addresses. Common uses: connecting branch
offices over the Internet, carrying protocols that the transit network
doesn't support (e.g. IPv6 over an IPv4-only ISP), and creating
overlays for private routing domains.

**2. How would you make it secure?**

GRE by itself provides **no encryption** — the outer packet hides the
inner addresses from the ISP, but anyone who captures the outer packet
can strip the GRE header and read the original data. To secure it, you
wrap GRE inside **IPsec** (Internet Protocol Security). IPsec encrypts
and authenticates the entire GRE packet before sending it. The typical
combination is called a **GRE over IPsec tunnel**. This gives you both
the flexibility of GRE (carrying any protocol) and the security of
IPsec (confidentiality and integrity).

---

## Summary — What We Built

```
  172.16.1.0/24                                       172.16.2.0/24
  ┌───────┐      ┌──────┐ ═══GRE Tunnel═══ ┌──────┐      ┌───────┐
  │ PC-A  ├──S1──│ WEST ├──────ISP─────────│ EAST ├──S3──│ PC-C  │
  └───────┘      └──────┘    (unaware)     └──────┘      └───────┘

  ✓ Physical connectivity through ISP          (Part 1)
  ✓ GRE tunnel up between WEST and EAST        (Part 2)
  ✓ OSPF running inside the tunnel              (Part 3)
  ✓ PC-A ↔ PC-C full end-to-end connectivity   (Part 3)
  ✓ ISP sees only 10.x.x.x — private networks are hidden
```

**DEMONSTRATE TO YOUR TUTOR**
