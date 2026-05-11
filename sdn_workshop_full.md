# Workshop 8 — Software Defined Networking (SDN) — Companion Guide

## Part 1 of 3: Topology Construction & Device Configuration

---

### What is SDN?

**SDN** = **Software Defined Networking** — a networking approach where the **control plane** (the brain that decides where traffic goes) is separated from the **data plane** (the hardware that actually forwards packets). Instead of every router and switch making its own independent decisions, a centralised **controller** makes decisions for the entire network from one place.

- Traditional networking: each device has its own brain. You configure them one by one via CLI.
- SDN: one central brain (the controller) manages all devices. You configure the network from a single dashboard.

In this workshop, you will build a traditional network first (topology, IPs, routing, SSH), then hand control to a **Network Controller** in Packet Tracer — so you can see the SDN approach in action.

---

### Layer Map — Workshop 8 (SDN)

| # | Section | Status |
|---|---------|--------|
| 1 | Introduction + Topology Construction (Part 1) | **⬅ YOU ARE HERE** |
| 2 | Device Configuration — Switches, Routers, RIP, SSH (Part 1) | **⬅ YOU ARE HERE** |
| 3 | Network Controller — Provisioning & Discovery (Part 2) | |
| 4 | Network Controller — Assurance, Dashboard & Push Config (Part 3) | |

---

### ⚠ Errors in the Original Workshop Brief

Before we begin — the original document has a few issues worth flagging:

1. **Section 3 title says "Configure S1, S2 R1 and R3"** — there is no R3 in this topology. It should say **R1 and R2**.
2. **East IP table says R2 G0/0 connects to "Switch 2 G0/1"** — this is misleading. R2 G0/0 (192.168.1.1) is on the East subnet, so it connects to **S1** (which has 192.168.1.2 on VLAN 1). S2 is the West switch.
3. **"East PC2/"** appears as a device name with a trailing slash — this is a copy-paste artefact. It refers to a PC behind an IP phone on F0/3.
4. **West table says R1 connects to S2 F0/1, but the S2 config sets `g0/0` as trunk** — contradictory. F0/1 is a FastEthernet port; the uplink to a router should be a GigabitEthernet port. In this guide, we use `g0/1` consistently — adjust to match whichever port you actually cable in Packet Tracer.

These are noted so you don't waste lab time second-guessing yourself.

---

### Network Topology — ASCII Diagram

```
                        WEST BUILDING                                      EAST BUILDING
                   (10.10.10.0/24 subnet)                            (192.168.1.0/24 subnet)

                                                 WAN Link
                                           192.168.2.0/24
                       +--------+                                    +--------+
                       |   R1   |                                    |   R2   |
                       |        |                                    |        |
                       | G0/1   |  G0/0                      G0/1   | G0/0   |
                       |10.10.  |  192.168.                  192.168.| 192.168.|
                       |10.1    |  2.2            ←-------→  2.1    | 1.1    |
                       +---+----+                                    +---+----+
                           |                                             |
                           | (to S2 G0/1)                                | (to S1 G0/1)
                           |                                             |
                       +---+----+                                    +---+----+
                       |   S2   |                                    |   S1   |
                       | VLAN1: |                                    | VLAN1: |
                       |10.10.  |                                    |192.168.|
                       |10.2    |                                    | 1.2    |
                       +--------+                                    +--------+
                        |      |                              |    |    |    |    |
                       F0/2   F0/3                          F0/1  F0/2 F0/3 F0/4 F0/5 F0/6
                        |      |                             |     |    |    |    |    |
                     Server [IP Phone]                     DNS    AP  [IP  Lap1  NC  PC2
                     .4       |                            Srv        Phn]  .5   .7   .6
                           Admin-PC                        .8          |
                            .3                                       PC(.4)
                                                                           [WiFi]
                                                                             |
                                                                          Laptop-2
                                                                            .3
```

**Key:**
- **AP** = Access Point (wireless)
- **NC** = Network Controller (192.168.1.7) — this is the SDN controller device
- **DNS** = DNS Server (192.168.1.8)
- Dotted connections through **[IP Phone]** mean the PC plugs into the phone, and the phone plugs into the switch port

---

### IP Addressing Tables

#### EAST Building — Subnet: 192.168.1.0/24

| Device | IP Address | Default Gateway | Switch Port |
|--------|-----------|----------------|-------------|
| R2 — G0/0 | 192.168.1.1 /24 | — | S1 G0/1 |
| S1 — VLAN 1 | 192.168.1.2 | 192.168.1.1 | — (management IP) |
| East Wireless Laptop-2 | 192.168.1.3 | 192.168.1.1 | via Access Point (WiFi) |
| East PC (behind IP Phone) | 192.168.1.4 | 192.168.1.1 | through IP Phone → F0/3 |
| East Laptop 1 | 192.168.1.5 | 192.168.1.1 | F0/4 |
| East PC2 | 192.168.1.6 | 192.168.1.1 | F0/6 |
| Network Controller | 192.168.1.7 | 192.168.1.1 | F0/5 |
| DNS Server | 192.168.1.8 | 192.168.1.1 | F0/1 |

#### WAN Link — Subnet: 192.168.2.0/24

| Device | IP Address |
|--------|-----------|
| R2 — G0/1 | 192.168.2.1 /24 |
| R1 — G0/0 | 192.168.2.2 /24 |

#### WEST Building — Subnet: 10.10.10.0/24

| Device | IP Address | Default Gateway | Switch Port |
|--------|-----------|----------------|-------------|
| R1 — G0/1 | 10.10.10.1 /24 | — | S2 G0/1 |
| S2 — VLAN 1 | 10.10.10.2 | 10.10.10.1 | — (management IP) |
| Admin PC | 10.10.10.3 | 10.10.10.1 | through IP Phone → F0/3 |
| Server-PT | 10.10.10.4 | 10.10.10.1 | F0/2 |

---

### Construction Notes — What to Do in Packet Tracer

1. **Drag devices from the palette** — you need:
   - 2 × Router (any model with at least 2 GigabitEthernet interfaces)
   - 2 × Switch (2960 or similar with FastEthernet + GigabitEthernet ports)
   - 1 × Network Controller (found under End Devices or Network Controller category in PT)
   - 1 × Server-PT for East (DNS server)
   - 1 × Server-PT for West
   - 2 × PC, 2 × Laptop
   - 1 × Access Point (AccessPoint-PT)
   - 2 × IP Phone (7960 or similar)

2. **Cable everything** according to the tables above
   - Router-to-Switch: use **straight-through** cables
   - PC/Server-to-Switch: use **straight-through** cables
   - Router-to-Router (R1 G0/0 ↔ R2 G0/1): use **crossover** cable (or straight-through if using auto-MDIX capable ports)
   - PC-to-IP Phone: straight-through into the phone's PC port; phone's switch port to the switch

3. **Set IP addresses and default gateways** on every end device using the Desktop > IP Configuration tab

4. **Do NOT configure switches or routers yet** — that comes next

---

### Device Configuration — Switches

#### Why are we configuring switches with IPs?

Switches are Layer 2 devices — they don't normally need IP addresses to forward frames. But here we're giving S1 and S2 management IPs on **VLAN 1** so that the **Network Controller can reach them over the network** via SSH. Without an IP, the controller has no way to talk to the switch.

#### S1 Configuration (East Switch)

```
Switch> enable
Switch# configure terminal

! --- Set hostname ---
Switch(config)# hostname S1

! --- Assign a management IP on VLAN 1 ---
S1(config)# interface Vlan 1
S1(config-if)# ip address 192.168.1.2 255.255.255.0
S1(config-if)# no shutdown

! --- Set the uplink port to R2 as a trunk ---
S1(config-if)# interface g0/1
S1(config-if)# switchport mode trunk
S1(config-if)# no shutdown
S1(config-if)# exit

! --- Default gateway so S1 can reach other subnets ---
S1(config)# ip default-gateway 192.168.1.1
```

**What is happening here:**
- **VLAN 1 IP** — gives the switch an IP address for remote management. VLAN 1 is the default VLAN on Cisco switches; all ports belong to it unless moved.
- **Trunk on G0/1** — the uplink to R2. A **trunk** port carries traffic for all VLANs (even though we only have one here). The original says `g0/0` but check your topology — use whichever GigabitEthernet port connects to the router.
- **Default gateway** — points to R2 (192.168.1.1). Without this, the switch can't send management traffic beyond its own subnet.

#### S2 Configuration (West Switch)

```
Switch> enable
Switch# configure terminal
Switch(config)# hostname S2

S2(config)# interface Vlan 1
S2(config-if)# ip address 10.10.10.2 255.255.255.0
S2(config-if)# no shutdown

S2(config-if)# interface g0/1
S2(config-if)# switchport mode trunk
S2(config-if)# no shutdown
S2(config-if)# exit

S2(config)# ip default-gateway 10.10.10.1
```

Same logic, different subnet. Gateway points to R1 (10.10.10.1).

---

### Device Configuration — Routers & RIP

#### R2 (East Router)

```
R2> enable
R2# configure terminal

! --- Configure interfaces ---
R2(config)# interface g0/0
R2(config-if)# ip address 192.168.1.1 255.255.255.0
R2(config-if)# no shutdown

R2(config-if)# interface g0/1
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit

! --- Enable RIP routing ---
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# network 192.168.1.0
R2(config-router)# network 192.168.2.0
R2(config-router)# no auto-summary
R2(config-router)# exit
```

#### R1 (West Router)

```
R1> enable
R1# configure terminal

R1(config)# interface g0/0
R1(config-if)# ip address 192.168.2.2 255.255.255.0
R1(config-if)# no shutdown

R1(config-if)# interface g0/1
R1(config-if)# ip address 10.10.10.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# network 192.168.2.0
R1(config-router)# network 10.0.0.0
R1(config-router)# no auto-summary
R1(config-router)# exit
```

#### What is RIP?

**RIP** = **Routing Information Protocol** — a dynamic routing protocol that lets routers automatically learn about networks they're not directly connected to. Instead of manually typing static routes, RIP lets R1 and R2 exchange their routing tables so they both know how to reach all three subnets.

- **version 2** — RIPv2 supports **CIDR** (**Classless Inter-Domain Routing** — the ability to use subnet masks other than the old classful defaults) and sends subnet mask information with route updates. RIPv1 does not.
- **no auto-summary** — prevents RIP from collapsing subnets into their classful boundaries. The risk here is on R1: 10.10.10.0/24 is a Class A address, so without `no auto-summary`, RIP would advertise it as 10.0.0.0/8 across the WAN link — a massively overbroad route. The 192.168.x.x networks are Class C, so their classful boundary is already /24 and wouldn't be affected, but `no auto-summary` is best practice on every RIP router regardless.
- **network 10.0.0.0** — note RIP `network` commands use **classful** network addresses. Even though the actual subnet is 10.10.10.0/24, you enter `10.0.0.0` because RIP's `network` command identifies which interfaces to activate RIP on, and it matches using classful boundaries. 10.x.x.x is Class A → `10.0.0.0`.

#### Verify Connectivity

After configuring both routers, test from **any East PC** to **any West device**:

```
C:\> ping 10.10.10.4
```

If this works, RIP has converged and full connectivity is established across all three subnets. If it fails:
- Check interface status: `show ip interface brief` on both routers
- Check RIP: `show ip route` — look for `R` entries (RIP-learned routes)
- Check end device default gateways

---

### SSH Configuration — All Four Devices

#### Why SSH?

**SSH** = **Secure Shell** — a protocol for encrypted remote access to a device's CLI (Command Line Interface). The Network Controller needs SSH to log into switches and routers over the network. Without SSH configured, the controller cannot manage these devices.

The alternative is **Telnet**, which sends everything in **plaintext** (including passwords) — SSH encrypts the session.

#### The Configuration (Same on All Four Devices)

Apply this block to **S1, S2, R1, and R2** — only the hostname changes:

```
! --- Set the enable password ---
S1(config)# enable password class

! --- Create a local user account ---
! (Use your actual student number for both username and password)
S1(config)# username [student-number] password [student-number]

! --- Set a domain name (required for RSA key generation) ---
S1(config)# ip domain-name 6cs029.com

! --- Generate RSA keys for SSH encryption ---
S1(config)# crypto key generate rsa general-keys modulus 1024

! --- Force SSH version 2 ---
S1(config)# ip ssh version 2

! --- Configure VTY lines to accept SSH connections ---
S1(config)# line vty 0 4
S1(config-line)# login local
S1(config-line)# transport input ssh
S1(config-line)# exit
```

**What each line does:**

- **enable password class** — sets the privileged EXEC mode password to `class`. When the controller SSH's in and tries to enter `enable` mode, it needs this password.
- **username / password** — creates a **local** user account stored on the device itself. `login local` on the VTY lines tells the device to check this local database when someone connects.
- **ip domain-name 6cs029.com** — SSH requires a **FQDN** (**Fully Qualified Domain Name** — the complete hostname + domain, e.g. `S1.6cs029.com`) to generate the encryption keys. The domain name is combined with the hostname internally.
- **crypto key generate rsa general-keys modulus 1024** — generates an **RSA** (**Rivest-Shamir-Adleman** — a public-key cryptographic algorithm) key pair of 1024 bits. SSH uses this to encrypt the session. The `modulus` is the key length in bits — 768 is the minimum for SSHv2 on Cisco IOS, but 1024 is the recommended size.
- **ip ssh version 2** — forces **SSHv2** only. Version 1 has known vulnerabilities.
- **line vty 0 4** — configures **VTY** (**Virtual Teletype** — virtual terminal lines for remote access) lines 0 through 4. That's 5 simultaneous remote sessions maximum.
- **login local** — authenticate using the local username/password database (not a RADIUS/TACACS+ server).
- **transport input ssh** — only allow SSH connections on these VTY lines. Blocks Telnet.

#### Repeat for All Devices

Replace `S1` with `S2`, `R1`, and `R2` respectively. The **username, password, domain name, RSA keys, and VTY config are identical** across all four devices. The only difference is the hostname already set earlier.

---

### Checkpoint — What You Should Have at This Point

Before moving on to Part 2 (Network Controller):

- [ ] All devices placed and cabled in Packet Tracer
- [ ] All end devices have correct IPs and default gateways
- [ ] S1 and S2 have VLAN 1 management IPs, trunks, and default gateways
- [ ] R1 and R2 have interface IPs configured and `no shutdown`
- [ ] RIP v2 running on both routers with `no auto-summary`
- [ ] Full connectivity verified: East PC can ping West Server (and vice versa)
- [ ] SSH configured on S1, S2, R1, R2 with matching credentials
- [ ] Enable password set to `class` on all four devices

If all boxes are ticked, your traditional network is ready. In **Part 2**, you will configure the Network Controller to take over management of these devices — the SDN part begins.

---

*Companion guide by Lazizbek (Course Tutor, IDU). Unofficial supplementary material for 6CS029 Advanced Networking. Module leader: Ian Coulson, University of Wolverhampton.*

---

# Workshop 8 — Software Defined Networking (SDN) — Companion Guide

## Part 2 of 3: Network Controller — Provisioning & Discovery

---

### Layer Map — Workshop 8 (SDN)

| # | Section | Status |
|---|---------|--------|
| 1 | Introduction + Topology Construction (Part 1) | ~~Done~~ |
| 2 | Device Configuration — Switches, Routers, RIP, SSH (Part 1) | ~~Done~~ |
| 3 | Network Controller — Provisioning & Discovery (Part 2) | **⬅ YOU ARE HERE** |
| 4 | Network Controller — Assurance, Dashboard & Push Config (Part 3) | |

---

### What Happens in This Part

In Part 1, you built a traditional network — every device configured individually via **CLI** (**Command Line Interface** — the text-based terminal where you type commands). Now you hand management over to the **Network Controller**. This is where SDN begins.

The controller needs three things before it can manage your network:

1. **Network access** — the controller must be reachable from a browser (HTTP setup)
2. **Credentials** — it needs the SSH username/password you configured on S1, S2, R1, R2 in Part 1
3. **Device inventory** — it needs to know the IP addresses of the devices it should manage

Once it has those, it can **discover** the devices — reach out via SSH, pull their configs, and build a picture of your network.

---

### Step 1: Configure the Network Controller Device

The Network Controller is a special device in Packet Tracer (IP: 192.168.1.7 on the East subnet). Before you can access its web interface, you need to enable **HTTP** (**HyperText Transfer Protocol** — the standard protocol for serving web pages) access on it.

#### How to do it:

1. **Click on the Network Controller** device in Packet Tracer
2. Go to the **Config** tab (not CLI — this is a GUI-configured device)
3. Find the **HTTP** settings
4. Set the **Controller Port** to **58000**
5. **Tick** the "Access enabled" checkbox

#### Why port 58000?

By default, HTTP uses port 80 and **HTTPS** (**HyperText Transfer Protocol Secure** — the encrypted version of HTTP) uses port 443. The controller uses a non-standard port (58000) so it doesn't clash with any other web services. This also means when you type the **URL** (**Uniform Resource Locator** — the web address in the browser bar), you must include the port number explicitly.

---

### Step 2: Access the Controller from the Admin PC

You access the controller's web **GUI** (**Graphical User Interface** — point-and-click interface as opposed to CLI) from the **Admin PC** in the West building (10.10.10.3).

#### Why the Admin PC?

Any device with a browser that has IP connectivity to the controller would work. The Admin PC is chosen because that's the administrator's workstation. Full connectivity was established in Part 1 via **RIP** (**Routing Information Protocol** — the dynamic routing protocol configured on R1 and R2), so even though the Admin PC (10.10.10.3) is on the West subnet and the controller (192.168.1.7) is on the East subnet, they can reach each other through R1 and R2.

#### How to do it:

1. **Click on the Admin PC** in Packet Tracer
2. Go to **Desktop** tab → **Web Browser**
3. In the URL bar, type:

```
http://192.168.1.7:58000
```

4. This opens the controller's web interface for the first time

---

### Step 3: Create the Admin Account

The first time you access the controller, it forces you to create a new user account. This is the **controller's own login** — separate from the SSH credentials on the network devices.

#### Fill in:

| Field | Value |
|-------|-------|
| Username | `admin` |
| Password | Your student number |

Then **log in** with those same credentials.

#### Don't confuse the three credentials:

| Credential | What it's for | Where it lives |
|-----------|---------------|----------------|
| `admin` / student number | Logging into the **controller's web GUI** | Stored on the controller itself |
| student number / student number | SSH access to **S1, S2, R1, R2** | Stored locally on each network device (`username` command from Part 1) |
| `class` | Enable password on S1, S2, R1, R2 | Stored locally on each network device (`enable password` command from Part 1) |

The controller will use both the SSH credentials (second row) **and** the enable password (third row) when it connects to your network devices. You'll set that up next.

---

### Step 4: The Initial Dashboard

After logging in, you'll see the controller's **Dashboard**. Right now it's empty — no devices, no topology, no stats. That's expected. The controller doesn't know about your network yet.

You need to **provision** it — which means telling it what devices exist and how to log into them.

**Provisioning** = the process of registering network devices with the controller so it can manage them. Think of it as introducing the controller to your network.

---

### Step 5: Create a Credential Profile

Before adding devices, the controller needs to know **how to log into them**. You set this up as a reusable credential profile.

#### How to do it:

1. From the controller's web interface, navigate to **Provisioning** (look for it in the left-hand menu or top navigation)
2. Select **Credentials** from the top menu
3. Click **+ Credential** (the button to add a new credential set)
4. Fill in the form:

| Field | Value |
|-------|-------|
| Username | Your student number |
| Password | Your student number |
| Enable Password | `class` |

5. **Save** the credential

#### What this does:

This stores the SSH login and enable password that the controller will use when it connects to any network device. Since you configured all four devices (S1, S2, R1, R2) with the **same** username, password, and enable password in Part 1, you only need **one** credential profile.

If different devices had different credentials, you'd create multiple profiles and assign the right one to each device.

---

### Step 6: Add Network Devices

Now tell the controller which devices to manage.

#### How to do it:

1. In the **Provisioning** section, select **Network Device** from the menu
2. Click **+ Device**
3. Enter the **IP address** of the device
4. Select the **credential profile** you just created from the dropdown
5. Click **Add** (or Save)
6. **Repeat for all four network devices**

#### Which IPs to enter:

| Device | IP Address to Enter | Why this IP |
|--------|-------------------|-------------|
| S1 | 192.168.1.2 | S1's VLAN 1 management IP |
| S2 | 10.10.10.2 | S2's VLAN 1 management IP |
| R1 | 10.10.10.1 | R1's G0/1 interface (West-facing) |
| R2 | 192.168.1.1 | R2's G0/0 interface (East-facing) |

**Important:** You enter the **management IP** of each device — the IP the controller can reach via SSH. For switches, that's the VLAN 1 IP. For routers, use any interface IP that's reachable from the controller.

After adding all four, you should see them listed in the Network Device view.

---

### Step 7: Run Discovery

Adding devices tells the controller they exist. **Discovery** goes further — the controller actively reaches out to each device via SSH, pulls configuration data, and maps the relationships between them (who connects to whom, what interfaces are up, etc.).

#### How to do it:

1. Navigate to **Discovery** in the controller's menu
2. Click **+ Discovery** (or "New Discovery")
3. Fill in the **IP range** for the East subnet:

| Field | Value |
|-------|-------|
| Start IP | 192.168.1.1 |
| End IP | 192.168.1.8 |

4. Select the credential profile from the dropdown
5. Click **Start** (or Save/Run)
6. **Repeat** for the West subnet:

| Field | Value |
|-------|-------|
| Start IP | 10.10.10.1 |
| End IP | 10.10.10.4 |

#### What Discovery does behind the scenes:

1. The controller tries to SSH into every IP in the range
2. For each device it successfully connects to, it runs `show` commands to gather:
   - Hostname
   - Interface status (up/down)
   - Connected neighbours
   - Running configuration details
3. It uses this data to build a **topology map** and **device inventory**

Not every IP in the range is a manageable device — the controller will simply skip hosts it can't SSH into (like PCs and servers without SSH). It only picks up switches and routers that have SSH properly configured.

#### After Discovery — What to Check:

Once both discovery scans complete, select a discovery entry from the list to see results. You should see:

- **S1, S2** — listed as switches with their VLAN 1 IPs
- **R1, R2** — listed as routers with their interface IPs
- **Interface states** — which ports are up/down
- **Link information** — which device connects to which

If a device shows as "unreachable" or doesn't appear:
- Verify SSH is working: from the Admin PC CLI, try `ssh -l [username] [device-IP]`
- Check the enable password matches what you entered in the credential profile
- Check connectivity: can you ping the device from the controller's subnet?

---

### Checkpoint — What You Should Have at This Point

Before moving on to Part 3:

- [ ] Network Controller HTTP access enabled on port 58000
- [ ] Admin account created (`admin` / student number)
- [ ] Credential profile saved with SSH username/password + enable password
- [ ] All four network devices (S1, S2, R1, R2) added to the controller
- [ ] Discovery completed for both East (192.168.1.x) and West (10.10.10.x) subnets
- [ ] All four devices showing as discovered with correct interface states

If all boxes are ticked, the controller now has full visibility of your network. In **Part 3**, you'll use the controller's **Assurance** features to monitor the network and **Push Config** to apply centralised settings — the real payoff of SDN.

---

*Companion guide by Lazizbek (Course Tutor, IDU). Unofficial supplementary material for 6CS029 Advanced Networking. Module leader: Ian Coulson, University of Wolverhampton.*

---

# Workshop 8 — Software Defined Networking (SDN) — Companion Guide

## Part 3 of 3: Assurance, Dashboard & Push Config

---

### Layer Map — Workshop 8 (SDN)

| # | Section | Status |
|---|---------|--------|
| 1 | Introduction + Topology Construction (Part 1) | ~~Done~~ |
| 2 | Device Configuration — Switches, Routers, RIP, SSH (Part 1) | ~~Done~~ |
| 3 | Network Controller — Provisioning & Discovery (Part 2) | ~~Done~~ |
| 4 | Network Controller — Assurance, Dashboard & Push Config (Part 3) | **⬅ YOU ARE HERE** |

---

### What Happens in This Part

In Part 2, you provisioned the controller — gave it credentials, added devices, and ran discovery. The controller now knows **what** is on your network and **how** things are connected.

In this part, you use the controller's monitoring and management features:

- **Assurance** — real-time visibility into device health, host details, topology, and traffic paths
- **Dashboard** — a summary overview of the entire network at a glance
- **Push Config** — the **SDN** (**Software Defined Networking** — centralised network management from a single controller) payoff: push a centralised configuration (**NTP** — **Network Time Protocol** — and Syslog) to all four network devices in one action, instead of configuring them one by one via **CLI** (**Command Line Interface** — the text-based terminal)

---

### Assurance — Overview

**Assurance** is the controller's monitoring section. It gives you a live picture of your network without needing to **SSH** (**Secure Shell** — encrypted remote CLI access) into individual devices and run `show` commands manually.

Navigate to **Assurance** in the controller's menu. You'll see several sub-options. We'll go through each.

---

### Assurance → Discovery (Device Details)

You already ran discovery in Part 2. Now select a completed discovery entry from the list to view detailed results.

#### What you'll see:

For each discovered device, the controller shows:

- **Hostname** — S1, S2, R1, R2
- **Device type** — switch or router
- **Management IP** — the IP the controller uses to reach the device
- **Platform/model** — the hardware model (e.g., 2960 for switches)
- **Software version** — the **IOS** (**Internetwork Operating System** — Cisco's proprietary operating system that runs on their routers and switches) version running on the device
- **Interface count and states** — how many interfaces exist, which are up/down

#### Why this matters:

In a traditional network, you'd need to SSH into each device and run `show version`, `show ip interface brief`, etc. The controller aggregates all of that into a single view. For 4 devices, the time saving is modest. For a network with 400 devices, this is indispensable — the **raison d'être** (French: "reason for existing" — pronounced *reh-ZOHN det-ruh* — meaning the fundamental justification) of SDN controllers.

---

### Assurance → Hosts

This view shows **end devices** — PCs, laptops, servers — not network infrastructure.

#### What you'll see:

For each host the controller has detected:

- **Host IP address**
- **MAC address** — **MAC** = **Media Access Control** — the unique hardware address burned into every network interface card
- **Which switch port** the host is connected to
- **Which VLAN** (**Virtual Local Area Network** — a logical grouping of switch ports into separate broadcast domains) the host belongs to (VLAN 1 in our case for everything)

#### How does the controller know about hosts?

The controller doesn't SSH into PCs — they don't run SSH. Instead, it learns about hosts indirectly from the switches. Switches maintain a **MAC address table** (which MAC addresses are seen on which ports), and the controller pulls this data during discovery. It correlates MAC addresses with **ARP** (**Address Resolution Protocol** — the protocol that maps IP addresses to MAC addresses on a local network) entries from routers to associate IPs with MACs.

---

### Assurance → Topology

This is the visual network map — the controller draws a diagram of your switches and routers and the links between them.

#### What you'll see:

- **S1, S2, R1, R2** represented as icons
- **Lines between them** showing physical connections
- **Link status** — green for up, red for down
- **Interface labels** on each end of the link

#### What you won't see:

End hosts (PCs, servers, laptops) typically don't appear on this view — it focuses on network infrastructure only. Use the **Hosts** view for end devices.

#### The SDN perspective:

This topology was built **automatically** from discovery data. You didn't draw it — the controller figured out the connections by querying each device. In a real enterprise network, this auto-discovered topology saves hours of manual documentation and stays up to date as the network changes.

---

### Assurance → Path Trace

This feature shows the **exact path** traffic takes between two devices on the network.

#### How to use it:

1. Select the **Path** option in Assurance
2. Enter the **source** IP address (e.g., 192.168.1.6 — East PC2)
3. Enter the **destination** IP address (e.g., 10.10.10.3 — Admin PC)
4. The controller displays every hop along the path: which interfaces, which devices, in what order

#### What you'll see:

For East PC2 → Admin PC, the path would be:

```
PC2 (192.168.1.6) → S1 (F0/6 in, G0/1 out) → R2 (G0/0 in, G0/1 out) → R1 (G0/0 in, G0/1 out) → S2 (G0/1 in, F0/3 out) → Admin PC (10.10.10.3)
```

#### Why this matters:

In troubleshooting, knowing the exact path traffic follows is critical. Without a controller, you'd need to log into each device, check routing tables and MAC tables, and piece the path together manually. The controller does this in one click.

---

### Dashboard

Navigate back to the **Dashboard** — the main overview page. Now that the controller has discovered your devices, this page is no longer empty.

#### What you'll see:

- **Device count** — total number of managed switches and routers
- **Device health summary** — how many devices are reachable/healthy vs. unreachable/degraded
- **Network health indicators** — overall status of the network

The dashboard is the first place a network administrator looks when they sit down — a quick "is everything OK?" check before diving into specifics.

---

### Push Config — Centralised Configuration

This is the most important SDN feature in this workshop. Instead of logging into each device individually and typing the same commands four times, you define the configuration **once** on the controller and **push** it to all devices simultaneously.

You will push two services:

1. **NTP** — **Network Time Protocol** — synchronises clocks across all network devices so timestamps in logs are consistent
2. **Syslog** — a standard protocol for sending log messages to a centralised server for storage and analysis

#### Why NTP and Syslog together?

Log messages are only useful if you know **when** events happened. If every device has a different clock, correlating events across devices becomes a nightmare. NTP ensures all devices agree on the time; Syslog ensures all devices send their logs to one place. Together, they give you a unified, time-accurate audit trail.

---

### Step 1: Configure NTP on the Controller

1. From the **Dashboard**, look for the **QoS** (**Quality of Service** — network traffic prioritisation mechanisms) window or settings area
2. Click the **Settings** button (gear icon)
3. Select the **NTP** tab
4. Enter the **NTP server address** — this is the **DNS** (**Domain Name System** — translates domain names to IP addresses) server (192.168.1.8) which also acts as the NTP server in this topology
5. Click **Save**

#### What this does:

You're telling the controller: "When I push config, tell all managed devices to synchronise their clocks with this NTP server." You're not configuring NTP yet — you're staging the configuration for the push.

---

### Step 2: Configure Syslog on the Controller

1. Still in the **Settings** area, select the **Syslog** tab
2. Enter the **Syslog server address** — same server: 192.168.1.8
3. Click **Save**

#### What this does:

Same idea — you're staging the Syslog configuration. When pushed, all four devices will send their log messages to 192.168.1.8.

#### Both services pointing to the same server:

In this lab, the East DNS server (192.168.1.8) doubles as both the NTP and Syslog server. In a real network, these would often be separate servers, but Packet Tracer keeps it simple with one multi-role server.

---

### Step 3: Push the Configuration

1. Click **PUSH CONFIG** from the Dashboard
2. A summary window will appear showing what will be pushed:
   - NTP server: 192.168.1.8
   - Syslog server: 192.168.1.8
   - Target devices: S1, S2, R1, R2
3. Click **OK** to confirm

#### What happens behind the scenes:

The controller SSH's into each of the four devices and runs commands equivalent to:

```
! --- NTP configuration ---
ntp server 192.168.1.8

! --- Syslog configuration ---
logging host 192.168.1.8
```

These two commands are applied to **all four devices** in one action. That's the SDN value — centralised, consistent, simultaneous configuration.

In a traditional network, you would need to:
1. SSH into S1, type the commands, exit
2. SSH into S2, type the commands, exit
3. SSH into R1, type the commands, exit
4. SSH into R2, type the commands, exit

With 4 devices, the difference is minor. With 400 devices, it's transformational.

---

### Step 4: Verify the Push — Syslog Test

To confirm the pushed configuration is working, perform this test:

#### On S1 (East Switch):

1. Open S1's CLI in Packet Tracer
2. Shut down the G0/1 interface (the uplink to R2):

> **Note:** The original brief says `g0/0`. This guide uses `g0/1` consistently as the router uplink port — use whichever port you actually cabled in Packet Tracer. The point is to shut the trunk link so a syslog message is generated.

```
S1> enable
Password: class
S1# configure terminal
S1(config)# interface g0/1
S1(config-if)# shutdown
```

3. Wait a few seconds, then bring it back up:

```
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# exit
```

#### What to check:

1. Open the **Syslog server** (192.168.1.8) in Packet Tracer
2. Go to **Services** tab → **Syslog**
3. You should see log entries from S1 recording the interface going down and coming back up — something like:

```
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
```

#### What this proves:

- The **Push Config** worked — S1 is now sending logs to the Syslog server
- **NTP** is also working (the timestamps on the log entries should be synchronised, not showing the default device clock)
- The controller successfully configured a network device remotely without you typing `logging host` or `ntp server` on S1 manually

---

### Demonstrate to Your Tutor

The original brief says "Demonstrate to your tutor." What your tutor wants to see:

1. **The controller dashboard** — populated with device data (not empty)
2. **Topology view** — showing all four network devices and their connections
3. **Path trace** — a working path between an East and West device
4. **Syslog entries** — proving the pushed NTP/Syslog configuration is active
5. **The shut/no shut test** — showing that the interface state change was captured by the Syslog server

---

### Summary — What You Did in This Workshop

| Part | What you built | Traditional vs. SDN |
|------|---------------|-------------------|
| Part 1 | Physical topology, IP addressing, **RIP** (**Routing Information Protocol**) routing, SSH access | **Traditional** — every device configured individually via CLI |
| Part 2 | Controller provisioning — credentials, device inventory, discovery | **SDN setup** — teaching the controller about the network |
| Part 3 | Assurance monitoring, centralised NTP/Syslog push | **SDN in action** — one-click visibility and configuration |

The key takeaway: you built the **same network** in Part 1 that any traditional engineer would. The difference is what happened in Parts 2 and 3 — a single controller now provides centralised monitoring, topology discovery, path tracing, and the ability to push configuration changes to all devices at once. That's the core promise of SDN.

---

### Layer Map — Workshop 8 (SDN) — Final

| # | Section | Status |
|---|---------|--------|
| 1 | Introduction + Topology Construction (Part 1) | ~~Done~~ |
| 2 | Device Configuration — Switches, Routers, RIP, SSH (Part 1) | ~~Done~~ |
| 3 | Network Controller — Provisioning & Discovery (Part 2) | ~~Done~~ |
| 4 | Network Controller — Assurance, Dashboard & Push Config (Part 3) | ~~Done~~ |

**Workshop 8 complete.**

---

*Companion guide by Lazizbek (Course Tutor, IDU). Unofficial supplementary material for 6CS029 Advanced Networking. Module leader: Ian Coulson, University of Wolverhampton.*
