# Lecture 5 — Systems: DHCP & DNS

## Chunk 1 of 10 — Slides 1–3: What This Lecture Is About & Why DHCP Exists

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Full Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ← YOU ARE HERE
  2.  The address pool and how leasing works          (Slides 4–6)
  3.  The server responds and the deal is sealed      (Slides 7–8)
  4.  The client commits and the server confirms      (Slides 9–10)
  5.  When things go wrong: troubleshooting addresses (Slide 11)

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14)
  7.  How lookups actually work and why caching matters (Slides 15–18)
  8.  Zones, delegation, and network architecture      (Slides 19–22)
  9.  Public DNS, latency, and real-world performance   (Slides 23–25)
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32)
```

---

## How This Lecture Connects to Everything Before It

By this point in the module, you have spent four lectures building a mental model of how networks work at the infrastructure level. Lecture 1 covered graph theory and minimum spanning trees — the mathematics of connecting things efficiently. Lecture 2 was about characterising traffic and designing hierarchical networks — understanding what flows through the wires and how to structure those wires intelligently. Lecture 3 introduced IPv6 — the addressing system that gives every device on the planet a unique numerical identity. Lecture 4 was QoS (Quality of Service — the network's ability to prioritise certain traffic types over others) — the mechanisms that decide whose packets go first when the pipe gets crowded.

All of that work assumed one thing: that every device on the network already has an IP address. But how does a device actually *get* an IP address in the first place? And once it has an IP address, how does it know that typing `www.google.com` into a browser means sending packets to `142.250.80.46`?

Those are the two questions this lecture answers. **DHCP** gives devices their IP addresses automatically. **DNS** translates human-readable names into machine-readable IP addresses. Without these two services, the beautifully designed hierarchical network from Lecture 2 would be a pile of cables connecting devices that have no idea who they are or where anything is.

The title slide just says "Systems" — deliberately generic. Ian uses it because DHCP and DNS are **infrastructure services** — they sit underneath everything else and make the network usable for humans. They are not the network itself; they are the systems that allow the network to function as something people can actually use.

---

## Slide 1 — Title: "Systems"

**What is on the slide:** A single word — "Systems" — in large olive-brown text, positioned at the bottom-left of a mostly white slide. A vertical brown-and-tan stripe runs along the right edge. No images, no subtitle, no module code. University of Wolverhampton branded template.

**What it means:** This is the most generic title in the entire module. Every other lecture has a descriptive name — "Minimum Spanning Trees," "Characterising Network Traffic," "QoS Concepts," "Wireless Networks." This one just says "Systems." That vagueness is intentional. DHCP and DNS are not flashy topics. They do not have the conceptual depth of graph theory or the engineering drama of QoS queuing algorithms. They are plumbing. Essential, invisible, and utterly taken for granted until something breaks.

Think of it like electricity in a building. Nobody walks into an office and says "wow, the electrical system is working beautifully today." But the moment the lights go out, it is the only thing anyone talks about. DHCP and DNS are the electrical system of a network. This lecture is about understanding that plumbing so you can fix it when it breaks — and more importantly, design it so it does not break in the first place.

No speaker notes on this slide.

---

## Slide 2 — Objectives

**What is on the slide:** A heading "Objectives" followed by a parent bullet and two child bullets:

- "We will look at some services that are provided to make a network operate"
  - **DHCP** — dynamic host configuration protocol servers — "To automatically allocate IP addresses to hosts"
  - **DNS** — domain name servers — "To translate web addresses (and other things) to actual IP addresses of the corresponding web servers"

**Breaking this down:**

**DHCP** stands for **Dynamic Host Configuration Protocol**. Let us take each word apart:

- **Dynamic** — the address assignment changes. It is not permanent. A device gets an address now, and it might get a different address tomorrow. This is the opposite of **static** addressing, where a network administrator manually types an IP address into each device and it stays that way until someone manually changes it.
- **Host** — any device on a network that needs an IP address. A laptop, a phone, a desktop PC, a printer, an IoT (Internet of Things — the broad category of everyday physical objects connected to the internet, such as smart thermostats, security cameras, or industrial sensors) sensor — all hosts.
- **Configuration** — it is not just the IP address. DHCP also provides the subnet mask (tells the device how large its local network is), the default gateway (tells the device where to send packets that are not for the local network), and DNS server addresses (tells the device where to ask for name-to-IP translations). A full configuration package, not just a number.
- **Protocol** — a set of rules that both the client (the device asking for an address) and the server (the device handing out addresses) follow. Like a script in a play — each side knows exactly what to say and when to say it.

**DNS** stands for **Domain Name System** (the slide says "domain name servers," which is a slight simplification — DNS is the entire system, not just the servers). Its job is translation. Humans think in words: `www.wlv.ac.uk`. Computers think in numbers: something like `134.220.x.x`. DNS is the translator sitting between the two. The slide adds "(and other things)" in parentheses because DNS does more than just web addresses — it also handles email routing (MX records — MX stands for Mail Exchanger, a DNS record type that tells the internet which server handles email for a given domain), service discovery, reverse lookups (IP-to-name), and more. But for now, the core idea is: type a name, get a number.

**The "(and other things)" parenthetical is worth remembering.** It hints at the breadth of DNS that the later slides will unpack — DNS is not just a phone book for websites. It is a distributed database that the entire internet relies on for almost everything.

No speaker notes on this slide.

---

## Slide 3 — Why Use DHCP

**What is on the slide:** A heading "DHCP" followed by five bullet points under the sub-heading "Why use DHCP." Some text is colour-coded — green for "Laptops are dynamic," red for "no chance of duplicate IP addresses on network," and blue for "release the IP address." No images.

**The five reasons, one by one:**

### 1. "Automation of configuring IP addresses"

This is the headline reason. Without DHCP, every single device on a network needs its IP address configured by hand. Imagine a university with 3,000 students. Each student has a laptop. Each laptop needs an IP address, a subnet mask, a default gateway, and at least one DNS server address. That is four pieces of information, typed manually into 3,000 devices. If even one administrator makes a typo — say, types `192.168.1.15` instead of `192.168.1.150` — that device either does not work or, worse, conflicts with another device that already has that address.

DHCP eliminates all of that. The administrator configures the DHCP server once: "Here is a pool of 3,000 addresses. Here is the subnet mask. Here is the gateway. Here is the DNS server." Every device that connects to the network asks the server for an address and receives everything automatically. Zero manual typing on the client side.

In a small home network with three devices, manual configuration is annoying but manageable. In an enterprise network with thousands of devices, it is simply impossible. DHCP is not a convenience — it is a necessity.

### 2. "Laptops are dynamic and hence need to take on different IP addresses"

This bullet is highlighted in green on the slide, and it addresses a problem that did not exist in the early days of networking. Desktop computers stay in one place. You can assign them a static IP and forget about it. But laptops move. A student carries a laptop from the library to the lecture hall to the dormitory to the coffee shop. Each of those locations might be a different subnet (a logically separate segment of the network with its own IP address range). The laptop cannot keep the same IP address across all of them — an address that is valid in the library subnet is meaningless in the dormitory subnet.

DHCP handles this seamlessly. When the laptop connects to the library Wi-Fi, it asks the local DHCP server for an address and gets one that belongs to that subnet. When the student walks to the dormitory and connects there, the laptop releases its library address and asks the dormitory DHCP server for a new one. The user notices nothing. The transition is invisible.

This is what "dynamic" means in DHCP — the address is not permanently attached to the device. It is borrowed for as long as the device needs it on that particular network.

### 3. "Need to keep a record of the IP addresses that have been assigned"

Without centralised management, tracking which addresses are in use becomes a nightmare. Imagine an administrator maintaining a spreadsheet of 500 addresses: who has what, when it was assigned, whether it has been returned. One missed entry and two devices end up with the same address — an **IP conflict** — which causes both devices to experience intermittent connectivity problems as the network cannot reliably determine which physical device owns that address.

The DHCP server is that centralised record. It knows exactly which addresses are currently in use (leased), which are available in the pool, and which are permanently reserved (excluded). Everything is tracked automatically. No spreadsheets, no human error, no guessing.

### 4. "Once an IP address has been given out it is no longer available — no chance of duplicate IP addresses on network"

This is highlighted in red on the slide for emphasis. It is the direct consequence of point 3. Because the DHCP server tracks every active lease, it will never hand out the same address to two different devices. The moment an address is assigned, it is removed from the available pool. The moment the lease expires or the device releases the address, it goes back into the pool. One address, one device, at any given time. Guaranteed.

This guarantee is something static addressing cannot provide. If two administrators are both configuring devices and one assigns `10.0.0.50` without telling the other, both devices show up on the network with the same IP. The result is an IP conflict — both devices intermittently lose connectivity as the network gets confused about which MAC address (MAC — Media Access Control — the unique hardware address burned into every network interface) belongs to that IP. DHCP makes this scenario structurally impossible.

### 5. "You can release the IP address of host that is offline and reallocate it to another host that comes online"

This is highlighted in blue on the slide. It introduces the concept of **address recycling**, which leads directly into the next slide's discussion of **leases**. An IP address is a finite resource — a /24 subnet only has 254 usable addresses. In a university with 3,000 students but only 500 simultaneous connections (because not everyone is online at the same time), you do not need 3,000 addresses. You need enough for whoever is currently connected.

When a student turns off their laptop and goes home, their IP address should not sit reserved and unused for the next 12 hours. DHCP reclaims it and hands it to the next student who connects. This is the same principle as a parking garage — you do not reserve a permanent spot for every driver who has ever parked there. You make spots available as people leave and allocate them to whoever arrives next.

This concept of temporary allocation and recycling is what makes DHCP fundamentally different from static addressing. Static addressing is ownership. DHCP is a rental agreement.

---

## The Big Picture So Far

Three slides in, and the lecture has established one thing: **manually assigning IP addresses does not scale**. The moment a network grows beyond a handful of devices, or the moment mobile devices enter the picture, you need automation. DHCP is that automation.

The next slide (Slide 4) will introduce the mechanics — how DHCP actually manages its pool of addresses, what "leasing" means in practice, and what happens when the server needs to keep certain addresses out of the pool permanently. That is where the protocol starts getting technical.

---

*Chunk 1 of 10 complete. Next chunk covers Slides 4–6: Scope, Exclusions, Leases, and the DORA Discover step.*
