# Workshop 4 — NTP & Syslog Logging: Part 3 — Syslog

## What We Are Doing

NTP is running. Both routers agree on the time. Now we set up
the second half — syslog — so that log messages from both
routers are collected on a central server (PC-B).

By the end of this part:
- Both routers will timestamp every log message with date, time,
  and milliseconds
- Both routers will send their log messages to PC-B
- PC-B will be running a syslog server, ready to receive them

```
                                                          172.16.2.0/24
   ┌────────┐  Serial   ┌────────┐  G0/0  ┌────┐  ┌──────────┐
   │   R1   │───────────│   R2   │────────│ S1 │──│   PC-B   │
   └────────┘           └────────┘        └────┘  │  Syslog  │
       │                    │                     │  Server  │
       │   log messages     │    log messages     │ (UDP 514)│
       └────────────────────┴─────────────────────▶──────────┘
```

---

## Step 1: Set Up the Syslog Server on PC-B

The original worksheet uses **Tftpd32** software on a Windows
VM. In Packet Tracer, we use the **built-in Syslog service**
instead — but there is an important difference.

> **Packet Tracer requires a Server device, not a regular PC.**
> A regular PC in CPT has no syslog viewer. To see incoming log
> messages, you need a **Server** device which has a built-in
> Syslog service under its Services tab.
>
> If you placed a PC in Part 1, replace it now:
> 1. Delete the existing PC-B
> 2. Click **End Devices** → drag a **Server** onto the workspace
> 3. Label it **PC-B** (keeping the original name)
> 4. Connect it to the switch with a straight-through cable
> 5. Set its IP to **172.16.2.3**, mask **255.255.255.0**,
>    gateway **172.16.2.1** (same as before)
> 6. Verify connectivity: ping 10.1.1.1 from PC-B's command prompt

Now enable the Syslog service:

1. Click on **PC-B** (the Server device)
2. Go to the **Services** tab
3. Click **Syslog** in the left menu
4. Make sure the service is set to **On**

The syslog server is now listening on UDP port 514, ready to
receive messages from the routers.

> **If using physical equipment with Tftpd32:** Download Tftpd32
> from Canvas, install it on the Windows VM, run it, and set the
> "Syslog server" interface to 172.16.2.3 (PC-B's NIC address).
> The rest of the commands below are the same.

---

## Step 2: Configure Timestamps on Both Routers

By default, Cisco log messages look like this:

```
%LINK-5-CHANGED: Interface Loopback0, changed state to up
```

No date. No time. Useless for troubleshooting.

We need to add timestamps. The workshop says: *"Configure R1
and R2 such that the messages it generates are timestamped with
date time and msec."*

**On R1:**

```
R1# configure terminal
R1(config)# service timestamps log datetime msec
R1(config)# exit
```

**On R2:**

```
R2# configure terminal
R2(config)# service timestamps log datetime msec
R2(config)# exit
```

Now log messages will look like this instead:

```
*Feb 23 14:42:15.327: %LINK-5-CHANGED: Interface Loopback0, changed state to up
```

That timestamp — `Feb 23 14:42:15.327` — is what makes the log
readable. The `.327` is the millisecond precision, which helps
when events happen in rapid succession.

> **Breaking down the command:**
> - `service timestamps` — enable timestamping
> - `log` — apply to log messages (not debug messages)
> - `datetime` — include the full date and time
> - `msec` — include millisecond precision
>
> Without `msec`, timestamps only show seconds. With it, you get
> three extra digits of precision — important when multiple events
> happen within the same second.

> **Why not enter config mode once and do everything?** Steps 2,
> 3, and 4 each enter and exit config mode separately. You could
> combine all three commands into a single config session — that
> works fine. We keep them separate here so each command gets its
> own explanation, but if you prefer efficiency:
>
> ```
> R1(config)# service timestamps log datetime msec
> R1(config)# logging host 172.16.2.3
> R1(config)# logging trap debugging
> ```

---

## Step 3: Direct Logging to the Syslog Server

Now tell both routers: "Send your log messages to PC-B at
172.16.2.3."

**On R1:**

```
R1# configure terminal
R1(config)# logging host 172.16.2.3
R1(config)# exit
```

**On R2:**

```
R2# configure terminal
R2(config)# logging host 172.16.2.3
R2(config)# exit
```

> **What happens behind the scenes:** When you issue
> `logging host 172.16.2.3`, the router starts sending copies
> of every log message as a UDP packet to port 514 on that IP
> address. The messages still appear on the console — the router
> just sends a copy to the server as well.

> **Why 172.16.2.3?** That is PC-B's IP address on R2's LAN.
> Both routers can reach it — R2 directly, and R1 through the
> serial link and RIPv2 routing (set up in Part 1).

---

## Step 4: Set the Trap Level

The workshop says: *"Set the level to debugging."*

This controls **which severity levels** get forwarded to the
syslog server. As covered in Part 0, the 8 levels range from
0 (Emergency) to 7 (Debugging).

**On R1:**

```
R1# configure terminal
R1(config)# logging trap debugging
R1(config)# exit
```

**On R2:**

```
R2# configure terminal
R2(config)# logging trap debugging
R2(config)# exit
```

Setting the trap to `debugging` (level 7) means: "Send
**everything** — all severity levels from 0 through 7."

> **How the trap level works:** The number you set is the
> threshold. The router sends every message **at that level or
> more severe** (lower number). Setting `debugging` (7) captures
> all 8 levels. Setting `warning` (4) would capture only levels
> 0–4 and discard 5, 6, and 7.

> **Why debugging for this lab?** We want to see every possible
> message so we can verify that the syslog system is working.
> In a production network, you would typically set this to
> `informational` (level 6) or `warning` (level 4) to reduce
> noise. The research questions in Part 5 ask you to think about
> this tradeoff.

---

## Step 5: Verify the Logging Configuration

Run `show logging` on both routers to confirm everything is
set up correctly.

**On R1:**

```
R1# show logging
```

**On R2:**

```
R2# show logging
```

You should see output that includes lines similar to:

```
Syslog logging: enabled (0 messages dropped, 0 messages rate-limited,
                0 flushes, 0 overflows, xml disabled, filtering disabled)

No Active Message Discriminator.

No Inactive Message Discriminator.

    Console logging: level debugging, 0 messages logged, xml disabled,
                     filtering disabled
    Monitor logging: level debugging, 0 messages logged, xml disabled,
                     filtering disabled
    Buffer logging:  level debugging, 0 messages logged, xml disabled,
                     filtering disabled
    Logging Exception size (4096 bytes)
    Count and timestamp logging messages: disabled
    Persistent logging: disabled

No active filter modules.

    Trap logging: level debugging, 0 message lines logged
        Logging to 172.16.2.3  (udp port 514, audit disabled,
              link up),
              0 message lines logged,
              0 message lines rate-limited,
              0 message lines dropped-by-MD,
              xml disabled, sequence number disabled
              filtering disabled
```

The key lines to confirm:

- **Trap logging: level debugging** — confirms your trap level
- **Logging to 172.16.2.3** — confirms the syslog server target
- **udp port 514** — confirms the standard syslog port

> **In Packet Tracer:** The output may be shorter or formatted
> differently. Look specifically for the trap level and the
> logging host address. If you see both, syslog is configured.

> **If `show logging` does not show 172.16.2.3:** Re-enter the
> `logging host 172.16.2.3` command. Make sure you are in global
> config mode, not interface config mode.

---

## Step 6: Save the Configuration

Save on both routers:

**On R1:**

```
R1# copy running-config startup-config
```

**On R2:**

```
R2# copy running-config startup-config
```

---

## Summary of Commands Entered

Here is everything configured in this part, for quick reference:

```
 Command                              │ What It Does
 ═════════════════════════════════════╪═══════════════════════════════════
 service timestamps log datetime msec │ Adds date/time/ms to log messages
 logging host 172.16.2.3              │ Sends logs to the syslog server
 logging trap debugging               │ Sends all severity levels (0–7)
```

---

## Part 3 Checklist

Before moving on, confirm all of the following:

- [ ] `service timestamps log datetime msec` configured on R1 and R2
- [ ] `logging host 172.16.2.3` configured on R1 and R2
- [ ] `logging trap debugging` configured on R1 and R2
- [ ] `show logging` on both routers confirms trap level and host address
- [ ] Configuration saved on both routers

No messages will appear in the syslog server yet — we have not
generated any events. Part 4 fixes that.

Everything configured? Proceed to Part 4.
