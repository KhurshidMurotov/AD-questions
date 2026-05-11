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
# Lecture 5 — Systems: DHCP & DNS

## Chunk 2 of 10 — Slides 4–6: The Address Pool, Leasing, and the First Step of DORA

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ← YOU ARE HERE
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

## Slide 4 — DHCP: Available Addresses (Slide 4 of 32)

**What is on the slide:** A heading "DHCP – Available addresses" followed by three bullet points. The word **scope** is bold and coloured red. The phrase "Certain addresses" is coloured red. The phrase "leased to a client" is coloured purple. Two sub-bullets nest under the third main bullet. No images.

This slide introduces three foundational concepts that the entire DHCP process relies on: the **scope**, **exclusions**, and **leases**. Everything that follows — the four-step **DORA** handshake (**D**iscover, **O**ffer, **R**equest, **A**cknowledge — the four messages exchanged between client and server to complete an IP address assignment) on the next six slides — only makes sense if you understand these three ideas first.

---

### Concept 1: The Scope

The slide says: *"When a DHCP server receives a request from a client, it chooses an IP address and other related information from a pool of addresses known as a **scope** and offers it to the client."*

A **scope** is simply the range of IP addresses that the DHCP server is authorised to hand out. Think of it as the server's inventory. If the server manages the `192.168.1.0/24` subnet, the scope might be defined as `192.168.1.1` through `192.168.1.254` — that is 254 possible addresses. The administrator defines this range when setting up the DHCP server.

But the scope is not just a list of IP addresses. It also includes the "other related information" the slide mentions. When you configure a scope, you also tell the server:

- What **subnet mask** to hand out (e.g. `255.255.255.0`)
- What **default gateway** to tell clients about (e.g. `192.168.1.1`)
- What **DNS server** addresses to provide (e.g. `8.8.8.8` and `8.8.4.4`)
- How long the **lease** should last (e.g. 86400 seconds, which equals 24 hours)

So the scope is the complete package — not just "which addresses exist" but also "what configuration goes with them." A single DHCP server can manage multiple scopes for multiple subnets, each with its own pool and settings. In a university, one scope might serve the library network (`10.0.1.0/24`), another the dormitory network (`10.0.2.0/24`), and a third the staff network (`10.0.3.0/24`), each with different gateways, different DNS servers, and different lease durations.

---

### Concept 2: Exclusions

The slide says: *"Certain addresses within the scope may be **excluded** from distribution because they have already been assigned as static addresses. E.g application servers, printers etc."*

Not every address in the scope should be handed out dynamically. Some devices on the network need a **permanent, predictable IP address** that never changes. A file server, for example — if its IP address changed every day, every user and application configured to connect to `192.168.1.5` would suddenly stop working. Printers are another classic example. Nobody wants to reconfigure the office printer's IP address in every employee's laptop because the DHCP server decided to give it a different address this morning.

These devices get **static** (manually configured) IP addresses. But their addresses still fall within the scope's range. If the scope covers `192.168.1.1` to `192.168.1.254`, and the file server is statically set to `192.168.1.5`, the DHCP server needs to know: "Do not give `192.168.1.5` to anyone. It is taken." That is what an exclusion does. You tell the DHCP server: "Your scope is `.1` through `.254`, but never touch `.1` through `.9` — those are reserved for infrastructure devices with static addresses."

In Cisco IOS (Internetwork Operating System — the software that runs on Cisco routers and switches), this is configured with the command:

```
ip dhcp excluded-address 192.168.1.1 192.168.1.9
```

This tells the server: "The first nine addresses are off-limits. Start handing out from `.10` onward." You will see this exact command in the DHCP workshop (Workshop 5) when configuring R2 as a DHCP server.

**Why this matters practically:** If you forget to exclude an address that is already statically assigned to a server, and the DHCP server hands that same address to a student's laptop, you get an IP conflict. The server and the laptop both claim the same address. The server — a critical piece of infrastructure — starts experiencing intermittent connectivity. Users notice. Your boss notices. All because of one missing exclusion line.

---

### Concept 3: Leases

The slide says: *"IP address is **leased to a client** for a fixed amount of time."* Two sub-bullets follow:
- *"New address obtained if client logs off and in again."*
- *"If the lease period expires without a renewal, the IP address is returned to the pool for reassignment."*

A **lease** is a time-limited rental agreement between the DHCP server and the client device. When the server assigns an address, it does not say "this is yours forever." It says "this is yours for the next 24 hours" (or whatever the lease duration is configured to be). The typical default is **86400 seconds**, which is exactly one day.

During that 24-hour window, the address belongs to that client. No other device will receive it. The DHCP server tracks the lease: which address, which client (identified by its MAC — Media Access Control — address), and when the lease expires.

**What happens at the halfway point:** Most DHCP clients do not wait for the full lease to expire before acting. At the **50% mark** (T1 — 12 hours into a 24-hour lease), the client quietly sends a renewal request to the server: "I am still here, can I keep this address?" If the server agrees, the lease timer resets to the full duration. The client does not need to go through the entire DORA process again — just a quick renewal exchange. This happens silently in the background. The user never knows.

**What happens at 87.5%:** If the T1 renewal failed (maybe the original DHCP server was temporarily unreachable), the client tries again at T2 (87.5% of the lease, which is roughly 21 hours into a 24-hour lease). This time, it broadcasts to any DHCP server on the network, not just the original one.

**What happens when the lease expires:** If both renewal attempts fail and the full lease duration passes, the client simply loses the address. It does not send any message — the address just expires. The client's network interface effectively goes offline, and it must start the entire DORA process from scratch — broadcasting a Discover message as if it had just been switched on for the first time. On the server's side, the expired address is automatically returned to the scope's available pool and can be assigned to someone else.

**What happens on log-off:** If the client disconnects cleanly (shutting down the computer, disconnecting from Wi-Fi), it sends a DHCPRELEASE message to the server, which immediately returns the address to the pool. No waiting for expiry. The sub-bullet "New address obtained if client logs off and in again" describes this — disconnecting and reconnecting typically results in the DORA process running fresh, which might give you a different address than the one you had before.

**Why lease duration matters for network design:** Short leases (e.g. 4 hours) mean addresses are recycled quickly — good for networks with lots of transient users (a coffee shop Wi-Fi, a conference venue). But short leases generate more DHCP traffic because devices renew more often. Long leases (e.g. 7 days) reduce DHCP traffic but tie up addresses longer — bad if you have more users than addresses and rely on turnover. The administrator sets the lease duration based on how mobile the users are and how large the address pool is relative to the number of users.

---

## Slide 5 — DORA: Discover (Slide 5 of 32)

**What is on the slide:** A heading "**D**ORA - **D**iscover" with the letter D highlighted in red in both words. Below it, a network diagram showing the Discover process.

**The diagram shows:**

On the left side, a PC (drawn as a monitor and keyboard). On the right side, a server (drawn as a tower). In the middle, a switch. Between them, multiple yellow envelope icons radiate outward from the PC through the switch in all directions — to the server, and also to other implied devices on the network. The envelopes represent the broadcast nature of the Discover message — it goes everywhere, not just to the server.

Below the diagram, two information boxes:

**Destination box:**
- IP: `255.255.255.255` — this is the **broadcast address**. It means "everyone on this local network." The PC does not know the server's IP address (it does not know anything yet — that is the whole problem), so it shouts to everyone.
- MAC: `FF.FF.FF.FF.FF.FF` — this is the **broadcast MAC address**. Just as the IP broadcast goes to every device at Layer 3 (the network layer), this MAC broadcast goes to every device at Layer 2 (the data link layer). Every switch port forwards this frame. Every device on the local network receives it.
- Port: **67** — this is the destination **UDP** (User Datagram Protocol — a connectionless transport protocol that sends data without establishing a handshake first, unlike TCP which requires a three-way handshake) port. Port 67 is reserved for DHCP servers. Any device listening on port 67 will pick up this message.

**Source box:**
- IP: `0.0.0.0` — the PC has no IP address yet. That is literally the entire reason it is sending this message. So it uses `0.0.0.0` as its source, which means "I do not have an address."
- MAC: `AB.67.87.45.C5.A1` — this is the PC's hardware address. Unlike IP addresses, MAC addresses are burned into the network interface at the factory. The PC always knows its own MAC, even before DHCP gives it anything. This is how the server will later identify and respond to this specific client.
- Port: **68** — the source UDP port. Port 68 is reserved for DHCP clients. When the server replies, it will send the reply to port 68.

Below those boxes: "UDP packet" — confirming that DHCP uses UDP, not TCP (Transmission Control Protocol — a connection-oriented transport protocol that guarantees delivery through acknowledgments and retransmissions). Why UDP? Because a TCP connection requires both sides to already have IP addresses (to complete the three-way handshake: SYN → SYN-ACK → ACK, where SYN stands for Synchronize and ACK for Acknowledge — a sequence that establishes a reliable connection before any data is sent). The client has no IP yet. It cannot do TCP. UDP does not care — it just fires the packet and hopes someone is listening. This is one of the few protocols where the deliberate unreliability of UDP is not a drawback but a structural necessity.

**The text on the slide says:** "PC switched on. Sends out Discover packet."

That is the Discover step in two sentences. The PC wakes up, realises it has no IP address, and screams into the void: "IS THERE A DHCP SERVER OUT THERE? I NEED AN ADDRESS!" It does not know where the server is. It does not know if a server even exists. It just broadcasts and waits.

---

## Slide 6 — DHCPDISCOVER Packet Structure (Slide 6 of 32)

**What is on the slide:** A detailed packet structure table showing the exact contents of a DHCPDISCOVER message, field by field, in hexadecimal. This is the raw anatomy of the message the PC sent on Slide 5. A large label on the right says "DHCPDISCOVER message."

This slide is heavy reference material. It is not something you need to memorise byte-by-byte, but understanding what each field does gives you a much deeper grasp of what is actually happening on the wire when a device asks for an IP address.

**The packet header (UDP layer):**

```
UDP Src=0.0.0.0  sPort=68
Dest=255.255.255.255  dPort=67
```

This matches exactly what Slide 5 showed — source `0.0.0.0:68` (the client with no IP, using the DHCP client port), destination `255.255.255.255:67` (broadcast to all, targeting the DHCP server port).

**The DHCP message fields, row by row:**

| Field | Hex Value | What It Means |
|-------|-----------|---------------|
| **OP** | `0x01` | Operation code. `0x01` = request (client to server). `0x02` would mean reply (server to client). This is a request — the client is asking for something. |
| **HTYPE** | `0x01` | Hardware type. `0x01` = Ethernet. This tells the server what kind of network the client is on. Almost always Ethernet in modern networks. |
| **HLEN** | `0x06` | Hardware address length. `0x06` = 6 bytes. An Ethernet MAC address is 6 bytes long (e.g. `AB:67:87:45:C5:A1` is six two-digit hex pairs). |
| **HOPS** | `0x00` | Hop count. Set to `0x00` by the client. If the message passes through a **relay agent** (a router that forwards DHCP broadcasts across subnets), the relay agent increments this field. More on relay agents in the workshop. |
| **XID** | `0x3903F326` | Transaction ID — a random number generated by the client to match requests with responses. When the server replies, it includes the same XID so the client knows "this reply is for me, not for some other device that also sent a Discover at the same time." Think of it like a ticket number at a deli counter. |
| **SECS** | `0x0000` | Seconds elapsed since the client started trying to get an address. Zero here because this is the first attempt. If the client has been trying for 30 seconds, this would reflect that. |
| **FLAGS** | `0x0000` | The most significant bit is the **broadcast flag**. If set to 1, it tells the server "I cannot receive unicast replies yet, so please broadcast your response." Set to 0 here. |
| **CIADDR** | `0x00000000` | Client IP Address. All zeros — the client does not have one yet. That is why it is sending this message in the first place. |
| **YIADDR** | `0x00000000` | "Your" IP Address. This is where the server will write the offered IP in the Offer reply. Empty in the Discover because the client is not offering itself anything — it is asking. |
| **SIADDR** | `0x00000000` | Server IP Address. Empty because the client does not know which server will respond. |
| **GIADDR** | `0x00000000` | Gateway IP Address (relay agent). Zero because there is no relay agent involved in this example. If the client were on a different subnet from the server, the relay agent's IP would appear here. |
| **CHADDR** | `0x00053C04 8D590000...` | Client Hardware Address — the client's MAC address, padded to 16 bytes. This is the only identity the client has at this point. The server uses this to track which physical device asked for an address. |

**Below the CHADDR field:**

*"192 octets of 0s, or overflow space for additional options. BOOTP legacy"*

This block of 192 zero bytes exists for historical reasons. DHCP evolved from an older protocol called **BOOTP** (Bootstrap Protocol — an early protocol designed to assign IP addresses to diskless workstations during boot-up). BOOTP had a fixed-size packet format with this 192-byte field reserved for server name and boot filename. DHCP inherited the format but does not use this space for its intended BOOTP purpose. It is kept for backward compatibility and can occasionally serve as overflow space for options that do not fit in the standard options area.

**The Magic Cookie:**

```
0x63825363
```

This is a fixed value that marks the beginning of the DHCP options section. Every DHCP packet has this exact number in this exact position. It tells the receiver: "What follows is a series of DHCP option fields, not BOOTP data." The name "magic cookie" is the actual technical term — it comes from the original RFC (Request For Comments — the numbered standards documents published by the IETF, the Internet Engineering Task Force, that define how internet protocols work).

**The DHCP Options (the interesting part):**

The bottom of the table shows three DHCP options:

**Option 53: DHCP Discover** — This identifies the message type. Option 53 is always present and always tells you which DORA step this message belongs to. Here it says "DHCP Discover" — step 1.

**Option 50: 192.168.1.100 requested** — The client is asking for a specific IP address. This happens when the client has been on this network before and remembers the address it had last time. It is saying: "I used to have `192.168.1.100` — can I have it again?" The server is not obligated to honour this request, but it usually will if the address is still available.

**Option 55: Parameter Request List** — The client is telling the server what information it wants in addition to an IP address. The numbers in parentheses correspond to specific DHCP option codes:
- **(1)** = Subnet Mask
- **(3)** = Router (default gateway)
- **(15)** = Domain Name
- **(6)** = Domain Name Server (DNS)

This is the client saying: "Give me an IP, but also tell me the subnet mask, the gateway, my domain name, and the DNS servers." The full configuration package, not just a number.

---

## What Just Happened — The Discover Step Summarised

A device with no IP address just did the only thing it could: broadcast a plea for help to every device on the local network. It identified itself by the only identity it has (its MAC address), asked for a specific address it vaguely remembers from last time (option 50), listed what configuration details it needs (option 55), and stamped the message with a random transaction ID (XID) so it can match the reply to its own request.

Now it waits. The next slide shows what happens when a DHCP server hears this plea and decides to respond with an Offer.

---

*Chunk 2 of 10 complete. Next chunk covers Slides 7–8: DORA Offer with full packet decode.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 3 of 10 — Slides 7–8: The Server Responds — DORA Offer

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ← YOU ARE HERE
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

## Where We Are in DORA

The client has just broadcast a DHCPDISCOVER message (Slides 5–6). It screamed into the void: "I need an IP address." Now a DHCP server on the network has heard that broadcast, checked its scope for an available address, and is about to respond. That response is the **Offer** — the second letter in DORA (Discover, **O**ffer, Request, Acknowledge).

The Offer is not a commitment. It is a proposal. The server is saying: "I have this address available — do you want it?" The client has not accepted anything yet. In fact, if multiple DHCP servers exist on the network, the client might receive multiple Offers from different servers, each proposing a different address. The client will choose one and reject the rest. But that decision comes in the next step (Request). For now, the server is just making its pitch.

---

## Slide 7 — DORA: Offer (Slide 7 of 32)

**What is on the slide:** A heading "D**O**RA - **O**ffer" with the letter O highlighted in red in both words. Below it, a network diagram showing the Offer process — visually the reverse of the Discover diagram from Slide 5.

**The diagram shows:**

On the left, the same PC (monitor and keyboard). On the right, the same server (tower). In the middle, the same switch. But this time, a single yellow envelope travels from the server, through the switch, to the PC. Unlike the Discover's spray of envelopes going everywhere, the Offer has just one envelope going to one destination. The server knows exactly who asked — it read the client's MAC (Media Access Control) address from the CHADDR field in the Discover message — so it sends the reply specifically to that client.

Below the diagram, two information boxes — but notice the labels are reversed from Slide 5. The **Destination** is now the PC, and the **Source** is now the server:

**Destination (PC):**
- IP: `255.255.255.255` — still a broadcast IP address. This might seem strange. The server knows who the client is (it has the MAC address), so why broadcast? Because the client does not have an IP address yet. You cannot send a unicast IP packet to a device that has no IP. The server uses the broadcast IP to ensure the packet reaches the client at Layer 3 (the network layer), while using the client's specific MAC address at Layer 2 (the data link layer) to target it more precisely.
- MAC: `AB.67.87.45.C5.A1` — the client's MAC address, copied from the CHADDR field of the Discover. Because this is a unicast MAC (not the broadcast `FF.FF.FF.FF.FF.FF`), the switch consults its MAC address table and forwards this frame only to the specific port where the client is connected. Other devices on the network never see this frame at all — the switch filters it out. This is a key difference from the Discover step, where the broadcast MAC caused the switch to flood the frame out of every port.
- Port: **68** — DHCP client port. The reply goes to the port where DHCP clients listen.

**Source (server):**
- IP: `172.177.91.42` — the server's own IP address. Unlike the Discover, where the source was `0.0.0.0`, the server has a real, configured IP address and uses it.
- MAC: `F1.87.90.12.DE.5A` — the server's hardware address.
- Port: **67** — DHCP server port.

**Two key notes on the slide:**

*"Offer will have a lease for a fixed time"* — the server is not just offering an IP address. It is offering a complete rental agreement: an IP, a subnet mask, a gateway, DNS servers, and a specific lease duration. The full package.

*"The PC may actually get many offers from different DHCP servers"* — this is important. In enterprise networks, you often have multiple DHCP servers for redundancy (if one goes down, the other can still serve addresses). When the client broadcasts its Discover, all DHCP servers on the network hear it, and all of them may respond with an Offer. The client will receive multiple proposals and must choose one. Typically, clients accept the first Offer that arrives — first come, first served. The others are silently discarded. This selection happens in the Request step (Slide 9).

---

## Slide 8 — DHCPOFFER Packet Structure (Slide 8 of 32)

**What is on the slide:** The detailed packet structure table for a DHCPOFFER message. Same format as the DHCPDISCOVER table on Slide 6, but now the fields are filled in from the server's perspective. Two annotations on the right side: "Client offered IP address 192.168.1.100" (in a red box) and "Client MAC address" (in a red box pointing to the CHADDR field). A label says "DHCP Offer message From the Server" (note: "Server" is misspelt as "Sever" in the label — a typo on the slide).

**A note on the example addresses:** Slide 7's diagram uses `172.177.91.42` as the server's IP, while Slide 8's packet uses `192.168.1.1`. These are different example addresses on different slides — Ian used separate examples to illustrate the concept and the packet format independently. Do not try to reconcile them as part of the same scenario. The important thing is the structure and the field values, not the specific IPs.

**The packet header (UDP — User Datagram Protocol — layer):**

```
UDP Src=192.168.1.1  sPort=67
Dest=255.255.255.255  dPort=68
```

The directions are flipped from the Discover. Source is now the server (`192.168.1.1` on port 67), destination is still the broadcast address (`255.255.255.255` on port 68). The server sends from its own IP on the server port, targeting the broadcast address on the client port.

**What changed from the Discover packet — field by field:**

Rather than repeating every field (many are identical), here are the fields that changed and why:

| Field | Discover Value | Offer Value | What Changed and Why |
|-------|---------------|-------------|---------------------|
| **OP** | `0x01` (request) | `0x02` (reply) | The direction flipped. `0x01` means client-to-server. `0x02` means server-to-client. This is now a reply. |
| **CIADDR** | `0x00000000` | `0x00000000` | Still all zeros. The client still does not have an IP — this Offer has not been accepted yet. In fact, CIADDR stays zero throughout the entire initial DORA exchange. It is only used during **lease renewals**, when a client that already has an address asks the server to extend the lease. |
| **YIADDR** | `0x00000000` | `0xC0A80164` | **This is the main event.** The server is writing the proposed IP address into the "Your IP Address" field. The hex `0xC0A80164` decodes to `192.168.1.100` (C0=192, A8=168, 01=1, 64=100 in decimal). This is the address the server is offering to the client. |
| **SIADDR** | `0x00000000` | `0xC0A80101` | The server identifies itself. `0xC0A80101` = `192.168.1.1`. Now the client knows which server made this offer. |
| **CHADDR** | Client's MAC | Client's MAC (unchanged) | The server echoes back the client's MAC address. This is how the client confirms "this Offer is for me, not for someone else." |
| **XID** | `0x3903F326` | `0x3903F326` (unchanged) | The transaction ID is the same as the Discover. The server copied it from the client's request. This is the "deli counter ticket" — the client checks this number to match the Offer to its own Discover. If a client sent multiple Discovers, the XIDs would differ, and each Offer would match to the correct one. |

**Fields that stayed identical:** HTYPE (`0x01`, Ethernet), HLEN (`0x06`, 6-byte MAC), HOPS (`0x00`), SECS (`0x0000`), FLAGS (`0x0000`), and the BOOTP (Bootstrap Protocol — DHCP's predecessor, designed for diskless workstations) legacy block (192 octets of zeros). The Magic Cookie is also the same (`0x63825363`) — it is always the same in every DHCP message.

---

### The DHCP Options — What the Server Is Actually Offering

This is where the real content of the Offer lives. The server is not just offering an IP — it is offering a complete network configuration:

**Option 53: DHCP Offer** — Message type identifier. Tells the client "this is an Offer, not a Discover, not a Request, not an ACK." Every DORA step has its own Option 53 value.

**Option 1: `255.255.255.0` subnet mask** — Tells the client the size of the local network. With a /24 mask, the client knows that any address in the `192.168.1.x` range is on its local subnet, and anything outside that range needs to go through the router (default gateway).

**Option 3: `192.168.1.1` router** — The default gateway. When the client needs to reach a device outside its local subnet (like a web server on the internet), it sends the packet to this address. The router then forwards it onward. Without this, the client could only communicate with other devices on the `192.168.1.0/24` subnet — completely isolated from the rest of the network.

**Option 51: `86400s (1 day)` IP address lease time** — The rental duration. The client can use `192.168.1.100` for exactly 24 hours (86,400 seconds). After that, it must renew or lose the address. As covered in Slide 4, the client will attempt renewal at the 50% mark (T1 = 12 hours) and again at 87.5% (T2 = 21 hours) before the lease expires.

**Option 54: `192.168.1.1` DHCP server** — The server's identifier. This is critical in networks with multiple DHCP servers. When the client sends its Request in the next step, it includes this identifier to tell all servers on the network: "I am accepting the offer from `192.168.1.1` specifically." Any other server that also sent an Offer will see this and withdraw its proposal, returning the offered address to its own pool.

**Option 6: DNS servers `9.7.10.15`, `9.7.10.16`, `9.7.10.18`** — Three DNS (Domain Name System) server addresses. The client will use these to resolve domain names (like `www.wlv.ac.uk`) into IP addresses. Three servers are provided for redundancy — if the first one is unreachable, the client tries the second, then the third. Notice how DHCP ties into DNS here — the very lecture that teaches you about DNS also shows you how devices learn *where* their DNS servers are. They learn it from DHCP.

---

## The Offer Summarised — What Just Happened

The server received the client's broadcast plea, looked into its scope, found an available address (`192.168.1.100`), and responded with a complete proposal:

```
"Here is what I am offering you:

  IP address:     192.168.1.100
  Subnet mask:    255.255.255.0
  Default gateway: 192.168.1.1
  DNS servers:    9.7.10.15, 9.7.10.16, 9.7.10.18
  Lease duration: 24 hours
  This offer comes from server: 192.168.1.1

  Take it or leave it."
```

The client has received this proposal. Maybe it received others from other servers too. Now it must decide: accept this one and formally request it (the Request step, Slide 9), or accept a different server's offer instead.

The deal is not done yet. Two of the four DORA steps are complete. The Offer is a proposal on the table. The next chunk covers what happens when the client picks up the pen and signs.

---

*Chunk 3 of 10 complete. Next chunk covers Slides 9–10: DORA Request and Acknowledge — the client commits and the server confirms.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 4 of 10 — Slides 9–10: The Client Commits and the Server Confirms — DORA Request & Acknowledge

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ✓ DONE
  4.  The client commits and the server confirms      (Slides 9–10)  ← YOU ARE HERE
  5.  When things go wrong: troubleshooting addresses (Slide 11)

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14)
  7.  How lookups actually work and why caching matters (Slides 15–18)
  8.  Zones, delegation, and network architecture      (Slides 19–22)
  9.  Public DNS, latency, and real-world performance   (Slides 23–25)
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32)
```

---

## Where We Are in DORA

Two steps done. The client broadcast a Discover ("I need an address"), and a server replied with an Offer ("Here is `192.168.1.100` — want it?"). Now the client must formally accept the deal (Request) and the server must confirm it (Acknowledge). These are the final two letters in DORA (**D**iscover, **O**ffer, **R**equest, **A**cknowledge).

What makes the Request step interesting is that it is not a private conversation between the client and the server that made the offer. The client broadcasts the Request to the entire network. This is deliberate — it serves as a public announcement: "I am accepting the offer from server `192.168.1.1`." Every other DHCP server on the network hears this and knows the client chose someone else. They withdraw their offers and return those addresses to their pools. One broadcast, multiple servers informed.

---

## Slide 9 — DORA: Request (Slide 9 of 32)

**What is on the slide:** A heading "DO**R**A - **R**equest" with the letter R highlighted in red. The left half contains four explanatory bullet points. The right half contains the DHCPREQUEST packet structure table. Above the packet, a large red-circled annotation shows `192.168.1.100` — the address the client is formally requesting.

Unlike the Discover and Offer steps, which had separate slides for the diagram and the packet, the Request combines both the explanation and the packet structure on a single slide. There is no separate diagram slide for the Request — Ian condensed it.

---

### What the Slide Text Explains

**Bullet 1:** *"Client now replies with a DHCP request, broadcast to the server, requesting the offered address."*

The client has chosen which Offer to accept. It responds with a DHCPREQUEST message. The critical word here is **broadcast**. The client still does not have a confirmed IP address — the Offer was a proposal, not a confirmation. So the client still uses `0.0.0.0` as its source IP and sends to `255.255.255.255` as the destination. Same broadcast mechanics as the original Discover.

But why broadcast? The client already knows which server it wants to talk to (it has the server's IP from Option 54 in the Offer). It could, in theory, send a targeted message. The reason it broadcasts is strategic, not technical. It broadcasts so that **all** DHCP servers on the network can hear the announcement — not just the one the client chose.

**Bullet 2:** *"A client can receive DHCP offers from multiple servers, but it will accept only one DHCP offer."*

This reinforces what we covered in Slide 7. If three DHCP servers all heard the Discover and each sent an Offer, the client now has three proposals on the table. It picks one (typically the first one to arrive) and ignores the rest. The Request message is the formal "I choose you" moment.

**Bullet 3:** *"Based on required server identification option in the request and broadcast messaging, servers are informed whose offer the client has accepted."*

The "server identification option" is **Option 54** in the DHCP Options section — it contains the IP address of the server whose offer the client is accepting. Because the Request is broadcast, every DHCP server on the network receives it. Each server looks at Option 54. If it sees its own IP, it knows it won. If it sees someone else's IP, it knows it lost.

**Bullet 4:** *"When other DHCP servers receive this message, they withdraw any offers that they might have made to the client and return the offered address to the pool of available addresses."*

This is the cleanup. The losing servers take back their offered addresses and make them available for other clients. Without this mechanism, those addresses would be stuck in limbo — reserved for a client that will never use them — until some internal timeout freed them. The broadcast Request ensures instant cleanup.

---

### The DHCPREQUEST Packet — What Changed

**UDP (User Datagram Protocol) header:**

```
UDP Src=0.0.0.0  sPort=68
Dest=255.255.255.255  dPort=67
```

Back to the Discover's addressing pattern. Source `0.0.0.0` (client still has no confirmed IP), destination `255.255.255.255` (broadcast). The Request is client-to-server, so it uses port 68 (client) as source and port 67 (server) as destination.

**Key field changes from the Offer packet:**

| Field | Offer Value | Request Value | What Changed and Why |
|-------|------------|---------------|---------------------|
| **OP** | `0x02` (reply) | `0x01` (request) | Direction flipped again. This is a client-to-server message, so OP returns to `0x01`. The pattern alternates: Discover (`0x01`) → Offer (`0x02`) → Request (`0x01`) → Acknowledge (`0x02`). Client sends = `0x01`. Server sends = `0x02`. |
| **CIADDR** | `0x00000000` | `0x00000000` | Still zero. CIADDR (Client IP Address) remains empty because the client has not configured its interface yet — the address is still just a proposal, not confirmed. CIADDR is only used during lease renewals when the client already has an established address. |
| **YIADDR** | `0xC0A80164` | `0x00000000` | Back to zero. YIADDR ("Your IP Address") is a field the server fills in, not the client. The client does not write the requested address here — it puts it in Option 50 instead. |
| **SIADDR** | `0xC0A80101` | `0xC0A80101` | Unchanged — still `192.168.1.1`. SIADDR (Server IP Address) — the client echoes back the server's IP to confirm which server it is talking to. |

**The DHCP Options — the client's formal acceptance:**

**Option 53: DHCP Request** — Message type. Step 3 of DORA.

**Option 50: `192.168.1.100` requested** — The client formally requests the specific IP that was offered. This is the same Option 50 that appeared in the original Discover (where the client asked "can I have my old address back?"), but now it carries more weight — the client is saying "I was offered this address and I want it."

**Option 54: `192.168.1.1` DHCP server** — The server identification option. This is the key field that makes the broadcast useful. Every DHCP server on the network reads this. Server `192.168.1.1` sees its own IP and knows the client accepted its offer. Any other server sees a different IP and knows it was rejected — time to withdraw the offer and reclaim the address.

**What is NOT in the Request options:** Notice that the Request does not include Option 55 (Parameter Request List). The client already told the server what it needed in the Discover. The server already included everything in the Offer. There is no need to ask again.

---

## Slide 10 — DORA: Acknowledgment (Slide 10 of 32)

**What is on the slide:** A heading "DOR**A** - **A**cknowledgment" with the letter A highlighted in red. The left half contains three bullet points and a critical note about ARP (Address Resolution Protocol — a protocol that maps IP addresses to MAC — Media Access Control — addresses on a local network). The right half contains the DHCPACK packet structure table.

This is the final step. The server receives the Request, verifies everything checks out, and sends a confirmation — the **DHCPACK** (DHCP Acknowledgment). Once the client receives this, the deal is sealed. The client configures its network interface with the assigned IP address, subnet mask, gateway, and DNS servers. DORA is complete.

---

### What the Slide Text Explains

**Bullet 1:** *"The acknowledgement phase involves sending a DHCPACK packet to the client."*

Straightforward. The server confirms the assignment.

**Bullet 2:** *"Packet includes: lease duration, any other configuration information that the client might have requested."*

The DHCPACK contains the same configuration details as the Offer — subnet mask, router, lease time, DNS servers. This is the final, binding version. The Offer was a proposal. The ACK is the signed contract.

**Bullet 3:** *"The protocol expects the DHCP client to configure its network interface with the negotiated parameters."*

Once the ACK arrives, the client's job is clear: take the IP address from YIADDR, the subnet mask from Option 1, the gateway from Option 3, and the DNS servers from Option 6, and apply them to the network interface. From this moment, the device has a fully working network configuration. It can communicate with any device on its local subnet, reach the internet through the gateway, and resolve domain names through the DNS servers.

**Bullet 4 (the important one):** *"After the client obtains an IP address, it should probe the newly received address (e.g. with ARP) to prevent address conflicts caused by overlapping address pools of DHCP servers."*

This is a safety check that most students overlook. Even after DORA completes successfully, there is still a tiny risk: what if two different DHCP servers have overlapping scopes (both include `192.168.1.100` in their pool) and both assigned it to different clients? Or what if someone manually configured a static device with `192.168.1.100` without telling the DHCP server?

To catch this, the client performs an **ARP probe** immediately after receiving the ACK. It sends an ARP request for its own new IP address: "Who has `192.168.1.100`?" If nobody responds, the address is clear — nobody else is using it. The client proceeds to configure the interface.

If something DOES respond — meaning another device already has that IP — the client knows there is a conflict. It sends a **DHCPDECLINE** message back to the server, refuses the address, and restarts the entire DORA process from scratch. The server marks that address as unavailable and offers a different one next time.

This ARP probe is a last line of defence. In a properly managed network with clean exclusions and non-overlapping scopes, it should never find a conflict. But networks are messy, humans make mistakes, and this safety check prevents a misconfiguration from silently breaking connectivity for two devices at once.

---

### The DHCPACK Packet — What Changed

**UDP header:**

```
UDP Src=192.168.1.1  sPort=67
Dest=255.255.255.255  dPort=68
```

Server-to-client direction again, same as the Offer. Server sends from `192.168.1.1:67`, destination is broadcast `255.255.255.255:68`.

**Key field changes from the Request packet:**

| Field | Request Value | ACK Value | What Changed and Why |
|-------|-------------|-----------|---------------------|
| **OP** | `0x01` (request) | `0x02` (reply) | Back to `0x02` — server sending to client. The DORA pattern completes: `0x01` → `0x02` → `0x01` → `0x02`. |
| **YIADDR** | `0x00000000` | `0xC0A80164` | The server writes `192.168.1.100` into YIADDR again — confirming the assignment. This is no longer a proposal (as it was in the Offer). This is the finalised address. |
| **SIADDR** | `0xC0A80101` | `0xC0A80101` | Unchanged — `192.168.1.1`. The server identifies itself. |

**A subtle label change on the slide:** GIADDR's description on Slide 10 says "Gateway IP address switched by relay" — slightly different wording from previous slides, which just said "Gateway IP address." The added "switched by relay" is a hint about relay agents. When a relay agent (a router that forwards DHCP messages across subnets) is involved, it writes its own IP into GIADDR so the server knows which subnet the client is on. In this example GIADDR is still `0x00000000` (no relay agent), but the label change foreshadows the relay concept used in the Workshop 5 DHCP lab.

---

### The DHCP Options — The Final Confirmation

**Option 53: DHCP ACK (value=5) or DHCP NAK (value=6)** — This is the only option that can go two ways.

**DHCPACK (value 5)** means "approved — here is your address." This is the normal, successful outcome.

**DHCPNAK (value 6)** — NAK stands for **N**egative **A**c**k**nowledgment — means "rejected." The server is saying "no, you cannot have that address." This can happen if:
- The requested address is no longer available (someone else grabbed it between the Offer and the Request)
- The client moved to a different subnet and is requesting an address that does not belong to this network
- The server's configuration changed between the Offer and the Request

If the client receives a DHCPNAK, it must restart the entire DORA process. Back to square one.

**Options 1, 3, 51, 54, 6** — identical to the Offer. The same configuration is repeated in the ACK as a final, binding confirmation:
- **Option 1:** `255.255.255.0` subnet mask
- **Option 3:** `192.168.1.1` router (default gateway)
- **Option 51:** `86400s` (1 day) lease duration
- **Option 54:** `192.168.1.1` DHCP server identifier
- **Option 6:** DNS (Domain Name System) servers `9.7.10.15`, `9.7.10.16`, `9.7.10.18`

---

## DORA Complete — The Full Picture

The entire four-step process, start to finish:

```
CLIENT (no IP)                                         SERVER (192.168.1.1)
     |                                                       |
     |  1. DHCPDISCOVER (broadcast)                          |
     |  "I need an IP address!"                              |
     |------------------------------------------------------>|
     |                                                       |
     |  2. DHCPOFFER                                         |
     |  "Here, take 192.168.1.100 for 24 hours"             |
     |<------------------------------------------------------|
     |                                                       |
     |  3. DHCPREQUEST (broadcast)                           |
     |  "I accept the offer from 192.168.1.1"               |
     |------------------------------------------------------>|
     |                                                       |
     |  4. DHCPACK                                           |
     |  "Confirmed. 192.168.1.100 is yours."                |
     |<------------------------------------------------------|
     |                                                       |
     |  [Client runs ARP probe for 192.168.1.100]           |
     |  [No conflict detected → configures interface]       |
     |                                                       |
     |  CLIENT IS NOW ONLINE                                 |
     |  IP: 192.168.1.100                                    |
     |  Mask: 255.255.255.0                                  |
     |  Gateway: 192.168.1.1                                 |
     |  DNS: 9.7.10.15, 9.7.10.16, 9.7.10.18               |
```

Four messages. Two broadcasts from the client (Discover and Request), two replies from the server (Offer and ACK). The entire process typically completes in milliseconds on a local network. The user turns on their laptop, and by the time the desktop loads, they are already online with a fully configured IP stack.

The next slide — and the next chunk — covers what happens when this process fails. What does a device look like when DHCP cannot reach a server? That is where `169.254.x.x` and `0.0.0.0` enter the picture.

---

*Chunk 4 of 10 complete. Next chunk covers Slide 11: Troubleshooting addresses — APIPA and 0.0.0.0.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 5 of 10 — Slide 11: When DHCP Fails — Troubleshooting Addresses

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ✓ DONE
  4.  The client commits and the server confirms      (Slides 9–10)  ✓ DONE
  5.  When things go wrong: troubleshooting addresses (Slide 11)     ← YOU ARE HERE

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14)
  7.  How lookups actually work and why caching matters (Slides 15–18)
  8.  Zones, delegation, and network architecture      (Slides 19–22)
  9.  Public DNS, latency, and real-world performance   (Slides 23–25)
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32)
```

---

## Why This Slide Matters

The previous six slides (5–10) described DORA (Discover, Offer, Request, Acknowledge) as a clean, four-step process that always works. But what happens when it does not work? What does a device look like when it tries to get an IP address from a DHCP server and fails?

This slide answers that question. It covers two specific IP addresses that a network administrator should immediately recognise as distress signals. If you see either of these addresses on a client device, something has gone wrong with DHCP — and this slide tells you what.

This is a short slide — just two bullet clusters — but in a troubleshooting scenario, it is one of the most practically useful slides in the entire lecture. Every helpdesk technician, every network administrator, and every student doing a Packet Tracer lab will eventually see `169.254.x.x` or `0.0.0.0` and need to know what it means.

---

## Slide 11 — Addresses to Troubleshoot (Slide 11 of 32)

**What is on the slide:** A heading "Addresses to troubleshoot" followed by two main bullet points — one for `169.254.x.x` and one for `0.0.0.0`. The phrase "when not connected to a TCP/IP network" is highlighted in red. The words "TCP/IP" and "DHCP" appear as underlined hyperlinks (likely linking to external resources in Ian's original presentation). No images.

**Typo on the slide:** The third sub-bullet under `169.254.x.x` reads "DHCP server is down or not **repsonding**" — this should be "responding." A copy-paste or typing error that was never caught.

---

### Address 1: `169.254.x.x` — APIPA

The slide says: *"169.254.x.x: This is what's called an Automatic Private IP address."*

**APIPA** stands for **Automatic Private IP Addressing** — a fallback mechanism built into most modern operating systems (Windows, macOS, Linux). When a device is configured to use DHCP but cannot reach a DHCP server, it does not simply sit there with no address at all. Instead, it assigns itself a random IP address from the range `169.254.1.0` to `169.254.254.255` (the first and last /24 blocks within `169.254.0.0/16` are reserved and cannot be used), with a subnet mask of `255.255.0.0` (a /16 network).

**Why this exists:** A device with absolutely no IP address cannot do anything on a network — it cannot even communicate with the device sitting next to it on the same switch. APIPA gives it a minimal, temporary identity so that at least basic local communication is possible while the device continues to search for a proper DHCP server.

**How the device ends up here — the timeline:**

1. The device boots up and sends a DHCPDISCOVER broadcast.
2. No DHCP server responds. Maybe the server is down, maybe the network cable is faulty, maybe a relay agent is misconfigured, maybe a firewall is blocking UDP (User Datagram Protocol) ports 67/68.
3. The device waits a few seconds and retries. Still no response.
4. After multiple failed attempts (typically 3–4 retries over about 30 seconds), the operating system gives up on DHCP — for now — and assigns itself an APIPA address from `169.254.x.x`.
5. Before using that self-assigned address, the device performs an ARP (Address Resolution Protocol — maps IP addresses to MAC — Media Access Control — addresses on a local network) probe to make sure no other device has already claimed the same `169.254.x.x` address. If there is a conflict, it picks a different random address from the range and probes again.
6. The device configures its interface with the APIPA address and continues to periodically send DHCPDISCOVER broadcasts in the background, hoping a server will eventually come online.

**What the slide tells you about APIPA's capabilities:**

*"Gives you basic comms to other resources with 169.254 IP addresses, but will periodically send out a DHCP request."*

This is a critical limitation. APIPA only allows communication with other devices that are also in the `169.254.x.x` range on the same local network segment. It does not provide:

- A **default gateway** — so the device cannot reach anything outside the local network. No internet, no access to servers on other subnets.
- **DNS (Domain Name System) servers** — so the device cannot resolve domain names. Even if it could somehow reach the internet, typing `www.google.com` would produce nothing because there is no DNS server to translate the name into an IP address.
- Any **subnet mask** other than `255.255.0.0` — so the device thinks its local network is the entire `169.254.0.0/16` range.

In practical terms, a device with an APIPA address is nearly useless for real work. It can talk to other APIPA devices on the same switch — which is almost never what anyone wants. The only purpose APIPA serves is keeping the network interface alive while waiting for DHCP to recover. It is a holding pattern, not a solution.

**What to do when you see `169.254.x.x` on a device:**

This is a diagnostic signal that means "DHCP failed." Your troubleshooting path should be:

1. **Is the network cable connected?** (Physical layer — Layer 1)
2. **Is the switch port active?** (Check for link lights)
3. **Is the DHCP server running?** (Log into the server and check the DHCP service)
4. **Is there a relay agent between the client and the server?** (If they are on different subnets, the router must have `ip helper-address` configured — this is exactly what Workshop 5 covers)
5. **Is a firewall blocking UDP ports 67/68?** (DHCP uses UDP, not TCP — Transmission Control Protocol — a firewall that allows TCP but blocks UDP will silently kill DHCP)

**Real-world example for students:** You are in the university lab. You open a command prompt and type `ipconfig` (on Windows) or `ifconfig` / `ip addr` (on Linux). You see `169.254.47.132` as your IP address. You know immediately: "My laptop tried to get an address from the DHCP server and failed. The server is either down, the network is broken between me and the server, or the relay agent on the router is not configured." You do not need to guess. The address itself tells you the story.

---

### Address 2: `0.0.0.0` — No Address At All

The slide says: *"0.0.0.0 on a client - PCs and other client devices normally show this address when not connected to a TCP/IP network."*

Where APIPA is a fallback ("I tried and failed, but here is a temporary address"), `0.0.0.0` is more absolute — it means the device has no IP address whatsoever. The network interface exists, but it has not been configured with any address at all.

**When you see `0.0.0.0`:**

**Scenario 1 — The device is offline.** The network cable is unplugged, Wi-Fi is disconnected, or the network adapter is disabled. The device is not on any network, so it has no address. This is the most common cause.

**Scenario 2 — DHCP assignment failure.** The slide says: *"It might also be automatically assigned by DHCP in the case of address assignment failures."* This happens when the DHCP process starts but fails catastrophically — for example, the client sent a DHCPREQUEST but received a DHCPNAK (Negative Acknowledgment — the server rejected the request), and the operating system has not yet fallen back to APIPA. In this brief window between rejection and APIPA kicking in, the device may show `0.0.0.0`.

**The difference between `0.0.0.0` and `169.254.x.x`:**

Think of it as two stages of failure:

```
STAGE 1: 0.0.0.0
  "I have nothing. I have not even tried yet, or I tried and
   was actively rejected, or I am not connected to anything."

STAGE 2: 169.254.x.x
  "I tried to get a real address, failed, and gave myself
   a temporary fallback. I am still trying in the background."
```

A device showing `0.0.0.0` is either not on the network at all or is in the earliest stages of a DHCP failure. A device showing `169.254.x.x` has been on the network long enough to try DHCP, fail, and fall back to APIPA. Both are problems, but they point to different diagnostic starting points.

**TCP/IP note from the slide:** The phrase "when not connected to a **TCP/IP** network" uses the term TCP/IP (Transmission Control Protocol / Internet Protocol — the fundamental protocol suite that underlies virtually all modern networking). This is just another way of saying "not connected to any network that uses IP addressing," which in practice means any network at all. The slide uses TCP/IP as a formal term rather than simply saying "network."

---

## End of DHCP — The Complete Picture

Slide 11 closes the DHCP section of this lecture. Across slides 1–11, the lecture has covered:

- **Why DHCP exists** — manual addressing does not scale (Slides 1–3)
- **How the address pool works** — scopes, exclusions, and leases (Slide 4)
- **How a device gets an address** — the four-step DORA process, with full packet-level detail (Slides 5–10)
- **What failure looks like** — APIPA and 0.0.0.0 as diagnostic signals (Slide 11)

Starting with the next slide, the lecture shifts to the second half: **DNS** — the system that translates human-readable names into the IP addresses that DHCP just assigned. DHCP answers "what is my IP?" DNS answers "what is their IP?" Together, they form the two essential infrastructure services that make a network usable.

---

*Chunk 5 of 10 complete. Next chunk covers Slides 12–14: DNS namespace tree, domain name servers, and resolver types.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 6 of 10 — Slides 12–14: The Naming Tree and Who Translates Names to IPs

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ✓ DONE
  4.  The client commits and the server confirms      (Slides 9–10)  ✓ DONE
  5.  When things go wrong: troubleshooting addresses (Slide 11)     ✓ DONE

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14) ← YOU ARE HERE
  7.  How lookups actually work and why caching matters (Slides 15–18)
  8.  Zones, delegation, and network architecture      (Slides 19–22)
  9.  Public DNS, latency, and real-world performance   (Slides 23–25)
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32)
```

---

## Entering DNS — The Second Half of the Lecture

The DHCP section answered one question: "How does a device get an IP address?" Now the lecture shifts to the second question: "How does a device find someone else's IP address when all it knows is a human-readable name?"

Every time you type `www.wlv.ac.uk` into a browser, your device needs to discover which IP address that name corresponds to before it can send a single packet. The system that performs this translation is the **DNS** — the **Domain Name System**. It is one of the most critical and most invisible services on the internet. When it works, nobody notices. When it breaks, the entire internet appears to be "down" — even though the network itself is fine. Users say "the internet is broken" when really the only thing broken is name resolution.

DNS is also the service that DHCP hands to clients as part of the configuration package. Remember Option 6 in the DORA (Discover, Offer, Request, Acknowledge) exchange? That was the DNS server addresses. DHCP tells the device "here is where to ask for name translations." This section explains what happens when the device actually asks.

---

## Slide 12 — Domain Namespace System (Slide 12 of 32)

**What is on the slide:** A heading "Domain Namespace System" (note: the slide title says "Namespace" not "Name" — both terms are used interchangeably in the industry, but the official protocol name is Domain Name System). Two bullet points of text and a tree diagram showing the hierarchical structure of DNS.

**The slide text:**

*"The DNS Namespace is based on a 'tree' structure, with a small number of generic Top Level Domains (eg, .com, .edu, .org) and a large number of country-based domains (eg .au, .my, .uk). Each TLD supports a group of 'second-level' domains, and so on, all the way down to individual hosts."*

**TLD** stands for **Top Level Domain** — the rightmost segment of a domain name (the part after the last dot).

---

### The Tree Diagram — DNS Hierarchy Visualised

The diagram on the slide shows a tree growing downward from a single root, with five levels:

```
                           . (root)
                            │
            ┌───────┬───────┼───────┬───────┐
           .net   .edu    .com    .net    .gov         ← Top Level Domains (TLDs)
                            │
                      example.com                      ← Second Level Domain
                            │
            ┌───────────────┼───────────────┐
   cluster.example.com  mail.example.com  www.example.com   ← Subdomains
            │
  node1.cluster.example.com                            ← Individual Host
```

(Note: `.net` appears twice in Ian's original diagram — this is a copy-paste error on the slide. One of them was likely intended to be a different TLD such as `.org`. The structure is still correct; just the label is duplicated.)

**Level 1 — The Root (`.`):** At the very top of the tree is a single dot. Most people have never seen it, but technically every fully qualified domain name ends with a dot: `www.google.com.` (with a trailing dot). The root is managed by 13 sets of root nameservers distributed globally (labelled A through M). These are the starting point for every DNS lookup that cannot be answered from cache.

**Level 2 — Top Level Domains (TLDs):** Directly below the root sit the TLDs. The slide shows two types:

- **Generic TLDs (gTLDs):** `.com`, `.edu`, `.org`, `.net`, `.gov` — originally designated for specific purposes (commercial, educational, organisations, network infrastructure, government). In practice, `.com` is used for almost everything.
- **Country-code TLDs (ccTLDs):** `.au` (Australia), `.my` (Malaysia), `.uk` (United Kingdom). Every country has its own. Uzbekistan has `.uz`. These often have their own internal structure — for example, `.uk` has `.ac.uk` for academic institutions and `.co.uk` for commercial entities.

**Level 3 — Second Level Domains:** These are the names that organisations register. `example.com`, `google.com` are genuine second-level domains (sitting directly under a TLD). In countries with structured namespaces like the UK, the registration level is actually one step deeper — `wlv.ac.uk` is technically a third-level domain (under `.ac` under `.uk`), but it functions as the equivalent of a second-level domain because `.ac.uk` is the effective TLD for UK academic institutions. This is the level where ownership begins — someone paid a registrar to control this name.

**Level 4 — Subdomains:** The owner of a second-level domain can create any number of subdomains underneath it. `www.example.com`, `mail.example.com`, `cluster.example.com` — these are all controlled by whoever owns `example.com`. No additional registration is needed. The domain owner simply configures their DNS server to include these names.

**Level 5 — Individual Hosts:** The leaves of the tree. `node1.cluster.example.com` is a specific machine — a server, a workstation, a container — with a specific IP address. This is where DNS resolution ends: a name that maps directly to an IP.

**Why this tree structure matters:** DNS is not one giant database. It is a distributed system where each level of the tree is managed by different organisations. The root servers know about TLDs. The TLD servers know about second-level domains. The second-level domain servers know about their own subdomains and hosts. Nobody knows everything. Each server knows its own piece and can point you to the next server in the chain. This distributed architecture is what makes DNS scalable to billions of names — and also what makes it resilient. No single point of failure can take down the entire system (though individual points of failure can take down specific domains).

---

## Slide 13 — Domain Name Servers (Slide 13 of 32)

**What is on the slide:** A heading "Domain Name Servers" followed by two short bullet points, and below them a screenshot of a Windows Command Prompt showing the output of the `ipconfig /all` command on a real machine at the University of Wolverhampton.

**The slide text:**

*"Need to translate the web address used in a browser such as www.wlv.ac.uk into actual IP address so a request can be made to that web browser to access a web page."*

(Note: the slide says "a request can be made to that web browser" — this should say "web server," not "web browser." The browser is on the user's device; the request goes to the web server. A minor slip in Ian's wording.)

*"The DNS system is the mechanism by which this happens."*

These two bullets restate what was introduced on Slide 2 — DNS translates names to IPs. Simple and direct.

---

### The Command Prompt Screenshot — Real DNS in Action

The screenshot is the most valuable part of this slide. It shows `ipconfig /all` output from a real Wolverhampton university machine, with several fields highlighted in yellow:

```
Ethernet adapter Ethernet 3:

  Connection-specific DNS Suffix . : wlv.ac.uk
  Description . . . . . . . . . . : Lenovo USB Ethernet
  Physical Address. . . . . . . . : F8-75-A4-F8-F9-07
  DHCP Enabled. . . . . . . . . . : Yes
  Autoconfiguration Enabled . . . : Yes
  Link-local IPv6 Address . . . . : fe80::81f5:774e:ea35:73c2%13(Preferred)
  IPv4 Address. . . . . . . . . . : 134.220.250.109(Preferred)
  Subnet Mask . . . . . . . . . . : 255.255.254.0
  Lease Obtained. . . . . . . . . : 18 February 2022 13:00:39
  Lease Expires . . . . . . . . . : 02 March 2022 09:33:28
  Default Gateway . . . . . . . . : 134.220.250.1
  DHCP Server . . . . . . . . . . : 134.220.1.39
  DNS Servers . . . . . . . . . . : 134.220.1.39
                                     134.220.1.20
```

**What this screenshot proves — DHCP and DNS connected in the real world:**

Everything this lecture has taught so far is visible in one screenshot:

- **DHCP Enabled: Yes** — this machine got its IP via DHCP, exactly as described in the DORA process.
- **IPv4 Address: `134.220.250.109`** — the address assigned by the university's DHCP server.
- **Subnet Mask: `255.255.254.0`** — a /23 network (512 total addresses, 510 usable after subtracting the network and broadcast addresses), handed out as part of the DHCP configuration package (Option 1).
- **Lease Obtained / Lease Expires** — highlighted in yellow. The lease started on 18 February 2022 and expires on 02 March 2022. That is approximately a 12-day lease — much longer than the 24-hour example in the DORA slides. Wolverhampton's network administrators chose a longer lease because university devices tend to stay connected for extended periods.
- **Default Gateway: `134.220.250.1`** — Option 3 from DHCP.
- **DHCP Server: `134.220.1.39`** — the server that handed out this configuration. This is the real-world equivalent of `192.168.1.1` from the DORA examples.
- **DNS Servers: `134.220.1.39` and `134.220.1.20`** — highlighted in yellow. Two DNS servers, provided by DHCP (Option 6). Notice that `134.220.1.39` is both the DHCP server and a DNS server — a common configuration where one infrastructure server handles multiple roles.

**The bridge between DHCP and DNS:** This screenshot is the physical proof that DHCP Option 6 works. The DHCP server told this machine "your DNS servers are `134.220.1.39` and `134.220.1.20`." Now, whenever this machine needs to resolve a domain name, it knows exactly where to ask. That is the handoff from Part 1 of this lecture to Part 2.

---

## Slide 14 — Types of DNS Resolvers (Slide 14 of 32)

**What is on the slide:** A heading "Types of DNS resolvers" followed by two sections — one describing the **stub resolver** (in red text) and one describing the **recursive resolver** (in teal/cyan text, underlined as a hyperlink). No images.

A **resolver** is any software component that performs DNS lookups — it takes a domain name as input and returns an IP address as output. But not all resolvers work the same way. This slide introduces two types that sit at different points in the DNS lookup chain.

---

### Type 1: The Stub Resolver

The slide says: *"A **stub resolver** is a software component normally found in endpoint hosts that generates DNS queries when application programs running on desktop computers or mobile devices need to resolve DNS domain names. Keeps a cache of recently resolved queries."*

The **stub resolver** lives inside your device — your laptop, your phone, your desktop PC. It is built into the operating system. Every time an application on your device needs to look up a domain name (your browser needs to reach `www.google.com`, your email client needs to find `mail.wlv.ac.uk`, a system update needs to contact `update.microsoft.com`), it asks the stub resolver.

The stub resolver is called "stub" because it is intentionally simple — a stripped-down, minimal component. It does not do the heavy lifting of actually walking the DNS tree from root to TLD to authoritative server. It is not smart enough for that. All it does is:

1. Check its local cache — "Have I looked this up recently? If so, I already know the answer."
2. If not cached, forward the query to a **recursive resolver** (described below) and wait for the answer.
3. Cache the answer for next time (for a duration defined by the TTL — Time To Live — a timer value included in every DNS response that tells the recipient how long the answer remains valid).

Think of the stub resolver as a lazy but efficient assistant. It does not know how to research anything itself. When you ask it a question, it checks its notes first (the cache). If the answer is not there, it passes the question to someone smarter (the recursive resolver) and writes down the answer when it comes back.

**Why caching matters here:** If you visit `www.google.com` ten times in an hour, the stub resolver looks up the IP once and remembers it for the remaining nine visits. Without this local cache, every single request would generate network traffic to the DNS server — thousands of unnecessary queries per day from every device on the network.

---

### Type 2: The Recursive Resolver

The slide says: *"The **recursive resolver** may reside in a home router, be hosted by an internet service provider or be provided by a third party, such as Google's Public DNS recursive resolver at 8.8.8.8 or the Cloudflare DNS service at 1.1.1.1."*

The **recursive resolver** is the workhorse. When the stub resolver on your device does not know the answer, it sends the query to the recursive resolver. The recursive resolver then does the actual work of walking the DNS tree — querying the root servers, then the TLD servers, then the authoritative servers — until it finds the IP address that matches the requested domain name.

The word "recursive" describes how it works: it starts at the top of the DNS tree and works its way down, level by level, making multiple queries to multiple servers until it reaches the final answer. It recursively breaks the problem down from the general (root) to the specific (the authoritative server for that exact domain).

**Where the recursive resolver lives — three common locations:**

**In your home router:** Many consumer routers include a built-in recursive resolver. When your laptop asks the router "what is the IP for `www.google.com`?", the router does the lookup itself rather than passing the query upstream. It also maintains its own cache, so if anyone else on your home network asks the same question within the cache lifetime, the answer comes back instantly.

**At your ISP (Internet Service Provider — the company that provides your internet connection):** Most ISPs run recursive resolvers for their customers. When you connect to your ISP, your device is typically configured (via DHCP) to use the ISP's DNS servers. These servers handle millions of queries from thousands of customers and maintain large caches.

**At a third-party provider:** Google (`8.8.8.8` and `8.8.4.4`) and Cloudflare (`1.1.1.1`) run public recursive resolvers that anyone can use. These are popular alternatives to ISP-provided DNS for several reasons: they are often faster (geographically distributed with heavy caching), more reliable (massive infrastructure), and more privacy-conscious (some ISPs log and sell DNS query data, while providers like Cloudflare explicitly promise not to). The slide specifically names both Google and Cloudflare as examples.

**The relationship between stub and recursive:**

```
YOUR DEVICE                         RECURSIVE RESOLVER              DNS TREE
(stub resolver)                     (ISP / Google / Cloudflare)     (root → TLD → auth)
     │                                      │                            │
     │  "What is www.wlv.ac.uk?"            │                            │
     │─────────────────────────────────────>│                            │
     │                                      │  "Who handles .uk?"        │
     │                                      │───────────────────────────>│ root
     │                                      │<───────────────────────────│
     │                                      │  "Who handles .ac.uk?"     │
     │                                      │───────────────────────────>│ .uk TLD
     │                                      │<───────────────────────────│
     │                                      │  "What is www.wlv.ac.uk?" │
     │                                      │───────────────────────────>│ wlv.ac.uk auth
     │                                      │<───────────────────────────│
     │  "It is 134.220.x.x"                │                            │
     │<─────────────────────────────────────│                            │
     │                                      │                            │
     │  [stub caches the answer]            │  [recursive caches too]    │
```

The stub resolver asks one question and gets one answer. The recursive resolver does all the legwork — multiple queries across multiple servers — and delivers the final result. Both cache the answer so neither has to do the work again until the TTL expires.

---

## What These Three Slides Established

Slide 12 showed the structure — DNS is a hierarchical tree, not a flat database. Slide 13 proved it works in the real world — a screenshot of an actual university machine with DHCP-assigned DNS servers. Slide 14 introduced the players — the stub resolver on your device and the recursive resolver that does the actual lookups. The next four slides (15–18) will show exactly how those lookups work, step by step, in two different modes: iterative and recursive.

---

*Chunk 6 of 10 complete. Next chunk covers Slides 15–18: Iterative vs recursive queries, DNS caching, and replicas/load balancing.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 7 of 10 — Slides 15–18: How Lookups Actually Work and Why Caching Matters

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ✓ DONE
  4.  The client commits and the server confirms      (Slides 9–10)  ✓ DONE
  5.  When things go wrong: troubleshooting addresses (Slide 11)     ✓ DONE

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14) ✓ DONE
  7.  How lookups actually work and why caching matters (Slides 15–18) ← YOU ARE HERE
  8.  Zones, delegation, and network architecture      (Slides 19–22)
  9.  Public DNS, latency, and real-world performance   (Slides 23–25)
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32)
```

---

## What This Section Covers

The previous chunk introduced the players: the stub resolver on your device and the recursive resolver that does the heavy lifting. But it left a critical question unanswered: when the recursive resolver needs to walk the DNS tree, **how exactly does that conversation unfold?**

It turns out there are two fundamentally different ways a DNS query can be processed: **iterative** and **recursive**. These two slides (15 and 16) are the most important diagrams in the entire DNS section because they show the actual message flow — who talks to whom, in what order, and who does the work. After that, Slide 17 explains why the whole system does not collapse under the weight of billions of queries per day (the answer: caching), and Slide 18 reveals a clever trick that large websites use to distribute traffic across multiple servers using DNS.

---

## Slide 15 — DNS Resolution: Iterated Query (Slide 15 of 32)

**What is on the slide:** A heading "DNS RESOLUTION: ITERATED QUERY" (no title in the extracted text — the title is embedded in the image). A diagram showing the step-by-step flow of an iterative DNS query with eight numbered arrows. On the left, a text block from the perspective of the local DNS server. No speaker notes.

**The slide has no title in the PPTX text layer** — only the diagram image carries the heading. This is one of the slides Ian flagged in our earlier overview as having no title, just a diagram. Students might not immediately know which query mode they are looking at without the instructor pointing it out.

---

### The Iterative Query Diagram — Step by Step

The diagram shows five actors arranged in a diamond-like layout:

- **Bottom-left:** Requesting host (`cis.myuni.edu`) — the device that originally needs the IP address
- **Centre-left:** Local DNS server (`dns.myuni.edu`) — the recursive resolver
- **Top-centre:** Root DNS server
- **Centre-right:** TLD (Top Level Domain) DNS server
- **Bottom-right:** Authoritative DNS server (`dns.cs.yours.edu`) — the server that owns the final answer
- **Below the authoritative server:** The target: `www.cs.yours.edu`

Eight numbered arrows trace the query flow:

```
Step 1: Requesting host → Local DNS server
  "What is the IP for www.cs.yours.edu?"
  (This is the stub resolver asking the recursive resolver.)

Step 2: Local DNS server → Root DNS server
  "Who handles .edu?"
  (The local server starts at the top of the tree.)

Step 3: Root DNS server → Local DNS server
  "I don't know the final answer, but try the .edu TLD server."
  (The root gives a REFERRAL — a pointer to the next server.)

Step 4: Local DNS server → TLD DNS server
  "Who handles yours.edu?"
  (The local server follows the referral and asks the next level.)

Step 5: TLD DNS server → Local DNS server
  "I don't know the final answer either, but try dns.cs.yours.edu."
  (Another referral — one level deeper.)

Step 6: Local DNS server → Authoritative DNS server
  "What is the IP for www.cs.yours.edu?"
  (The local server follows the second referral.)

Step 7: Authoritative DNS server → Local DNS server
  "It is [IP address]."
  (The authoritative server has the definitive answer.)

Step 8: Local DNS server → Requesting host
  "It is [IP address]."
  (The local server passes the final answer back to the client.)
```

**The text on the slide** captures the local server's attitude in iterative mode:

*"Hey, I need the IP address for this domain. Please let me know the address of the next DNS server in the lookup process so I can look it up myself."*

**What "iterative" means:** In iterative mode, each server the local DNS server contacts gives back a **referral** — "I don't know the answer, but here is who might." The local DNS server then has to do the next query itself. It iterates — step by step, server by server — working its way down the tree. The root does not go ask the TLD on the local server's behalf. The TLD does not go ask the authoritative server on the local server's behalf. Each server says "not my job, try them" and the local server does all the walking.

Think of it like asking for directions in a city. You ask a stranger, "Where is the post office?" They say, "I don't know, but that guy over there might." You walk to that person. They say, "Two blocks east, ask at the shop." You walk there. The shop says, "It's right behind this building." You did all the walking. Each person just pointed you in the right direction.

**Who does the work:** The local DNS server (the recursive resolver) does almost everything. It makes three separate outbound queries (steps 2, 4, 6) and receives three responses (steps 3, 5, 7). The requesting host only sends one query (step 1) and receives one answer (step 8). The root, TLD, and authoritative servers each answer a single query with minimal effort.

---

## Slide 16 — DNS Resolution: Recursive Query (Slide 16 of 32)

**What is on the slide:** A heading "DNS RESOLUTION: RECURSIVE QUERY" (again, embedded in the image). Same four actors, same layout, but the arrows follow a different path — forming a chain instead of a star. A text block captures the local server's attitude. A note says: *"Generally faster than iterative, but vulnerable to amplification and DNS poisoning attacks."* A YouTube link is included at the bottom.

---

### The Recursive Query Diagram — Step by Step

Same actors, very different flow:

```
Step 1: Requesting host → Local DNS server
  "What is the IP for www.cs.yours.edu?"
  (Same as before — the stub resolver asks the recursive resolver.)

Step 2: Local DNS server → Root DNS server
  "What is the IP for www.cs.yours.edu?"
  (But this time, the local server asks the ROOT to find the full answer.)

Step 3: Root DNS server → TLD DNS server
  "What is the IP for www.cs.yours.edu?"
  (The root doesn't just give a referral — it FORWARDS the query onward.)

Step 4: TLD DNS server → Authoritative DNS server
  "What is the IP for www.cs.yours.edu?"
  (The TLD also forwards, continuing the chain.)

Step 5: Authoritative DNS server → TLD DNS server
  "It is [IP address]."
  (The answer starts flowing back up the chain.)

Step 6: TLD DNS server → Root DNS server
  "It is [IP address]."
  (The TLD passes the answer back to who asked it.)

Step 7: Root DNS server → Local DNS server
  "It is [IP address]."
  (The root passes the answer back to the local server.)

Step 8: Local DNS server → Requesting host
  "It is [IP address]."
  (The local server delivers the final answer to the client.)
```

**The text on the slide** captures the local server's attitude in recursive mode:

*"Hey, I need the IP address for this domain, please hunt it down and don't get back to me until you have it."*

**What "recursive" means:** In recursive mode, the local DNS server asks the root to **do all the work**. The root does not just give a referral — it actually forwards the query to the TLD, which forwards it to the authoritative server. The answer then travels back up the chain: authoritative → TLD → root → local → client. Each server in the chain accepts responsibility for finding the answer, not just pointing in the right direction.

Think of it like calling a customer service line. You explain your problem to Agent 1. Instead of saying "call this other number," Agent 1 puts you on hold and calls Agent 2 on your behalf. Agent 2 calls Agent 3. Agent 3 finds the answer, tells Agent 2, who tells Agent 1, who tells you. You made one call. The agents did the forwarding internally.

---

### The Key Difference — Side by Side

```
                    ITERATIVE                    RECURSIVE
                    ─────────                    ─────────
Who does the work:  Local DNS server             Every server in the chain

Local server sends: 3 queries (root, TLD, auth)  1 query (root only)

Root server does:   Answers 1 referral            Forwards query + waits

Pattern:            Star (local ↔ each server)   Chain (root → TLD → auth → back)

Advantage:          Less load on root/TLD         Faster for the client
                    servers — they just answer     (one round trip to local
                    and forget                     server, rest is internal)

Disadvantage:       More round trips for the      Root and TLD servers do
                    local server — slower          more work — vulnerable to
                    overall                        amplification and poisoning
```

**The slide's warning:** *"Generally faster than iterative, but vulnerable to amplification and DNS poisoning attacks."*

This is a critical security point that comes back in Slides 26–32 (the DNS attacks section). In recursive mode, the root and TLD servers are doing work on behalf of the requester. An attacker can exploit this by sending a small query that triggers a chain of work across multiple servers, ultimately generating a large response directed at a victim. That is the amplification attack the slide warns about. DNS poisoning is also easier in recursive mode because an attacker has more opportunities to inject fake responses as the query bounces through multiple servers. More detail on both attacks in Chunk 10.

**Which mode is actually used in practice:** Most queries from client devices to their local recursive resolver use **recursive mode** — the client asks the local server to do everything. The local recursive resolver, however, typically uses **iterative mode** when talking to root, TLD, and authoritative servers. So in practice, both modes are used in a single lookup: recursive between client and local server, iterative between local server and the DNS tree. The slide presents them as alternatives, but in reality they are complementary layers.

---

## Slide 17 — DNS Caching (Slide 17 of 32)

**What is on the slide:** A heading "DNS Caching" followed by three bullet points. No images, no speaker notes.

If every DNS query had to walk the full tree from root to TLD to authoritative server every single time, the system would collapse. There are trillions of DNS queries per day globally. The root servers — just 13 sets of them — cannot possibly handle that load if every query starts at the top. Caching is the mechanism that prevents this.

---

### Bullet 1: Why Caching Reduces Overhead

*"Caching can substantially reduce overhead — The top-level servers very rarely change — Popular sites visited often — Local DNS server often has the information cached"*

Three reasons caching is so effective:

**Top-level information is almost static.** The root servers rarely change. The list of TLD servers for `.com`, `.uk`, `.edu` almost never changes. Once your local DNS server learns "the TLD servers for `.com` are at these IP addresses," it can cache that information for days or weeks. It never needs to ask the root again for `.com` queries during that period. This alone eliminates the vast majority of root server traffic.

**Popular sites are looked up constantly.** If one person at your university looks up `www.google.com`, the local DNS server caches the answer. When the next 999 people look up the same domain, the local server answers instantly from cache without contacting any external server. For a popular site like Google, a single cached entry serves thousands of queries.

**The local DNS server becomes a knowledge base.** Over time, a busy recursive resolver accumulates cached answers for thousands of domains. The more users it serves, the more queries it can answer from cache. A large ISP (Internet Service Provider) DNS server might cache answers for millions of domains, serving the majority of queries without ever leaving its own memory.

---

### Bullet 2: How DNS Caching Works

*"How DNS caching works — DNS servers cache responses to queries — Response includes a TTL field — Server deletes the cached entry after TTL expires (typically 12 hours)"*

**TTL** stands for **Time To Live** — a numerical value (in seconds) included in every DNS response. The TTL tells the recipient: "This answer is valid for this many seconds. After that, throw it away and ask again."

When a recursive resolver receives an answer (say, `www.google.com` = `142.250.80.46` with a TTL of 300 seconds), it stores the answer in its cache with a countdown timer set to 300 seconds. For the next 5 minutes, any query for `www.google.com` gets an instant answer from cache. After 300 seconds, the entry is deleted. The next query for that domain triggers a fresh lookup.

**Why not cache forever?** Because IP addresses change. A website might migrate to new servers, a company might change hosting providers, a load balancer might rotate addresses. If cached entries never expired, devices would keep connecting to old, possibly dead IP addresses. The TTL is a balance between performance (longer TTL = fewer lookups = less traffic) and accuracy (shorter TTL = fresher data = correct answers). The slide mentions 12 hours as a typical TTL, but in practice it varies widely: Google uses TTLs of 300 seconds (5 minutes), while some rarely-changing domains use TTLs of 86400 seconds (1 day) or longer.

**Who sets the TTL?** The owner of the domain. When you configure DNS records for your domain, you specify the TTL for each record. If you know your IP address will not change anytime soon, you set a long TTL (reducing DNS traffic at the cost of slow propagation if you do change it). If you anticipate changes, you set a short TTL (ensuring fresh data at the cost of more queries).

---

### Bullet 3: Negative Caching

*"DNS negative queries are cached — Save time for non-existent sites, e.g., misspelling"*

This is a subtlety that most people do not think about. If you type `www.gooogle.com` (with three O's) into your browser, the DNS system goes through the entire lookup process and discovers: this domain does not exist. That is called an **NXDOMAIN** response (Non-Existent Domain — a DNS response code indicating that the queried domain name does not exist in the DNS system).

Without negative caching, every single misspelling or non-existent domain would trigger a full DNS tree walk every time. If a user has a misconfigured application that repeatedly queries a non-existent domain, it would generate thousands of useless queries per day.

With negative caching, the first failed lookup is cached as "this domain does not exist" with its own TTL. For the next few minutes (or hours), any repeat query for the same non-existent name gets an instant "does not exist" answer from cache, saving the entire lookup chain the work of confirming what it already knows.

---

## Slide 18 — Directing Web Clients to Replicas (Slide 18 of 32)

**What is on the slide:** A heading "DIRECTING WEB CLIENTS TO REPLICAS" in all caps, followed by five bullet points. No images, no speaker notes.

This slide shows a clever use of DNS that goes beyond simple name-to-IP translation. Large websites do not run on a single server — they run on hundreds or thousands of servers spread across data centres worldwide. DNS is part of the mechanism that distributes users across those servers.

---

### The Problem: One Name, One Server Does Not Scale

The slide starts: *"Large websites use multiple webservers - replicas."*

A **replica** is a copy of a web server — same content, same software, same capability — running on a different physical machine, possibly in a different data centre, possibly in a different country. CNN, Google, Netflix, Amazon — they all run thousands of replicas. If every user on the planet connected to one single server, that server would melt.

---

### The Simple (Bad) Approach

*"Simple approach: different names — www1.cnn.com, www2.cnn.com, www3.cnn.com — But, this requires users to select specific replicas."*

This was an early, crude solution. Give each replica its own name. Users pick one. But this is terrible for multiple reasons: users have to know about the replicas, they have to choose manually, they have no way of knowing which one is closest or least loaded, and if one replica goes down, users who bookmarked that specific name are stuck.

---

### The Elegant Approach: One Name, Multiple IPs

*"More elegant approach: different IP addresses — Single name (e.g., www.cnn.com), multiple addresses — E.g., 64.236.16.20, 64.236.16.52, 64.236.16.84, ..."*

This is what actually happens. The domain `www.cnn.com` does not map to one IP address. It maps to many. The authoritative DNS server for `cnn.com` maintains a list of IP addresses, each pointing to a different replica server.

---

### How the Selection Works

*"Authoritative DNS server returns many addresses — The local DNS server selects one address — Authoritative server may vary the order of addresses."*

When a recursive resolver queries `www.cnn.com`, the authoritative server returns all of the IP addresses — but it can change the **order** of the list. The first address in the list is typically the one the client uses. By rotating the order with each query (a technique called **round-robin DNS**), the authoritative server distributes traffic across all replicas roughly equally.

The local DNS server can also influence the selection. Some resolvers pick the first address. Some pick randomly. Some test which address responds fastest and prefer that one. The result is that different users, in different locations, at different times, connect to different replicas — distributing the load.

---

### Load Balancing

*"Load balancing based on performance and proximity."*

More advanced DNS-based load balancing goes beyond simple round-robin. Modern authoritative DNS servers (especially those run by CDNs — Content Delivery Networks, which are globally distributed networks of servers designed to deliver web content from the server closest to the user) can detect where the query is coming from geographically and return the IP address of the replica closest to the user. A user in Tashkent gets directed to a server in Asia. A user in London gets directed to a server in Europe. Same domain name, different servers, based on proximity.

Some systems also factor in real-time server load — if one replica is handling too many connections, the DNS server temporarily deprioritises it and sends new users to a less busy replica. This is more sophisticated than pure round-robin but achieves the same goal: spread the traffic, reduce the load on any single server, and give users the fastest response.

**Connection to the bigger picture:** This is a practical example of why DNS is more than a "phone book." It is an active traffic management tool. The authoritative DNS server is not just translating a name to a number — it is making a real-time decision about which server should handle this particular user. That decision affects performance, reliability, and the user's experience. DNS is infrastructure that thinks.

---

## What These Four Slides Established

Slide 15 showed iterative DNS resolution — the local server does all the walking, each upstream server just gives a referral. Slide 16 showed recursive resolution — the query chains through the tree and the answer flows back down. Slide 17 explained why the system does not collapse under load — caching at every level, with TTL controlling freshness. Slide 18 revealed that DNS is not just translation but also traffic distribution — one name can map to many servers, enabling load balancing on a global scale.

The next chunk covers the administrative side of DNS: how the namespace is divided into zones, how zone data is replicated between servers, and how organisations architect their internal and external DNS infrastructure.

---

*Chunk 7 of 10 complete. Next chunk covers Slides 19–22: Zones, zone transfer, DNS architecture, and DNS proxy.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 8 of 10 — Slides 19–22: Zones, Delegation, and Network Architecture

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ✓ DONE
  4.  The client commits and the server confirms      (Slides 9–10)  ✓ DONE
  5.  When things go wrong: troubleshooting addresses (Slide 11)     ✓ DONE

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14) ✓ DONE
  7.  How lookups actually work and why caching matters (Slides 15–18) ✓ DONE
  8.  Zones, delegation, and network architecture      (Slides 19–22) ← YOU ARE HERE
  9.  Public DNS, latency, and real-world performance   (Slides 23–25)
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32)
```

---

## What This Section Covers

The previous chunks showed how DNS works from the user's perspective — a name goes in, an IP comes out, caching makes it fast. But behind the scenes, someone has to actually maintain those DNS records. Someone has to decide which server is authoritative for which domain. Someone has to keep backup copies. Someone has to architect the system so that internal hosts are protected from the outside world while public hosts are visible.

This section shifts from "how DNS works" to "how DNS is administered." It answers four questions: What is a zone and who owns it? (Slide 19). How do backup servers get their data? (Slide 20). How do you split internal and external DNS? (Slide 21). And what role does a DNS proxy play in the architecture? (Slide 22).

---

## Slide 19 — Zones (Slide 19 of 32)

**What is on the slide:** A heading "Zones" followed by four bullet points. The phrase "DNS zone" is highlighted in red. The word "nameserver" is bold. No images, no speaker notes.

---

### What Is a DNS Zone?

The slide says: *"A DNS zone is a distinct part of the domain namespace which is delegated to a legal entity — a person, organization or company, who are responsible for maintaining the DNS zone."*

In the tree structure from Slide 12, we saw domains cascading from root to TLD (Top Level Domain) to second-level to subdomains to individual hosts. A **zone** is a slice of that tree that a specific organisation controls. It is an administrative boundary, not a technical one.

**The distinction between a domain and a zone is subtle but important.** A domain is a node in the tree plus everything below it. The domain `example.com` includes `www.example.com`, `mail.example.com`, `support.example.com`, and everything else underneath. A zone, however, might not include everything below the domain. If the owner of `example.com` decides to delegate `support.example.com` to a separate team with its own DNS server, then `support.example.com` becomes its own zone. The `example.com` zone contains records for `www.example.com` and `mail.example.com`, but for `support.example.com` it simply contains a **delegation record** — a pointer saying "for anything under `support.example.com`, ask this other server."

Think of zones like departments in a company. The CEO (root) delegates the `.com` TLD to a registrar (like a vice president). The registrar delegates `example.com` to Company X. Company X delegates `support.example.com` to its support team. Each delegation creates a new zone with its own administrators, its own servers, and its own responsibility.

---

### Zone Structure — The Slide's Examples

*"The Domain Name System (DNS) defines a domain namespace, which specifies Top Level Domains (such as '.com'), second-level domains (such as 'acme.com') and lower-level domains, also called subdomains (such as 'support.acme.com'). Each of these levels can be a DNS zone."*

The example uses `acme.com` (a common placeholder domain name used in technical documentation — "acme" is a Greek-derived word meaning the peak or pinnacle of something, and it has been a stock example name in networking standards for decades). The point is that zones can exist at any level:

- The root zone (managed by ICANN — Internet Corporation for Assigned Names and Numbers, the organisation that coordinates the global DNS root)
- A TLD zone (`.com`, managed by Verisign)
- A second-level zone (`acme.com`, managed by whoever registered it)
- A subdomain zone (`support.acme.com`, optionally managed by a different team)

---

### Nameservers and Responsibility

*"The name servers in their respective zones are responsible for answering queries for their zones."*

Each zone has at least one DNS server that holds the authoritative records for that zone. When a recursive resolver eventually reaches this server in the lookup chain, it gets the definitive answer — not a cached copy, not a referral, but the actual record maintained by the zone administrator.

*"There is usually one primary **nameserver** and one or more secondary nameservers."*

**Primary nameserver:** The master copy. This is where the administrator creates, edits, and deletes DNS records. All changes happen here first.

**Secondary nameserver(s):** Backup copies. They hold replicas of the primary's zone data. If the primary goes down, the secondaries can still answer queries. Clients and recursive resolvers do not know or care whether they are talking to the primary or a secondary — both give authoritative answers.

Why have secondaries? Redundancy. If the only DNS server for your domain dies, your entire domain disappears from the internet. Every website, every email server, every service — all unreachable. Not because the servers themselves are down, but because nobody can find them anymore. DNS is the map. Without it, the destination still exists but nobody knows how to get there. Having multiple nameservers spread across different physical locations protects against this.

---

## Slide 20 — Zone Transfer (Slide 20 of 32)

**What is on the slide:** A heading "Zone transfer" (in red text) followed by three main bullets and four sub-bullets. No images, no speaker notes.

Zone transfer is the mechanism by which secondary nameservers get their data from the primary. If the primary is the master copy, zone transfer is the photocopier.

---

### How Records Get to the Primary

*"A primary server will have an administrator entering the records or receive dynamic updates."*

Two ways records arrive on the primary:

**Manual entry:** A network administrator logs into the DNS server and types in records. "The A record for `www.example.com` is `93.184.216.34`." This is how most static records are created — web servers, mail servers, nameservers themselves.

**Dynamic updates:** Some systems can automatically add or update DNS records without human intervention. For example, when a DHCP (Dynamic Host Configuration Protocol) server assigns an IP address to a device, it can simultaneously tell the DNS server: "This device is called `pc-123.internal.example.com` and its IP is `192.168.1.47`." The DNS record is created automatically. This integration of DHCP and DNS is common in enterprise environments running Windows Active Directory.

---

### How Records Get to the Secondaries

*"A secondary server gets a copy of all the records by taking a copy — **zone transfer**."*

A **zone transfer** is exactly what it sounds like: the entire contents of a zone are copied from the primary server to a secondary server. Every record, every setting — a complete replica. The secondary can then serve queries using this copy.

**Three scenarios that trigger a zone transfer:**

**1. "When starting the DNS Service on the secondary DNS server."**

The secondary has just been turned on (or restarted). It has no data. The first thing it does is request a full zone transfer from the primary — pulling down the complete set of records. This is called an **AXFR** (Full Zone Transfer — a DNS query type that requests the complete contents of a zone in one transfer; the 'A' is commonly understood to mean "All," distinguishing it from **IXFR**, Incremental Zone Transfer, which only transfers records that changed since the last update).

**2. "When the refresh time expires."**

Even without changes, the secondary periodically checks whether the primary has newer data. The zone's **SOA record** (Start of Authority — a special DNS record that contains administrative information about the zone, including serial number, refresh interval, retry interval, and expiry time) defines a **refresh interval** — a timer that tells the secondary how often to check. When the refresh timer expires, the secondary contacts the primary and compares serial numbers. If the primary's serial number is higher (meaning records have been updated), the secondary requests a new zone transfer. If the serial numbers match, nothing happens.

**3. "When changes are saved to the Primary Zone file and there is a Notify List."**

Instead of waiting for the refresh timer, the primary can proactively push a notification. When an administrator changes a record on the primary, the primary sends a **DNS NOTIFY** message to all servers on its **notify list** (a configured list of secondary servers). The notification says: "Something changed. Come get a fresh copy." The secondary then initiates a zone transfer immediately, rather than waiting for its refresh interval.

**A critical detail the slide emphasises:**

*"Zone Transfers are always initiated by the secondary DNS server. The primary DNS server simply answers the request for a Zone Transfer."*

The primary never pushes data to the secondary. It only sends a notification ("hey, something changed"). The secondary then pulls the data. This is a pull model, not a push model. The distinction matters for security — it is harder for a random external attacker to inject fake zone data by impersonating a push from the primary. However, if the primary itself is compromised, the secondary will still pull poisoned data on the next transfer, since it trusts whatever the configured primary sends. The pull model is not a defence against a compromised primary — only against unsolicited injections from outside.

---

## Slide 21 — Architecture (Slide 21 of 32)

**What is on the slide:** A heading "Architecture" followed by a short text bullet and a diagram. Substantial speaker notes.

**The slide text:**

*"Having local DNSs reduces traffic in network and lookup latency."*

*"Outward facing DNS advertises certain outside hosts to world — MI-Linux.wlv.ac.uk"*

This slide introduces the concept of **split DNS** — running separate internal and external DNS servers that present different views of your network to different audiences.

---

### The Problem Split DNS Solves

From Ian's speaker notes: *"Once embarked on the mission to create a DNS infrastructure, you will soon realize that you need a way to isolate certain portions of your DNS hierarchy."*

Imagine you are running the network for a company (`company.com`). You have:

- A public website at `www.company.com` — needs to be visible to the entire internet
- An email server at `mail.company.com` — needs to be reachable from external senders
- Internal databases at `db1.internal.company.com` — must NOT be visible outside
- Employee workstations at `alice-pc.internal.company.com` — must NOT be visible outside
- Department servers at `acct.company.com` and `engr.company.com` — internal use only

If you put all of these records on one DNS server that answers queries from the internet, anyone can discover the names and IP addresses of your internal infrastructure. An attacker could query your DNS and learn that `db1.internal.company.com` exists at `10.0.5.20` — valuable reconnaissance (preliminary information gathering that an attacker uses to plan an attack) for planning an attack.

---

### The Solution — Internal and External DNS Servers

The speaker notes explain: *"Not only will you want a separate DNS server external to your network with limited information, you will want servers distributed throughout your network to prevent unnecessary DNS traffic on the WAN (Wide Area Network — the links connecting geographically separate sites) and LAN (Local Area Network — the network within a single building or campus) and reduce lookup latency."*

**External DNS server (outward-facing):** This server sits in the DMZ (Demilitarised Zone — a network segment between the internal network and the internet, typically protected by firewalls on both sides) or at a hosting provider. It only contains records for services that need to be publicly visible: `www.company.com`, `mail.company.com`, and perhaps a few others. It knows nothing about internal hosts. When someone on the internet queries your DNS, they see only what you choose to advertise.

**Internal DNS server(s):** These sit inside your private network. They contain all records — both public and internal. An employee querying `db1.internal.company.com` gets an answer. An employee querying `www.google.com` gets an answer too (via the recursive resolver function of the internal server, which forwards external queries upstream).

The slide mentions `MI-Linux.wlv.ac.uk` as an example of a host that the University of Wolverhampton advertises externally — it is a public-facing Linux server. Internal lab machines, staff workstations, and infrastructure devices are not advertised.

---

### Strategic Placement from the Speaker Notes

Ian's notes add: *"Once you have a hierarchy established, you can start thinking about strategic locations for the primary servers for those subdomains. You will want to co-locate them with the bulk of the users in those departments so those users can use the server as an initial DNS contact."*

This connects directly to Lecture 2's discussion of hierarchical network design. Just as you place access switches close to users and distribution switches at aggregation points, you place DNS servers close to the users who need them. The accounting department's DNS server sits in the accounting building. Engineering's DNS server sits in engineering. Each server handles the queries from its local users, reducing traffic on the WAN links between buildings and reducing latency for the most common lookups.

---

## Slide 22 — DNS Proxy (Slide 22 of 32)

**What is on the slide:** A heading "DNS proxy" followed by four bullet points (each with a bold lead-in) and a diagram showing the proxy's position in the architecture. The diagram is sourced from Akamai (a major CDN — Content Delivery Network — provider). No speaker notes.

A **DNS proxy** is an intermediary that sits between the recursive resolver and the authoritative nameservers. It intercepts DNS queries, caches responses, and can apply security filtering — all without the client knowing it exists. The slide lists four benefits:

---

### Benefit 1: Faster DNS Resolution

*"Because it caches information about previously accessed domain names, a DNS proxy server can return results more quickly."*

This sounds identical to the caching described on Slide 17, and it largely is. The difference is where the cache sits. The recursive resolver has its own cache. The DNS proxy adds another layer of caching between the resolver and the authoritative servers. In a large organisation, the proxy might serve hundreds of recursive resolvers, each of which benefits from the shared cache. A query that one resolver has never seen might already be cached by the proxy because another resolver asked the same question earlier.

---

### Benefit 2: Less Latency

*"By enabling name resolution from a DNS cache, DNS proxies also help reduce network latency."*

Latency (the delay between sending a query and receiving a response) is reduced because the proxy can answer from its cache without forwarding the query to the authoritative server, which might be geographically distant. If the authoritative server for `example.com` is in the United States and you are querying from Tashkent, the round trip adds hundreds of milliseconds. A local proxy with the answer cached eliminates that entire round trip.

---

### Benefit 3: Easier Management

*"DNS proxies may help IT teams to simplify management of DNS configuration."*

Instead of configuring every device or every recursive resolver with specific DNS server addresses, you point everything at the proxy. The proxy handles the routing of queries to the appropriate upstream servers. If you need to change which authoritative servers you query, or add filtering rules, or redirect certain domains — you change the proxy configuration once, and every device behind it is immediately affected.

---

### Benefit 4: Stronger Security

*"A DNS proxy may offer security features that help protect networks from cyberattacks such as domain hijacking and DNS spoofing."*

The proxy can inspect every DNS query passing through it. It can block queries to known malicious domains (phishing sites, malware command-and-control servers), enforce company policies (blocking social media during work hours), and detect suspicious patterns (a device suddenly making thousands of queries to random domains, which might indicate a malware infection). This positions the DNS proxy as a security tool, not just a performance tool.

**The diagram on the slide** shows the proxy sitting between the recursive DNS resolver and two authoritative DNS nameservers. The flow is:

```
Device → Recursive DNS resolver → Proxy → Authoritative DNS nameservers
```

The proxy sits between the recursive resolver and the authoritative nameservers. In some architectures, the proxy is transparent to the recursive resolver — the resolver forwards queries upstream without knowing a proxy is intercepting them. In other setups, the proxy itself acts as the recursive resolver, combining both roles into one device. Either way, the end-user device is unaware of the proxy's existence — it simply sends queries to its configured DNS server and receives answers.

---

## What These Four Slides Established

Slide 19 introduced zones — the administrative boundaries of DNS, where ownership and responsibility are defined. Slide 20 explained zone transfer — how secondary servers get copies of zone data, triggered by startup, refresh timers, or notifications, always pulled by the secondary. Slide 21 introduced split DNS architecture — separating internal and external DNS to protect private infrastructure while advertising public services. Slide 22 added the DNS proxy — a caching and security layer that sits between resolvers and authoritative servers.

Together, these slides shift the perspective from "how does DNS answer a query" to "how do you build and maintain the DNS infrastructure that makes those answers possible." The next chunk covers real-world performance: public DNS providers, latency factors, and a concrete example of how DNS lookup times affect web page loading.

---

*Chunk 8 of 10 complete. Next chunk covers Slides 23–25: Public DNS, latency breakdown, and page speed demo.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 9 of 10 — Slides 23–25: Public DNS, Latency, and Real-World Performance

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ✓ DONE
  4.  The client commits and the server confirms      (Slides 9–10)  ✓ DONE
  5.  When things go wrong: troubleshooting addresses (Slide 11)     ✓ DONE

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14) ✓ DONE
  7.  How lookups actually work and why caching matters (Slides 15–18) ✓ DONE
  8.  Zones, delegation, and network architecture      (Slides 19–22) ✓ DONE
  9.  Public DNS, latency, and real-world performance   (Slides 23–25) ← YOU ARE HERE
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32)
```

---

## What This Section Covers

The previous chunks explained how DNS (Domain Name System) works in theory — the tree, the resolvers, the caching, the zones, the architecture. This section shifts to performance: how fast is DNS in the real world? What causes DNS to be slow? And can you see the impact of DNS lookups on an actual web page loading?

These three slides bridge theory and practice. They give students a reason to care about DNS optimisation — because the speed of DNS directly affects the speed of everything users do on the internet.

---

## Slide 23 — Free and Public DNS (Slide 23 of 32)

**What is on the slide:** A heading "Free and Public DNS" followed by a parent bullet and three child bullets. Speaker notes are substantial.

**The slide text:**

*"Public DNS servers are alternative to ones given by your ISP"*

When you connect to the internet, your ISP (Internet Service Provider — the company that provides your internet connection) typically configures your device (via DHCP — Dynamic Host Configuration Protocol) to use the ISP's own DNS servers. This is the default for most home and office connections. But you are not locked into using them. You can manually configure your device — or your router — to use a different set of DNS servers. These are called **public DNS** servers.

---

### The Two Most Famous Public DNS Providers

*"Google DNS and OpenDNS are the reported to be the best and most reliable"*

(Note: the slide has a grammatical error — "are the reported" should read "are reported." The original slide also spells it "OpenDns" with inconsistent capitalisation — the standard spelling is "OpenDNS.")

**Google Public DNS:** `8.8.8.8` and `8.8.4.4`. Launched in 2009. One of the most widely used public DNS services in the world. Google operates it from data centres on every continent, using **anycast** routing (a networking technique where the same IP address is advertised from multiple geographical locations; when you send a packet to `8.8.8.8`, the network automatically routes it to the nearest Google data centre, not a single fixed server).

**OpenDNS:** Now owned by Cisco Systems. Provides DNS resolution with optional content filtering and security features (blocking known malicious domains, phishing protection). Popular in schools and businesses where administrators want to control what users can access via DNS.

The slide also includes a URL to an old article listing free public DNS servers, and notes: *"These 2 most famous have geographically distributed servers with enough clusters and load balancing to reduce latency."*

---

### Why People Switch from ISP DNS to Public DNS — From the Speaker Notes

Ian's speaker notes provide three reasons DNS can be slow, which explain why people seek alternatives:

**1. Latency between client and resolving server.** The physical distance between your device and the DNS server matters. If your ISP's DNS server is in another city — or worse, another country — every DNS query adds that round-trip delay. Google and Cloudflare reduce this by having servers everywhere, so the anycast route to `8.8.8.8` or `1.1.1.1` is almost always short.

The notes specifically mention: *"geographical distance between client and server machines; network congestion; packet loss and long retransmit delays (one second on average); overloaded servers, denial-of-service attacks and so on."*

RTT (Round-Trip Time — the time it takes for a packet to travel from your device to the server and back) is the dominant factor here. Every millisecond of RTT is added to every DNS query. If your ISP's DNS server has 80ms RTT and Google's has 15ms, you save 65ms on every uncached lookup. Over a complex web page with 13 DNS lookups (as Slide 25 will show), that adds up.

**2. Latency between resolving servers and authoritative nameservers.** Even after your query reaches the recursive resolver, the resolver might need to contact external servers to find the answer. The notes list three causes of this inter-server latency:

- **Cache misses** — if the answer is not cached, the resolver must recursively query root, TLD (Top Level Domain), and authoritative servers. If those servers are geographically remote, the added network latency is considerable.
- **Under-provisioning** — if the DNS resolver is overloaded (too many queries, not enough capacity), queries get queued, responses are delayed, and in severe cases packets are dropped and must be retransmitted. This is common with smaller ISPs that underinvest in their DNS infrastructure.
- **Malicious traffic** — DDoS (Distributed Denial of Service — an attack where thousands of compromised devices flood a target with traffic to overwhelm it) attacks and Kaminsky-style cache poisoning attacks (a technique where an attacker floods a resolver with fake responses to redirect traffic — explained in detail in the next chunk) can place undue load on resolvers even if they are not the primary target.

**Why Google and Cloudflare win here:** They operate massive, globally distributed resolver networks. Their cache hit rates are extremely high (because they serve so many users, popular domains are almost always cached). Their infrastructure is heavily provisioned (they can absorb traffic spikes and attacks that would cripple a small ISP's DNS). And their anycast deployment means the resolver-to-authoritative path is often shorter because they have nodes near the major authoritative servers.

---

## Slide 24 — DNS Latency (Slide 24 of 32)

**What is on the slide:** A heading "DNS latency" followed by two main bullets with sub-bullets explaining the two components of DNS latency. No images, no speaker notes. There is a typo: "under-provissioning" (double 's').

This slide condenses the speaker notes from Slide 23 into a more structured format for the actual presentation. It formally names the two components of DNS latency and lists the causes of each.

---

### Component 1: Between Client and DNS Resolving Server

*"Between client and DNS resolving server — usually due to round trip time in network etc."*

This is the latency you feel directly. Every DNS query travels from your device to the recursive resolver and back. The delay depends on:

- **Physical distance** — a resolver in Tashkent serves local users faster than a resolver in London
- **Network congestion** — if the path between you and the resolver is congested, packets queue at intermediate routers
- **Packet loss** — if a DNS query packet is lost, the client must wait for a timeout (typically 1–2 seconds) before retrying. One lost packet can add seconds to a single lookup. Since DNS uses UDP (User Datagram Protocol — a connectionless transport protocol with no built-in retransmission), lost packets are not automatically recovered at the transport layer — the DNS client software must handle retransmission itself.

---

### Component 2: Between Resolving Servers

*"Between resolving servers"*

This is the latency the resolver experiences when it needs to walk the DNS tree. Three causes:

**Cache misses:** *"require further recursive queries to other nameservers and can cause considerable latency as more authoritative nameservers may be geographically far away."*

A cache miss means the resolver does not have the answer stored. It must query the root, then the TLD server, then the authoritative server — each query adding its own round trip. If the authoritative server for an obscure domain is hosted on a single server in Brazil and you are querying from Central Asia, the round trip could be hundreds of milliseconds. Multiply that by three levels of the tree (root → TLD → authoritative) and a single uncached lookup could take over a second.

This is exactly why caching is so critical (Slide 17). A cached answer is served in under 1ms. An uncached answer might take 200–500ms. The difference is massive when a single web page triggers a dozen lookups.

**Under-provisioning:** *"if servers cannot cope they will queue or may even drop requests and require retransmissions."*

If a DNS server receives more queries than it can process, it starts queuing them. Queries that would normally be answered in 1ms now wait 50ms or 100ms in the queue. If the queue overflows, packets are dropped entirely, and the client must wait for a timeout and retry — adding seconds of delay. This is particularly problematic for ISP DNS servers during peak usage hours.

**Malicious traffic:** *"DNS traffic can slow down responses even they cannot overwhelm the server."*

Even if a DDoS attack does not fully bring down a DNS server, it can degrade performance by consuming processing capacity and bandwidth. Legitimate queries get slower because the server is spending resources handling the flood of malicious queries. This is the "traffic jam" effect — even if the road does not close, enough extra cars make everyone slower.

---

## Slide 25 — Page Speed Screenshot (Slide 25 of 32)

**What is on the slide:** A screenshot from Google's Page Speed Activity tool showing a waterfall chart (a visual timeline that displays each resource a browser loads when rendering a web page, showing when each request starts, how long it takes, and what is happening at each stage) of a web page loading. On the left, four annotations from Ian. The image is sourced from Google Developers documentation. No speaker notes.

This is the "proof" slide — a real-world demonstration of how DNS latency affects something users actually see: web page loading time.

---

### What the Screenshot Shows

The waterfall chart spans from `0ms` on the left to `11 seconds` on the right. Each horizontal bar represents one resource the browser needs to load: an HTML file, a CSS stylesheet, a JavaScript file, an image, etc. The bars are colour-coded:

- **Black segments = DNS lookups** — this is the DNS portion of each request
- **Grey segments = Connection Wait** — waiting for a connection to be established
- **Yellow segments = Connect** — TCP (Transmission Control Protocol) handshake
- **Orange segments = Cache Hit** — resource served from browser cache, no network download needed
- **Green segments = Data Available** — actual content being received from the server
- **Other colours** = Send, Receive, Connected, JS Parse, JS Execute

The black (DNS) segments appear at the start of many bars — before the browser can download a resource, it must first look up the IP address of the server hosting that resource. Each black segment is a DNS query adding delay before any data flows.

---

### Ian's Four Annotations

**"11 secs to load"** — The entire page took 11 seconds from first request to fully rendered. This is slow by modern standards (users typically expect pages to load in 2–3 seconds), but this example is deliberately chosen to show a complex page with many external dependencies.

**"13 lookups"** — The browser performed 13 separate DNS lookups while loading this single page. Why so many? Because modern web pages pull resources from multiple domains: the main website's server, a CDN (Content Delivery Network) for images, Google Analytics for tracking, a fonts server for custom typography, an advertising network for ads, a social media widget for sharing buttons. Each unique domain requires its own DNS lookup.

**"Some in parallel"** — The browser does not wait for one DNS lookup to finish before starting the next. It fires off multiple queries simultaneously. This is visible in the waterfall chart where several black segments overlap vertically — they start at the same time. Parallel lookups reduce total time because overlapping queries share the same wall-clock time instead of adding up sequentially.

**"5 serial look ups"** — Despite parallelisation, five of the lookups had to happen in sequence — one after another, not simultaneously. This happens when the browser discovers a new domain reference only after loading and parsing a previous resource. For example, the HTML page loads first, and inside it is a reference to a JavaScript file on `analytics.google.com`. The browser cannot look up `analytics.google.com` until it has finished loading and parsing the HTML. Then the JavaScript file loads, and inside it is a reference to `tracking.example.com`. Another serial lookup. Each one in the chain adds its full DNS resolution time to the total page load.

---

### Why This Matters — The DNS Tax on Web Performance

The screenshot makes a powerful point: DNS is not a one-time cost per page. It is a per-domain cost. A complex page with resources from 13 different domains pays the DNS tax 13 times. Even if each lookup takes only 50ms (a fairly fast cached result from a nearby resolver), 5 serial lookups add 250ms of pure DNS delay that cannot be parallelised. If any of those lookups are uncached and require a full tree walk, the delay climbs significantly.

This is why:

- **Caching matters** (Slide 17) — a cached answer is nearly instant. An uncached one might take 200ms+. Caching turns 13 expensive lookups into 13 cheap ones.
- **Public DNS providers matter** (Slide 23) — a faster resolver with better cache hit rates and lower RTT reduces the cost of every single lookup.
- **DNS prefetching exists** — modern browsers look ahead in the HTML and start DNS lookups for domains they see referenced, even before those resources are needed. This converts some serial lookups into parallel ones, reducing total delay.
- **Reducing the number of external domains matters** — every unique domain on a page adds a DNS lookup. Web performance optimisation (minimising the number of external services a page depends on) directly reduces the DNS overhead.

**Connection to Lecture 4 (QoS — Quality of Service):** Remember the discussion of latency as one of the four QoS metrics? DNS latency is a concrete, measurable example of network delay that directly affects user experience. A web page that takes 11 seconds to load — partly because of slow DNS — is a QoS failure from the user's perspective. This slide connects the abstract QoS concepts from Lecture 4 to something every user has experienced: waiting for a page to load.

---

## What These Three Slides Established

Slide 23 introduced public DNS as an alternative to ISP-provided servers, with Google and Cloudflare as the dominant options — fast because of global anycast deployment and massive caching. Slide 24 formally broke DNS latency into two components: client-to-resolver (RTT-dominated) and resolver-to-authoritative (cache miss, under-provisioning, and attack-dominated). Slide 25 proved the impact with a real waterfall chart: 13 DNS lookups, 5 serial, contributing to an 11-second page load time.

The final chunk covers the dark side of DNS: what happens when attackers exploit the system. Weaknesses, poisoning, reflection, amplification, and the DNSSEC defence.

---

*Chunk 9 of 10 complete. Next chunk covers Slides 26–32: DNS security — weaknesses, poisoning, reflection, amplification, and DNSSEC.*
# Lecture 5 — Systems: DHCP & DNS

## Chunk 10 of 10 — Slides 26–32: DNS Under Attack and How DNSSEC Fights Back

**Module:** 6CS029 Advanced Networking
**Source file:** `Lecture_5_DHCP_and_DNS.pptx` (32 slides)

---

## Lecture Map

```
PART 1 — DHCP (Dynamic Host Configuration Protocol)

  1.  What this lecture covers and why DHCP exists    (Slides 1–3)   ✓ DONE
  2.  The address pool and how leasing works          (Slides 4–6)   ✓ DONE
  3.  The server responds and the deal is sealed      (Slides 7–8)   ✓ DONE
  4.  The client commits and the server confirms      (Slides 9–10)  ✓ DONE
  5.  When things go wrong: troubleshooting addresses (Slide 11)     ✓ DONE

PART 2 — DNS (Domain Name System)

  6.  The naming tree and who translates names to IPs  (Slides 12–14) ✓ DONE
  7.  How lookups actually work and why caching matters (Slides 15–18) ✓ DONE
  8.  Zones, delegation, and network architecture      (Slides 19–22) ✓ DONE
  9.  Public DNS, latency, and real-world performance   (Slides 23–25) ✓ DONE
  10. DNS under attack and how DNSSEC fights back       (Slides 26–32) ← YOU ARE HERE
```

---

## What This Section Covers

Every previous section in Part 2 assumed DNS is trustworthy — that when you ask "what is the IP for `www.wlv.ac.uk`?" the answer you receive is correct. This final section destroys that assumption. DNS (Domain Name System) was designed in the 1980s, when the internet was a small academic network where everyone broadly trusted everyone else. Security was not a consideration. The result is a system that, in its original form, has no built-in mechanism to verify that a DNS answer is genuine.

These seven slides walk through the consequences: what weaknesses exist (Slides 26–27), how specific attacks exploit them (Slides 28–31), and what defence was built to fix it (Slide 32). The section flows logically — first the problems, then the attacks, then the solution.

---

## Slide 26 — Weaknesses in the DNS System (Slide 26 of 32)

**What is on the slide:** A heading "Weaknesses in the DNS system" followed by two main bullets, each with sub-bullets. The phrases "unencrypted and unsigned" and "spoofed authority or additional records" are highlighted in blue. No images, no speaker notes.

This slide identifies the two fundamental design flaws in DNS — the root causes that every attack in the following slides exploits.

---

### Weakness 1: Unencrypted and Unsigned

*"DNS queries and responses may be sent over UDP, unencrypted and unsigned."*

**UDP** stands for **User Datagram Protocol** — a connectionless transport protocol that sends packets without establishing a handshake or verifying delivery. DNS uses UDP port 53 for most queries because it is fast and lightweight. But UDP provides zero security:

**Unencrypted** means anyone who can observe network traffic between you and your DNS server can read every query you send. They can see that you looked up `www.yourbank.com` at 14:32, then `webmail.company.com` at 14:35. Your DNS queries are a complete browsing diary, transmitted in plaintext for anyone on the network path to read. This is a serious privacy concern — an ISP (Internet Service Provider), a government agency, or an attacker on the same Wi-Fi network can build a detailed profile of your internet activity just by watching DNS traffic.

**Unsigned** means there is no way to verify that a DNS response actually came from the server you asked. When your recursive resolver receives a response that says "`www.yourbank.com` is at `93.184.216.34`," it has no mechanism to confirm that this response genuinely came from the authoritative server for `yourbank.com`. An attacker who can inject packets into the network can forge a response, and the resolver will accept it as genuine.

Two consequences follow from this:

*"eavesdropping on sensitive DNS data"* — passive observation. The attacker just watches. No manipulation, just surveillance.

*"queries or responses can be injected or modified, and will be treated as valid"* — active manipulation. The attacker changes the answers. The resolver accepts the fake answer because it has no way to distinguish it from a real one. This is the foundation of DNS poisoning, covered on Slide 29.

---

### Weakness 2: Additional Records

*"DNS responses contain additional information"*

A DNS response does not just contain the answer to the query. It also contains an **authority section** (which nameservers are authoritative for the domain) and an **additional section** (extra records the server thinks might be useful — for example, the IP addresses of the nameservers it just mentioned in the authority section). These extra sections are designed to be helpful — they reduce the need for follow-up queries.

But an attacker can exploit this: *"attacker may return the correct answer to a query, but include spoofed authority or additional records."*

Imagine querying for `www.example.com`. The attacker returns the correct IP in the answer section, so nothing seems wrong. But in the authority section, the attacker writes: "The nameserver for `example.com` is now `evil-ns.attacker.com`." And in the additional section: "`evil-ns.attacker.com` is at `6.6.6.6`." The resolver caches all of this. Now, every future query for anything under `example.com` goes to the attacker's server instead of the real one. The attacker has hijacked the entire domain by piggybacking on a legitimate response.

---

## Slide 27 — DNS Weaknesses (Continued) (Slide 27 of 32)

**What is on the slide:** A heading "DNS weaknesses" followed by five bullets covering three additional attack surfaces. The words "subverting" and "distributed DOS" are highlighted in red. "A full zone transfer" is also highlighted. No images, no speaker notes.

---

### Weakness 3: Subverting High-Value Targets

*"subverting a DNS proxy, or a DNS server near the root of the DNS tree would also be an excellent attack"*

The higher up the DNS tree an attacker can compromise, the more damage they can do. A compromised root server could redirect queries for any TLD — `.com`, `.uk`, `.edu`, all of them. A compromised TLD (Top Level Domain) server could redirect every query for an entire country's domains. A compromised DNS proxy (as described on Slide 22) could redirect every query from every user behind it. The value of the target scales with its position in the hierarchy.

---

### Weakness 4: DoS and DDoS

*"DNS, like much of the Internet, is susceptible to DOS and distributed DOS"*

**DoS** stands for **Denial of Service** — an attack where the attacker floods a target with traffic to make it unavailable to legitimate users. **DDoS** stands for **Distributed Denial of Service** — the same attack but launched from thousands of compromised devices (a **botnet** — a network of malware-infected computers that an attacker controls remotely) simultaneously, making it much harder to block because the traffic comes from everywhere.

DNS servers are particularly attractive DDoS targets because taking down a DNS server does not just affect one service — it makes every domain that server is responsible for unreachable. Attacking a company's web server takes down one website. Attacking their DNS server takes down every service under their domain: web, email, VPN (Virtual Private Network — a technology that creates an encrypted tunnel over the internet for secure remote access), API (Application Programming Interface — a set of rules that allows software applications to communicate with each other) endpoints, everything.

---

### Weakness 5: Zone Transfer Exposure

*"DNS allows a client to request a full zone transfer from the zone's SOA (start of authority – zones authoritative DNS) server."*

Zone transfers (covered on Slide 20) were designed to replicate data between primary and secondary nameservers. But if the server is not configured to restrict who can request a transfer, anyone can request it. An attacker sends an AXFR (Full Zone Transfer) request and receives the complete list of every hostname, IP address, and alias in the zone — a map of the entire network.

*"This allows an attacker to learn all sorts of useful information about the zone: names, IP addresses, aliases etc."*

This is **reconnaissance** (preliminary information gathering that an attacker uses to plan further attacks). Knowing that `db-prod.internal.company.com` exists at `10.0.5.20` tells the attacker exactly where the production database lives and what its internal IP is. That information makes targeted attacks far more effective.

*"Can be mitigated by only allowing zone transfers to certain IP addresses"*

The fix is simple: configure the DNS server to only accept zone transfer requests from the IP addresses of known, legitimate secondary nameservers. All other requests are denied. This is standard practice but is not always implemented — particularly on older or poorly maintained servers.

---

## Slide 28 — DNS Security Attack Diagram (Slide 28 of 32)

**What is on the slide:** A heading "DNS security" with a subtitle "Authoritative DNS sever" (note: "server" is misspelt as "sever" — the same typo that appeared on Slide 8). A large, detailed diagram showing the complete DNS architecture with attack vectors marked. Three text annotations at the bottom.

(Note: the slide title says "Authoritative DNS sever" — this is Ian's recurring typo. "Sever" should be "server.")

---

### The Diagram Explained

The diagram is split into two halves — the top half shows the **normal DNS data flow** and the bottom half shows the **attack vectors** that target each point.

**Top half — Normal flow (left to right):**

```
Local OS on PC → STUB resolver → Caching resolver (recursive) → MASTER server ← zone file (text, DB)
                                                                      ↕                ↑
                                                              Zone Transfer      dynamic updates
                                                                      ↕
                                                                  SLAVES
```

This is the standard architecture from Slides 14 and 19–20: the device's stub resolver talks to the caching recursive resolver, which queries the master authoritative server. The master server gets its data from the zone file and dynamic updates. Slave (secondary) servers receive copies via zone transfer.

**Bottom half — Attack vectors (matching each component):**

| Normal Component | Attack Vector | What the Attack Does |
|---|---|---|
| Stub resolver on PC | **Man in the middle** | An attacker positioned between the PC and the network intercepts DNS queries and injects fake responses |
| Caching resolver | **Cache poisoning** | Attacker injects false records into the resolver's cache, redirecting all users of that resolver |
| Zone transfer path | **Modified data** | Attacker intercepts or corrupts zone transfer data in transit |
| Master server | **Spoofing master (routing/DoS)** | Attacker impersonates the master server or overwhelms it with traffic |
| Dynamic updates | **Spoofed updates** | Attacker sends fake dynamic update messages to inject records |
| Zone file / database | **Corrupted data** | If the zone file itself is compromised (via file system access), all data is tainted at the source |

**Three annotations at the bottom of the slide:**

*"Trojan horse attack will alter resolver and send you to spoof paypal etc"* — This describes a malware attack on the local PC. A Trojan (a type of malware disguised as legitimate software) modifies the operating system's DNS server settings — changing the configured DNS server address from the legitimate one to an attacker-controlled server. The stub resolver itself is unchanged, but it obediently uses whatever DNS server the OS points it at. From that point on, every DNS query from that PC goes to the attacker. When the user types `www.paypal.com`, the attacker's server returns the IP of a fake PayPal site that looks identical but steals login credentials. The user sees a perfect copy of PayPal and has no visual clue that anything is wrong.

*"Overwhelm slave - believe it is receiving a dns entry from authoritative server"* — The attacker floods a secondary (slave) DNS server with fake zone transfer data, timed to arrive when the slave expects a legitimate update. If the slave accepts the fake data, it starts serving poisoned records to anyone who queries it.

*"Spoofing of master IP address to inject false DNS entries"* — The attacker sends packets to the slave server with a forged source IP matching the master server's address. The slave thinks the packets came from the master and accepts them. This is IP spoofing (forging the source IP address in a packet header to impersonate another device) applied to DNS infrastructure.

---

## Slide 29 — DNS Poisoning Attack (Slide 29 of 32)

**What is on the slide:** A heading "DNS poisoning attack" followed by three bullets and a diagram showing the attack flow. No speaker notes.

**DNS poisoning** (also called **cache poisoning**) is one of the most dangerous DNS attacks because it does not require the attacker to be on the victim's network. The attacker can be anywhere on the internet.

---

### How the Attack Works — Step by Step

*"Without eavesdropping, an attacker can still spoof and poison a DNS server"*

This is the key insight — the attacker does not need to intercept traffic. They can attack from outside, without seeing the victim's queries.

**Step 1:** *"Attacker queries the server for a name that it doesn't know"*

The attacker sends a DNS query to the target resolver for a domain the resolver has never seen — something like `random123.example.com`. The resolver does not have this in its cache, so it starts the normal iterative lookup process: querying root servers, TLD servers, and eventually the authoritative server for `example.com`.

**Step 2:** *"Then attacker floods the server with faked responses with authoritative data for that name"*

While the resolver is waiting for the legitimate response from the real authoritative server, the attacker floods the resolver with thousands of fake responses. These fake responses claim to be from the authoritative server and contain the attacker's IP address instead of the real one. They also include poisoned authority records (as described on Slide 26) to redirect future queries.

**Step 3:** *"Attacker has to guess the transaction ID for the response to be valid, but with only 32 bits, this is quickly done in 64K responses."*

Every DNS query carries a **transaction ID** — a random number that the response must match for the resolver to accept it (this is the same concept as the XID field in DHCP — Chunk 2). The resolver checks: "Does this response's transaction ID match the query I sent?" If it does not match, the response is discarded.

The DNS transaction ID field is **16 bits**, giving 65,536 possible values — roughly 64K. (Note: the slide says "32 bits" but then says "64K responses." These numbers contradict each other — 32 bits would mean over 4 billion possibilities, not 64K. The 64K figure is correct and matches the actual 16-bit transaction ID field defined in the DNS protocol. The "32 bits" on the slide is an error.) The attacker sends 64K fake responses, each with a different transaction ID, covering every possibility. At least one will match. If the fake response with the correct transaction ID arrives at the resolver **before** the legitimate response from the real authoritative server, the resolver accepts the fake and discards the real one when it eventually arrives (because the query has already been answered).

**The diagram on the slide** shows three actors: a DNS server (left), an authoritative nameserver (right), and an attacker (bottom). The DNS server asks the authoritative server "What's the IP for example.com?" The attacker, positioned below, simultaneously sends a flood of fake responses saying "I am an authoritative nameserver. IP address is 192.0.0.17" — a fake IP under the attacker's control. If the attacker's response arrives first, the DNS server caches the fake IP.

**Why this is devastating:** The poisoned cache entry is now served to every user who queries that resolver. If the resolver serves a university with 3,000 students, all 3,000 students are redirected to the attacker's server when they try to visit the poisoned domain. The attack is silent — no error messages, no visual warnings. The fake site can look identical to the real one.

---

## Slide 30 — Reflection Attack (Slide 30 of 32)

**What is on the slide:** A heading "REFLECTION ATTACK" in red, followed by four bullets and a diagram showing the attack flow. No speaker notes.

A **reflection attack** is not about poisoning data — it is about flooding a victim with traffic by bouncing it off innocent third parties.

---

### How Reflection Works

*"Attacker sends packets to a known service on the intermediary with a spoofed source address of the actual target system"*

The attacker sends DNS queries to legitimate DNS servers, but with a critical twist: the **source IP address** in the query is forged. Instead of the attacker's real IP, the source field contains the **victim's IP address**. This is IP spoofing — and it is possible because UDP (unlike TCP — Transmission Control Protocol, which verifies connections via a three-way handshake) does not verify the source address.

*"When intermediary responds, the response is sent to the target"*

The DNS server receives the query, looks up the answer, and sends the response to the address in the source field — which is the victim, not the attacker. The DNS server is an unwitting accomplice. It is just doing its job: answering a query and sending the response to who (it thinks) asked.

*"'Reflects' the attack off the intermediary (reflector)"*

The name comes from the mechanics: the attacker's traffic bounces (reflects) off the DNS server toward the victim. The victim receives a flood of DNS responses it never requested, from servers it never contacted. The attacker's real IP is hidden — the victim only sees the IP addresses of legitimate DNS servers.

*"Goal is to generate enough volumes of packets to flood the link to the target system without alerting the intermediary"*

The DNS servers being used as reflectors handle the queries normally. Each individual query is tiny and legitimate-looking. The DNS server has no reason to suspect anything is wrong. But the victim receives thousands of these reflected responses simultaneously, overwhelming its network link.

**The diagram** shows the full flow: an attacker controls a botnet (a network of malware-infected computers). The botnet sends small spoofed DNS requests to multiple open resolvers (DNS servers that accept queries from anyone). Each resolver sends an amplified DNS response to the victim's ecommerce server. The victim is flooded from multiple directions with traffic it cannot block because it comes from legitimate DNS servers.

---

## Slide 31 — Amplification Attack (Slide 31 of 32)

**What is on the slide:** A heading "Amplification attack" in red, followed by six bullets. No images, no speaker notes.

Amplification is reflection's deadlier variant. The reflection attack bounces traffic off intermediaries. The amplification attack does the same thing but multiplies the traffic volume in the process.

---

### The Amplification Multiplier

*"Variation of the reflection attack"*

Same mechanics — spoofed source IP, innocent DNS server as reflector, victim receives the responses.

*"Use packet(s) directed at a legitimate DNS server as the intermediary system — a vulnerable DNS may allow a zone transfer"*

The attacker specifically targets DNS servers that are misconfigured to allow zone transfers from anyone (the weakness from Slide 27). A zone transfer request is small — perhaps 60 bytes. The response is the entire zone file — potentially thousands of records.

*"Exploit DNS behaviour to convert a small request to a much larger response (amplification) — 60 bytes query"*

This is the core of the attack: the **amplification factor**. The attacker sends a small query (60 bytes) but the DNS server responds with a much larger answer.

**The numbers from the slide:**

*"512 bytes answer (8.5x)"* — A standard DNS response can be up to 512 bytes. That is an 8.5x amplification: the attacker sends 60 bytes, the victim receives 512 bytes. For every 1 Mbps of traffic the attacker generates, the victim receives 8.5 Mbps.

*"RFC 2671 allows larger answers"* — **RFC** stands for **Request For Comments** — the numbered standards documents published by the **IETF** (Internet Engineering Task Force — the international body that develops internet standards). RFC 2671 introduced **EDNS** (Extension Mechanisms for DNS — an upgrade to the original DNS protocol that removes the 512-byte response limit and allows much larger responses). With EDNS enabled, DNS responses can be significantly larger than 512 bytes.

*"Answers larger than 4000 bytes possible (>60x)"* — With EDNS, a single DNS response can exceed 4000 bytes. That is a 60x amplification factor: the attacker sends 60 bytes, the victim receives over 4000 bytes. For every 1 Mbps the attacker generates, the victim receives 60 Mbps. An attacker with a modest 100 Mbps botnet can generate a 6 Gbps flood aimed at the victim — enough to overwhelm most servers and many network links.

*"Target is flooded with amplified response(s)"*

The victim's network link saturates. Legitimate traffic cannot get through. The service goes offline — not because the server crashed, but because the pipe to the server is completely full of junk DNS responses.

**Why DNS is perfect for amplification:** DNS is one of the few protocols where a small request reliably generates a large response, the transport protocol (UDP) does not verify the source address, and millions of misconfigured open resolvers exist on the internet ready to be used as reflectors. This combination makes DNS amplification one of the most common DDoS (Distributed Denial of Service) attack methods in the real world.

---

## Slide 32 — DNSSEC (Slide 32 of 32)

**What is on the slide:** A heading "DNSSEC" followed by two main bullets with two sub-bullets. A YouTube link at the bottom. No speaker notes.

**DNSSEC** stands for **Domain Name System Security Extensions** — a set of protocol extensions that add cryptographic verification to DNS responses. DNSSEC is the answer to the weaknesses described on Slides 26–31. It does not prevent all attacks, but it addresses the most fundamental one: the inability to verify that a DNS response is genuine.

---

### What DNSSEC Guarantees

*"Guarantees — Authenticity of DNS answer origin — Integrity of reply"*

Two guarantees:

**Authenticity** — the response genuinely came from the authoritative server for that domain. It was not forged by an attacker, intercepted and modified in transit, or injected by a man-in-the-middle. The resolver can cryptographically verify: "This answer was created by someone who holds the private key for `example.com`'s DNS zone."

**Integrity** — the response has not been tampered with since it was created. Not a single bit has been changed. If an attacker tried to modify the response in transit (changing the IP address, adding a fake authority record), the cryptographic signature would break and the resolver would reject the response.

---

### How DNSSEC Works

*"Accomplishes this by signing DNS replies at each step of the way"*

Every authoritative DNS server signs its responses using a **digital signature** — a mathematical value computed from the response data and the server's private key. The corresponding **public key** is published in the DNS itself (as a DNSKEY record). The resolver retrieves the public key and uses it to verify the signature. If the signature is valid, the response is genuine and unmodified. If it is not valid, the response is discarded.

*"Uses public-key cryptography to sign responses"*

**Public-key cryptography** (also called asymmetric cryptography) uses a pair of mathematically related keys: a **private key** (kept secret by the zone owner, used to create signatures) and a **public key** (published openly, used by anyone to verify signatures). The private key can sign data, and only the matching public key can verify that signature. An attacker without the private key cannot forge a valid signature, even if they know the public key.

The "at each step of the way" part means that DNSSEC operates across the entire chain of trust. The root zone signs its responses. The TLD zone signs its responses. The authoritative zone for `example.com` signs its responses. Each level includes a pointer to the public key of the level below it, creating a **chain of trust** from the root all the way down to the specific record you queried. If any link in the chain is missing or broken, validation fails.

---

### What DNSSEC Does NOT Provide

*"DNSSEC does not provide — Confidentiality"*

This is a deliberate and important limitation. DNSSEC proves that a response is **genuine** and **unmodified**, but it does not **encrypt** the response. Everyone on the network can still see your DNS queries and the responses — the data is still transmitted in plaintext. DNSSEC adds a signature alongside the data, but the data itself remains readable.

To put it in analogy terms: DNSSEC is like a notarised letter. The notary stamp proves the letter is genuine and has not been altered. But the letter is not in an envelope — anyone can read it. If you want confidentiality (nobody can read the queries), you need separate technologies like **DoH** (DNS over HTTPS — HTTPS stands for HyperText Transfer Protocol Secure, the encrypted version of the standard web protocol; DoH sends DNS queries inside encrypted HTTPS connections so nobody on the network can see them) or **DoT** (DNS over TLS — TLS stands for Transport Layer Security, the encryption protocol that also underpins HTTPS websites; DoT encrypts DNS queries using this protocol).

DNSSEC and DoH/DoT solve different problems. DNSSEC prevents fake answers. DoH/DoT prevent eavesdropping. A fully secured DNS setup uses both.

---

## End of Lecture 5 — The Complete Picture

Across 32 slides, this lecture covered the two infrastructure services that make networks usable for humans:

**DHCP (Slides 1–11):** How devices get their IP addresses automatically — the scope/exclusion/lease model, the four-step DORA handshake at packet level, and what failure looks like (APIPA, 0.0.0.0).

**DNS (Slides 12–32):** How names become IP addresses — the hierarchical tree, stub and recursive resolvers, iterative and recursive query modes, caching with TTL, zones and zone transfer, split architecture, public DNS performance, and the security landscape from weaknesses through attacks to DNSSEC.

The two services are deeply interconnected. DHCP tells devices where the DNS servers are (Option 6). DNS tells devices where everything else is. Without DHCP, devices have no address. Without DNS, devices have no way to find each other by name. Together, they are the plumbing that makes the internet feel seamless — invisible when working, catastrophic when broken.

---

*Chunk 10 of 10 complete. All slides covered. Full reading guide for Lecture 5 is now available across 10 markdown files.*
