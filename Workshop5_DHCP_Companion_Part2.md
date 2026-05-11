# Workshop 5 — Configuring Basic DHCPv4 on a Router

## Companion Guide — Part 2: Topology 2 + DNS Research Question

---

## What's Different in Part 3?

Topology 1 had one DHCP server (R2) serving two remote LANs via a relay agent (R1). Part 3 flips the design:

- **Both routers now have a LAN** (unlike topology 1 where R2 had no LAN)
- **Each router is the DHCP server for its own LAN** — R1 serves 100.1.1.0/24, R2 serves 200.2.2.0/24
- Only the **first 5 addresses** excluded (not 9)
- The original doc gives you almost nothing — no commands, no addressing table, just requirements

> **Missing topology alert:** The original workshop says "the topology shown below" but the image is missing from the docx. The topology below is reconstructed from the requirements.

> **Original doc copy-paste error:** Part 3 says "verify that the PCs have received IP address information from the DHCP server on R2." That's carried over from Part 2 where R2 was the only server. In topology 2, PC-A gets its address from **R1**, and only PC-B gets from R2. Also, the original misspells "DCHP" instead of "DHCP" in the requirements.

---

## Topology 2

```
                        10.0.0.0/30
           R1 ========================== R2
     S0/0/0 (.1)  Serial Link (DCE)  S0/0/0 (.2)
       |          clock rate 128000          |
       |                                     |
      G0/0                                  G0/0
       |.1                                   |.1
       |                                     |
   [Switch S1]                          [Switch S2]
       |                                     |
     PC-A                                  PC-B
   (DHCP client)                       (DHCP client)
                                    
   100.1.1.0/24                        200.2.2.0/24
   DHCP server: R1                     DHCP server: R2
```

### Addressing Table

```
+--------+--------------+------------------+-----------------+-----------------+
| Device | Interface    | IP Address       | Subnet Mask     | Default Gateway |
+--------+--------------+------------------+-----------------+-----------------+
| R1     | G0/0         | 100.1.1.1        | 255.255.255.0   | N/A             |
|        | S0/0/0 (DCE) | 10.0.0.1         | 255.255.255.252 | N/A             |
+--------+--------------+------------------+-----------------+-----------------+
| R2     | G0/0         | 200.2.2.1        | 255.255.255.0   | N/A             |
|        | S0/0/0       | 10.0.0.2         | 255.255.255.252 | N/A             |
+--------+--------------+------------------+-----------------+-----------------+
| PC-A   | NIC          | DHCP             | DHCP            | DHCP            |
| PC-B   | NIC          | DHCP             | DHCP            | DHCP            |
+--------+--------------+------------------+-----------------+-----------------+
```

> **WAN subnet choice:** The original doesn't specify the serial link addresses. 10.0.0.0/30 is used here as a clean, distinct range that won't collide with either LAN. Any /30 works.

> **Key difference from topology 1:** Each router now serves its OWN directly-connected LAN. Since the DHCP server is local to each LAN, **no relay agent is needed.** The DHCP broadcasts reach the server without crossing a router. This tests whether students understood the mechanics — relay is only required when the server is on a different subnet.

---

## Step 1: Build the Topology in Packet Tracer

Start fresh — new file, or `erase startup-config` + `reload` on both routers.

1. Place **R1** (left) and **R2** (right) — same router models as topology 1
2. Place **S1** under R1, **S2** under R2
3. Place **PC-A** connected to S1, **PC-B** connected to S2
4. Cable:
   - R1 G0/0 → S1 F0/1
   - R2 G0/0 → S2 F0/1
   - PC-A → S1 F0/6
   - PC-B → S2 F0/6
   - R1 S0/0/0 → R2 S0/0/0 (DCE cable — click R1 first)

---

## Step 2: Configure R1

```
enable
configure terminal
hostname R1
!
interface GigabitEthernet0/0
 ip address 100.1.1.1 255.255.255.0
 no shutdown
!
interface Serial0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 128000
 no shutdown
!
end
```

## Step 3: Configure R2

```
enable
configure terminal
hostname R2
!
interface GigabitEthernet0/0
 ip address 200.2.2.1 255.255.255.0
 no shutdown
!
interface Serial0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
!
end
```

## Step 4: Configure RIPv2

**R1:**
```
configure terminal
router rip
 version 2
 network 100.0.0.0
 network 10.0.0.0
 no auto-summary
end
```

**R2:**
```
configure terminal
router rip
 version 2
 network 200.2.2.0
 network 10.0.0.0
 no auto-summary
end
```

> **Why `network 100.0.0.0` and not `network 100.1.1.0`?** RIP uses classful network boundaries in its `network` command. 100.x.x.x is Class A, so the classful boundary is 100.0.0.0. RIP then advertises the actual subnets it finds (100.1.1.0/24) thanks to `version 2` and `no auto-summary`.

> **Same for 200.2.2.0** — Class C, so `network 200.2.2.0` works directly.

### Verify:

From R1:
```
ping 10.0.0.2
ping 200.2.2.1
```

From R2:
```
ping 10.0.0.1
ping 100.1.1.1
```

All must succeed before continuing. If R2 can't reach 100.1.1.1, wait 30 seconds for RIP convergence and retry.

---

## Step 5: Configure DHCP on R1 (for 100.1.1.0/24)

```
configure terminal
!
ip dhcp excluded-address 100.1.1.1 100.1.1.5
!
ip dhcp pool R1LAN
 network 100.1.1.0 255.255.255.0
 default-router 100.1.1.1
 exit
!
end
```

## Step 6: Configure DHCP on R2 (for 200.2.2.0/24)

```
configure terminal
!
ip dhcp excluded-address 200.2.2.1 200.2.2.5
!
ip dhcp pool R2LAN
 network 200.2.2.0 255.255.255.0
 default-router 200.2.2.1
 exit
!
end
```

> **Pool names:** The original doesn't specify pool names for topology 2. `R1LAN` and `R2LAN` are used here. Any name works — it's a local label.

> **Exclusion range:** First 5 addresses only (.1 through .5). Smaller than topology 1's 9. The first available lease will be **.6**.

---

## Step 7: Set PCs to DHCP and Verify

On both PCs: Desktop → IP Configuration → select **DHCP**

**PC-A** expected result (`ipconfig /all`):
```
IP Address...........: 100.1.1.6
Subnet Mask..........: 255.255.255.0
Default Gateway......: 100.1.1.1
```

**PC-B** expected result (`ipconfig /all`):
```
IP Address...........: 200.2.2.6
Subnet Mask..........: 255.255.255.0
Default Gateway......: 200.2.2.1
```

> **No relay agent needed here.** PC-A's DHCPDISCOVER broadcast hits R1's G0/0 directly — R1 is the server, it responds immediately. Same for PC-B and R2. No `ip helper-address` commands required.

### Verify on routers:

R1:
```
show ip dhcp binding
show ip dhcp pool
```

R2:
```
show ip dhcp binding
show ip dhcp pool
```

### End-to-end test:

From PC-A command prompt:
```
ping 200.2.2.6
```

This should succeed — packet goes PC-A → S1 → R1 → serial → R2 → S2 → PC-B.

---

## Demonstrate to Tutor — Checklist (Topology 2)

1. R1 and R2 can ping each other's LANs (RIP working)
2. PC-A has 100.1.1.6 via DHCP from R1
3. PC-B has 200.2.2.6 via DHCP from R2
4. `show ip dhcp binding` on both routers shows leases
5. PC-A can ping PC-B across the WAN

---

## Why No Relay Agent This Time?

This is the teaching point of topology 2. In topology 1, the DHCP server (R2) was on a **different subnet** from the clients — broadcasts couldn't reach it, so R1 had to relay. Here, each DHCP server sits on the **same subnet** as its clients. Broadcasts arrive directly.

If you wanted to test understanding, ask: "What would you change if R1 were the DHCP server for BOTH LANs?" Answer: R2 would need `ip helper-address` on G0/0 pointing to R1's serial IP, and R1 would need a second pool for 200.2.2.0/24.

---

## Research Question: DNS Strengths and Weaknesses

This question is in the portfolio section of the workshop. It seems disconnected from DHCP, but it's bundled here because DHCP and DNS are in the same lecture week (Week 5 on Canvas — "DHCP and DNS").

### Framing for Students

The question asks about **DNS and associated servers** — meaning the DNS system as a whole (root servers, TLD servers, authoritative servers, recursive resolvers, caching).

**Strengths to cover:**
- Hierarchical distributed design — no single point of failure at the top level
- Caching at every level reduces load and speeds up resolution
- Scalability — handles billions of queries daily across millions of domains
- Transparent to users — works silently behind every URL
- Supports multiple record types (A, AAAA, MX, CNAME, TXT, NS) for flexible configuration

**Weaknesses to cover:**
- Originally designed without security — queries and responses are unencrypted plaintext (UDP port 53)
- Vulnerable to DNS spoofing/cache poisoning — attackers can inject false records
- DDoS amplification — small queries can generate large responses, exploited in reflection attacks
- Single point of failure at the local level — if your configured DNS server goes down, resolution fails (mitigated by secondary DNS)
- Propagation delay — DNS record changes (TTL expiry) can take hours to propagate globally
- Privacy concerns — DNS queries reveal browsing habits to ISPs and resolvers

**Mitigations to mention (for higher marks):**
- DNSSEC — adds cryptographic signatures to DNS records to prevent spoofing
- DoH (DNS over HTTPS) and DoT (DNS over TLS) — encrypt DNS queries
- Anycast routing — distributes root/TLD servers across multiple locations for resilience

> **Portfolio tip:** The brief says "high marks to complete fuller well researched answers." Students should include specific examples and reference sources (RFCs, Cisco documentation, textbooks). A two-paragraph answer gets low marks. A page with examples, diagrams, and references gets high marks.

---

## Summary: What Students Submit for Workshop 5

**Demo (in session):**
- Topology 1 working — show tutor PC-A and PC-B with DHCP addresses, `show ip dhcp binding` on R2
- Topology 2 working — same verification on both routers

**Portfolio document (submitted via Canvas as .docx):**
1. First available IPs for PC-A and PC-B (topology 1) → .10 each
2. Client identification in `show ip dhcp binding` → MAC address
3. What "current index" means → next address to be offered
4. What is a DHCP relay agent and why needed → forwards broadcasts as unicast across subnets
5. `ipconfig /all` screenshots from both topologies
6. DNS strengths and weaknesses research question (referenced, in own words)

---

**End of Workshop 5 Companion Guide.**
