# Workshop 6 — Autonomous Systems & eBGP: Companion Guide (Part 1 of 3)

## ⚠️ Disclaimer

This companion guide is an **unofficial supplementary resource** prepared by your course tutor.
It is **not** a replacement for the original workshop brief issued by the module leader (Ian Coulson, University of Wolverhampton).
Always refer to the original workshop document as the authoritative source.
Use this guide alongside it to fill in gaps and deepen your understanding.

Recommended reading tool: [Obsidian](https://obsidian.md) (free). Alternative: VS Code.

---

## What This Workshop Is About (Big Picture)

Up to this point in the module, every routing protocol you have seen — static routes, RIP, OSPF, EIGRP — has been about routing **inside** a single organisation's network. That is called **interior routing**.

This workshop is about **exterior routing** — how completely separate organisations pass traffic between each other to form the Internet.

By the end of this lab you will have:

- Two separate organisations (a Company and an ISP) each running as their own **Autonomous System**
- A single **eBGP** peering session connecting them
- End-to-end reachability from a Company router to a web server "somewhere on the Internet"

Before you touch Packet Tracer, read the theory sections below. Everything you configure in Parts 2 and 3 flows directly from these concepts.

---

## Core Theory: Autonomous Systems

### What is an Autonomous System (AS)?

- An Autonomous System is a **large network (or group of networks) under a single administrative control**
- "Single administrative control" means one organisation sets the routing policy — the University of Wolverhampton is one AS, BT is another, Google is another
- Every AS is assigned a unique **AS Number (ASN)** so that it can be identified on the Internet
- Think of it this way: your home router has an IP address to identify it on a network; an entire organisation has an ASN to identify it among other organisations

### AS Numbers: Private vs Public

- **Public ASN**: assigned by a Regional Internet Registry (ARIN, RIPE, etc.) — used on the real Internet
- **Private ASN range**: 64512–65534 — used for labs, testing, and internal purposes (similar concept to private IP addresses like 192.168.x.x)
- In this workshop, both AS numbers (65000 and 65001) fall within the **private range** — this is normal for a lab environment

### Why Do Autonomous Systems Exist?

- The Internet is too large for any single routing protocol to handle
- Each organisation needs to make its own internal routing decisions independently
- AS boundaries create a clean separation: inside your AS, you run whatever interior protocol you like (OSPF, EIGRP, etc.); between ASes, everyone speaks the same exterior protocol
- That exterior protocol is **BGP**

---

## Core Theory: BGP (Border Gateway Protocol)

### What Problem Does BGP Solve?

- Interior protocols (OSPF, EIGRP, RIP) are designed to find the best path **within** a network
- They do not scale to the entire Internet (hundreds of thousands of networks)
- They also cannot express **policy** — an organisation might prefer to route traffic through a cheaper provider even if a faster path exists
- BGP solves both problems: it handles Internet-scale routing and lets each AS set its own routing policy

### Key Facts About BGP

- BGP is a **path-vector** protocol — it does not just track cost/metric like OSPF or EIGRP; it tracks the **full path of AS numbers** traffic must cross to reach a destination
- BGP peers (neighbours) form a **TCP session on port 179** — this is unlike OSPF/EIGRP which use their own transport; BGP rides on reliable TCP
- BGP neighbours must be **explicitly configured** — there is no auto-discovery; you manually tell your router "my neighbour is at IP address X and belongs to AS Y"
- BGP is **slow and cautious by design** — it is not optimised for fast convergence like OSPF; it is optimised for stability, because a routing flap on the Internet can affect millions of users

### eBGP vs iBGP

- **eBGP (External BGP)**: peering between routers in **different** AS numbers — this is what you configure in this workshop
- **iBGP (Internal BGP)**: peering between routers in the **same** AS number — not covered here, but you should know the distinction exists
- The "e" and "i" simply refer to whether the neighbour is inside or outside your AS

### How a BGP Peering Session Works (Simplified)

1. You configure Router A: "my neighbour is 209.165.200.1, and they are in AS 65001"
2. You configure Router B: "my neighbour is 209.165.200.2, and they are in AS 65000"
3. Both routers attempt to open a TCP connection to each other on port 179
4. They exchange **OPEN** messages (hello, I am AS 65000, here are my capabilities)
5. If both sides agree, the session moves to **ESTABLISHED** state
6. They exchange **UPDATE** messages — "here are the networks I can reach"
7. Both sides now have routes to each other's advertised networks

This is exactly what will happen between R2 and ISP-1 in this lab.

### What Does "Advertising a Network" Mean in BGP?

- When you say `network 198.133.219.0 mask 255.255.255.248` inside BGP config, you are telling BGP: "announce to my peers that I can reach this network"
- Your peer receives this announcement and installs a route in its routing table
- This is how the ISP learns about the Company's network, and vice versa
- **Important**: the `network` command in BGP does **not** enable BGP on an interface (unlike OSPF's `network` command, which activates OSPF on matching interfaces). In BGP, `network` simply says "advertise this prefix to my peers, provided it already exists in my routing table." If the prefix is not in the routing table, BGP will not advertise it.

---

## Core Theory: Default Routes and Loopback Addresses

### Default Route (Gateway of Last Resort)

- A default route is `0.0.0.0/0` — it matches **everything**
- When a router has no specific route to a destination, it falls back to the default route
- In real life, your ISP gives your edge router a default route: "send me anything you don't know what to do with, and I'll figure it out"
- In this lab, ISP-1 originates the default route and advertises it via BGP to R2 — R2 then knows "if I don't have a specific route, send it to the ISP"
- R1, which does not run BGP at all, gets a manually configured static default route pointing to R2

### Loopback Interfaces

- A loopback interface is a **virtual interface** — it has no physical cable
- It is **always up** as long as the router is running (unlike a physical interface which can go down if a cable is pulled)
- Common uses:
  - **Testing and simulation**: in this lab, Loopback0 on ISP-1 simulates the web server (10.10.10.10) — no real server hardware needed
  - **Router ID**: OSPF and BGP often use loopback addresses as stable router identifiers
  - **Management access**: a loopback gives administrators a stable address to SSH into, regardless of which physical interface is up or down

---

## Topology

### ASCII Diagram

```
  COMPANY (AS 65000)                          ISP (AS 65001)
 ========================                    ================================

                  Serial link                  Serial link
                198.133.219.0/29             209.165.200.0/30
                (Company internal)           (WAN to ISP)

  +------+  S0/0/0        S0/0/0  +------+  S0/0/1        S0/0/1  +---------+
  |  R1  |-----(DCE)-------------|  R2  |-----(DCE)-------------|  ISP-1  |
  +------+  .1              .2    +------+  .2              .1    +---------+
                                                                    |
                                                                    | Loopback0
                                                                    | 10.10.10.10/32
                                                                    | (Simulated Web Server)
                                  <------>
                                eBGP peering
                              (R2 <-> ISP-1)

  R1: no BGP                     R2: BGP speaker             ISP-1: BGP speaker
      static default                 AS 65000                     AS 65001
      route to R2                    (border router)              (originates default route)
```

### What Is Connected to What

- **R1 S0/0/0 (DCE)** ↔ **R2 S0/0/0** — the Company's internal serial link (198.133.219.0/29)
- **R2 S0/0/1 (DCE)** ↔ **ISP-1 S0/0/1** — the WAN link between AS 65000 and AS 65001 (209.165.200.0/30)
- **ISP-1 Loopback0** — virtual interface simulating the web server (10.10.10.10/32)

### Which Router Does What

| Router | Role | Runs BGP? | AS Number |
|--------|------|-----------|-----------|
| R1 | Internal Company router | No | Part of AS 65000 but not a BGP speaker |
| R2 | Company's **border router** (edge between Company and ISP) | Yes — eBGP | AS 65000 |
| ISP-1 | ISP's border router | Yes — eBGP | AS 65001 |

- R2 is the critical device — it sits on the AS boundary and speaks BGP to the outside world
- R1 only knows how to reach the outside via a static default route pointing to R2
- ISP-1 simulates the entire Internet behind a single loopback address

---

## Addressing Table (Corrected and Complete)

The original workshop table is correct on the IPs but sparse on detail. Here is the full picture:

| Device | Interface | IP Address | Subnet Mask | Network | Role |
|--------|-----------|------------|-------------|---------|------|
| R1 | S0/0/0 (DCE) | 198.133.219.1 | 255.255.255.248 (/29) | 198.133.219.0/29 | Company internal link |
| R2 | S0/0/0 | 198.133.219.2 | 255.255.255.248 (/29) | 198.133.219.0/29 | Company internal link |
| R2 | S0/0/1 (DCE) | 209.165.200.2 | 255.255.255.252 (/30) | 209.165.200.0/30 | WAN to ISP |
| ISP-1 | S0/0/1 | 209.165.200.1 | 255.255.255.252 (/30) | 209.165.200.0/30 | WAN to Company |
| ISP-1 | Loopback0 | 10.10.10.10 | 255.255.255.255 (/32) | 10.10.10.10/32 | Simulated web server |

### Subnet Breakdown

**198.133.219.0/29** (Company link between R1 and R2):
- Usable hosts: 198.133.219.1 – 198.133.219.6 (6 addresses)
- Broadcast: 198.133.219.7
- /29 = 255.255.255.248

**209.165.200.0/30** (Point-to-point WAN between R2 and ISP-1):
- Usable hosts: 209.165.200.1 – 209.165.200.2 (2 addresses — just enough for a point-to-point link)
- Broadcast: 209.165.200.3
- /30 = 255.255.255.252

**10.10.10.10/32** (Loopback):
- /32 means this is a single address, not a range — it identifies exactly one endpoint
- This is standard for loopback interfaces

### DCE/DTE and Why It Matters

- In Packet Tracer, serial links need one side to provide the **clock signal** (DCE side)
- The DCE side must have the `clock rate` command configured
- In this lab:
  - R1 S0/0/0 is **DCE** → needs `clock rate`
  - R2 S0/0/1 is **DCE** → needs `clock rate`
- The original workshop does not mention this, but without it your serial interfaces will not come up

---

## What Comes Next

**Part 2** will walk through the full device configuration for all three routers — including every command the original workshop left out.

**Part 3** will cover verification (`show` commands with expected outputs), troubleshooting if your ping fails, and guidance on the portfolio research tasks.

---

*Workshop 6 Companion Guide — Part 1 of 3*
*Prepared as unofficial supplementary material for 6CS029 Advanced Networking (IDU)*
