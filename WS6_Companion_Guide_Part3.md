# Workshop 6 — Autonomous Systems & eBGP: Companion Guide (Part 3 of 3)

## ⚠️ Disclaimer

This companion guide is an **unofficial supplementary resource** prepared by your course tutor.
It is **not** a replacement for the original workshop brief issued by the module leader (Ian Coulson, University of Wolverhampton).
Always refer to the original workshop document as the authoritative source.

---

## What This Part Covers

All three routers are configured. Now you need to **prove** it works. The original workshop gives you four verification tasks but does not tell you what the output should look like or what to look for. This part fills that gap.

After verification, there is a troubleshooting section (what to check if something is broken) and guidance on the portfolio research tasks.

---

## Theory: Why Verification Matters

In networking, "I typed the commands" does not mean "it works." Configuration errors, typos, missing commands, and wrong interface assignments can all silently break things. Professional network engineers never assume — they verify with `show` commands and read the output carefully.

The verification commands in this workshop fall into three categories:

- **Routing table** (`show ip route`) — does the router know how to reach the destination?
- **BGP table** (`show ip bgp`) — what prefixes has BGP learned and is it advertising?
- **BGP session status** (`show ip bgp summary`) — is the peering session actually up?

Each command answers a different question. Together, they give you a complete picture.

---

## Verification Task 1: Display the IPv4 Routing Table on R2

### The Command

```
R2# show ip route
```

### What You Are Looking For

This command displays every route R2 knows about. You are specifically looking for the **BGP-learned default route** from ISP-1.

### Expected Output (Key Lines)

**Note**: the exact formatting of `show ip route` varies between Packet Tracer versions (some group routes under classful parents, some say "variably subnetted," etc.). Do not worry if your screen does not match character-for-character — focus on finding the **key entries** described below.

```
Gateway of last resort is 209.165.200.1 to network 0.0.0.0

     198.133.219.0/29 is subnetted, 1 subnets
C       198.133.219.0 is directly connected, Serial0/0/0
     209.165.200.0/30 is subnetted, 1 subnets
C       209.165.200.0 is directly connected, Serial0/0/1
B*   0.0.0.0/0 [20/0] via 209.165.200.1, 00:05:12
```

### Reading the Output Line by Line

**`Gateway of last resort is 209.165.200.1 to network 0.0.0.0`**
- This confirms R2 has a default route and it points to 209.165.200.1 (ISP-1)
- "Gateway of last resort" = the default route = where packets go when no specific route matches

**`C 198.133.219.0 is directly connected, Serial0/0/0`**
- `C` = Connected route — this network is on one of R2's own interfaces
- This is the Company internal link to R1

**`C 209.165.200.0 is directly connected, Serial0/0/1`**
- The WAN link to ISP-1

**`B* 0.0.0.0/0 [20/0] via 209.165.200.1`**
- `B` = learned via **BGP** — this is the important one
- `*` = this route is the **candidate default route** (the gateway of last resort)
- `0.0.0.0/0` = the default route (matches everything)
- `[20/0]` = administrative distance 20, metric 0. The number 20 is the default administrative distance for eBGP routes. For comparison: OSPF is 110, static is 1, connected is 0. Lower = more trusted. eBGP at 20 means the router trusts BGP-learned routes quite highly — second only to directly connected and static routes
- `via 209.165.200.1` = the next hop is ISP-1

**If you do NOT see the `B*` line**: the BGP session is not working. Jump to the Troubleshooting section below.

---

## Verification Task 2: Display the BGP Table on R2

### The Command

```
R2# show ip bgp
```

### Theory: What Is the BGP Table?

The BGP table (also called the BGP RIB — Routing Information Base) is **separate** from the routing table. Think of it this way:

- The **BGP table** stores every prefix BGP has learned from all peers — this is BGP's own internal database
- The **routing table** stores the best routes from all sources (connected, static, OSPF, BGP, etc.) — this is what the router actually uses to forward packets

BGP first puts learned routes into its own table, then the best ones get installed into the main routing table.

### Expected Output

```
BGP table version is 2, local router ID is 209.165.200.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop         Metric  LocPrf  Weight  Path
*> 0.0.0.0          209.165.200.1         0              0  65001 i
*> 198.133.219.0/29 0.0.0.0               0          32768  i
```

### Reading the Output Line by Line

**`local router ID is 209.165.200.2`**
- The BGP router ID — by default, IOS picks the highest IP on any active interface. R2's highest IP is 209.165.200.2

**`*> 0.0.0.0`**
- `*` = valid route
- `>` = best route (this one is installed in the routing table)
- `0.0.0.0` = the default route
- `Next Hop 209.165.200.1` = learned from ISP-1
- `Path: 65001 i` = this route was advertised by AS 65001 (ISP-1). The `i` means the origin is IGP (the route was originated with a BGP `network` command). The "Path" column is the **AS path** — the core of how BGP works. It lists every AS the route has passed through. Here it only crossed one AS (65001), because ISP-1 is our direct peer

**`*> 198.133.219.0/29`**
- This is the Company's own network
- `Next Hop 0.0.0.0` = next hop of 0.0.0.0 means "this originated from me" — R2 is advertising this network, not receiving it
- `Weight 32768` = default weight for locally originated routes
- `Path: i` = no AS path (it did not cross any AS — it started here)

### What This Tells You

R2's BGP table has exactly two entries:
1. The default route, learned from ISP-1 (AS 65001)
2. The Company network, originated locally by R2

This is correct. If you see fewer entries, something is misconfigured.

---

## Verification Task 3: Display BGP Session Status on R2

### The Command

```
R2# show ip bgp summary
```

### Theory: What Does This Show?

This command gives you a quick overview of all BGP peering sessions — who is the peer, what AS are they in, how long has the session been up, and how many prefixes have been exchanged. This is the **first command** most network engineers run when troubleshooting BGP.

### Expected Output

```
Neighbor        V    AS  MsgRcvd  MsgSent  TblVer  InQ  OutQ  Up/Down   State/PfxRcd
209.165.200.1   4  65001      10       10       2    0     0  00:05:30         1
```

### Reading the Output Column by Column

| Column | Value | Meaning |
|--------|-------|---------|
| Neighbor | 209.165.200.1 | The peer's IP address (ISP-1) |
| V | 4 | BGP version 4 (the current and only version in use) |
| AS | 65001 | The peer's AS number |
| MsgRcvd / MsgSent | 10 / 10 | Number of BGP messages exchanged (these increment over time — exact numbers will vary) |
| TblVer | 2 | BGP table version — increments every time the table changes |
| InQ / OutQ | 0 / 0 | Messages queued (should always be 0 in a healthy session — a non-zero value means BGP is falling behind) |
| Up/Down | 00:05:30 | How long the session has been in its current state |
| State/PfxRcd | 1 | **This is the most important column.** A number means the session is ESTABLISHED and shows how many prefixes were received from this peer. Here, 1 prefix received = the default route |

### The Critical Check

**If the last column shows a number (like `1`)**: the session is up and working. You are good.

**If the last column shows a word (like `Idle`, `Active`, `Connect`, or `OpenSent`)**: the session is **NOT** established. These are BGP state names indicating the peering is stuck. Jump to Troubleshooting.

Common stuck states and what they mean:
- **Idle** — BGP has not even tried to connect. Usually means the `neighbor` command is missing or has the wrong IP
- **Active** — BGP is actively trying to connect but failing. Usually means the peer is unreachable (interface down, wrong IP, no physical connectivity)
- **Connect** — TCP connection attempt is in progress but not completing
- **OpenSent** — TCP connected but the BGP OPEN message was rejected (usually AS number mismatch)

---

## Verification Task 4: Ping the Loopback from R1

### The Command

```
R1# ping 10.10.10.10
```

### Expected Output

```
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.10.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 28/28/28 ms
```

### What Each Character Means

- `!` = a successful reply was received
- `.` = the request timed out (no reply within 2 seconds)
- `U` = destination unreachable message received

**Five `!` marks** = all five pings succeeded. The lab is complete.

**If you get `.....` or `...!!`**: some or all pings failed. The first 1-2 pings sometimes fail because ARP needs to resolve — this is normal. But if you get all dots, there is a real problem. See Troubleshooting below.

**Why might the first ping fail?** In Packet Tracer, the first ICMP packet sometimes times out due to initial processing delay on the router (building the CEF entry for the new destination). This is normal simulator behavior and not a configuration error. If you get `.!!!!` (4 out of 5), that is still a pass. Note: this is **not** caused by ARP — serial links use point-to-point encapsulation (HDLC by default) and do not need MAC address resolution like Ethernet does.

---

## Troubleshooting: What to Check If Things Do Not Work

If any verification step above failed, work through this checklist **in order**. Each step builds on the previous one — do not skip ahead.

### Step 1: Are the Interfaces Up?

```
R2# show ip interface brief
```

Every interface you configured should show `up` in both the Status and Protocol columns:

```
Interface        IP-Address      OK?  Method  Status    Protocol
Serial0/0/0      198.133.219.2   YES  manual  up        up
Serial0/0/1      209.165.200.2   YES  manual  up        up
```

**If Status = administratively down**: you forgot `no shutdown` on that interface.

**If Status = up but Protocol = down**: the other end is not connected or not configured. Check:
- Is the cable plugged in on both ends?
- Is the other router's interface configured with `no shutdown`?
- Is the DCE side configured with `clock rate`? If the clock rate is missing on the DCE end, the Protocol column stays `down` on **both** sides.

Run the same command on all three routers.

### Step 2: Can Neighbours Ping Each Other Directly?

Test each link individually:

```
R1# ping 198.133.219.2        (R1 pinging R2 — Company internal link)
R2# ping 209.165.200.1        (R2 pinging ISP-1 — WAN link)
```

**If these pings fail**: the basic IP configuration is wrong. Double-check:
- IP addresses match the addressing table exactly
- Subnet masks are correct (/29 on the Company link, /30 on the WAN link)
- You did not accidentally put the wrong IP on the wrong interface (e.g., swapping S0/0/0 and S0/0/1 on R2)

### Step 3: Is the BGP Session Established?

```
R2# show ip bgp summary
```

Check the last column. If it shows a **number**, the session is up. If it shows a **word** (Idle, Active, etc.), the session is stuck.

**If Idle**: check the `neighbor` command on both R2 and ISP-1. The IP address in each `neighbor` statement must match the other router's actual interface IP.

**If Active**: BGP is trying to open a TCP connection to the peer but failing. This usually means the peer is unreachable (interface down, wrong IP, no physical connectivity) or the remote router's BGP process is not running.

**If OpenSent**: the TCP session opened but BGP negotiation failed. Check that the AS numbers in the `remote-as` statements are correct — R2 should say `remote-as 65001` (ISP-1's AS), and ISP-1 should say `remote-as 65000` (R2's AS). If these are swapped, BGP rejects the session.

### Step 4: Does R2 Have the Default Route?

```
R2# show ip route
```

Look for `B* 0.0.0.0/0 via 209.165.200.1`. If missing:
- Check that ISP-1 has the static route `ip route 0.0.0.0 0.0.0.0 Loopback0`
- Check that ISP-1's BGP config has `network 0.0.0.0`
- Both must be present: the static route puts 0.0.0.0/0 in ISP-1's routing table, and the `network` command tells BGP to advertise it

### Step 5: Does ISP-1 Have a Route Back to R1?

```
ISP-1# show ip route
```

Look for `B 198.133.219.0/29 via 209.165.200.2`. If missing:
- Check that R2's BGP config has `network 198.133.219.0 mask 255.255.255.248`
- Make sure the mask keyword is present — without it, BGP tries to advertise a /24 which does not match R2's routing table and silently fails

### Step 6: Does R1 Have a Default Route?

```
R1# show ip route
```

Look for `S* 0.0.0.0/0 [1/0] via 198.133.219.2`. The `S` means static. If missing:
- Check that you typed `ip route 0.0.0.0 0.0.0.0 198.133.219.2` on R1

---

## Portfolio Research Tasks

The original workshop lists three research questions. These are for your portfolio — the guide below gives you **direction**, not complete answers. You should research and write these in your own words.

### Task 1: What is a loopback address and why is it used?

Direction to research:
- Start with what you already learned in this guide: a loopback is a virtual interface that is always up
- Expand on the three uses mentioned: testing/simulation, router ID for protocols like OSPF and BGP, and management access
- Research how a loopback differs from a physical interface in terms of reliability
- Consider why network engineers prefer loopback addresses as BGP router IDs rather than physical interface IPs — what happens to the BGP session if a physical interface goes down?

### Task 2: What is an Autonomous System and what role does BGP have in their use?

Direction to research:
- Define an AS in your own words — a network or group of networks under single administrative control, identified by an ASN
- Explain the difference between interior routing (within an AS) and exterior routing (between ASes)
- Explain that BGP is the protocol that connects autonomous systems together to form the Internet
- Mention the distinction between eBGP (between different ASes) and iBGP (within the same AS)
- You can reference the scale of BGP: the global Internet routing table contains over 900,000 prefixes — only BGP is designed to handle this

### Task 3: The University of Wolverhampton main site has an AS number, what is it?

Direction to research:
- Use a public looking glass or AS lookup tool to find this. Useful sites:
  - https://bgp.he.net (Hurricane Electric BGP Toolkit) — search for "University of Wolverhampton"
  - https://www.peeringdb.com — search by organisation name
- The answer is a public ASN assigned by RIPE NCC (the Regional Internet Registry for Europe)
- Once you find the ASN, you can explore further: what IP prefixes does the University advertise? Who are its upstream providers? This is all public information visible in BGP

---

## Workshop Complete

If your ping from R1 to 10.10.10.10 succeeded with five `!` marks, you have:

- Built a two-AS topology from scratch
- Configured eBGP peering between a Company border router and an ISP
- Advertised networks across AS boundaries
- Verified the BGP session, routing tables, and end-to-end reachability

This is the fundamental mechanism by which every organisation on the Internet exchanges traffic. Every website you visit, every video you stream — at some point the traffic crosses an eBGP peering session exactly like the one you just built.

---

*Workshop 6 Companion Guide — Part 3 of 3*
*Prepared as unofficial supplementary material for 6CS029 Advanced Networking (IDU)*
