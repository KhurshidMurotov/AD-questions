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
