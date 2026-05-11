# Workshop 4 — NTP & Syslog Logging: Part 2 — NTP

## What We Are Doing

In Part 1, you saw that both routers have unset clocks showing
the wrong date (1993). In this part, you will:

1. Set the correct time and date on R1
2. Make R1 the NTP master (the time authority for the network)
3. Make R2 sync its clock to R1 automatically
4. Verify that both routers show the same time

After this part, every device on the network will agree on
what time it is — the foundation that makes syslog useful.

```
                    NTP sync
          ┌──────────────────────┐
          │                      │
          ▼                      │
    ┌───────────┐  Serial   ┌────┴──────┐
    │    R1     │───────────│    R2     │
    │ NTP Master│  10.1.1.x │ NTP Client│
    │ Stratum 5 │           │ Stratum 6 │
    └───────────┘           └───────────┘

    R1 is the source of truth.
    R2 asks R1 "what time is it?" and adjusts its own clock.
```

---

## Step 1: Set the Clock on R1

The workshop says: *"Using a search engine find and then issue
a command to set the current time and date on R1."*

Here is the command. It must be run from **privileged exec mode**
(the `#` prompt), not from config mode:

```
R1# clock set 14:30:00 23 Feb 2026
```

The format is: `clock set HH:MM:SS DD Month YYYY`

> **Use today's actual date and time** when you run this command.
> The example above is just a sample — replace it with the real
> current time.

Verify immediately:

```
R1# show clock
```

You should now see something like:

```
14:30:05.123 UTC Mon Feb 23 2026
```

Notice two changes from before:
- The asterisk (*) is **gone** — the clock is now considered set
- The date shows today's real date, not 1993

> **Why set the clock manually?** In a real network, R1 would
> sync to a public NTP server on the internet (like
> pool.ntp.org). In this lab, there is no internet access, so
> we set the time by hand and then tell R1 to act as though it
> is a reliable time source.

---

## Step 2: Configure R1 as the NTP Master

Now tell R1: "You are the authoritative time source for this
network, at stratum level 5."

```
R1# configure terminal
R1(config)# ntp master 5
R1(config)# end
```

The `ntp master 5` command does two things:
- R1 will now **respond to NTP requests** from other devices
- R1 advertises itself as a **Stratum 5** time source

> **What does stratum 5 mean?** As explained in Part 0, stratum
> measures how many hops away a clock is from an atomic clock
> source. Stratum 1 is directly connected to an atomic clock.
> We use 5 because R1 is not actually connected to any real time
> source — it is just using the time we typed in manually.
> Setting it to 1 would falsely claim atomic-clock accuracy.

---

## Step 3: Configure R2 as the NTP Client

Now tell R2: "Your time source is R1, at address 10.1.1.1.
Check with that server periodically to stay in sync."

```
R2# configure terminal
R2(config)# ntp server 10.1.1.1
R2(config)# ntp update-calendar
R2(config)# end
```

What each command does:

- `ntp server 10.1.1.1` — tells R2 to synchronize its clock
  with the device at 10.1.1.1 (which is R1)
- `ntp update-calendar` — tells R2 to periodically update its
  hardware calendar from the NTP time. Without this, only the
  software clock syncs; the hardware clock (which persists
  across reboots) stays unchanged

> **Packet Tracer note:** If R2 rejects the `ntp update-calendar`
> command with an error, skip it — some CPT versions do not
> support this command. NTP will still synchronize the software
> clock, which is what matters for this lab.

> **Why 10.1.1.1?** That is R1's serial interface address — the
> only address R2 can reach R1 on. You must use an IP that is
> reachable, not just any IP belonging to R1.

---

## Step 4: Wait for Synchronization

NTP does not sync instantly. R2 needs to exchange several
packets with R1 before it trusts the time source. This can
take **1 to 5 minutes** in Packet Tracer.

> **Tip:** While waiting, you can speed up time in Packet Tracer.
> In the bottom-right corner, there is a speed slider or
> **Fast Forward** button. Use it to accelerate the simulation,
> then switch back to Realtime when you are done waiting.

You can monitor the progress with:

```
R2# show ntp associations
```

Look for this output:

```
  address         ref clock     st  when  poll  reach  delay  offset   disp
*~10.1.1.1        127.127.1.1   5   32    64    377    2.00   0.00     0.12
```

Key things to check:

- The `*` at the start means R2 has **selected** R1 as its time
  source. If you see a space or a `~` without `*`, it has not
  synced yet — wait longer.
- `st` column shows **5** — that is R1's stratum level
- `reach` should be **377** (meaning all recent polls succeeded).
  If it shows a lower number (like 1, 3, 7), NTP is still
  establishing contact.

> **Note:** Your Packet Tracer output may look slightly different
> from the example above — column widths and spacing can vary
> between CPT versions. Focus on finding the `*` symbol and the
> stratum number, not on matching the exact formatting.

Also run:

```
R2# show ntp status
```

Look for:

```
Clock is synchronized, stratum 6, reference is 10.1.1.1
```

This confirms:
- R2's clock is **synchronized**
- R2 is now **Stratum 6** (one level below R1's Stratum 5)
- R2's time reference is **10.1.1.1** (R1)

> **If `show ntp status` says "Clock is unsynchronized":**
> NTP has not finished yet. Wait another 2–3 minutes and try
> again. Do not proceed to Step 5 until you see "synchronized."

---

## Step 5: Verify Both Clocks Match

The workshop says: *"Now issue the show clock commands on both
routers and the times should be the same."*

**On R1:**

```
R1# show clock
```

**On R2:**

```
R2# show clock
```

Both should now show the same time (within a second or two)
and the same date. R2's asterisk should be gone — its clock
is now set via NTP.

```
R1:  14:35:22.456 UTC Mon Feb 23 2026
R2:  14:35:23.789 UTC Mon Feb 23 2026
```

> **The example above is for illustration only.** The actual
> output will not include the "R1:" or "R2:" prefix — those are
> labels added here so you can see both outputs side by side.
> In Packet Tracer, you will see each output separately in each
> router's CLI window.

If the times match — **NTP is working.** Every log message
generated from this point forward will carry a reliable,
synchronized timestamp.

---

## Step 6: Save the Configuration

Save on both routers so the NTP settings persist:

**On R1:**

```
R1# copy running-config startup-config
```

**On R2:**

```
R2# copy running-config startup-config
```

---

## What Just Happened — The Big Picture

```
 BEFORE (Part 1):                    AFTER (Part 2):

 R1 clock: *00:04:32 (1993)          R1 clock: 14:35:22 (today)
 R2 clock: *00:04:30 (1993)          R2 clock: 14:35:23 (today)

 ● Clocks unset                      ● R1 is the NTP master
 ● No time authority                 ● R2 syncs automatically
 ● Timestamps meaningless            ● Timestamps trustworthy
```

Now both routers agree on the time. In Part 3, we will set up
syslog so that log messages from both routers flow to a central
server — and thanks to NTP, those messages will have accurate,
synchronized timestamps.

---

## Part 2 Checklist

Before moving on, confirm all of the following:

- [ ] R1's clock shows today's correct date and time
- [ ] R1 is configured as NTP master stratum 5
- [ ] R2 has `ntp server 10.1.1.1` and `ntp update-calendar`
- [ ] `show ntp status` on R2 says "Clock is synchronized, stratum 6"
- [ ] `show clock` on both routers shows matching times
- [ ] Configuration saved on both routers

**Demonstrate to your tutor.** This is the NTP checkpoint.

Everything synced? Proceed to Part 3.
