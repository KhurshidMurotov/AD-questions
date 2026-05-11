# Workshop 5 — Configuring Basic DHCPv4 on a Router

> **Numbering note:** The original file is labeled "6CS029 Workshop 4" but Canvas lists this as **Workshop 5 - DHCP**. We follow the Canvas numbering here.

## Companion Guide — Part 1: Theory + Topology 1

---

## The Problem This Workshop Solves

Every device on a network needs an IP address, a subnet mask, a default gateway, and usually a DNS server. On a network with 10 PCs — you configure those manually. On a network with 500 PCs across multiple floors — you don't.

DHCP automates this. One server, one pool of addresses, and every device that connects gets configured automatically.

But there's a catch. DHCP uses **broadcast** messages. A PC that just booted has no IP — it screams "IS THERE A DHCP SERVER OUT THERE?" to everyone on its local network. Broadcasts don't cross routers. So if your DHCP server sits on a different subnet (behind a router), the PC's cry for help never reaches it.

That's where the **relay agent** comes in. The router hears the broadcast, wraps it into a **unicast** packet, and forwards it to the DHCP server on the other subnet. Server replies. Router passes the reply back. PC gets its address. Problem solved.

This workshop does exactly that:

- **R2** = DHCP server (holds the address pools)
- **R1** = relay agent (forwards DHCP requests from its LANs to R2)
- **PC-A and PC-B** = DHCP clients (receive addresses automatically)

---

## DHCP Message Flow (4 Steps — "DORA")

```
PC-A (no IP yet)                    R1 (relay)                    R2 (DHCP server)
      |                                |                                |
      |--- DHCPDISCOVER (broadcast) -->|                                |
      |                                |--- DHCPDISCOVER (unicast) ---->|
      |                                |                                |
      |                                |<--- DHCPOFFER (unicast) -------|
      |<--- DHCPOFFER (broadcast) -----|                                |
      |                                |                                |
      |--- DHCPREQUEST (broadcast) --->|                                |
      |                                |--- DHCPREQUEST (unicast) ----->|
      |                                |                                |
      |                                |<--- DHCPACK (unicast) ---------|
      |<--- DHCPACK (broadcast) -------|                                |
      |                                |                                |
      |  PC-A now has: 192.168.0.10    |                                |
```

**D** — Discover: "Anyone out there?"
**O** — Offer: "I have 192.168.0.10 for you."
**R** — Request: "I'll take that one."
**A** — Acknowledge: "It's yours. Lease starts now."

Without the relay agent on R1, the Discover broadcast dies at R1's interface and never reaches R2.

---

## Topology

```
                     192.168.2.252/30
          R1 ========================== R2
    S0/0/0 (.253)  Serial Link (DCE)  S0/0/0 (.254)
      |                                     (DHCP Server)
      |
      +--- G0/0 ---[Switch S1]--- PC-A
      |   .1                      (DHCP client)
      |   192.168.0.0/24
      |
      +--- G0/1 ---[Switch S2]--- PC-B
          .1                      (DHCP client)
          192.168.1.0/24
```

### Addressing Table

```
+--------+--------------+------------------+-----------------+-----------------+
| Device | Interface    | IP Address       | Subnet Mask     | Default Gateway |
+--------+--------------+------------------+-----------------+-----------------+
| R1     | G0/0         | 192.168.0.1      | 255.255.255.0   | N/A             |
|        | G0/1         | 192.168.1.1      | 255.255.255.0   | N/A             |
|        | S0/0/0 (DCE) | 192.168.2.253    | 255.255.255.252 | N/A             |
+--------+--------------+------------------+-----------------+-----------------+
| R2     | S0/0/0       | 192.168.2.254    | 255.255.255.252 | N/A             |
+--------+--------------+------------------+-----------------+-----------------+
| PC-A   | NIC          | DHCP             | DHCP            | DHCP            |
| PC-B   | NIC          | DHCP             | DHCP            | DHCP            |
+--------+--------------+------------------+-----------------+-----------------+
```

**Key detail:** R2 has NO LAN interface — only a serial link. It serves DHCP pools for networks it's not directly connected to. That's why R1 must relay.

---

## Part 1 — Build the Network and Configure Basic Settings

### Step 1: Build the Topology in Packet Tracer

1. Place **two routers** (1941 or 2911). Label them **R1** (left) and **R2** (right).
2. Place **two switches** (2960). Label **S1** under R1's G0/0, **S2** under R1's G0/1.
3. Place **PC-A** connected to S1, **PC-B** connected to S2.
4. Cable it:
   - R1 G0/0 → S1 F0/1 (straight-through)
   - R1 G0/1 → S2 F0/1 (straight-through)
   - PC-A → S1 F0/6 (straight-through)
   - PC-B → S2 F0/6 (straight-through)
   - R1 S0/0/0 → R2 S0/0/0 (**DCE cable — click R1 first** so R1 gets DCE)

### Step 2: Configure R1

```
enable
configure terminal
hostname R1
!
interface GigabitEthernet0/0
 ip address 192.168.0.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface Serial0/0/0
 ip address 192.168.2.253 255.255.255.252
 clock rate 128000
 no shutdown
!
end
```

> **Why clock rate on R1?** Because R1 holds the DCE end of the serial cable. The DCE side provides the clocking signal. If you connected the cable R2-first, R2 would need the clock rate instead. You can verify which side is DCE with `show controllers serial 0/0/0` — look for "DCE" in the output.

### Step 3: Configure R2

```
enable
configure terminal
hostname R2
!
interface Serial0/0/0
 ip address 192.168.2.254 255.255.255.252
 no shutdown
!
end
```

R2 has **only one interface** in this topology. No LAN, no GigabitEthernet. Just the serial link back to R1.

### Step 4: Configure RIPv2 on Both Routers

**R1:**
```
configure terminal
router rip
 version 2
 network 192.168.0.0
 network 192.168.1.0
 network 192.168.2.252
 no auto-summary
end
```

**R2:**
```
configure terminal
router rip
 version 2
 network 192.168.2.252
 no auto-summary
end
```

> **Why does R2 advertise only one network?** R2 is only directly connected to 192.168.2.252/30. It doesn't know about 192.168.0.0 or 192.168.1.0 yet — it learns those from R1 via RIP.

> **Why `no auto-summary`?** Without it, RIPv2 advertises subnets at their classful boundaries. The 192.168.x.x addresses are Class C, so each /24 is already at its classful boundary — but the /30 WAN link (192.168.2.252/30) would be summarized to 192.168.2.0/24, which could cause routing ambiguity since R2 also connects to that network. Best practice: always include `no auto-summary` with RIPv2 to preserve exact subnet information.

### Step 5: Verify Connectivity

From R1:
```
ping 192.168.2.254
```

From R2:
```
ping 192.168.0.1
ping 192.168.1.1
```

All three must succeed. If R2 can't reach 192.168.0.1 or 192.168.1.1, RIP hasn't converged yet — wait 30 seconds and retry, or check `show ip route` on R2 for the learned routes.

### Step 6: Set PCs to DHCP

On **PC-A**: Desktop tab → IP Configuration → select **DHCP**
On **PC-B**: Same.

They won't get an address yet. That's expected — no DHCP server is configured.

---

## Part 2 — Configure DHCP Server (R2) and Relay Agent (R1)

### Step 7: Configure DHCP Pools on R2

The logic: we create two pools on R2 — one for each of R1's LANs. We also exclude the first 9 addresses from each pool so that .1 through .9 are reserved (for routers, servers, printers — anything that needs a static IP).

```
configure terminal
!
ip dhcp excluded-address 192.168.0.1 192.168.0.9
ip dhcp excluded-address 192.168.1.1 192.168.1.9
!
ip dhcp pool R1G1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 exit
!
ip dhcp pool R1G0
 network 192.168.0.0 255.255.255.0
 default-router 192.168.0.1
 exit
!
end
```

> **Excluded addresses first.** The workshop brief says this explicitly — best practice is to configure exclusions before pools. If you create the pool first and a PC requests an address in the meantime, it might get .1 (the router's own IP), causing a conflict.

> **`default-router` = default gateway.** This tells the DHCP client which IP to use as its gateway. For the G0/0 LAN, that's R1 at 192.168.0.1. For the G0/1 LAN, R1 at 192.168.1.1.

### Step 8: Test — Still No Address?

Go to PC-A → Command Prompt:
```
ipconfig /all
```

**You still won't have an address.** Why?

PC-A broadcasts a DHCPDISCOVER. It hits R1's G0/0 interface. R1 is a router — routers block broadcasts by default. The request never reaches R2. This is the whole point of the relay agent.

### Step 9: Configure Relay Agent on R1

The fix: tell R1 "when you hear a DHCP broadcast on this interface, forward it as a unicast to 192.168.2.254 (R2)."

```
configure terminal
!
interface GigabitEthernet0/0
 ip helper-address 192.168.2.254
 exit
!
interface GigabitEthernet0/1
 ip helper-address 192.168.2.254
 exit
!
end
```

> **You must configure `ip helper-address` on BOTH G0/0 and G0/1.** Each interface serves a different LAN. Without the helper on G0/1, PC-B's DHCP requests would still die at R1.

> **Original doc typo:** The workshop shows `R1(config-if)# exit` followed by `R1(config-if)# interface g0/1`. After `exit`, the prompt returns to `R1(config)#`, not `R1(config-if)#`. Cosmetic error in Ian's doc — commands still work.

> **`ip helper-address` doesn't just forward DHCP.** By default, it forwards 8 UDP protocols including DHCP (ports 67/68), DNS (port 53), TFTP (port 69), and others. For this workshop, only DHCP matters.

### Step 10: Renew DHCP on PCs

PCs may have given up trying. Force a new request:

**PC-A** → Command Prompt:
```
ipconfig /release
ipconfig /renew
```

Same on **PC-B**.

> **Packet Tracer shortcut:** If `ipconfig /renew` doesn't work immediately, go to Desktop → IP Configuration → switch to Static, then back to DHCP. This forces a fresh DORA cycle.

### Step 11: Verify — Expected Results

**PC-A** (`ipconfig /all`):
```
IP Address...........: 192.168.0.10
Subnet Mask..........: 255.255.255.0
Default Gateway......: 192.168.0.1
```

**PC-B** (`ipconfig /all`):
```
IP Address...........: 192.168.1.10
Subnet Mask..........: 255.255.255.0
Default Gateway......: 192.168.1.1
```

> **Why .10?** Addresses .1 through .9 are excluded. The first available address in each pool is .10.

### Step 12: Verify on R2 (Server Side)

```
show ip dhcp binding
```

Expected output:
```
IP address      Client-ID/         Lease expiration    Type
                Hardware address
192.168.0.10    0060.3E12.ABCD --  --                  Automatic
192.168.1.10    00D0.BA45.6789 --  --                  Automatic
```

> **Portfolio hint:** "What other useful client identification information is in the output?" — the **hardware address (MAC address)** of each client. This lets you identify exactly which physical device got which IP.

```
show ip dhcp pool
```

Expected output shows pool statistics including:
- Total addresses in pool (254 per /24, minus excluded = 245 available)
- Current index — the **next address the server will try to offer**

> **Portfolio hint:** "What does current index refer to?" — it's the next IP address R2 will attempt to assign to the next DHCP request. It increments after each lease.

---

## Portfolio Answer Hints (Part 2)

**"What are the first available IP addresses that PC-A and PC-B can lease?"**
→ PC-A: **192.168.0.10** (first 9 excluded from 192.168.0.0/24)
→ PC-B: **192.168.1.10** (first 9 excluded from 192.168.1.0/24)

**"What other piece of useful client identification information is in `show ip dhcp binding`?"**
→ The **Client-ID / Hardware address** — the MAC address of each DHCP client.

**"What does current index refer to in `show ip dhcp pool`?"**
→ The **next IP address** the DHCP server will attempt to offer to the next client.

**"What is a DHCP relay agent and why is it needed?"**
→ A relay agent is a router that **forwards DHCP broadcast messages as unicast** across subnet boundaries. It's needed because DHCP clients use broadcasts to discover servers, and routers don't forward broadcasts. Without a relay agent, clients on subnets without a local DHCP server can never receive an address.

---

## Demonstrate to Tutor — Checklist

Before calling the tutor over, verify:

1. `ping 192.168.2.254` from R1 — serial link works
2. `ping 192.168.0.1` from R2 — RIP route learned
3. PC-A has 192.168.0.10 via DHCP
4. PC-B has 192.168.1.10 via DHCP
5. `show ip dhcp binding` on R2 shows both leases
6. PC-A can ping PC-B (end-to-end through R1)

---

**End of Part 1. Part 2 covers the second topology (Part 3 of the original workshop) + DNS research question.**
