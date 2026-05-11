# Workshop 4 — NTP & Syslog Logging: Part 4 — Generate & Analyze

## What We Are Doing

NTP is running (Part 2). Syslog is configured (Part 3). But the
syslog server log is still empty — nothing interesting has
happened on the network yet.

In this part, you will deliberately create network events by
toggling loopback interfaces on and off, then read and analyze
the log messages that arrive at the syslog server.

---

## Step 1: Create Loopback Interfaces

A loopback interface is a virtual interface inside the router.
It does not connect to any physical cable — it exists only in
software. Loopback interfaces are useful for testing because
you can bring them up and down without affecting real traffic.

**On R1:**

```
R1# configure terminal
R1(config)# interface Loopback0
R1(config-if)# ip address 1.1.1.1 255.255.255.0
R1(config-if)# exit
R1(config)# interface Loopback1
R1(config-if)# ip address 1.1.2.1 255.255.255.0
R1(config-if)# exit
R1(config)# end
```

**On R2:**

```
R2# configure terminal
R2(config)# interface Loopback0
R2(config-if)# ip address 2.2.1.1 255.255.255.0
R2(config-if)# exit
R2(config)# interface Loopback1
R2(config-if)# ip address 2.2.2.1 255.255.255.0
R2(config-if)# exit
R2(config)# end
```

> **Why two loopbacks on each router?** More interfaces means
> more events when we toggle them — more log messages to analyze.
> The IP addresses do not matter for this exercise — they just
> need to be valid and on different subnets so Cisco does not
> reject them with an overlap error.

When you create a loopback, it comes up automatically. You
should already see messages on the router console:

```
*Feb 23 14:50:03.112: %LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback0, changed state to up
```

This is your first syslog message. It should also appear on
PC-B's syslog server.

---

## Step 2: Generate Events — Toggle Interfaces

Now create a series of up/down events. Enter interface config
mode and use `shutdown` to bring an interface down, then
`no shutdown` to bring it back up.

> **Tip:** Wait a few seconds between `shutdown` and
> `no shutdown` so the timestamps are visibly different.
> This makes analysis easier in Step 3.

**On R1:**

```
R1# configure terminal
R1(config)# interface Loopback0
R1(config-if)# shutdown
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface Loopback1
R1(config-if)# shutdown
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# end
```

**On R2:**

```
R2# configure terminal
R2(config)# interface Loopback0
R2(config-if)# shutdown
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface Loopback1
R2(config-if)# shutdown
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# end
```

Each `shutdown` generates a "down" message. Each `no shutdown`
generates an "up" message. In practice, each event produces
**two** log entries (one for the link layer, one for the line
protocol), so you should see **16 or more messages** on the
syslog server across both routers. Do not worry if the count
is higher than expected — that is normal.

---

## Step 3: Examine the Syslog Server Log

Now check PC-B to see the collected messages.

1. Click on **PC-B** (the Server device)
2. Go to the **Services** tab
3. Click **Syslog** in the left menu
4. You should see a list of log messages

The messages will look similar to this:

```
 Seq │ Time                  │ Host       │ Message
 ════╪═══════════════════════╪════════════╪══════════════════════════════════════
  1  │ Feb 23 14:50:03.112   │ 10.1.1.1   │ %LINK-3-UPDOWN: Interface Loopback0
     │                       │            │ changed state to up
  2  │ Feb 23 14:50:04.556   │ 10.1.1.1   │ %LINEPROTO-5-UPDOWN: Line protocol
     │                       │            │ on Interface Loopback0 changed to up
  3  │ Feb 23 14:52:10.891   │ 10.1.1.1   │ %LINK-5-CHANGED: Interface Loopback0
     │                       │            │ changed state to administratively down
  4  │ Feb 23 14:52:15.223   │ 10.1.1.1   │ %LINK-3-UPDOWN: Interface Loopback0
     │                       │            │ changed state to up
  5  │ Feb 23 14:52:17.445   │ 172.16.2.1 │ %LINK-5-CHANGED: Interface Loopback0
     │                       │            │ changed state to administratively down
  6  │ Feb 23 14:52:22.667   │ 172.16.2.1 │ %LINK-3-UPDOWN: Interface Loopback0
     │                       │            │ changed state to up
```

> **In Packet Tracer:** The syslog display format in CPT may
> differ from the table above. CPT typically shows a simpler
> list with the device IP, timestamp, and message text. The
> important thing is that you can see messages from both routers.
>
> **Why does R1 show 10.1.1.1 but R2 shows 172.16.2.1?** The
> Host column shows the source IP of the syslog packet. R1's
> only interface is its serial port (10.1.1.1), so that is the
> source. R2 reaches PC-B directly through G0/0 (172.16.2.1),
> so it uses that address instead of its serial IP.
>
> **If the syslog log is empty:** Check that PC-B can still
> ping both routers. Then verify `show logging` on R1 and R2
> shows `Logging to 172.16.2.3` with a non-zero message count.

---

## Step 4: Read a Syslog Message

Every Cisco syslog message follows the same structure. Take
this example:

```
*Feb 23 14:52:10.891: %LINK-5-CHANGED: Interface Loopback0, changed state to administratively down
```

Here is how to break it apart:

```
 Part             │ Value                       │ Meaning
 ═════════════════╪═════════════════════════════╪═════════════════════════
 Timestamp        │ Feb 23 14:52:10.891         │ When (date, time, ms)
 Facility         │ LINK                        │ What subsystem generated it
 Severity         │ 5                           │ Notification level
 Mnemonic         │ CHANGED                     │ Short name for this event
 Description      │ Interface Loopback0,        │ What happened
                  │ changed state to admin down │
```

> **The severity number is the key.** It tells you how serious
> the event is. Refer back to Part 0 for the full severity table
> (0 = Emergency through 7 = Debugging).

---

## Step 5: Analyze Your Log — Identify Patterns

Look at the messages on PC-B's syslog and answer these
questions for yourself (not for submission — just to build
understanding):

**For each message, identify:**
- What is the timestamp?
- What is the severity level? (the number after the facility name)
- Which router sent it? (look at the Host/IP column)
- What interface was affected?
- Did the interface go up or down?

**Then look at the whole log:**
- Can you tell which events happened on R1 vs R2?
- Are the timestamps in chronological order?
- If you shutdown Loopback0 on R1 at 14:52:10, and the syslog
  shows R1's message at 14:52:10 — does this confirm NTP is
  doing its job?

> **This is the payoff.** In Part 0, we described the scenario:
> "Finance can't reach the server Monday morning — what
> happened?" With NTP and syslog in place, you could now look at
> the centralized log, find the exact time a link went down,
> identify which device reported it, and trace the sequence of
> events. That is exactly what you just did with loopback
> interfaces — the same principle applies to real network
> failures.

---

## Step 6: Save and Demonstrate

Save the configuration on both routers:

**On R1:**

```
R1# copy running-config startup-config
```

**On R2:**

```
R2# copy running-config startup-config
```

**Demonstrate to your tutor.** Show:
1. The syslog server on PC-B with collected log messages
2. `show logging` on both routers confirming the configuration
3. `show clock` on both routers confirming NTP synchronization

This is the final demonstration checkpoint for the lab.

---

## Part 4 Checklist

Before moving on, confirm all of the following:

- [ ] Loopback interfaces created on both routers
- [ ] Interfaces toggled with shutdown/no shutdown
- [ ] Syslog server on PC-B shows log messages from both routers
- [ ] You can identify the timestamp, severity, facility,
      and source in each message
- [ ] Configuration saved on both routers
- [ ] Demonstrated to your tutor

Everything logged? Proceed to Part 5 (Research Questions).
