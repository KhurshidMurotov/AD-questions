# IPv6 Network Configuration Workshop

## Cisco Packet Tracer Tutorial

---

## Scenario

Your company has two sites in Telford and Wolverhampton. Each site requires a LAN, and the two sites must be connected via a WAN link. In this workshop, you will configure IPv6 addressing on routers and PCs, and implement RIPng (RIP next generation) as the routing protocol.

> **Why RIPng?**
> The traditional RIP protocol does not support IPv6. RIPng was developed specifically for
> IPv6 networks while maintaining the same distance-vector metric as RIP. Unlike RIP for IPv4,
> RIPng runs as a process on each interface.

---

## IP Addressing Scheme

Use the following IPv6 addresses for this workshop:

```
+----------------+-----------+---------------------+---------+
| Device         | Interface | IPv6 Address        | Prefix  |
+----------------+-----------+---------------------+---------+
| WOLVES Router  | G0/0      | 2011:314:271:1::1   | /64     |
| WOLVES Router  | S0/0/0    | FC00::12:1          | /112    |
| TELFORD Router | G0/0      | 2011:314:271:2::1   | /64     |
| TELFORD Router | S0/0/0    | FC00::12:2          | /112    |
| WOLVES PC      | NIC       | 2011:314:271:1::10  | /64     |
| TELFORD PC     | NIC       | 2011:314:271:2::10  | /64     |
+----------------+-----------+---------------------+---------+
```

---

## Network Diagram

```
[WOLVES PC]                                        [TELFORD PC]
2011:314:271:1::10                          2011:314:271:2::10
      |                                            |
  [Switch]                                    [Switch]
      |                                            |
     G0/0                                        G0/0
2011:314:271:1::1                          2011:314:271:2::1
      |                                            |
[WOLVES Router]---S0/0/0----WAN----S0/0/0---[TELFORD Router]
              FC00::12:1    <-->    FC00::12:2
```

---

## Part 1: Building the Network Topology

### 1.1 Open Cisco Packet Tracer

Step 1: Launch Cisco Packet Tracer on your computer.
Step 2: Create a new file (File > New) or use the default blank workspace.

### 1.2 Add the Routers

Step 3: In the bottom-left device panel, click on Network Devices (the icon that looks like a hub).
Step 4: Click on Routers in the submenu.
Step 5: Select the 2911 router (or any router with serial interfaces).
Step 6: Click on the workspace to place the first router. This will be the WOLVES router.
Step 7: Place a second 2911 router to the right. This will be the TELFORD router.

#### Rename the Routers

Step 8: Click on the first router to open its configuration window.
Step 9: Go to the Config tab.
Step 10: In the Display Name field, type WOLVES and press Enter.
Step 11: Repeat for the second router, naming it TELFORD.

### 1.3 Add the Switches

Step 12: In the device panel, click on Switches.
Step 13: Select the 2960 switch.
Step 14: Place one switch below the WOLVES router.
Step 15: Place another switch below the TELFORD router.

### 1.4 Add the PCs

Step 16: In the device panel, click on End Devices.
Step 17: Select PC.
Step 18: Place one PC below the WOLVES switch (this is the WOLVES PC).
Step 19: Place another PC below the TELFORD switch (this is the TELFORD PC).

### 1.5 Cable the Network

Now connect all devices with the appropriate cables.

#### Connect Routers to Switches (Straight-through cables)

Step 20: Click on the Connections icon (lightning bolt) in the device panel.
Step 21: Select Copper Straight-Through cable.
Step 22: Click on the WOLVES router, then select GigabitEthernet0/0.
Step 23: Click on the switch below it, then select any FastEthernet port (e.g., Fa0/1).
Step 24: Repeat to connect TELFORD router (G0/0) to its switch.

#### Connect PCs to Switches (Straight-through cables)

Step 25: Using the same Copper Straight-Through cable:
Step 26: Connect WOLVES PC (FastEthernet0) to its switch (any available port, e.g., Fa0/2).
Step 27: Connect TELFORD PC (FastEthernet0) to its switch.

#### Connect Routers Together (Serial cable for WAN link)

> **HWIC-2T Module Required:** Don't forget to insert a serial interface module into each router first. Choose HWIC-2T — it's simple and reliable. You do it via drag and drop. First turn the router off, then insert the module, then turn it back on.

Step 28: Click on Connections, then select Serial DCE cable.
Step 29: Click on WOLVES router and select Serial0/0/0.
Step 30: Click on TELFORD router and select Serial0/0/0.

> **Important:** The DCE end of the cable (WOLVES side) must provide the clock signal. We will configure the clock rate on the WOLVES router.

---

## Part 2: Configuring the WOLVES Router

Click on the WOLVES router, then go to the CLI tab. Type 'no', confirm and then press Enter to start.

### 2.1 Enter Global Configuration Mode

```
Router> enable
Router# configure terminal
Router(config)# hostname WOLVES
```

### 2.2 Enable IPv6 Routing

This command enables the router to forward IPv6 packets:

```
WOLVES(config)# ipv6 unicast-routing
```

### 2.3 Configure the LAN Interface (G0/0)

Configure the interface that connects to the local network:

```
WOLVES(config)# interface GigabitEthernet0/0
WOLVES(config-if)# ipv6 address 2011:314:271:1::1/64
WOLVES(config-if)# ipv6 enable
WOLVES(config-if)# ipv6 rip comms enable
WOLVES(config-if)# no shutdown
```

> **Understanding the Commands:**
> `ipv6 address` — Assigns the IPv6 address to this interface
> `ipv6 enable` — Activates IPv6 processing on the interface
> `ipv6 rip comms enable` — Enables RIPng process named 'comms' on this interface
> `no shutdown` — Turns the interface on

### 2.4 Configure the WAN Interface (S0/0/0)

Configure the serial interface that connects to the TELFORD router:

```
WOLVES(config-if)# interface Serial0/0/0
WOLVES(config-if)# ipv6 address FC00::12:1/112
WOLVES(config-if)# ipv6 enable
WOLVES(config-if)# ipv6 rip comms enable
WOLVES(config-if)# clock rate 4000000
WOLVES(config-if)# no shutdown
```

> **Why clock rate?**
> The WOLVES router has the DCE end of the serial cable, so it must provide the clocking signal. The `clock rate` command sets the speed of the serial link.
>
> **Important!** It does matter which router you connected to the DCE cable first! If it was WOLVES, then it's the one that is DCE, if not — vice versa. Only one router may become a clock manager. No two clock initiators are possible.
>
> If you forgot, just delete the wire (press Del button — it moves into deleting mode, and Esc to leave that mode) and then replug it first to WOLVES and then to TELFORD.

### 2.5 Exit and Save

```
WOLVES(config-if)# end
WOLVES# copy running-config startup-config
```

Press Enter when prompted to confirm the filename.

---

## Part 3: Configuring the TELFORD Router

Click on the TELFORD router, then go to the CLI tab.

### 3.1 Basic Configuration

```
Router> enable
Router# configure terminal
Router(config)# hostname TELFORD
TELFORD(config)# ipv6 unicast-routing
```

### 3.2 Configure the LAN Interface (G0/0)

```
TELFORD(config)# interface GigabitEthernet0/0
TELFORD(config-if)# ipv6 address 2011:314:271:2::1/64
TELFORD(config-if)# ipv6 enable
TELFORD(config-if)# ipv6 rip comms enable
TELFORD(config-if)# no shutdown
```

### 3.3 Configure the WAN Interface (S0/0/0)

```
TELFORD(config-if)# interface Serial0/0/0
TELFORD(config-if)# ipv6 address FC00::12:2/112
TELFORD(config-if)# ipv6 enable
TELFORD(config-if)# ipv6 rip comms enable
TELFORD(config-if)# no shutdown
```

> **Note:** TELFORD does not need the `clock rate` command because it has the DTE end of the serial cable. Verify that you did actually connect the cable to WOLVES side first. It does matter!

### 3.4 Exit and Save

```
TELFORD(config-if)# end
TELFORD# copy running-config startup-config
```

---

## Part 4: Configuring the PCs

### 4.1 Configure WOLVES PC

Step 1: Click on the WOLVES PC to open its configuration window.
Step 2: Go to the Desktop tab.
Step 3: Click on IP Configuration.
Step 4: Select the IPv6 Configuration section (you may need to scroll down).
Step 5: Enter the following settings:

```
IPv6 Address:    2011:314:271:1::10
Prefix Length:   64
Default Gateway: 2011:314:271:1::1
```

### 4.2 Configure TELFORD PC

Step 6: Click on the TELFORD PC.
Step 7: Go to Desktop > IP Configuration.
Step 8: Enter the following settings:

```
IPv6 Address:    2011:314:271:2::10
Prefix Length:   64
Default Gateway: 2011:314:271:2::1
```

---

## Part 5: Testing Connectivity

### 5.1 Ping from WOLVES PC to TELFORD PC

Step 1: Click on WOLVES PC.
Step 2: Go to Desktop > Command Prompt.
Step 3: Type the following command:

```
ping 2011:314:271:2::10
```

You should see successful replies. If not, wait a few seconds for RIPng to converge and try again.

### 5.2 Verify Router Interfaces

On each router, run these verification commands:

#### Show interface status:

```
WOLVES# show ipv6 interface brief
```

This displays all interfaces with their IPv6 addresses and status (up/down).

#### Show routing table:

```
WOLVES# show ipv6 route
```

This shows all known IPv6 routes. You should see:
- Connected routes (C) for directly connected networks
- Local routes (L) for the router's own addresses
- RIP routes (R) learned from the other router via RIPng

---

## Part 6: Experimenting with EUI-64

EUI-64 is a method for automatically generating the interface ID (last 64 bits) of an IPv6 address from the interface's MAC address.

### 6.1 Add EUI-64 Address to WOLVES Router

On the WOLVES router CLI:

```
WOLVES# configure terminal
WOLVES(config)# interface GigabitEthernet0/0
WOLVES(config-if)# ipv6 address 2011:314:271:1::/64 eui-64
WOLVES(config-if)# end
```

### 6.2 Add EUI-64 Address to TELFORD Router

On the TELFORD router CLI:

```
TELFORD# configure terminal
TELFORD(config)# interface GigabitEthernet0/0
TELFORD(config-if)# ipv6 address 2011:314:271:2::/64 eui-64
TELFORD(config-if)# end
```

### 6.3 Examine the Results

Now check the interface addresses:

```
WOLVES# show ipv6 interface brief
```

> **What do you notice?**
> The interface now has TWO IPv6 addresses:
> 1. The manually configured address (2011:314:271:1::1)
> 2. The EUI-64 generated address (with a longer interface ID derived from the MAC address)

### 6.4 Testing Connectivity

From the WOLVES PC, try to ping both IPv6 addresses on your partner's router G0/0 interface. Were you able to ping both addresses?

### 6.5 Link-Local Addresses

Every IPv6 interface automatically generates a link-local address starting with FE80::. Check the `show ipv6 interface brief` output to find these addresses.

Try to ping your partner's link-local address from the TELFORD PC:

```
ping FE80::xxxx:xxxx:xxxx:xxxx
```

> **Were you successful?**
> Link-local addresses are only valid within the local network segment. They cannot be routed across the WAN link. This ping should fail.

---

## Part 7: Research Topics for Your Portfolio

Write notes in your own words on the following topics. Use approximately 400 words total.

### Topic 1: IPv6 Addressing Scheme

Research and explain:
- Address scope: global unicast, unique local, link-local
- Address structure: the 128-bit format and hexadecimal notation
- Address types: unicast, multicast, anycast

### Topic 2: EUI-64 Address Construction

Research and explain how an EUI-64 interface identifier is constructed from a MAC address. Include:
- How FFFE is inserted into the MAC address
- Why the 7th bit (U/L bit) is flipped
- A worked example showing the conversion

---

## Appendix: Quick Reference

### Essential IPv6 Commands

```
+-------------------------------+--------------------------------------------+
| Command                       | Purpose                                    |
+-------------------------------+--------------------------------------------+
| ipv6 unicast-routing          | Enable IPv6 routing on the router          |
| ipv6 address <addr>/<prefix>  | Assign IPv6 address to interface           |
| ipv6 enable                   | Enable IPv6 on the interface               |
| ipv6 rip <n> enable           | Enable RIPng on the interface              |
| show ipv6 interface brief     | Show IPv6 addresses and interface status   |
| show ipv6 route               | Display the IPv6 routing table             |
+-------------------------------+--------------------------------------------+
```

---

*End of Workshop*
