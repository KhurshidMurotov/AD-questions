# Workshop 4 — NTP & Syslog Logging: Part 0 — Why This Matters

## The Problem

Your network is running. VLANs are separating traffic. Tunnels connect
remote offices. Routing protocols are sharing routes. Everything works.

Then on Monday morning, the Finance department can't reach the database
server. By the time you walk to the server room and check, it's working
again. Your boss asks: "What happened?"

You have no answer. You weren't watching. The network doesn't keep a
diary — it just does its job and forgets.

Now imagine something worse. A security breach. Someone accessed a router
at 3am and changed the configuration. When did it happen? From where?
What did they change? You don't know, because nobody was recording.

This workshop gives you two tools that solve this:

```
 SYSLOG  ──── What happened?
 NTP     ──── When exactly did it happen?
```

---

## Tool 1: Syslog — The Network's Diary

Every Cisco device already generates messages when things happen:

```
 ● Interface goes up              → message generated
 ● Interface goes down            → message generated
 ● Someone fails a login          → message generated
 ● OSPF loses a neighbor          → message generated
 ● Configuration changes          → message generated
```

These messages scroll across the console screen if you are watching.
If nobody is watching, they disappear forever.

**Syslog** is a standard protocol (UDP port 514) that lets you redirect
all those messages to a central server — one machine that collects logs
from every device on the network.

```
    ┌──────┐         ┌──────┐         ┌──────┐
    │  R1  │         │  R2  │         │  S1  │
    └──┬───┘         └──┬───┘         └──┬───┘
       │                │                │
       │  log messages  │  log messages  │  log messages
       │                │                │
       ▼                ▼                ▼
    ┌──────────────────────────────────────┐
    │         SYSLOG SERVER (PC-B)         │
    │                                      │
    │  14:32:05  R1  Interface G0/0 down   │
    │  14:32:07  R2  OSPF neighbor lost    │
    │  14:32:09  S1  Port Fa0/1 down       │
    │  14:35:22  R1  Interface G0/0 up     │
    │  14:35:24  R2  OSPF neighbor found   │
    │  ...                                 │
    └──────────────────────────────────────┘
```

Instead of checking each device one by one, you look at one screen
and see everything that happened across the entire network.

---

## Severity Levels — Not All Messages Are Equal

A router telling you "interface went down" is urgent.
A router telling you "debug: ARP packet received" is noise.

Syslog defines **8 severity levels** to separate the serious from
the routine:

```
 Level │ Name          │ Meaning                   │ Example
 ══════╪═══════════════╪═══════════════════════════╪════════════════════════════
   0   │ Emergency     │ System is unusable        │ Router hardware failure
   1   │ Alert         │ Act immediately           │ Temperature critical
   2   │ Critical      │ Critical condition        │ Memory allocation failed
   3   │ Error         │ Error occurred            │ Interface error, link down
   4   │ Warning       │ Warning condition         │ Config change unsaved
   5   │ Notification  │ Normal but significant    │ Interface up/down toggle
   6   │ Informational │ General information       │ ACL match logged
   7   │ Debugging     │ Maximum detail            │ Every packet, every step
```

**The lower the number, the more serious the problem.**

When you configure syslog, you set a **trap level** — the threshold
for what gets sent to the server. Setting `logging trap warning`
(level 4) means: "Send me everything from level 0 to 4. Ignore
levels 5, 6, and 7."

```
 logging trap debugging  →  sends levels 0-7  →  EVERYTHING (noisy)
 logging trap warning    →  sends levels 0-4  →  problems only (quiet)
 logging trap error      →  sends levels 0-3  →  serious problems only
```

The tradeoff:

- Set it too high (debugging) → thousands of messages per hour, hard to find the important ones in the flood.

- Set it too low (error) → clean log, but you miss early warning signs that could have prevented a bigger failure.

---

## Tool 2: NTP — Making the Diary Trustworthy

Your syslog server is now collecting messages from R1, R2, and a
switch. You see this:

```
 R1:     Interface went down at  14:32:05
 R2:     Lost OSPF neighbor at   14:29:17
 Switch: Port flapped at         14:35:42
```

Which happened first? Did R1's failure cause R2's OSPF loss?
Or was it the other way around?

**You cannot tell.** Each device has its own internal clock, and
they drift independently. R1 might be 3 minutes ahead. R2 might
be 2 minutes behind. The timestamps are meaningless for
reconstructing a sequence of events.

**NTP (Network Time Protocol)** solves this by synchronizing every
device's clock to one authoritative source.

```
 BEFORE NTP:                          AFTER NTP:

 R1 clock: 14:32:05                   R1 clock: 14:32:05
 R2 clock: 14:29:17  ← 3 min off!    R2 clock: 14:32:07  ← 2 sec after R1
 S1 clock: 14:35:42  ← wrong!        S1 clock: 14:32:09  ← 4 sec after R1

 "Which came first??"                 "R1 went down first,
                                       then R2 lost OSPF,
                                       then S1 port flapped.
                                       Clear chain of events."
```

---

## Stratum — How Trustworthy Is Your Clock?

NTP uses a hierarchy called **stratum** to measure how far a clock
is from the original time source:

```
 ┌─────────────────────────────┐
 │  ATOMIC CLOCK / GPS         │  ← Stratum 0  (the ultimate source)
 └─────────────┬───────────────┘
               │
 ┌─────────────▼───────────────┐
 │  NTP Server directly        │  ← Stratum 1  (connected to atomic clock)
 │  connected to Stratum 0     │
 └─────────────┬───────────────┘
               │
 ┌─────────────▼───────────────┐
 │  Public NTP Server          │  ← Stratum 2  (syncs from Stratum 1)
 │  (e.g. pool.ntp.org)        │
 └─────────────┬───────────────┘
               │
 ┌─────────────▼───────────────┐
 │  Your company's NTP server  │  ← Stratum 3  (syncs from Stratum 2)
 └─────────────┬───────────────┘
               │
 ┌─────────────▼───────────────┐
 │  Your routers and switches  │  ← Stratum 4  (sync from your server)
 └─────────────────────────────┘
```

Each hop from the atomic clock adds 1 to the stratum number.
**Lower stratum = more accurate time.**

In this workshop, we set R1 as `ntp master 5` — this tells R1:
"Pretend you are a Stratum 5 time source." R2 then syncs to R1
and becomes Stratum 6. In a real production network, R1 would
point to an actual internet NTP server instead of being its own
master.

---

## How They Work Together

```
         NTP alone                     Syslog alone
    ┌──────────────────┐          ┌──────────────────┐
    │ All clocks agree │          │ Messages arrive  │
    │ but nothing is   │          │ but timestamps   │
    │ being recorded   │          │ don't match      │
    └──────────────────┘          └──────────────────┘
             │                             │
             └─────────────┬───────────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │   NTP + Syslog   │
                  │                  │
                  │  Accurate time   │
                  │  on all devices  │
                  │       +          │
                  │  All messages    │
                  │  collected in    │
                  │  one place       │
                  │       =          │
                  │  A reliable,     │
                  │  chronological   │
                  │  record of your  │
                  │  entire network  │
                  └──────────────────┘
```

---

## What You Will Build in This Workshop

```
                    NTP sync
          ┌──────────────────────┐
          │                      │
          ▼                      │
    ┌───────────┐  Serial   ┌────┴──────┐      ┌──────────┐
    │    R1     │───────────│    R2     │──────│   PC-B   │
    │ NTP Master│  10.1.1.x │ NTP Client│ G0/0 │  Syslog  │
    │ Stratum 5 │           │ Stratum 6 │      │  Server  │
    └───────────┘           └───────────┘      └──────────┘
          │                      │                   ▲
          │     log messages     │    log messages   │
          └──────────────────────┴───────────────────┘
```

- **Part 1:** Build the topology and configure basic connectivity
- **Part 2:** Set up NTP so both routers agree on the time
- **Part 3:** Set up Syslog so both routers send logs to PC-B
- **Part 4:** Generate events and watch them appear on the server

Now proceed to Part 1.
