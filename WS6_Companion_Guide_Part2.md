# Workshop 6 — Autonomous Systems & eBGP: Companion Guide (Part 2 of 3)

## ⚠️ Disclaimer

This companion guide is an **unofficial supplementary resource** prepared by your course tutor.
It is **not** a replacement for the original workshop brief issued by the module leader (Ian Coulson, University of Wolverhampton).
Always refer to the original workshop document as the authoritative source.

---

## What This Part Covers

You are now configuring all three devices in Packet Tracer. This part walks through every command for every router — including commands the original workshop left out.

The order matters. Configure in this sequence:

1. **ISP-1** first
2. **R2** second
3. **R1** last

Why this order? BGP requires **both** sides to be configured before the session establishes. If you configure ISP-1 first, then when you finish R2's BGP config, the session comes up immediately and you see a clear confirmation message (`%BGP-5-ADJCHANGE: neighbor ... Up`). If you configure in random order, you will see confusing error messages on partially-configured routers that might make you think something is wrong when it is not.

Before each device, a short theory block explains **why** you are typing what you are typing.

---

## Step 0: Build the Physical Topology in Packet Tracer

Before any configuration, you need to place devices and cable them.

1. Place three routers (use model **1841** or **2811** — they have serial interfaces)
2. Cable them using the **Serial DCE** cable (the one with the clock icon in the connections panel):
   - **First link**: pick up the Serial DCE cable → click **R1** first (this makes R1 the DCE side) → select **S0/0/0** → then click **R2** → select **S0/0/0**
   - **Second link**: pick up the Serial DCE cable → click **R2** first (this makes R2 the DCE side) → select **S0/0/1** → then click **ISP-1** → select **S0/0/1**

**Critical**: in Packet Tracer, the **first router you click** when placing a Serial DCE cable becomes the DCE side. If you click the wrong router first, the DCE/DTE assignment flips, the clock rate ends up on the wrong device, and the link will not come up. If this happens, delete the cable and redo it.

**How to verify**: on any router, run `show controllers s0/0/0` (or s0/0/1). The output will say either `DCE` or `DTE`. Check that:
- R1 S0/0/0 = DCE
- R2 S0/0/0 = DTE
- R2 S0/0/1 = DCE
- ISP-1 S0/0/1 = DTE

---

## Device 1: ISP-1

### Theory: What Is ISP-1 Doing?

ISP-1 represents the Internet Service Provider. In the real world, an ISP:

- Owns a public AS number
- Runs BGP to exchange routes with its customers and with other ISPs
- Provides a **default route** to its customers — meaning "send me anything you cannot route yourself, and I will forward it into the Internet"

In this lab, ISP-1 does exactly that on a small scale:

- It has AS number **65001**
- It peers with R2 (the Company's border router) via eBGP
- It advertises a default route (`0.0.0.0/0`) to R2
- It has a loopback interface (10.10.10.10) pretending to be a web server somewhere on the Internet

### Configuration: ISP-1

```
enable
configure terminal

! --- Step 1: Set the hostname ---
! This is cosmetic but important for identification during verification.

hostname ISP-1

! --- Step 2: Configure the serial interface toward R2 ---
! This is the physical link that carries the eBGP session.
! ISP-1 is the DTE side (R2 is DCE), so no clock rate needed here.

interface Serial0/0/1
 ip address 209.165.200.1 255.255.255.252
 no shutdown

! --- Step 3: Configure the loopback interface ---
! This virtual interface simulates a web server at 10.10.10.10.
! The original workshop tells you to configure this but does not give the commands.
! A loopback is created simply by entering the interface — no cable, no shutdown needed.

interface Loopback0
 ip address 10.10.10.10 255.255.255.255

! --- Step 4: Create a static default route pointing to the loopback ---
! Why? ISP-1 needs a default route in its own routing table so that BGP has
! something to advertise. The next hop is Loopback0 (Lo0), which means
! "send unmatched traffic to myself" — in a real ISP this would point upstream,
! but in our lab the Internet is simulated by this one router.

ip route 0.0.0.0 0.0.0.0 Loopback0

! --- Step 5: Enable BGP with AS number 65001 ---
! This is the core of the lab. Each command is explained below.

router bgp 65001

! bgp log-neighbor-changes: tells the router to print a console message
! every time a BGP neighbour comes up or goes down. Essential for debugging.

 bgp log-neighbor-changes

! network 0.0.0.0: tells BGP to advertise the default route (0.0.0.0/0)
! to its peers. This works because we just created a static default route
! in Step 4, so the prefix exists in the routing table.

 network 0.0.0.0

! neighbor 209.165.200.2 remote-as 65000: this is the eBGP neighbour statement.
! It says: "the router at 209.165.200.2 (R2) belongs to AS 65000, and I want
! to form a BGP peering session with it."
! Because 65000 ≠ 65001 (my own AS), this is an eBGP peering — external BGP.

 neighbor 209.165.200.2 remote-as 65000

end
```

### What Just Happened on ISP-1

After this config, ISP-1 will:

1. Bring up its serial interface and be reachable at 209.165.200.1
2. Have a loopback at 10.10.10.10 (always up)
3. Have a static default route in its routing table pointing to Lo0
4. Start attempting to open a TCP session on port 179 to 209.165.200.2 (R2)
5. The BGP session will **not** establish yet — R2 is not configured. That is normal. You will see console messages like `%BGP-3-NOTIFICATION: ... connection refused`. This stops once R2 is configured.

---

## Device 2: R2

### Theory: What Is R2 Doing?

R2 is the **Company's border router**. It sits at the boundary between the Company (AS 65000) and the ISP (AS 65001). Its job:

- Run eBGP with ISP-1 to receive the default route (so the Company can reach the Internet)
- Advertise the Company's internal network (198.133.219.0/29) to the ISP via BGP (so the ISP knows how to send return traffic back to the Company)
- Forward traffic between R1 (internal) and ISP-1 (external)

In the real world, this is exactly what a company's edge router does: it speaks BGP to the ISP, advertises the company's public IP ranges, and receives a default route or full routing table from the ISP.

### Why Does R2 Advertise 198.133.219.0/29?

Think about it from the ISP's perspective. If a web server on the Internet wants to send a reply back to R1 (198.133.219.1), the traffic arrives at ISP-1. ISP-1 needs a route to 198.133.219.0/29 — otherwise it drops the packet. By advertising this network via BGP, R2 tells ISP-1: "I can reach 198.133.219.0/29 — send traffic for that network to me."

Without this advertisement, the ping in Part 3 would work **outbound** (R1 → ISP-1) but the reply would be **dropped** at ISP-1 because ISP-1 would have no return route.

### Configuration: R2

```
enable
configure terminal

! --- Step 1: Hostname ---

hostname R2

! --- Step 2: Configure the serial interface toward R1 (Company internal) ---
! R2 is DTE on this link (R1 is DCE), so no clock rate here.

interface Serial0/0/0
 ip address 198.133.219.2 255.255.255.248
 no shutdown

! --- Step 3: Configure the serial interface toward ISP-1 (WAN) ---
! R2 is DCE on this link, so clock rate IS needed.
! 128000 is a common lab clock rate. The original workshop omits this.

interface Serial0/0/1
 ip address 209.165.200.2 255.255.255.252
 clock rate 128000
 no shutdown

! --- Step 4: Enable BGP with AS number 65000 ---

router bgp 65000

! neighbor 209.165.200.1 remote-as 65001: peer with ISP-1.
! ISP-1 is in AS 65001, which is different from our AS 65000 → this is eBGP.

 neighbor 209.165.200.1 remote-as 65001

! network 198.133.219.0 mask 255.255.255.248: advertise the Company's network.
!
! Why the explicit mask? Because BGP defaults to classful boundaries.
! 198.133.219.0 is a Class C address, so without the mask keyword,
! BGP would try to advertise 198.133.219.0/24 — but that does not exist
! in our routing table (we have a /29, not a /24). The advertisement
! would silently fail. The explicit mask ensures BGP advertises the
! exact /29 prefix that matches our connected route.

 network 198.133.219.0 mask 255.255.255.248

end
```

### ⚠️ Error in the Original Workshop

The original brief says under R2's config:

> `R2(config)# router BGP 65000`

The uppercase `BGP` works because IOS is case-insensitive, but it is not how Cisco documentation writes it. The standard form is lowercase: `router bgp 65000`. Follow the standard form in your work.

### What Just Happened on R2

After this config:

1. Both serial interfaces are up with correct IPs
2. R2 starts a BGP session with ISP-1 at 209.165.200.1
3. Since ISP-1 is already configured and waiting, the session should move to **ESTABLISHED** within a few seconds
4. You will see a console message: `%BGP-5-ADJCHANGE: neighbor 209.165.200.1 Up`
5. R2 receives the default route (0.0.0.0/0) from ISP-1
6. R2 sends 198.133.219.0/29 to ISP-1
7. Both routers now have routes to each other's advertised networks

### Why R2 Does Not Have `bgp log-neighbor-changes`

The original workshop includes this command on ISP-1 but not on R2. In modern IOS versions (15.x+), `bgp log-neighbor-changes` is **enabled by default** — you do not need to type it. ISP-1's config includes it explicitly, which is redundant but harmless. If you want to be consistent, you can add it to R2 as well, but it is not required.

---

## Device 3: R1

### Theory: What Is R1 Doing?

R1 is a simple internal Company router. It does **not** run BGP — it has no direct connection to the ISP and no reason to participate in inter-AS routing.

R1's only job:

- Be reachable on the Company's internal network (198.133.219.0/29)
- Have a way to reach everything outside the Company — this is done with a **static default route** pointing to R2

### Why a Static Default Route, Not BGP?

Not every router in an organisation needs to run BGP. BGP is for border routers that talk to other autonomous systems. Internal routers just need to know: "if I don't have a specific route to a destination, send it to the border router (R2), and let R2 figure it out."

This is exactly what a default route does. It is simpler, uses fewer resources, and is the standard design for internal routers in small to medium networks.

### About the Original Workshop's Instruction

The original brief says:

> "A static default route to the loopback address must be configured on R1, use a search engine to compose this command and then deploy it."

This is **misleading**. The static default route on R1 does not point "to the loopback address." It points to **R2's serial interface IP** (198.133.219.2) as the next hop. The loopback (10.10.10.10) is the ultimate destination you are trying to reach, not the next hop of the route.

The correct way to think about it:

- R1 says: "for any destination I don't know, send the packet to 198.133.219.2 (R2)"
- R2 receives the packet, checks its own routing table, and forwards it to ISP-1 via BGP
- ISP-1 receives the packet at its loopback and replies

### Configuration: R1

```
enable
configure terminal

! --- Step 1: Hostname ---

hostname R1

! --- Step 2: Configure the serial interface toward R2 ---
! R1 is DCE on this link, so clock rate IS needed.

interface Serial0/0/0
 ip address 198.133.219.1 255.255.255.248
 clock rate 128000
 no shutdown

! --- Step 3: Configure the static default route ---
! ip route 0.0.0.0 0.0.0.0 = "for any destination (0.0.0.0/0)..."
! 198.133.219.2 = "...send the packet to R2 as the next hop"
!
! This is the command the original workshop tells you to Google.
! Now you know what it does and why.

ip route 0.0.0.0 0.0.0.0 198.133.219.2

end
```

### What Just Happened on R1

After this config:

1. The serial interface comes up and R1 is reachable at 198.133.219.1
2. R1's routing table now has:
   - A **connected route** to 198.133.219.0/29 (its own link to R2)
   - A **static default route** (0.0.0.0/0) via 198.133.219.2 (R2)
3. If R1 needs to reach anything outside 198.133.219.0/29 — for example 10.10.10.10 — it will forward the packet to R2
4. R2, having learned the default route from ISP-1 via BGP, will forward it onward to ISP-1

---

## The Full Traffic Flow (End-to-End)

Now that all three devices are configured, here is what happens when you ping 10.10.10.10 from R1:

```
Step 1: R1 wants to reach 10.10.10.10
        R1 checks its routing table → no specific route
        R1 has a default route → next hop 198.133.219.2 (R2)
        R1 sends the ICMP echo request to R2

Step 2: R2 receives the packet destined for 10.10.10.10
        R2 checks its routing table → default route learned via BGP → next hop 209.165.200.1 (ISP-1)
        R2 forwards the packet to ISP-1

Step 3: ISP-1 receives the packet destined for 10.10.10.10
        ISP-1 checks its routing table → 10.10.10.10/32 is directly connected (Loopback0)
        ISP-1 processes the ping and generates an ICMP echo reply

Step 4: ISP-1 sends the reply to 198.133.219.1 (R1's address)
        ISP-1 checks its routing table → 198.133.219.0/29 learned via BGP → next hop 209.165.200.2 (R2)
        ISP-1 forwards the reply to R2

Step 5: R2 receives the reply destined for 198.133.219.1
        R2 checks its routing table → 198.133.219.0/29 is directly connected (S0/0/0)
        R2 forwards the reply to R1

Step 6: R1 receives the ICMP echo reply
        Ping succeeds: "!!!!!"
```

This is the complete round trip. If any single step in this chain is broken — a missing route, a down interface, a wrong IP — the ping will fail. Part 3 will show you how to verify each step and troubleshoot failures.

---

## Summary of All Commands

For quick reference, here is every command on every device without the explanations.

**Note**: before pasting any block below, you must first enter privileged exec mode and then global configuration mode on the router. Type `enable` then `configure terminal` before entering these commands.

### ISP-1

```
hostname ISP-1
interface Serial0/0/1
 ip address 209.165.200.1 255.255.255.252
 no shutdown
interface Loopback0
 ip address 10.10.10.10 255.255.255.255
ip route 0.0.0.0 0.0.0.0 Loopback0
router bgp 65001
 bgp log-neighbor-changes
 network 0.0.0.0
 neighbor 209.165.200.2 remote-as 65000
end
```

### R2

```
hostname R2
interface Serial0/0/0
 ip address 198.133.219.2 255.255.255.248
 no shutdown
interface Serial0/0/1
 ip address 209.165.200.2 255.255.255.252
 clock rate 128000
 no shutdown
router bgp 65000
 neighbor 209.165.200.1 remote-as 65001
 network 198.133.219.0 mask 255.255.255.248
end
```

### R1

```
hostname R1
interface Serial0/0/0
 ip address 198.133.219.1 255.255.255.248
 clock rate 128000
 no shutdown
ip route 0.0.0.0 0.0.0.0 198.133.219.2
end
```

---

## What Comes Next

**Part 3** will cover:

- Every `show` command from the workshop with expected output explained line by line
- The final ping test
- What to check if the ping fails (troubleshooting checklist)
- Guidance on the portfolio research tasks

---

*Workshop 6 Companion Guide — Part 2 of 3*
*Prepared as unofficial supplementary material for 6CS029 Advanced Networking (IDU)*
