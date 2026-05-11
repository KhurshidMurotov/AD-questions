# Workshop 4 — NTP & Syslog Logging: Part 1 — Build & Connect

## What We Are Building

Two routers connected by a serial WAN link, with a PC on R2's LAN
side that will later become our syslog server. Before we can set
up NTP or syslog, the network itself must be working.

```
                  Serial WAN
                  10.1.1.0/30
                                                     172.16.2.0/24
   ┌────────┐                      ┌────────┐       ┌────┐  ┌──────┐
   │   R1   │──────────────────────│   R2   │───────│ S1 │──│ PC-B │
   │        │  .1 (DCE)       .2   │        │  .1   └────┘  │  .3  │
   └────────┘                      └────────┘               └──────┘
```

---

## Addressing Table

Keep this table visible while you work. Every IP in this lab
comes from here.

```
 Hostname │ Interface    │ IP Address  │ Subnet Mask     │ Default GW
 ═════════╪══════════════╪═════════════╪═════════════════╪═══════════
 R1       │ S0/0/0 (DCE) │ 10.1.1.1    │ 255.255.255.252 │ N/A
 R2       │ S0/0/0       │ 10.1.1.2    │ 255.255.255.252 │ N/A
 R2       │ G0/0         │ 172.16.2.1  │ 255.255.255.0   │ N/A
 PC-B     │ NIC          │ 172.16.2.3  │ 255.255.255.0   │ 172.16.2.1
```

> **Why /30 on the serial link?** A /30 subnet gives exactly
> 2 usable addresses — perfect for a point-to-point link between
> two routers. No wasted addresses.

---

## Step 1: Place Devices in Packet Tracer

Open Cisco Packet Tracer and start with a blank workspace.

**Routers:**
1. Click **Network Devices** → **Routers**
2. Select **1941** (or 2901 if 1941 is unavailable)
3. Place two routers on the workspace
4. Label the left one **R1**, the right one **R2**

**Switch (for the LAN):**
1. Click **Network Devices** → **Switches**
2. Select **2960**
3. Place it between R2 and PC-B

> **Note:** The original worksheet does not mention a switch, but
> in Packet Tracer you need one between R2's GigabitEthernet port
> and PC-B. On physical equipment, you could connect a PC directly
> to a router's Ethernet port with a crossover cable — in CPT,
> using a switch with straight-through cables is the standard
> approach.

**PC:**
1. Click **End Devices** → **PC**
2. Place one PC to the right of the switch
3. Label it **PC-B**

---

## Step 2: Add the Serial Module to the Routers

The 1941 router does not have serial ports by default.
You must add them:

1. Click on **R1**
2. Go to the **Physical** tab
3. **Power off** the router (click the power switch)
4. From the module list on the left, find **WIC-1T**
   (or **HWIC-2T** depending on your version)
5. Drag it into an empty slot on the router
6. **Power on** the router
7. Repeat for **R2**

> **Important:** You must power off the router before adding
> modules. Packet Tracer will not let you insert them while
> the router is running.

---

## Step 3: Cable the Devices

| Connection        | Cable Type       | From           | To            |
|-------------------|------------------|----------------|---------------|
| R1 ↔ R2           | Serial DCE       | R1 S0/0/0      | R2 S0/0/0     |
| R2 ↔ S1           | Straight-through | R2 G0/0        | S1 Fa0/1      |
| S1 ↔ PC-B         | Straight-through | S1 Fa0/2       | PC-B Fa0      |

> **How to identify DCE:** The worksheet says R1's serial is DCE.
> In Packet Tracer, use a **Serial DCE** cable and connect from
> R1 first. The clock icon appears on the DCE end. The DCE side
> is responsible for providing the clock rate.

> **About link lights:** The LAN connections (R2 → S1 → PC-B)
> should turn green shortly after cabling. The serial link
> between R1 and R2 will stay **red** until both sides are
> configured with IP addresses and `no shutdown` in Steps 5–6.
> This is normal — do not wait for it to turn green now.

---

## Step 4: Erase Any Previous Configuration

The worksheet says: *"Check that the router has previous
configuration on it by issuing the show running-config command.
If it has then refer to the handout on Canvas on erasing the
settings."*

On **both** R1 and R2, run:

```
Router> enable
Router# show running-config
```

If you see any configuration beyond the defaults, erase it:

```
Router# erase startup-config
Router# reload
```

When prompted to save, type **no**. The router will reboot clean.

---

## Step 5: Configure R1

```
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# no ip domain-lookup
```

> **What is `no ip domain-lookup`?** Without this, every time you
> mistype a command, the router thinks it is a hostname and tries
> to look it up via DNS — freezing your CLI for 30+ seconds.
> This command disables that behavior. Not required, but saves
> frustration.

**Configure the serial interface:**

```
R1(config)# interface Serial0/0/0
R1(config-if)# ip address 10.1.1.1 255.255.255.252
R1(config-if)# clock rate 128000
R1(config-if)# no shutdown
R1(config-if)# exit
```

> **Why clock rate?** On a serial link, one side must provide the
> clocking signal — this is the DCE side. The workshop specifies
> R1 as DCE, so R1 sets the clock rate. R2 (the DTE side) does
> not need this command.

---

## Step 6: Configure R2

```
Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# no ip domain-lookup
```

**Configure the serial interface:**

```
R2(config)# interface Serial0/0/0
R2(config-if)# ip address 10.1.1.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
```

**Configure the LAN interface:**

```
R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip address 172.16.2.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit
```

> **Note:** R2 has two interfaces — serial toward R1, and
> GigabitEthernet toward the LAN where PC-B sits. R1 only
> has the serial interface.

---

## Step 7: Configure PC-B

1. Click on **PC-B**
2. Go to **Desktop** → **IP Configuration**
3. Set the following:

```
 IP Address:      172.16.2.3
 Subnet Mask:     255.255.255.0
 Default Gateway: 172.16.2.1
```

> **Why 172.16.2.3 and not .2?** The .1 address is already
> taken by R2's G0/0 interface. The .2 address is skipped
> (possibly reserved for a second router or future device).
> The worksheet assigns .3 to PC-B.

---

## Step 8: Configure RIPv2

The routers need a routing protocol so that R1 knows about
R2's LAN (172.16.2.0/24) and R2 knows about R1's networks.
The workshop uses RIPv2.

**On R1:**

```
R1(config)# router rip
R1(config-router)# version 2
R1(config-router)# network 10.0.0.0
R1(config-router)# no auto-summary
R1(config-router)# exit
```

**On R2:**

```
R2(config)# router rip
R2(config-router)# version 2
R2(config-router)# network 10.0.0.0
R2(config-router)# network 172.16.0.0
R2(config-router)# no auto-summary
R2(config-router)# exit
```

> **Why `no auto-summary`?** RIPv1 automatically summarizes
> networks at classful boundaries, which can cause routing
> problems. Version 2 supports classless routing, but
> auto-summary is still on by default. Turning it off ensures
> the exact subnet information is shared.

> **Why R2 has two network statements?** R2 sits between two
> networks: the 10.1.1.0/30 serial link and the 172.16.2.0/24
> LAN. It must advertise both so R1 learns how to reach the LAN.
> R1 only touches the 10.0.0.0 network.

---

## Step 9: Save Your Configuration

Before testing, save the running configuration on both routers
so your work survives a reload or CPT crash:

**On R1:**

```
R1(config)# end
R1# copy running-config startup-config
```

**On R2:**

```
R2(config)# end
R2# copy running-config startup-config
```

When prompted for the destination filename, press **Enter** to
accept the default.

---

## Step 10: Verify Connectivity

**Test 1 — R1 to R2 (serial link):**

```
R1# ping 10.1.1.2
```

Expected: **Success.** If this fails, check your serial
interface configurations and cable connections.

**Test 2 — R2 to PC-B (LAN):**

```
R2# ping 172.16.2.3
```

Expected: **Success.** If this fails, check R2's G0/0
configuration and PC-B's IP settings.

**Test 3 — PC-B to R1 (end-to-end through RIP):**

This is the verification the workshop asks for: *"Verify that
the network is functioning by pinging from the PC to S0/0/0
port on router R1."*

On PC-B, open **Command Prompt** (Desktop → Command Prompt):

```
C:\> ping 10.1.1.1
```

Expected: **Success.** This proves:
- PC-B can reach its default gateway (R2)
- R2 knows how to forward to 10.1.1.0/30 (directly connected)
- R1 knows how to reply back to 172.16.2.0/24 (learned via RIP)

If this fails but Tests 1 and 2 passed, the problem is RIP.
Run `show ip route` on R1 and check that you see a route
to 172.16.2.0:

```
R1# show ip route
```

You should see something like:

```
R    172.16.2.0/24 [120/1] via 10.1.1.2
```

The **R** means it was learned via RIP. If this line is missing,
re-check your RIP configuration on both routers.

---

## Step 11: Check the Clocks

The workshop says: *"On R1 and R2 issue the show clock command
to see the difference in them."*

**On R1:**

```
R1# show clock
```

**On R2:**

```
R2# show clock
```

You will likely see something like this on each router:

```
*00:04:32.123 UTC Mon Mar 1 1993
```

Note the asterisk (*) at the start — this means the clock has
never been set and is not considered authoritative. The date
will show 1993 (the Cisco default), not today's actual date.

> **In Packet Tracer:** The times on R1 and R2 may be very close
> or even identical, since both routers start at the same moment
> in the simulation. On real hardware, the difference would be
> more obvious because routers boot at different times. Either
> way, the key observation is that these clocks have never been
> set to the correct real-world time.

> **This is exactly the problem from Part 0.** If these two
> routers were sending syslog messages right now, the timestamps
> would not match. You could not tell which event happened first.
>
> Part 2 fixes this with NTP.

---

## Part 1 Checklist

Before moving on, confirm all of the following:

- [ ] R1 and R2 are cabled with a serial DCE connection
- [ ] R2 and PC-B are connected through a switch
- [ ] All IP addresses match the addressing table
- [ ] Clock rate 128000 is set on R1's S0/0/0
- [ ] RIPv2 is running on both routers
- [ ] Configuration saved on both routers (`copy running-config startup-config`)
- [ ] PC-B can successfully ping 10.1.1.1 (R1)
- [ ] `show clock` run on both routers — both show unset clocks (year 1993)

Everything green? Proceed to Part 2.
