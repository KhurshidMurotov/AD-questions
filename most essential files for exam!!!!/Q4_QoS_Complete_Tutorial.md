# Q4 — Quality of Service: Complete Exam Tutorial

**Module:** 6CS029 Advanced Networks
**Topic:** Quality of Service — written exam preparation
**Source materials:** Lecture 4 (QoS Concepts, 38 slides) + Lecture 2 (Network Design, 43 slides)
**Tutorial length:** Six parts, designed to be read in order or jumped into for revision.

---

## Table of Contents

```
PART 1 — Understanding Q4
  Part 1: Reading the Question (anatomy + time budget)
  Part 2: The QoS Problem (queuing, parameters, delay, loss)
  Part 3: Traffic Characteristics + Queuing Algorithms

PART 2 — Answering Q4
  Part 4: Writing the 80-Mark Theory Answer
  Part 5: Bandwidth Formula Deep Dive
  Part 6: Mock Practice Table — Full Solution
```

---

# Part 1: Reading the Question: The Anatomy of a QoS Exam Beast

## Why this question matters (a real-world story)

Picture this. You graduate. Your first job is at a Tashkent **ISP** (**I**nternet **S**ervice **P**rovider — the company that sells the internet connection to homes and businesses) — Uztelecom, Beeline, Ucell, Mobiuz, pick one. Three months in, your team lead drops a problem on your desk:

> "A new sales office is opening with 100 hard phones, 25 soft phones, 10 video-call agents, and 8 field reps on mobile WiFi. We need to size the **WAN** (**W**ide **A**rea **N**etwork — the connection from their office to the wider internet) link they buy from us. How much bandwidth do they need?"

You can't guess. You can't sell them too little — calls will drop, video will stutter, the customer's business breaks and they leave you for a competitor. You can't sell them too much — the deal collapses on price and the same thing happens.

You need a formula. A defensible number on a piece of paper that says "this is exactly what your office needs, and here is how we got it."

That formula exists. You've already been taught it — on one slide buried inside Lecture 2 (Network Design, Slide 21). That formula is **half** of the QoS exam question.

The other half is the conversation that happens after the customer signs:

> "OK we bought the bandwidth, but our voice calls still sound choppy on busy mornings. Why?"

The answer to that question is **Q**uality **o**f **S**ervice (**QoS**) — the set of techniques used to make sure voice traffic gets priority over web browsing, video gets enough room to breathe, and email waits its turn. That's the other half of the exam question.

So a QoS exam question is a small, paid version of the real job you're being trained for. Worth taking seriously.

---

## The shape of the question

QoS exam questions on this module follow a very consistent pattern. They come in two parts that always sum to **100 marks**:

```
┌─────────────────────────────────────────────────────────┐
│  PART (a) — The Theory Question         ~80 marks       │
│  ──────────────────────────────────                     │
│  Describe, with diagrams, the problems QoS exists to    │
│  solve, and the queuing mechanisms used to solve them.  │
│                                                          │
│  Long-form answer. Multiple sub-topics expected.        │
│  Diagrams are required, not optional.                   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  PART (b) — The Calculation              ~20 marks      │
│  ────────────────────────────                           │
│  A table of endpoint types is given, with counts,       │
│  headroom %, simultaneous-use %, and per-endpoint       │
│  bandwidth.                                              │
│                                                          │
│  Compute the total WAN bandwidth needed for the site.   │
└─────────────────────────────────────────────────────────┘

                  TOTAL: 100 marks
```

The 80/20 split tells you what the examiner cares about. **Four times more marks for theory than for calculation.** If you only have time to prepare one half thoroughly, prepare the theory.

But there's a catch. The calculation is far easier to score on. A correctly executed table calc is 18–20 marks every single time, no judgement involved. The theory is where students lose marks through omission — forgetting a topic, skipping a diagram, mixing up two queuing algorithms. So the right strategy is *never* to skip the easier 20-mark half.

---

## What does an 80-mark answer actually look like?

80 marks is a lot. At Level 6 mark density — roughly 1 mark per minute of focused writing — that's about **half an hour of writing** if you're attempting this whole question.

Thirty minutes of writing means **a lot of content**. The examiner does not expect a three-paragraph definition of QoS. They expect six distinct content blocks, each worth roughly 13 marks:

```
A properly built 80-mark QoS theory answer covers six blocks:

  1. The problem QoS exists to solve
     (traffic > capacity → queuing → delay → drops)

  2. The four parameters that define network quality
     (bandwidth, delay, jitter, packet loss)

  3. Where delay comes from
     (the six delay types: code, packetisation, queuing,
      serialisation, propagation, de-jitter)

  4. Why packet loss matters for voice and video
     (playout buffer, DSP interpolation, audio breaks
      when buffer can't keep up with arriving jitter)

  5. The queuing algorithms used to manage traffic
     (FIFO, WFQ, CBWFQ, LLQ — what each does, how they
      differ, when you would use each)

  6. At least one diagram
     (the three-priority-queues picture, or the LLQ
      strict-priority + CBWFQ-classes diagram)
```

Six content blocks × ~13 marks each = ~78 marks. Cover all six properly, even briefly, and you're nearly full marks. Skip one or two and you suddenly cap at 50 — and that's the difference between a first and a 2:2 on this question alone.

We'll build each of these six blocks in Parts 2 and 3, then walk through how to assemble them into a written answer in Part 4.

---

## What does a 20-mark calculation look like?

The 20-mark calc is mechanical. You're handed a table that looks like this:

```
┌──────────────────┬─────────┬──────────┬───────────┬──────────┐
│ Endpoint Type    │ Number  │ Headroom │ Sim. use  │ Mbps per │
│                  │   of    │   for    │  factor   │ endpoint │
│                  │ endpts  │  future  │           │   type   │
├──────────────────┼─────────┼──────────┼───────────┼──────────┤
│ Soft phone       │   30    │   30%    │    60%    │   0.1    │
│ Hard Phone       │  100    │   10%    │    50%    │   0.1    │
│ Two Party HD     │   10    │   20%    │    60%    │    2     │
│ Video call       │         │          │           │          │
│ Mobile Phone     │    8    │   10%    │    75%    │   0.1    │
│ on WiFi          │         │          │           │          │
└──────────────────┴─────────┴──────────┴───────────┴──────────┘
```

For each row, you apply this formula:

```
BW(row) = ( Endpoints × (1 + Headroom%) ) × Sim_Use% × Mbps_per_endpoint
```

Then you sum all rows. That's the total **Mb/s** (**M**ega**b**its per **s**econd — millions of bits transmitted per second) needed on the WAN link.

That's the whole thing. Four rows × one formula × one sum = 20 marks. Get the formula right, get the arithmetic right, write the units at the end, you're done.

We'll walk through this formula component by component in Part 5, then solve a full mock practice table end-to-end in Part 6.

---

## How to budget your time in the exam

The exam is **2 hours total** (including reading time). You answer **3 of 5 questions**. All five questions are equally weighted. That means:

```
  Total exam time:             120 minutes
  Reading time (suggested):    -10 minutes
                              ─────────────
  Writing time:                110 minutes
  Per question (110 ÷ 3):       ~36 minutes
```

If you choose to attempt this QoS question, your 36 minutes splits according to the 80/20 mark allocation:

```
  Theory part — 80 marks:      ~29 minutes
  Calculation — 20 minutes:    ~7 minutes
                               ───────────
                               ~36 minutes
```

That's tight, especially for the theory. Two habits make this manageable:

1. **Plan before writing.** Spend the first 2 of those 29 minutes pencil-listing your six content blocks down the side of the answer booklet. Then write each block in order. This stops you from writing yourself into a corner where you realise on minute 25 that you forgot to mention jitter.

2. **Do the calc on scrap paper first.** If scrap paper is provided, do all four row calculations there, then transcribe a clean table into your answer booklet with the sum line at the bottom. Doing it directly in the booklet leads to crossed-out arithmetic that costs presentation marks.

---

## Common mistakes that lose marks before you start

A few traps students fall into year after year:

1. **Treating Q4(a) as "explain QoS in your own words"** and writing 200 words that say nothing concrete. 200 words gets you 5 marks. The examiner wants the *named mechanisms* — **FIFO** (**F**irst **I**n, **F**irst **O**ut), **WFQ** (**W**eighted **F**air **Q**ueuing), **CBWFQ** (**C**lass-**B**ased **W**eighted **F**air **Q**ueuing), **LLQ** (**L**ow **L**atency **Q**ueuing). By name. With distinctions between them.

2. **Skipping the diagram in Q4(a).** The question explicitly asks for diagrams. No diagram = capped marks even if your prose is perfect. Draw at least one. The three-priority-queues diagram (Voice/Finance/Web feeding High/Medium/Low queues into a single output link) is the easiest to reproduce from memory.

3. **Confusing delay with jitter.** Delay is *how long* a packet takes to arrive. Jitter is the *variation* in delay from one packet to the next. They are different parameters with different units and different consequences. Mixing them up costs marks even in a one-line mention.

4. **Forgetting units in Q4(b).** The final answer is in megabits per second — Mbps, or Mb/s. Not megabytes (capital B). Not bits per second. Write "Mb/s" at the end of your total. Missing units = 1–2 marks gone for nothing.

5. **Adding the headroom % to the endpoint count as if it were a number, not a percentage.** If a row says 25 endpoints and 30% headroom, the count-with-headroom is `25 × 1.30 = 32.5`, not `25 + 30 = 55`. The headroom is a multiplier, not an addend. We'll hammer this in Parts 5 and 6.

6. **Doing the calc but forgetting to sum.** Each row is worth marks, but the **final total** is worth its own marks too. After all four rows are calculated, write one clean line: `Total = X + Y + Z + W = N.NNN Mb/s`. Don't make the examiner add for you.

---

## What's coming next

Parts 2 and 3 build the theory you need for Q4(a):

- **Part 2** — Why queuing exists in the first place, and the four parameters that QoS engineers actually measure: bandwidth, congestion, delay, jitter, and packet loss. The full delay-types table. Why voice and video care so much more about delay and jitter than email does. (Lecture 4 slides 4–9.)

- **Part 3** — The four queuing algorithms in order of increasing sophistication: FIFO (the dumbest), WFQ (fair-by-flow), CBWFQ (fair-by-user-defined-class), LLQ (CBWFQ plus a strict-priority lane for voice). One diagram per algorithm. By the end of Part 3 you can name and distinguish all four. (Lecture 4 slides 16–20.)

Parts 4 to 6 are how to actually answer:

- **Part 4** — Building the 80-mark answer from the six content blocks. Paragraph order. Which diagrams to draw. How to budget your 29 writing minutes minute by minute.

- **Part 5** — The bandwidth formula, broken open. Why each multiplier is there. Unit check. The mental model of "growth factor × utilisation factor × per-unit demand". (Lecture 2 slide 21.)

- **Part 6** — A mock practice table solved end-to-end, step by step, with the wrong answers students typically give and why those answers are wrong.

---

# Part 2: The QoS Problem: Queuing, the Four Parameters, and Why Things Break

## Opening picture — the Hadra junction at 5pm

Imagine the Hadra junction in central Tashkent at 5pm on a Friday. The road can carry roughly 1,000 cars per hour. At 4:55pm, 800 cars per hour are arriving — everything flows. Then 5pm hits and 1,500 cars per hour pile in. The road can still only carry 1,000.

The extra 500 cars per hour don't disappear. They form a line. The line causes everyone to wait longer than they should. If the line keeps growing past what the side roads can absorb, cars start parking on the shoulder and giving up.

Now zoom in. Some of those cars are ambulances. Some are delivery vans rushing perishable food. Most are commuters going home. Without traffic management, they all wait the same — the ambulance behind a Damas full of grocery bags. With traffic management — sirens, dedicated bus lanes, traffic-police signals — some vehicles get priority and reach their destination on time.

That's the entire story of **QoS** (**Q**uality **o**f **S**ervice — the set of techniques routers and switches use to give different traffic types different treatment when the network gets busy). Cars are packets. The road is your **WAN** (**W**ide **A**rea **N**etwork — the long-distance link from your office to the wider internet) link. The ambulance is a voice call. The Damas is an email attachment. The shoulder full of abandoned cars is your packet-loss statistics.

The rest of this part explains exactly how networking people measure all of this.

---

## Slide 4 — The Purpose of QoS

> **What's on the slide:** Title "The Purpose of QoS". Three bullet points. No images.

The three bullets describe a chain of cause and effect that you must reproduce in any 80-mark answer:

**Bullet 1 — When traffic volume is greater than what can be transported across the network, devices queue (hold) the packets in memory until resources become available to transmit them.**

This is the foundational statement. A router or switch can only transmit packets at the speed of its outgoing interface. If packets arrive faster than that interface can send them out, the router does not have a magic wand to push them through. It does the only thing it can do: hold them in **R**andom **A**ccess **M**emory (**RAM**, the device's short-term storage) until the interface is free. That holding zone is called a **queue** — and "queuing" is the verb for what happens inside it.

A common student error here is to say "the router slows down". It doesn't. The router's processing speed doesn't change. What changes is that packets now have to *wait* before being sent. That waiting is queuing.

**Bullet 2 — Queuing packets causes delay because new packets cannot be transmitted until previous packets have been processed.**

Queues operate **F**irst **I**n, **F**irst **O**ut by default. If your packet is the 50th in line and each packet ahead of you takes 0.1 milliseconds to transmit, your packet waits 5 milliseconds before its turn comes. That waiting time is **queuing delay** — one of the six delay types we'll meet on Slide 7.

**Bullet 3 — If the number of packets to be queued continues to increase, the memory within the device fills up and packets are dropped.**

Every router has a finite amount of RAM. When the queue fills the available buffer space, new arriving packets have nowhere to go. The router has no choice — it discards them. This is **packet loss**, and it happens silently from the sender's perspective. The router doesn't email the sender to apologise. The packet just vanishes.

For traffic like email (which uses **TCP**, **T**ransmission **C**ontrol **P**rotocol — a protocol that detects missing packets and retransmits them), this isn't fatal — TCP just resends. For traffic like voice (which uses **UDP**, **U**ser **D**atagram **P**rotocol — a protocol that does not detect or retransmit losses), a dropped packet means a missing slice of audio that's gone forever. The receiver hears a click, a pop, or a dropout.

**The bottom line of slide 4:** congestion → queuing → delay → eventual drops. Three words, three consequences. Memorise this chain. It's the spine of the whole topic.

---

## Slide 5 — Prioritizing Traffic

> **What's on the slide:** Title "Prioritizing Traffic". One bullet at the top. A diagram showing three types of traffic sources (Voice over IP phone in red, Financial Transaction terminal in yellow, Web Page server in orange) feeding into a router. Inside the router, three labelled queues: High Priority Queue (red envelopes), Medium Priority Queue (yellow envelopes), Low Priority Queue (orange envelopes). The queues then merge onto a single output line labelled "Link to Network", with an orange annotation box explaining "All communication types have some access to the media. One particular type of traffic cannot consume all of the bandwidth."

This is the diagram you should aim to reproduce in your exam answer. It captures the central idea of QoS in one picture: instead of one big queue where every packet waits the same, the router maintains **multiple queues** and chooses which queue to serve next based on priority.

The single bullet on the slide says: **"One QoS technique that can help with this problem is to classify data into multiple queues."**

That sentence is doing a lot of work. Unpack it:

- **Classify** — the router examines each arriving packet and decides what type of traffic it is. Voice? Video? Financial? Web? This decision is made using markings in the packet header (we'll meet **DSCP** and **CoS** in later module material).
- **Multiple queues** — instead of one **FIFO** (**F**irst **I**n, **F**irst **O**ut) line, the router maintains several lines in parallel, one per priority level.
- **Help with this problem** — the problem being the congestion → delay → drops chain from Slide 4.

The annotation in the diagram is also a deliberate teaching point: even the lowest-priority queue gets *some* service. A QoS-enabled router doesn't completely starve low-priority traffic — it just makes them wait longer. Web traffic still flows, just behind voice.

---

## Slide 6 — Bandwidth, Congestion, Delay, and Jitter

> **What's on the slide:** Title "Bandwidth, Congestion, Delay, and Jitter". Three bullets. A diagram with three panels labelled Aggregation, Speed Mismatch, and LAN to WAN, each showing arrows of different thicknesses to indicate traffic flow into a bottleneck.

This is the slide that names the four headline parameters you must mention in any 80-mark answer.

**Bullet 1 — Bandwidth is measured in the number of bits that can be transmitted in a single second.**

Bandwidth is the *capacity* of a link. It's measured in **bits per second** (bps), or more commonly kilobits (Kbps), megabits (Mbps), or gigabits (Gbps) per second. Note carefully: bits with a lowercase b, not bytes with an uppercase B. 1 byte = 8 bits, so 100 Mbps = 12.5 megabytes per second. Students lose marks every year by writing megabytes when they mean megabits.

**Bullet 2 — Network congestion causes delay.**

Causation, not correlation. Congestion is not a synonym for delay — congestion is the *cause*, delay is the *symptom*. Congestion happens when offered traffic exceeds available bandwidth. Delay is the result.

**Bullet 3 — Congestion points are ideal candidates for QoS mechanisms. (Aggregation, speed mismatch, and LAN to WAN.)**

The three sub-cases shown in the diagram describe *where* congestion typically happens in real networks:

```
1. AGGREGATION
   Multiple lower-speed links feed into a single
   higher-speed link. If too many feeds are active
   at once, the aggregating switch can't push all
   the packets onto the upstream link.
   
   Example: 24 PCs at 100 Mbps each (theoretically
   2.4 Gbps total) feeding a single 1 Gbps uplink.

2. SPEED MISMATCH
   A fast link feeds a slow link. Packets arrive
   faster than they can leave.
   
   The diagram shows 100 Mbps feeding into the
   router and 1000 Mbps coming out the other side.
   This particular diagram shows fast-out, slow-in
   labelling (Note: the slide labels 100 Mbps above
   the switch and 1000 Mbps below it — read carefully
   which direction is which in your version).

3. LAN to WAN
   The most common real-world bottleneck. The
   office LAN (Local Area Network) runs at gigabit
   speeds inside the building. The WAN link to
   the wider internet runs at a fraction of that —
   often 100 Mbps for a small office.
   
   The diagram shows 1000 Mbps inside and 100 Mbps
   on the WAN side. When everyone in the office
   tries to upload at once, the WAN link is where
   things back up.
```

The fourth headline parameter — **jitter** — isn't explicitly defined as a bullet on Slide 6, but it's in the title and the slide expects you to know it. **Jitter is the variation in delay between consecutive packets.** If packet 1 arrives 20 ms after it was sent, and packet 2 arrives 80 ms after it was sent, and packet 3 arrives 35 ms after it was sent — that variation (20, 80, 35) is jitter. We'll see in Slides 8 and 9 why jitter is the killer parameter for voice and video.

So the four headline parameters, in order, are:

```
1. Bandwidth       — the capacity of the link        (bits/sec)
2. Delay (latency) — how long a packet takes         (milliseconds)
3. Jitter          — the variation in delay          (milliseconds)
4. Packet loss     — percentage of packets dropped   (percent)
```

Write these four down in your exam booklet at the start of your Q4(a) answer. They're worth at least 12 marks together, and they're the easiest 12 marks on the whole question.

---

## Slide 7 — The Six Causes of Delay

> **What's on the slide:** Title "Bandwidth, Congestion, Delay, and Jitter" (same title repeated). A six-row table with two columns: Delay (type name) and Description.

This is the most fact-dense slide in the whole lecture. Memorise the table. Examiners love asking about it because it's specific, finite, and easy to mark.

Here it is, expanded:

```
┌────────────────────┬──────────┬─────────────────────────────────────┐
│ Delay Type         │ Fixed or │ What it is                          │
│                    │ Variable │                                     │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ Code delay         │  Fixed   │ Time to compress data at the source │
│                    │          │ before sending to the first network │
│                    │          │ device (usually a switch).          │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ Packetization      │  Fixed   │ Time to wrap the data into a packet │
│ delay              │          │ with all required header fields.    │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ Queuing delay      │ Variable │ Time a frame or packet waits in the │
│                    │          │ outgoing queue for its turn on the  │
│                    │          │ link.                               │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ Serialization      │  Fixed   │ Time to push the bits of a frame    │
│ delay              │          │ onto the physical wire.             │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ Propagation        │ Variable │ Time for the frame to travel between│
│ delay              │          │ source and destination.             │
├────────────────────┼──────────┼─────────────────────────────────────┤
│ De-jitter delay    │  Fixed   │ Time to buffer arriving packets and │
│                    │          │ play them out at evenly spaced      │
│                    │          │ intervals.                          │
└────────────────────┴──────────┴─────────────────────────────────────┘
```

A few teaching notes on individual rows:

**Code delay** is the time your computer takes to compress the voice signal (or video frame) into bytes before any networking happens. A voice **codec** (**co**der-**dec**oder — the algorithm that compresses voice into bytes and decompresses it back into sound) like G.711 or G.729 takes a fixed amount of time per sample. That time is code delay. Same for video: an **H.264** encoder takes a fixed time per frame.

**Packetization delay** is the next step: now that you have compressed bytes, the protocol stack wraps them in headers — adding the **UDP**, **IP** (**I**nternet **P**rotocol — the standard for addressing and routing packets), and Ethernet wrappers. This too takes a fixed amount of time per packet.

**Queuing delay** is the variable one we already met. It depends entirely on how busy the outgoing link is at the moment your packet arrives.

**Serialization delay** is the time the network adapter takes to transmit the actual bits, one after another, onto the cable. A 1500-byte packet on a 100 Mbps link takes `1500 × 8 / 100,000,000 = 120 microseconds` to serialize. On a 1 Gbps link, the same packet takes 12 microseconds. The faster the link, the lower the serialization delay.

**Propagation delay** is the time the bits spend actually travelling along the cable. Electricity in copper moves at roughly two-thirds the speed of light — about 200,000 km per second. So 1 km of cable adds 5 microseconds of propagation delay. From Tashkent to London (about 5,500 km of cable distance), that's roughly 27 milliseconds one way before *anything else* is added. (Note: the slide labels propagation as "variable", which is true if the routing path changes between packets — on a fixed point-to-point cable, propagation is actually constant. Cisco materials call it variable because real internet paths can change.)

**De-jitter delay** is something added on purpose at the receiver. We'll cover it next.

For exam purposes: name all six. Say whether each is fixed or variable. One sentence each. That's worth around 12 marks on its own.

---

## Slide 8 — Packet Loss and the Playout Delay Buffer

> **What's on the slide:** Title "Packet Loss". Two main bullets. A diagram on the right showing an "Audio stream of packets received with jitter" (five red envelopes arriving at irregular intervals) flowing through a purple box labelled "Playout delay buffer", then emerging as a "De-jittered stream sent to outbound interface" (five yellow envelopes spaced evenly). The diagram is captioned **"Playout Delay Buffer Compensates for Jitter"**.

Here's the problem this slide solves. When voice packets travel across a network, each one experiences a different amount of queuing delay along the way. So even though the sender transmits them at perfectly even intervals (one packet every 20 milliseconds, for example), they don't arrive at the receiver at perfectly even intervals. They arrive bunched, then gapped, then bunched again.

If you played them out the moment they arrived, you'd hear stuttering audio: a burst of sound, silence, more sound, silence. The pacing of the original voice would be lost.

**The fix is the playout delay buffer.** This is a small holding area on the receiver. As packets arrive, the receiver puts them in the buffer instead of playing them immediately. Then it plays them out of the buffer at perfectly even intervals — the original cadence is restored.

**Bullet 1 — Without QoS mechanisms, time-sensitive packets such as real-time video and voice are dropped with the same frequency as data that is not time-sensitive.**

In other words: if a router has no QoS configured, a voice packet sitting in a full queue gets dropped exactly as readily as an email packet. This is bad because email can recover (TCP retransmits) but voice cannot.

**Bullet 2 — When a router receives a Real-Time Protocol (RTP) digital audio stream for Voice over IP (VoIP), it compensates for the jitter that is encountered using a playout delay buffer.**

Three abbreviations to unpack:

- **RTP** — **R**eal-**T**ime **P**rotocol — the protocol layer that carries voice and video payloads inside UDP packets. RTP adds timestamps and sequence numbers so the receiver can reorder packets and detect missing ones.
- **VoIP** — **V**oice **o**ver **IP** — the general technique of carrying telephone-style voice traffic over an IP network instead of the traditional telephone system.
- **Playout delay buffer** — the holding area described above.

The playout buffer is a deliberate, intentional delay added at the receiver. It is the source of the "de-jitter delay" row in the Slide 7 table. Yes, it adds delay — but it adds *predictable* delay, which is far better than the *unpredictable* jitter it cancels out. A constant 40 ms of delay is easy to live with; a delay that swings between 10 ms and 100 ms is unlistenable.

---

## Slide 9 — When the Buffer Can't Keep Up

> **What's on the slide:** Title "Packet Loss" (same title as Slide 8). Two main bullets. A diagram on the right showing "Audio stream with excessive jitter" — five red envelopes arriving at very irregular intervals with large gaps, flowing into the playout buffer, but only some emerging as yellow envelopes on the output — with one envelope position empty in the output stream. The diagram is captioned **"Packet is dropped due to excessive jitter"**.

The playout buffer is not magic. It has a fixed size — typically tens of milliseconds. If a packet arrives so late that the playout slot it was supposed to fill has already passed, the buffer can't help. The packet is too late to be useful, and the receiver discards it. The audio has a gap.

**Bullet 1 — If the jitter is so large that it causes packets to be received out of the range of the playout buffer, the out-of-range packets are discarded and dropouts are heard in the audio.**

This is the breakdown case. Jitter exceeds buffer size. Late packets are dropped. The user hears the dropout.

**Bullet 2 — For losses as small as one packet, the digital signal processor (DSP) interpolates what it thinks the audio should be and no problem is audible to the user.**

**DSP** stands for **D**igital **S**ignal **P**rocessor — a specialised chip (often inside the IP phone itself) designed to manipulate audio in real time. When one packet is lost, the DSP looks at the audio sample just before the gap and the audio sample just after, then computes a plausible "in-between" waveform to fill the gap. The human ear cannot distinguish this from real audio for losses of one packet.

**Bullet 3 — When jitter exceeds what the DSP can do to make up for the missing packets, audio problems are heard.**

Two consecutive lost packets? Maybe still recoverable. Three or four in a row? You start to hear clicks, robotic artefacts, or actual silence. The DSP cannot invent a quarter-second of audio it has no input data for.

This is why voice traffic has such tight loss requirements — typically below 1% — while data traffic happily tolerates 5% loss because TCP will just resend.

---

## Summary — what this part gave you

Six distinct content blocks ready to drop into a Q4(a) answer:

```
1. The QoS problem (Slide 4)
   Congestion → queuing → delay → drops chain.
   Three sentences.

2. Prioritising traffic (Slide 5)
   Multiple queues classified by traffic type.
   Diagram opportunity #1.

3. The four parameters (Slide 6)
   Bandwidth, delay, jitter, packet loss.
   Three congestion scenarios: aggregation,
   speed mismatch, LAN-to-WAN. Diagram opportunity #2.

4. The six delay types (Slide 7)
   Code, packetisation, queuing, serialisation,
   propagation, de-jitter. Reproduce the table.

5. The playout buffer (Slide 8)
   How receivers de-jitter voice. Diagram opportunity #3.

6. When the buffer fails (Slide 9)
   DSP interpolation handles one lost packet.
   Multiple losses cause audible degradation.
```

That's already six blocks worth roughly 12–13 marks each. With the queuing algorithms in Part 3 it becomes seven blocks of material — comfortably more than enough for 80 marks.

---

# Part 3: Traffic Characteristics + The Four Queuing Algorithms

## Opening picture — the dispatcher's whiteboard

In Part 2 we left you at the Hadra junction watching cars queue. Now zoom out. Imagine you're the traffic dispatcher sitting in the control room, looking at a whiteboard. Your job is to decide which vehicles get priority lanes today.

You can't make rules until you understand what's on the road. A whiteboard with no information leads to the wrong decisions. So before any priority scheme can be applied, you need to know your *traffic*: who's on it, what they need, and what happens if they don't get it.

That's what the first half of this part covers — the three traffic types (voice, video, data) and their specific demands. The second half covers the actual mechanisms a router uses to enforce priority: the four queuing algorithms named in every Q4(a) marking scheme.

This part is the heaviest one in the tutorial. Take it in two sittings if you need to — pause after the voice/video/data block and come back fresh for the queuing algorithms.

---

## Section A — Traffic Characteristics (Slides 10–15)

## Slide 10 — Section divider

> **What's on the slide:** Title "Traffic Characteristics" centred on the cover-style background. No content. No image.

This is a section divider that tells you the next group of slides is about how different traffic types behave on the network. We move from "what is **QoS** (**Q**uality **o**f **S**ervice — the techniques routers use to give different traffic types different treatment when the network gets busy)" to "what traffic are we doing QoS *to*".

---

## Slide 11 — Network Traffic Trends

> **What's on the slide:** Title "Network Traffic Trends". Two paragraphs with bullets, no images.

This slide gives you the historical context that makes QoS necessary in the first place.

**Paragraph 1 — In the early 2000s, the predominant types of IP traffic were voice and data:**

- **Voice traffic has a predictable bandwidth need and known packet arrival times.** A voice call uses a fixed codec that produces packets at a fixed rate. You always know it's coming, and you always know how much it needs.
- **Data traffic is not real-time and has unpredictable bandwidth need.** A user reading email uses zero bandwidth between clicks, then a burst when they open a message. You cannot predict when the next click will happen.
- **Data traffic can temporarily burst.** A file download saturates the link briefly and then ends. This burstiness is data's defining feature.

**Paragraph 2 — More recently, video traffic has become increasingly important:**

- **Video traffic represented 70% of all traffic in 2017.**
- **By 2022, video will represent 82% of all traffic.**
- **Mobile video traffic will reach 60.9 exabytes per month by 2022.** An **exabyte** is a billion gigabytes. Sixty exabytes per month is the kind of number that justifies its own QoS strategy.

**The final sentence — "The type of demands that voice, video, and data traffic place on the network are very different."**

This is the punchline. The whole rest of the QoS topic exists because these three traffic types have incompatible requirements. You can't treat them all the same and expect everyone to be happy.

---

## Slide 12 — Voice

> **What's on the slide:** Title "Voice". Two groups of bullets: "Voice Traffic Characteristics" on the left (Smooth, Benign, Drop sensitive, Delay sensitive, UPD priority), and "Requirements" below (Latency ≤ 150ms, Jitter ≤ 30ms, Loss ≤ 1%, Bandwidth 30–128 Kbps). On the right, a chart titled "Voice Packets" with bytes on the Y-axis (0 to 1400) and time on the X-axis, showing small blue bars of about 200 bytes each, evenly spaced, with the label "Audio Samples" and an arrow pointing at one of the bars marked "20 ms" between bars.

The chart is a key teaching tool. Voice traffic on the wire looks like a steady drumbeat: small packets (about 200 bytes each), perfectly evenly spaced (one every 20 milliseconds), forever, for as long as the call lasts. There are no spikes, no bursts, no gaps. That's what "smooth" means.

**Characteristics — left bullet list:**

- **Smooth** — packets arrive at constant intervals, not in bursts. The chart on the slide makes this visible.
- **Benign** — voice does not try to grab more bandwidth than it needs. A single voice call uses what it uses; it does not expand to fill available capacity.
- **Drop sensitive** — losing voice packets causes audible degradation (clicks, dropouts, robotic artefacts). Voice cannot recover via retransmission because the audio has already moved on.
- **Delay sensitive** — voice needs to arrive quickly. Humans expect conversational latency below about 150 ms; beyond that, people start talking over each other because they can't tell when the other person stopped.
- **UDP priority** — voice runs over **UDP** (**U**ser **D**atagram **P**rotocol — a connectionless transport protocol with no retransmission, used when speed matters more than delivery guarantees). (Note: the slide writes this as "UPD priority" — that's a typo for **UDP**. Read it as UDP.)

**Requirements — bottom bullet list:**

- **Latency ≤ 150 ms** — one-way delay from microphone to speaker should stay under 150 milliseconds.
- **Jitter ≤ 30 ms** — variation in packet arrival should stay under 30 milliseconds (otherwise the playout buffer from Part 2 cannot smooth it out).
- **Loss ≤ 1%** — under one percent of voice packets may be lost without audible problems. Above that, the **DSP** (**D**igital **S**ignal **P**rocessor — the chip on the receiving phone that fills in for lost packets) can't keep up and dropouts become audible.
- **Bandwidth 30–128 Kbps** — that's **K**ilo**b**its **p**er **s**econd, thousands of bits per second. Voice is genuinely tiny: a low-quality codec uses 30 Kbps; high-quality G.711 uses about 80 Kbps; a top-end stereo codec might use 128 Kbps. Compare with video below.

---

## Slide 13 — Video

> **What's on the slide:** Title "Video". Bullets for Characteristics (Bursty, Greedy, Drop sensitive, Delay sensitive, UPD priority) and One-Way Requirements (Latency ≤ 200–400 ms, Jitter ≤ 30–50 ms, Loss ≤ 0.1–1%, Bandwidth 384 Kbps – 20 Mbps). On the right, a chart titled "Video Packets" showing tall blue bars at varying heights from about 500 to 1300 bytes, grouped in three clusters labelled "Video Frame", with "33 ms" between frame clusters.

The chart shows the opposite of the voice picture. Video packets are bigger (often near the 1500-byte Ethernet maximum), and they arrive in bursts — one for each video frame. At 30 frames per second, you get a burst every 33 milliseconds, but within each burst the packets pile out as fast as the encoder can push them.

**Characteristics — left bullet list:**

- **Bursty** — packets arrive in clumps (one clump per frame), not steadily. This is the opposite of voice.
- **Greedy** — video tries to consume as much bandwidth as it can. Higher quality always uses more. A 4K stream will use 25 Mbps if you let it; a 720p stream will use 4 Mbps; both will use what they're given.
- **Drop sensitive** — losing a video packet causes visual artefacts (blocks, freezes, smearing) until the next "key frame" arrives.
- **Delay sensitive** — interactive video (a call, a conference) needs low delay so participants don't talk over each other. One-way streaming (Netflix, YouTube) is less sensitive — a 5-second buffer is invisible to the user.
- **UDP priority** — like voice, real-time video uses UDP. (Note: this slide also writes "UPD priority" — same typo as Slide 12. It means UDP.)

**One-Way Requirements:**

- **Latency ≤ 200–400 ms** — a wider range than voice because the answer depends on whether the video is interactive (200 ms) or one-way streaming (400 ms is tolerable).
- **Jitter ≤ 30–50 ms** — similar to voice, slightly more forgiving.
- **Loss ≤ 0.1–1%** — generally tighter than voice because video artefacts last longer than audio artefacts.
- **Bandwidth 384 Kbps – 20 Mbps** — note the units jump from kilobits (low-quality video conferencing) to **M**ega**b**its **p**er **s**econd (high-definition broadcast streams). Even the bottom of video's range is several times the top of voice's range.

---

## Slide 14 — Data

> **What's on the slide:** Title "Data". Several paragraphs of running text plus a bullet list at the bottom.

This slide describes the third traffic class — everything that isn't voice or video. Email, web browsing, file transfers, database queries, software updates.

**First paragraph — Data applications that have no tolerance for data loss, such as email and web pages, use TCP to ensure that if packets are lost in transit, they will be resent.**

**TCP** stands for **T**ransmission **C**ontrol **P**rotocol — the transport protocol that detects missing packets and retransmits them, the opposite of UDP. Email cannot tolerate a corrupted word; if a packet is lost, TCP just sends it again. The cost is a slightly longer delivery time, which email users do not notice.

**Bullets in the middle:**

- **Data traffic can be smooth or bursty.** Unlike voice (always smooth) or video (always bursty), data is unpredictable. A user reading a webpage is silent for minutes, then clicks a link and bursts a few hundred kilobytes.
- **Network control traffic is usually smooth and predictable.** Routing protocol updates, monitoring traffic, **DHCP** lease renewals — these happen on regular timers and have known sizes.
- **Some TCP applications can consume a large portion of network capacity.** A backup job, a software update, or a large file download can saturate a link for minutes or hours.

**Characteristics — bottom bullet list:**

- **Smooth or bursty** — depending on application.
- **Benign or greedy** — depending on application. Email is benign; a backup is greedy.
- **Drop insensitive** — TCP will retransmit anything lost. The user never sees the loss.
- **Delay insensitive** — a 300 ms delay on a web page load is unnoticed. The same delay on a voice call is unacceptable.
- **TCP retransmits** — the defining recovery mechanism.

---

## Slide 15 — Data: Mission Critical vs Interactive

> **What's on the slide:** Title "Data" (same as Slide 14). A short intro paragraph and a 2×2 matrix table with rows for Interactive vs Not interactive and columns for Mission Critical vs Not Mission Critical.

The intro: **Data traffic is relatively insensitive to drops and delays compared to voice and video. Quality of Experience or QoE is important to consider with data traffic. Does the data come from an interactive application? Is the data mission critical?**

**QoE** is a new abbreviation: **Q**uality **o**f **E**xperience — what the user actually feels, as opposed to the strict technical parameters of QoS. A web page that loads in 5 seconds is technically working, but the QoE is terrible.

The matrix gives the four combinations:

```
┌─────────────────┬──────────────────────┬─────────────────────┐
│                 │  Mission Critical    │  Not Mission Critical│
├─────────────────┼──────────────────────┼─────────────────────┤
│  Interactive    │  Prioritise for      │  Could benefit from │
│                 │  lowest delay among  │  lower delay.       │
│                 │  data traffic; aim   │                     │
│                 │  for 1–2 second      │                     │
│                 │  response time.      │                     │
├─────────────────┼──────────────────────┼─────────────────────┤
│  Not            │  Delay can vary as   │  Gets any leftover  │
│  interactive    │  long as a minimum   │  bandwidth after    │
│                 │  bandwidth is        │  voice, video, and  │
│                 │  guaranteed.         │  other data are met.│
└─────────────────┴──────────────────────┴─────────────────────┘
```

Translated:

- **Interactive + Mission Critical**: a banking transaction terminal. The user is sitting there waiting for a result. Delay must be visibly low.
- **Interactive + Not Mission Critical**: a chat app. Lower delay feels nicer, but a one-second lag won't ruin your day.
- **Not Interactive + Mission Critical**: an overnight backup. Nobody is watching, but it absolutely must complete before morning. Bandwidth matters; delay does not.
- **Not Interactive + Not Mission Critical**: software auto-updates. The leftover bucket. Whatever's free at 3am.

This 2×2 is worth memorising — it gives you a tidy way to talk about why "data" isn't one thing in a QoS answer.

---

## Section B — Queuing Algorithms (Slides 16–20)

## Slide 16 — Section divider

> **What's on the slide:** Title "Queuing Algorithms" on the cover-style background. No content.

We now switch from "what traffic looks like" to "what the router does about it". The next four slides give the four named queuing algorithms in order of increasing sophistication. **All four must be named in any 80-mark Q4(a) answer.**

---

## Slide 17 — First In, First Out (FIFO)

> **What's on the slide:** Title "First in First Out". Four bullets. A diagram showing packets arriving at an ingress interface (a row of unsorted coloured squares — purple, gold, blue, grey), entering a box labelled "No Queuing" with a triangle, flowing through a box labelled "FIFO", and exiting at an egress interface as the same coloured squares in the same order.

This is the absolute baseline — no QoS at all.

**Bullets:**

- **First In First Out (FIFO) queuing buffers and forwards packets in the order of their arrival.** The first packet to enter the queue is the first to leave. Like a single-file checkout line.
- **FIFO has no concept of priority or classes of traffic and consequently, makes no decision about packet priority.** It doesn't look inside the packets; it doesn't care whether they're voice, video, or email. They're all just packets.
- **There is only one queue, and all packets are treated equally.** This is the defining property.
- **Packets are sent out an interface in the order in which they arrive.**

**The diagram** shows this visually: the same coloured squares come out in the same order they went in. No reordering. No prioritisation.

**Why FIFO matters as a baseline:** every queuing algorithm we'll meet next is described in terms of how it improves on FIFO. If you can explain FIFO clearly, you can explain the rest by saying what they add.

**Where FIFO is still used:** on links where there's no congestion. If the outgoing link is fast enough that the queue is always empty, FIFO is perfectly fine. The router doesn't need to make decisions if there's nothing to decide. FIFO becomes inadequate only when traffic exceeds capacity and packets pile up.

---

## Slide 18 — Weighted Fair Queuing (WFQ)

> **What's on the slide:** Title "Weighted Fair Queuing (WFQ)". Several bullets at the top. On the right, a diagram showing packets entering at an ingress interface (two unsorted squares — yellow and blue), passing through a Classify triangle, then splitting into four labelled queues (High in purple, Medium in yellow, Normal in blue, Low in grey), feeding into a WFQ box, and emerging at an egress interface as a reordered sequence. A legend at the bottom shows "Priority Classification" with all four colours and their corresponding labels.

**WFQ** stands for **W**eighted **F**air **Q**ueuing — the first algorithm that distinguishes between flows automatically.

**Bullets:**

- **Provides fair bandwidth allocation to all network traffic.**
- **WFQ applies priority to identified traffic, classifies it into conversations or flows, and then determines how much bandwidth each flow is allowed relative to other flows.**

A **flow** is a stream of packets between two specific endpoints. WFQ treats each conversation as a separate flow and gives each one its fair share of available bandwidth.

- **Classification can be based on source and destination IP addresses, MAC addresses, port numbers, protocol, and Type of Service (ToS) value.**

Five classification inputs the router can use:

- **Source and destination IP addresses** — who is talking to whom at the network layer.
- **MAC addresses** — **M**edia **A**ccess **C**ontrol addresses — the hardware-level addresses at the link layer.
- **Port numbers** — which application is in use (port 80 for web, 5060 for **SIP** signalling, and so on).
- **Protocol** — TCP, UDP, **ICMP**, others.
- **Type of Service (ToS) value** — an 8-bit field in the IPv4 header that the sender can use to mark traffic priority.

The combination of these inputs lets WFQ separate, say, a voice call from a file download even though both are crossing the same router.

- **WFQ is not supported with tunneling and encryption.**

This is a key limitation. If the packet is inside a **VPN** tunnel or encrypted, the router cannot see the inner port numbers or addresses, so WFQ cannot classify the flow. This is why WFQ alone is rarely deployed in modern enterprises — encrypted traffic is everywhere now.

**The diagram** shows the classify step splitting the input stream into four priority levels (High purple, Medium yellow, Normal blue, Low grey). WFQ then services these queues in proportion to their weight — high-priority queues get drained more frequently than low-priority ones, but nothing is completely starved.

---

## Slide 19 — Class-Based Weighted Fair Queuing (CBWFQ)

> **What's on the slide:** Title "Class-Based Weighted Fair Queuing (CBWFQ)". Several bullets and a diagram showing packets entering an ingress interface (numbers 4, 3, 2, 1 representing four different traffic types), passing through user-defined classification into three labelled queues (Class 1, Class 2, Class 3), then through a WFQ box, and emerging at egress in a reordered sequence (2, 1, 3, 4).

**CBWFQ** stands for **C**lass-**B**ased **W**eighted **F**air **Q**ueuing — the next step up. The difference from WFQ is that *you* define the classes instead of letting the router auto-detect flows.

**Bullets:**

- **Extends the standard WFQ functionality to provide support for user-defined traffic classes.**
- **Traffic classes are defined based on match criteria including protocols, access control lists (ACLs), and input interfaces.**

**ACL** stands for **A**ccess **C**ontrol **L**ist — a configured rule on the router that matches packets by their source/destination/protocol/port. With CBWFQ, you can write an ACL that says "every packet destined for IP 192.168.50.10 on port 5060 belongs to the Voice class", then assign that class a guaranteed amount of bandwidth.

- **A FIFO queue is reserved for each class.** Inside each class, packets are still served in arrival order — that's the FIFO part. The classes themselves are served in proportion to their weight — that's the WFQ part.
- **A class can be assigned characteristics, such as bandwidth, weight, and maximum packet limit.** You configure these explicitly. "Voice class gets 30% of the link. Video class gets 40%. Everything else shares the remaining 30%." This is administrative control that WFQ alone doesn't offer.

**Why CBWFQ is a step up from WFQ:**

- WFQ decides class membership *automatically* based on flow inspection.
- CBWFQ lets *you* decide class membership based on rules you write.

If your business cares that the accounting database always has bandwidth, CBWFQ lets you guarantee it. WFQ would just treat each accounting database connection as one fair flow among many, with no special protection.

**The diagram** shows traffic being grouped into three user-defined classes (Class 1, Class 2, Class 3), each maintaining its own FIFO queue, then WFQ-style fair service across the three classes.

---

## Slide 20 — Low Latency Queuing (LLQ)

> **What's on the slide:** Title "Low Latency Queuing (LLQ)". Several bullets and a complex diagram showing two paths: voice packets marked V go to a "Priority Queue" labelled "Class Priority" with a "PQ" oval; other packets (numbered 1, 2, 3, 4) go into CBWFQ classes (Class 1, Class 2, Class 3) with a "WFQ" oval; both paths merge into a single egress stream showing the sequence "2 1 3 4 V V".

**LLQ** stands for **L**ow **L**atency **Q**ueuing — the top tier. This is what real-world voice-carrying networks actually use.

**Bullets:**

- **The Low Latency Queuing (LLQ) feature brings strict priority queuing (PQ) to CBWFQ.**

**PQ** stands for **P**riority **Q**ueue. A strict priority queue is a queue that gets emptied first, *always*, before any other queue is even looked at. As long as there's a packet in the priority queue, the router serves it ahead of everything else.

- **Strict PQ allows delay-sensitive packets such as voice to be sent before packets in other queues.**
- **LLQ allows delay-sensitive packets to be sent first giving delay-sensitive packets preferential treatment over other traffic.**

**The mental model:** LLQ is CBWFQ + one extra special queue called the priority queue. The priority queue is reserved for the most delay-sensitive traffic — almost always voice. Whenever a voice packet arrives, it goes to the priority queue and gets sent first, before the router even considers what's in the CBWFQ classes.

This is the ambulance-with-sirens model from our Hadra junction analogy. The other vehicles in the CBWFQ classes still get their fair share — but the moment a voice packet appears, everything else waits.

**Important caveat:** strict priority is dangerous if not capped. If voice traffic ever exceeds the link's capacity, the priority queue could starve everything else. Real LLQ configurations always include a *policer* on the priority queue — a hard cap (say, 20% of link bandwidth) above which voice packets are dropped. This protects the other classes from being starved.

**The diagram** captures all this: voice packets (V) take the express lane through the priority queue. Other packets (1, 2, 3, 4) go through the regular CBWFQ classes. The output sequence shows voice packets emerging first whenever they're present.

---

## The Four Algorithms — Side by Side

This is the comparison you should be able to draw from memory in the exam:

```
┌─────────┬──────────────┬──────────────────┬────────────────────┐
│ Algo    │ Classes?     │ Priority?        │ When to use        │
├─────────┼──────────────┼──────────────────┼────────────────────┤
│ FIFO    │ One queue.   │ No priority.     │ No congestion;     │
│         │ All equal.   │ Order of arrival.│ baseline only.     │
├─────────┼──────────────┼──────────────────┼────────────────────┤
│ WFQ     │ Auto-detected│ Weighted fair    │ Mixed traffic,     │
│         │ flows.       │ across flows.    │ no tunnels/crypto. │
├─────────┼──────────────┼──────────────────┼────────────────────┤
│ CBWFQ   │ User-defined │ Weighted fair    │ Operator needs     │
│         │ classes.     │ across classes.  │ guaranteed BW per  │
│         │              │                  │ class.             │
├─────────┼──────────────┼──────────────────┼────────────────────┤
│ LLQ     │ User-defined │ One STRICT       │ Voice traffic      │
│         │ classes +    │ priority queue + │ present; production│
│         │ one priority │ weighted fair    │ enterprise default.│
│         │ queue.       │ for the rest.    │                    │
└─────────┴──────────────┴──────────────────┴────────────────────┘
```

If you reproduce this table in your exam booklet, you have just answered probably 20 marks of the queuing-algorithms section in one diagram.

---

## Summary — what this part gave you

Two more content blocks for the Q4(a) answer, on top of the six from Part 2:

```
7. Traffic characteristics (Slides 11–15)
   Voice (smooth, drop/delay-sensitive, 30–128 Kbps).
   Video (bursty, greedy, 384 Kbps – 20 Mbps).
   Data (TCP, drop/delay-insensitive, mission-critical
   vs interactive matrix).

8. Queuing algorithms (Slides 17–20)
   FIFO   — one queue, no priority.
   WFQ    — auto-detected flows, fair weights.
   CBWFQ  — user-defined classes, fair weights per class.
   LLQ    — CBWFQ + one strict-priority queue for voice.
   Reproduce the diagrams. Name all four. Distinguish them.
```

You now have eight content blocks ready. That's more than enough for an 80-mark answer. Part 4 will show you how to assemble them in the right order, with the right diagrams, in the 29 minutes you have.

---

# Part 4: Writing the 80-Mark Theory Answer

## Opening picture — knowing vs performing

Parts 2 and 3 made you a person who *knows* **QoS** (**Q**uality **o**f **S**ervice — the techniques routers use to give different traffic types different treatment when the network gets busy). Eight content blocks. Two dozen named concepts. Plenty of detail.

But the exam doesn't reward knowing. It rewards performing — getting the right material onto the page, in the right order, with the right diagrams, in under thirty minutes, under exam pressure, by hand, with no internet.

This part is about that performance. Same material as before, but viewed through a new question: *given that you only have 29 minutes to write, what do you actually put on the page first, second, third?*

Three things make a Q4(a) answer score well above the average:

1. A **plan written before the prose starts**.
2. **Two or three diagrams**, drawn with labels.
3. **All four queuing algorithms named explicitly** — **FIFO**, **WFQ**, **CBWFQ**, **LLQ** — by their full names on first mention.

If you do those three things, the rest takes care of itself. The rest of this part explains how.

---

## The shape of an 80-mark answer

The question has two visible halves. Lecturers tend to write QoS questions that look like:

> "Describe, using diagrams, the problems to overcome in delivering QoS solutions, and the queuing mechanisms used to achieve these."

That's two halves welded with the word "and":

- **Half A — The problems** (worth roughly 40 of the 80 marks)
- **Half B — The queuing mechanisms** (worth the other ~40)

So your answer should be split into two clear sections of roughly equal length. The skeleton looks like this:

```
┌─────────────────────────────────────────────────────────────┐
│  SECTION A — The Problems QoS Solves           ~40 marks    │
│                                                              │
│   A1. What is QoS, and what is the chain of                 │
│       cause and effect it tries to manage?                  │
│       (congestion → queuing → delay → drops)        ~5 mks  │
│                                                              │
│   A2. The four parameters used to measure                   │
│       network quality.                                       │
│       (bandwidth, delay, jitter, packet loss)      ~10 mks  │
│                                                              │
│   A3. The six sources of delay.                             │
│       (code, packetisation, queuing,                        │
│        serialisation, propagation, de-jitter)      ~10 mks  │
│                                                              │
│   A4. Packet loss in voice/video specifically.              │
│       Playout buffer compensates for jitter;                │
│       DSP fills small gaps; large jitter causes             │
│       audible loss.                                  ~8 mks │
│                                                              │
│   A5. Traffic characteristics — why different                │
│       types need different treatment.                       │
│       Voice (smooth, drop/delay-sensitive);                 │
│       video (bursty, greedy);                                │
│       data (TCP, drop/delay-insensitive).            ~7 mks │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  SECTION B — The Queuing Mechanisms            ~40 marks    │
│                                                              │
│   B1. The basic idea: multiple queues, classify              │
│       by traffic type, serve by priority.            ~5 mks │
│                                                              │
│   B2. FIFO (First In First Out).                            │
│       One queue, no priority, baseline only.         ~8 mks │
│                                                              │
│   B3. WFQ (Weighted Fair Queuing).                          │
│       Auto-detected flows, fair share each,                 │
│       not compatible with tunnels/encryption.        ~8 mks │
│                                                              │
│   B4. CBWFQ (Class-Based Weighted Fair Queuing).            │
│       User-defined classes, guaranteed BW each,             │
│       configured via ACLs.                            ~8 mks│
│                                                              │
│   B5. LLQ (Low Latency Queuing).                            │
│       CBWFQ + one strict-priority queue for voice.          │
│       The production standard for voice networks.   ~11 mks │
└─────────────────────────────────────────────────────────────┘

                                              TOTAL: 80 marks
```

Roughly 10 blocks. Eight of them worth 5–10 marks each, two of them slightly heavier. If you cover all ten in any depth, you have your 80 marks.

---

## The plan-then-write rule

Spend the first **two minutes** of your 29 not writing prose. Instead, draw this skeleton in the **margin** of your answer booklet:

```
A. PROBLEMS                          B. QUEUING
   1. QoS + chain                       1. multiple queues
   2. 4 params (BW/D/J/L)               2. FIFO
   3. 6 delay types                     3. WFQ
   4. packet loss + playout             4. CBWFQ
   5. voice/video/data                  5. LLQ ★
```

Just keywords. No prose. You'll tick each one off as you write that block.

Two reasons this matters:

1. **It stops you from forgetting a topic.** Without the marginal list, students routinely write a beautiful 600-word answer on the four parameters and the delay types, then look up at the clock with 5 minutes left and realise they haven't mentioned a single queuing algorithm. That answer gets capped at ~45 marks because half the question wasn't addressed. The marginal list prevents this exact failure.

2. **It lets you write in the right depth.** Knowing you have ten blocks tells you each block needs about 2–3 minutes of writing. That prevents you from over-investing in block 1 (the easy one) and under-investing in block 8 (the hard one).

Star (★) the LLQ block. It's the heaviest. Make sure you reach it before time runs out.

---

## Diagram strategy

The question explicitly says **"using diagrams"** — plural. One diagram is not enough. Aim for **three diagrams**, scattered through your answer where they fit naturally.

The three diagrams to memorise:

```
┌────────────────────────────────────────────────────────────┐
│  DIAGRAM 1 — The Three Congestion Scenarios                │
│  (Lecture 4 Slide 6)                                       │
│                                                             │
│  Three side-by-side panels showing where congestion        │
│  happens in real networks:                                 │
│                                                             │
│   Aggregation       Speed Mismatch       LAN to WAN        │
│  ┌──────────┐     ┌──────────────┐    ┌──────────────┐    │
│  │ many in  │     │ 100 Mbps in  │    │ 1000 Mbps in │    │
│  │  ↓↓↓↓↓   │     │      │       │    │      │       │    │
│  │   one    │     │      ▼       │    │      ▼       │    │
│  │   out    │     │ 1000 Mbps out│    │ 100 Mbps out │    │
│  └──────────┘     └──────────────┘    └──────────────┘    │
│                                                             │
│  Place this diagram in Block A1 or A2.                     │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│  DIAGRAM 2 — The Multi-Queue Prioritisation Picture        │
│  (Lecture 4 Slide 5)                                       │
│                                                             │
│      Voice ───┐                                            │
│               ├── Classify ──┬─ HIGH ─┐                    │
│  Financial ───┤              │        │                    │
│               │              ├─ MED ──┼── Link to network  │
│       Web ───┘              │        │                    │
│                              └─ LOW ──┘                    │
│                                                             │
│  Three traffic sources → classify → three priority         │
│  queues → single output link.                              │
│                                                             │
│  Place this diagram at the start of Section B (Block B1).  │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│  DIAGRAM 3 — The LLQ Architecture                          │
│  (Lecture 4 Slide 20)                                      │
│                                                             │
│   Voice ──────► Priority Queue ──┐                         │
│                                  │                         │
│                                  ▼                         │
│   Class 1 ──┐                ┌── PQ ──┐                    │
│   Class 2 ──┼──── CBWFQ ─────┤        ├── Output           │
│   Class 3 ──┘                └── WFQ ─┘                    │
│                                                             │
│  Voice on the express lane (PQ, served first always);     │
│  other classes share the rest via CBWFQ.                  │
│                                                             │
│  Place this diagram in Block B5 — your strongest move.    │
└────────────────────────────────────────────────────────────┘
```

These three together demonstrate visual understanding of the whole topic: where the problem is (Diagram 1), the basic solution (Diagram 2), and the most sophisticated form of the solution (Diagram 3). That's a clear progression and it scores well.

If you only have time for two diagrams, drop Diagram 1 — it's the most replaceable.

Label every box, arrow, and queue. Unlabelled diagrams score lower than labelled ones, even if they're the same shape.

---

## Minute-by-minute time budget

Here's how the 29 minutes break down:

```
┌──────────┬─────────────────────────────────────────────────┐
│   Time   │  What you should be doing                       │
├──────────┼─────────────────────────────────────────────────┤
│   0–2    │  Marginal plan. Skeleton of 10 blocks.          │
│          │  No prose yet.                                  │
├──────────┼─────────────────────────────────────────────────┤
│   2–4    │  Block A1 — opening paragraph + chain.          │
├──────────┼─────────────────────────────────────────────────┤
│   4–7    │  Block A2 — four parameters.                    │
│          │  (Optional: insert Diagram 1 here.)             │
├──────────┼─────────────────────────────────────────────────┤
│   7–10   │  Block A3 — six delay types table.              │
├──────────┼─────────────────────────────────────────────────┤
│  10–12   │  Block A4 — packet loss + playout buffer.       │
├──────────┼─────────────────────────────────────────────────┤
│  12–14   │  Block A5 — voice/video/data characteristics.   │
├──────────┼─────────────────────────────────────────────────┤
│  14–16   │  Block B1 — multiple-queues idea.               │
│          │  Insert Diagram 2 here.                         │
├──────────┼─────────────────────────────────────────────────┤
│  16–18   │  Block B2 — FIFO.                               │
├──────────┼─────────────────────────────────────────────────┤
│  18–20   │  Block B3 — WFQ.                                │
├──────────┼─────────────────────────────────────────────────┤
│  20–22   │  Block B4 — CBWFQ.                              │
├──────────┼─────────────────────────────────────────────────┤
│  22–26   │  Block B5 — LLQ + Diagram 3. ★                  │
├──────────┼─────────────────────────────────────────────────┤
│  26–29   │  Review: check units, labels, diagrams.         │
│          │  Tick each block on the marginal plan.          │
└──────────┴─────────────────────────────────────────────────┘
```

Two minutes per block sounds tight, but each block only needs 3–5 sentences. You're not writing essays per block — you're writing concise paragraphs that hit the named facts.

Watch the clock. If you hit minute 20 and you're still on block A4, **skip ahead to block B5**. You can come back to the missed blocks if there's time. But you cannot afford to leave the queuing algorithms unwritten.

---

## A sample opening paragraph

To make this concrete, here's an example of what Block A1 (the first 2 minutes of writing) might look like:

> Quality of Service (QoS) refers to a set of mechanisms used by routers and switches to manage network resources when traffic demand exceeds available capacity. Without QoS, all packets are treated equally — meaning time-sensitive traffic such as voice calls receives no priority over routine traffic such as email or file transfers. This becomes a problem during congestion. When packets arrive at a router faster than the outgoing link can transmit them, the router holds them in memory queues. Queuing introduces delay because newer packets must wait for earlier ones to be served. If the queue grows past the memory buffer's capacity, packets are dropped indiscriminately — regardless of how important they were. QoS exists to ensure that under these conditions, business-critical and time-sensitive traffic receives preferential treatment.

That's about 130 words, takes roughly 2 minutes to hand-write, and hits: the definition, why it matters, the queuing chain, and the loss outcome. It earns its ~5 marks cleanly.

Adapt this opening to your own phrasing — don't memorise it word for word. The structure is what matters.

---

## What to do if you run short on time

Reality check: most students don't finish Q4(a) cleanly. Time slips. Here's the priority order if you're behind schedule:

**At minute 20, if you haven't reached the queuing section yet:**

1. **Stop writing the current block.** Mid-sentence is fine.
2. **Skip directly to block B5 (LLQ).** Yes, you'll skip B1–B4. That's OK if you have to.
3. **Write LLQ with its diagram.** Mention that it builds on CBWFQ which builds on WFQ which extends FIFO. That sentence alone signals you know the others exist.
4. **Come back to whatever you skipped if any time remains.**

**At minute 25, if you still have blocks unwritten:**

1. **Use bullet lists**, not prose. Markers read bullets faster.
2. **Name the algorithm and its one defining feature**, then move on. "WFQ — auto-detects flows by classification, assigns weighted fair share, not compatible with encrypted tunnels."
3. **Always finish with a sum-up sentence** that names all four algorithms even if you didn't explain them all in depth.

A 65-mark answer that named all four algorithms is better than an 80-mark plan that only got to two of them.

---

## What NOT to do

Six failure modes that lose marks for no good reason:

1. **Don't write the answer without a plan.** Even 90 seconds of marginal skeleton beats diving straight into prose. Without a plan you will forget a topic.

2. **Don't write an answer without diagrams.** The question says "using diagrams". An answer with zero diagrams is capped around 50 marks no matter how good the prose is.

3. **Don't confuse delay with jitter.** Delay = how long a packet takes. Jitter = the variation in delay from packet to packet. Use both words correctly.

4. **Don't write "voice uses TCP".** Voice uses **UDP** (**U**ser **D**atagram **P**rotocol — connectionless, no retransmission). TCP would make voice worse, not better, because retransmitted voice arrives too late to be useful.

5. **Don't skip the LLQ section because it's last.** It's the heaviest block at ~11 marks. Failing to mention it caps you at ~69. Always reach LLQ.

6. **Don't draw unlabelled diagrams.** A box with three arrows and no words is not a diagram, it's a doodle. Label every queue, every input, every output. Use the words from the slides: "High Priority", "WFQ", "Priority Queue", "Output".

---

## Wrap-up — the checklist

Before you move on from Q4(a), look back at your answer and check:

```
□ Did I open with a definition of QoS?
□ Did I describe the congestion → queuing → delay → drops chain?
□ Did I name all four parameters (bandwidth, delay, jitter, packet loss)?
□ Did I list the six delay types?
□ Did I mention packet loss in voice (playout buffer + DSP)?
□ Did I describe at least two of voice/video/data characteristics?
□ Did I name all four queuing algorithms (FIFO, WFQ, CBWFQ, LLQ)?
□ Did I explain how each algorithm differs from the previous one?
□ Did I draw at least two labelled diagrams?
□ Did I include the LLQ architecture diagram?
```

Ten ticks = 75+ marks. Even seven ticks is comfortably a 2:1.

Now turn the page to Q4(b) — that's a clean, mechanical 20 marks waiting for you. Parts 5 and 6 prepare you for it.

---

# Part 5: The Bandwidth Formula — Broken Open

## Opening picture — back to the ISP desk

Remember the Tashkent **ISP** (**I**nternet **S**ervice **P**rovider — the company that sells the internet connection to homes and businesses) story from Part 1? Your team lead handed you the brief:

> "A new sales office is opening. 100 hard phones, 25 soft phones, 10 video-call agents, 8 field reps on mobile WiFi. Size the **WAN** (**W**ide **A**rea **N**etwork — the long-distance link from the office to the wider internet) link they buy from us."

This part is the formula you use to answer that question. It's the same formula the exam will hand you in Q4(b). It's also the formula a real network engineer at Uztelecom or Beeline would scribble on the back of a napkin during a customer meeting.

The full thing fits on one line:

```
                                                                
  BW(row) = (Endpoints × (1 + Headroom%)) × Sim_Use% × Mbps/EP  
                                                                
       Total = Σ BW(row)   (sum across all endpoint types)      
```

Looks innocent. Looks like algebra. But every symbol in that line answers a real question a network engineer has to think about. The rest of this part pulls each multiplier apart and asks "why is this here?"

---

## The mental model — three factors and a per-unit demand

Before grinding into the formula, build the right mental picture.

Sizing a WAN link is asking *how much bandwidth do I actually need*. The answer is built from four numbers:

```
   ┌─ how many users will I have in the future? ──┐
   │                                              │
   │   (Endpoints × (1 + Headroom%))              │
   │                                              │
   ├─ how many of them will be busy at once? ─────┤
   │                                              │
   │   × Simultaneous use %                       │
   │                                              │
   ├─ how much does each busy user need? ─────────┤
   │                                              │
   │   × Bandwidth per endpoint                   │
   │                                              │
   └──────────────────────────────────────────────┘
   
       = bandwidth needed for this endpoint type
```

Three factors that shrink or grow your count, multiplied by a per-unit demand. That's the whole shape. If you keep that picture in your head, you can rebuild the formula from scratch even if you forget the exact phrasing.

Then you do that calculation for every endpoint type and add the results.

---

## Component 1 — Endpoints (the count baseline)

The first number is just *how many of this thing exist right now*.

```
   Endpoints = current number of devices of this type
```

If the brief says "100 hard phones", then Endpoints = 100. No interpretation needed. Read it off the table.

A few things to watch for:

- **Read the row label carefully.** "Two-party HD video call" means each *call* uses 2 endpoints (the two people on the call), but the row count usually refers to the number of simultaneous *calls*, not the number of *people*. Read the bandwidth-per-endpoint value to confirm: if it says 2 Mbps per call, that's the full call (both directions, both parties). If it says 1 Mbps per person, multiply.

- **Don't double-count.** If the table lists "soft phones" and "hard phones" separately, those are two distinct device populations. Don't assume soft-phone users also have hard phones unless the brief says so.

- **Count is always a whole number.** You can't have 25.5 phones. Round up if the brief gives a fraction (rare, but possible in some variants).

---

## Component 2 — Headroom factor (1 + Headroom%)

This is the multiplier that bothers students most. It looks simple but is easy to get wrong.

```
   Headroom factor = (1 + Headroom%)
```

The word **headroom** means "spare capacity reserved for future growth". If the table says "30% headroom", that means "build the link big enough to handle 30% more endpoints than we have today".

So if today's count is 100 and headroom is 30%, the future-proofed count is:

```
   100 × (1 + 0.30)  =  100 × 1.30  =  130
```

**Why does headroom exist as a concept?**

Network links take weeks or months to provision. When the customer signs a contract for a 50 Mbps link today, they can't easily call up the ISP next month and say "we hired more staff, please make it 60 Mbps by Friday". The provisioning paperwork, the carrier negotiations, the equipment changes — these things take time. So the engineer building the design today must look 1–3 years ahead and ask "how big will this office be by the time we'd realistically upgrade the link?".

Headroom is the answer. It bakes future growth into today's number.

**The trap students fall into:**

```
   ✗ WRONG:  100 + 30 = 130     (treating 30% as if it were "30")
   ✓ RIGHT:  100 × 1.30 = 130   (treating 30% as a multiplier of 1.30)
```

In this specific case the two methods happen to give the same answer (130) by coincidence — because 100 × 0.30 = 30. But try it with a different count and you'll see the difference:

```
   25 endpoints, 30% headroom:
   ✗ WRONG:  25 + 30 = 55       (count somehow ballooned to 55)
   ✓ RIGHT:  25 × 1.30 = 32.5   (sensible — 30% bigger than 25)
```

The right answer 32.5 makes physical sense: "30% bigger than 25 is about 33". The wrong answer 55 makes no sense at all — it's more than double.

Always treat percentage headroom as a multiplier on the count, not an addition to the count.

---

## Component 3 — Simultaneous use factor

The third number is the share of users who are *actively using their device at the same moment*.

```
   Sim_Use% = fraction of endpoints active simultaneously
```

A call centre with 100 phones doesn't have 100 active calls at any single moment. Some agents are on break, some are between calls, some are typing notes. Maybe 50 are actually talking right now. That's a 50% simultaneous use factor.

**Why does this multiplier exist?**

Because you size your WAN for the *typical busy moment*, not for the *theoretical worst case*. If you sized for 100% simultaneous use, you'd be paying for capacity that is never used. If you sized for 10%, you'd run out of bandwidth every busy morning.

Different endpoint types have very different typical concurrency:

```
   Hard Phone (call centre):     ~50% (lots of agents talking at once)
   Soft Phone (general staff):   ~60% (busy office workers)
   HD Video call:                ~60% (each scheduled call is active)
   Mobile WiFi (field reps):     ~80% (when on-site, mostly active)
```

These numbers come from the customer's business profile. The brief gives you the value to use; you don't have to guess.

**The trap:**

```
   ✗ WRONG:  Add Sim_Use% to the count.
   ✓ RIGHT:  Multiply the count by Sim_Use% (after headroom).
```

Sim_Use% always behaves like a fraction between 0 and 1, never as an addend. 60% = 0.60. 80% = 0.80. Multiply, never add.

---

## Component 4 — Bandwidth per endpoint

The fourth and final number is *how many Mbps each active endpoint actually consumes*.

```
   Mbps_per_EP = bandwidth needed by one active endpoint of this type
```

From the traffic profiles we covered in Part 3:

```
   Voice phone (hard or soft):   0.08–0.10 Mbps  (often rounded to 0.1)
   HD Video call:                2–4 Mbps        (depending on quality)
   Mobile WiFi voice:            0.1 Mbps        (same codec as fixed)
```

The brief gives you the exact value to use. Don't try to look up "real" codec rates — use the table value.

**Why is the number so small for voice?**

A G.711 codec uses 64 Kbps for the audio payload, plus about 16 Kbps of header overhead, totalling about 80 Kbps = **0.08 Mbps**. The 0.1 figure on most tables is a slightly rounded-up convenience number.

**Why is video so much bigger?**

Because video carries thousands of times more information per second than audio. A speech sample at 8 kHz × 8 bits = 64 Kbps. A 720p video frame at 30 fps × roughly 50 KB compressed = 12 Mbps raw, compressed down to 2 Mbps with H.264.

---

## Putting it together — the per-row formula

Once you have the four numbers, the per-row calculation is one line:

```
   BW(row) = Endpoints × (1 + Headroom%) × Sim_Use% × Mbps_per_EP
```

Bracket order matters when you write it down, but mathematically multiplication is associative — you can compute it in any order:

```
   = (100 × 1.30) × 0.50 × 0.10
   =       130    × 0.50 × 0.10
   =          65.0      × 0.10
   =                  6.50  Mbps
```

Or:

```
   = 100 × (1.30 × 0.50 × 0.10)
   = 100 × 0.065
   = 6.50 Mbps
```

Same answer. Pick whichever order you find easiest to compute by hand under exam pressure. Most students prefer left-to-right because it matches how they read the formula.

---

## The final step — the sum

Once you have a BW value for every row, add them up:

```
   Total_BW = BW(row_1) + BW(row_2) + ... + BW(row_n)
```

This total is the minimum WAN bandwidth the office needs. In a real ISP quote, you'd round this up to the nearest available service tier (10, 25, 50, 100, 250 Mbps and so on) and recommend that.

For the exam, write the total to **three decimal places** and label it in **Mb/s** or **Mbps**.

---

## A tiny worked example — single row

Let's run the formula on one made-up scenario before tackling the bigger mock table in Part 6.

> A Yunusabad call centre has **40 hard phones** today. The customer expects **20% growth** over the next two years. On a typical busy morning, **50%** of phones are active. Each phone uses **0.1 Mbps**. What's the bandwidth requirement for the phones alone?

Step by step:

```
   Endpoints      = 40
   Headroom%      = 20%   →  factor (1 + 0.20) = 1.20
   Sim_Use%       = 50%   →  0.50
   Mbps_per_EP    = 0.1

   BW = 40 × 1.20 × 0.50 × 0.10
      = 48 × 0.50 × 0.10
      = 24 × 0.10
      = 2.40 Mbps
```

The phones at this site need **2.4 Mbps** of WAN bandwidth.

Notice the chain of intermediate values: 48 (future-proofed phone count), 24 (active phones at peak), 2.4 (their combined demand). Each intermediate number is meaningful in the real world — they're not just arithmetic mile-markers. Showing them in your exam working is good practice; it lets the marker see that you understand what each multiplier *means*.

---

## A two-row example — adding across types

Now suppose the same call centre also has **10 video-call agents** with **30% headroom**, **60% simultaneous use**, and **2 Mbps each**. What's the total?

```
   Row 1 — Phones (from above):
     40 × 1.20 × 0.50 × 0.10 = 2.400 Mbps

   Row 2 — Video calls:
     10 × 1.30 × 0.60 × 2    = 15.600 Mbps

   Total:
     2.400 + 15.600          = 18.000 Mbps
```

That's the WAN link size. Eighteen megabits per second. Round up to the nearest standard tier (25 Mbps in most markets) and that's your quote.

Notice how dominant the video row is — almost seven times bigger than the phone row, despite having only a quarter as many endpoints. That's the per-unit demand at work. Video is bandwidth-hungry; voice is not. One video call eats more than twenty phone calls.

---

## Unit sanity check

Always do a units check before you write down the final number.

```
   Endpoints           — unitless count    [—]
   (1 + Headroom%)     — unitless ratio    [—]
   Sim_Use%            — unitless fraction [—]
   Mbps_per_EP         — Mbps              [Mbps]
                                            ─────
   Product             — Mbps              [Mbps]
```

All four multipliers chain together to give Mbps as the unit of the answer. If your final number doesn't come out in Mbps, you've made an error — go back.

Common unit mistakes:

- Using **K**ilo**b**its (Kbps) instead of **M**egabits (Mbps). 0.1 Mbps = 100 Kbps. If the table gives 0.1 Mbps, use 0.1, not 100.
- Writing **MB/s** (capital B, megabytes) instead of **Mb/s** (lowercase b, megabits). These differ by a factor of 8. The exam expects **Mb/s** or **Mbps**.
- Mixing units within one calculation. If the table gives 0.1 Mbps for voice and 2 Mbps for video, both are already in the same unit — just multiply directly. Don't convert one without converting the other.

---

## Common pitfalls — the top six

The six errors graders see most often on this exact calculation:

1. **Adding headroom% to the count instead of multiplying.** `25 + 30 = 55` is wrong. `25 × 1.30 = 32.5` is right. Headroom is a multiplier, not an addend.

2. **Forgetting the "1 +" in the headroom factor.** If you multiply by 0.30 instead of 1.30, you've made the count *smaller*, not bigger. Always `1 + headroom%`.

3. **Treating Sim_Use% as a percentage to add.** Same trap as headroom but in the opposite direction. 60% = 0.60 (a multiplier). Multiply the count by it.

4. **Using megabytes (MB) instead of megabits (Mb).** This makes your answer 8× bigger than it should be. The exam expects Mb/s.

5. **Forgetting one row.** Each endpoint type contributes a row. Skip one and you've under-sized the link. Always count: the table has N rows, your working should have N row calculations.

6. **Not writing the units on the final answer.** A number with no unit is not an answer. Always end with "Mb/s" or "Mbps".

All six are unforced errors. Slow down, double-check, and write neatly — and you don't make any of them.

---

# Part 6: Mock Practice Table — Full Solution

## Opening picture — exam day, 7 minutes on the clock

You've finished the 80-mark theory part. Twenty-three minutes are gone. Six minutes remain for Q4(b) — the **20-mark** bandwidth calculation. You turn the page and see a table of four endpoint types with numbers in five columns.

Six minutes is plenty. The calculation is mechanical. The only enemy is sloppiness — arithmetic errors, forgotten units, missing rows. This part is the dress rehearsal: we'll work through a complete mock table end-to-end, showing every step, the way you should lay it out in your exam booklet.

---

## The mock practice table

```
A regional sales office handles customer enquiries through video and
telephone calls. Calculate the bandwidth requirements for its WAN link
to meet current demand and any future expansion.

┌──────────────────┬───────────┬───────────┬──────────────┬───────────┐
│ Endpoint Type    │ Number of │ % headroom│ Simultaneous │ Bandwidth │
│                  │ endpoints │ for future│ use factor   │ per EP    │
│                  │           │           │              │ (Mbps)    │
├──────────────────┼───────────┼───────────┼──────────────┼───────────┤
│ Soft phone       │    30     │    30%    │     60%      │    0.1    │
│ Hard Phone       │   100     │    10%    │     50%      │    0.1    │
│ Two Party HD     │    10     │    20%    │     60%      │     2     │
│ Video call       │           │           │              │           │
│ Mobile Phone     │     8     │    10%    │     75%      │    0.1    │
│ on WiFi          │           │           │              │           │
└──────────────────┴───────────┴───────────┴──────────────┴───────────┘

                                                            (20 Marks)

Formula:
  Minimum BW = Σ [(Endpoints × (1 + Headroom%)) × Sim_Use% × BW_per_EP]
```

Four rows, one formula per row, one sum. Let's go.

---

## Row 1 — Soft phone

> **What the numbers say:** 30 soft phones today. Customer expects 30% growth. On a typical busy moment 60% are active. Each uses 0.1 Mbps.

**Step 1 — Pull out the four numbers:**

```
   Endpoints      = 30
   Headroom%      = 30%        →  factor = 1 + 0.30 = 1.30
   Sim_Use%       = 60%        →  factor = 0.60
   Mbps_per_EP    = 0.1 Mbps
```

**Step 2 — Apply the formula:**

```
   BW(soft phone) = 30 × 1.30 × 0.60 × 0.1
```

**Step 3 — Compute left to right, showing each intermediate result:**

```
   30 × 1.30   = 39        (future-proofed count: 30% bigger than 30)
   39 × 0.60   = 23.4      (active count at peak: 60% of 39)
   23.4 × 0.1  = 2.34      (combined demand: 0.1 Mbps each)
```

**Step 4 — Write the answer:**

```
   BW(soft phone) = 2.340 Mbps
```

Notice the chain of meaningful intermediate numbers: **39** (future phone count), **23.4** (active phones at peak), **2.34** (their combined Mbps). Each one is a real-world quantity. Writing them all out shows the marker you understand what each multiplier means, not just that you punched buttons on a calculator.

---

## Row 2 — Hard Phone

> **What the numbers say:** 100 hard phones today. 10% growth expected. 50% active at peak. 0.1 Mbps each.

**Step 1 — The four numbers:**

```
   Endpoints      = 100
   Headroom%      = 10%        →  factor = 1.10
   Sim_Use%       = 50%        →  factor = 0.50
   Mbps_per_EP    = 0.1
```

**Step 2 — Apply the formula:**

```
   BW(hard phone) = 100 × 1.10 × 0.50 × 0.1
```

**Step 3 — Compute step by step:**

```
   100 × 1.10  = 110       (future-proofed count)
   110 × 0.50  = 55        (active count at peak)
   55 × 0.1    = 5.5       (combined demand)
```

**Step 4 — The answer:**

```
   BW(hard phone) = 5.500 Mbps
```

A useful sanity check: 100 phones is more than 3× the soft phone count (30), but the hard-phone result (5.5 Mbps) is only ~2.4× the soft-phone result (2.34 Mbps). Why? Because the headroom and simultaneous-use factors are smaller for hard phones, partially cancelling out the larger base count. The numbers make physical sense — they don't blow up unreasonably.

---

## Row 3 — Two Party HD Video call

> **What the numbers say:** 10 video-call endpoints today. 20% growth. 60% active. 2 Mbps per call.

This is the one to watch carefully. Even though only 10 endpoints exist, the per-call bandwidth is 20× higher than voice — so this row will dominate the total.

**Step 1 — The four numbers:**

```
   Endpoints      = 10
   Headroom%      = 20%        →  factor = 1.20
   Sim_Use%       = 60%        →  factor = 0.60
   Mbps_per_EP    = 2 Mbps
```

**Step 2 — Apply the formula:**

```
   BW(HD video) = 10 × 1.20 × 0.60 × 2
```

**Step 3 — Compute:**

```
   10 × 1.20   = 12         (future-proofed count)
   12 × 0.60   = 7.2        (active count at peak)
   7.2 × 2     = 14.4       (combined demand)
```

**Step 4 — The answer:**

```
   BW(HD video) = 14.400 Mbps
```

This single row is **bigger than the other three combined**. That's video for you — bandwidth-hungry compared to voice. If the customer plans to expand the video-call team further down the line, this row will grow even faster.

---

## Row 4 — Mobile Phone on WiFi

> **What the numbers say:** 8 mobile users today. 10% growth. 75% active at peak. 0.1 Mbps each.

The smallest row. Don't skip it — every row contributes marks.

**Step 1 — The four numbers:**

```
   Endpoints      = 8
   Headroom%      = 10%        →  factor = 1.10
   Sim_Use%       = 75%        →  factor = 0.75
   Mbps_per_EP    = 0.1
```

**Step 2 — Apply the formula:**

```
   BW(mobile) = 8 × 1.10 × 0.75 × 0.1
```

**Step 3 — Compute:**

```
   8 × 1.10    = 8.8        (future-proofed count)
   8.8 × 0.75  = 6.6        (active count at peak)
   6.6 × 0.1   = 0.66       (combined demand)
```

**Step 4 — The answer:**

```
   BW(mobile WiFi) = 0.660 Mbps
```

A small but non-zero contribution. Even 8 mobile users at high activity pull a notable fraction of a megabit. The 75% simultaneous-use factor is high here because field reps tend to be "on call" most of the time they're out — when their phone is on, they're using it.

---

## The Total

Sum all four rows:

```
   Soft phone        :   2.340 Mbps
   Hard Phone        :   5.500 Mbps
   Two Party HD Video:  14.400 Mbps
   Mobile on WiFi    :   0.660 Mbps
                       ─────────────
   Total Minimum BW  :  22.900 Mbps
```

The office needs **22.9 Mb/s** of WAN bandwidth. In a real ISP quote, you'd round this up to the nearest service tier — probably 25 Mbps — and recommend that.

Notice the breakdown of where the bandwidth is going:

```
   HD Video calls   →  14.4 / 22.9  =  63%  of the link
   Hard Phones      →   5.5 / 22.9  =  24%  of the link
   Soft Phones      →   2.3 / 22.9  =  10%  of the link
   Mobile WiFi      →   0.7 / 22.9  =   3%  of the link
```

Two-thirds of the link is for video calls, even though only 10 of the 148 total endpoints are video. That's the per-unit-demand factor at work. If the customer wanted to save money, the lever is the number of simultaneous video calls — that's where the bandwidth is going.

---

## How this looks in an actual exam booklet

You don't have to write all the prose above in the exam. Here's the bare minimum the marker needs to see for full marks:

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Q4(b)                                                         │
│                                                                │
│  Formula: BW = Endpoints × (1+Headroom%) × Sim_Use% × Mbps/EP  │
│                                                                │
│  Soft phone:    30 × 1.30 × 0.60 × 0.1   = 2.340 Mbps          │
│  Hard Phone:   100 × 1.10 × 0.50 × 0.1   = 5.500 Mbps          │
│  HD Video:      10 × 1.20 × 0.60 × 2     = 14.400 Mbps         │
│  Mobile WiFi:    8 × 1.10 × 0.75 × 0.1   = 0.660 Mbps          │
│                                            ─────────────       │
│  Total Minimum WAN BW                    = 22.900 Mb/s         │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

That layout has:

- The formula at the top (shows the marker you know it).
- One row per endpoint type (no skipped rows).
- The full chain of multiplication visible for each row.
- Each row's answer to three decimal places with units.
- A clean horizontal line and the final total at the bottom.
- The unit "Mb/s" on the total.

About 10 lines of writing. Fits comfortably in 5–6 minutes if you've practised. Full 20 marks.

---

## Common wrong answers — what students typically do

Six failure modes graders see year after year on this exact style of question:

**Wrong answer #1 — adding headroom%:**

```
   ✗ Soft phone: (30 + 30) × 0.60 × 0.1 = 3.6 Mbps
   ✓ Soft phone:  30 × 1.30 × 0.60 × 0.1 = 2.34 Mbps
```

The student treated "30% headroom" as if it meant "add 30 to the count". Easy to do under pressure. Results in a wildly overestimated answer.

**Wrong answer #2 — forgetting the "1 +" in the headroom factor:**

```
   ✗ Soft phone: 30 × 0.30 × 0.60 × 0.1 = 0.54 Mbps
   ✓ Soft phone: 30 × 1.30 × 0.60 × 0.1 = 2.34 Mbps
```

The student multiplied by the percentage as a fraction (0.30) instead of (1 + 0.30 = 1.30). Result: the future-proofed count comes out *smaller* than today's count, which makes no physical sense.

**Wrong answer #3 — using megabytes instead of megabits:**

The student gets the arithmetic right but writes "22.9 MB/s" instead of "22.9 Mb/s". That's a unit error worth at least 1 mark. Mb (lowercase b) means megabits. MB (capital B) means megabytes — eight times bigger. Always use the lowercase b.

**Wrong answer #4 — skipping the headroom step entirely:**

```
   ✗ Soft phone: 30 × 0.60 × 0.1 = 1.8 Mbps
   ✓ Soft phone: 30 × 1.30 × 0.60 × 0.1 = 2.34 Mbps
```

The student forgot the headroom column existed. Total comes out about 20% too small. The brief specifically asks for "current demand **and any future expansion**" — the headroom is the future expansion. Don't drop it.

**Wrong answer #5 — only computing one or two rows:**

The student runs out of time and only computes 2 of the 4 rows, giving a partial total. That's at most half marks. Always compute every row, even if you have to do it quickly.

**Wrong answer #6 — getting the sum wrong:**

```
   2.34 + 5.5 + 14.4 + 0.66 = 22.9    ✓ correct
   2.34 + 5.5 + 14.4 + 0.66 = 23.0    ✗ rounded too early
   2.34 + 5.5 + 14.4 + 0.66 = 21.9    ✗ arithmetic error
```

The student computed each row correctly but added them wrong. Double-check the addition. Lay out the four row answers in a column and add them carefully.

---

## Self-test — try this one on your own

Here's a different mock table for you to solve unaided. Set yourself a 6-minute timer and have a go.

```
A regional logistics company in Tashkent operates the following endpoints:

┌──────────────────┬───────────┬───────────┬──────────────┬───────────┐
│ Endpoint Type    │ Number of │ % headroom│ Simultaneous │ Bandwidth │
│                  │ endpoints │ for future│ use factor   │ per EP    │
│                  │           │           │              │ (Mbps)    │
├──────────────────┼───────────┼───────────┼──────────────┼───────────┤
│ IP Desk Phone    │    50     │    15%    │     40%      │    0.1    │
│ Video Conf Room  │     5     │    20%    │     50%      │     4     │
│ Mobile Field on  │    20     │    10%    │     90%      │    0.15   │
│ Cellular         │           │           │              │           │
│ Customer Service │    12     │    25%    │     70%      │    0.1    │
│ Softphone        │           │           │              │           │
└──────────────────┴───────────┴───────────┴──────────────┴───────────┘
```

Compute the minimum WAN bandwidth requirement. Show every row. Don't peek below.

. . .

. . .

. . .

. . .

. . .

### Answer key — check your work

```
   IP Desk Phone     :  50 × 1.15 × 0.40 × 0.10  =   2.300 Mbps
   Video Conf Room   :   5 × 1.20 × 0.50 × 4.00  =  12.000 Mbps
   Mobile Field      :  20 × 1.10 × 0.90 × 0.15  =   2.970 Mbps
   Softphone CS      :  12 × 1.25 × 0.70 × 0.10  =   1.050 Mbps
                                                  ─────────────
   Total Minimum BW                              =  18.320 Mb/s
```

**If your total is 18.320 Mb/s** — well done, you've nailed the formula and the arithmetic.

**If your total is within 0.1 of 18.320** — you have a rounding habit. Carry full precision through every row and only round at the end.

**If your total is 20.000 or higher** — check if you added headroom% instead of multiplying.

**If your total is below 15.000** — check if you forgot the "1 +" in the headroom factor, or skipped a row.

**If your video row is anything other than 12.000** — that row is the high-stakes one. 5 × 1.20 = 6 future-proofed rooms. 6 × 0.50 = 3 active at peak. 3 × 4 Mbps = 12 Mbps total. The arithmetic is clean; the only way to get it wrong is the headroom step.

---

## Tutorial complete — wrap-up

You've now been through the full Q4 toolkit:

```
Part 1 — How to read Q4 and budget your time.
Part 2 — The QoS problem (parameters, delay, loss).
Part 3 — Traffic types and the four queuing algorithms.
Part 4 — How to assemble the 80-mark theory answer.
Part 5 — The bandwidth formula, broken open.
Part 6 — Full mock practice table solved end-to-end.
```

What this should leave you with:

- **A definition** of QoS you can write from memory.
- **Eight content blocks** for the theory part, each worth roughly 10 marks.
- **Three diagrams** you can reproduce by hand: congestion scenarios, multi-queue prioritisation, and LLQ architecture.
- **The names** of all four queuing algorithms (FIFO, WFQ, CBWFQ, LLQ) and how each differs from the one before.
- **The bandwidth formula** as a mental model — four factors per row, sum across rows.
- **The discipline** to never add a percentage as if it were a number.

Practice the self-test one more time before the exam. Time yourself. Do it with a pen, not a calculator app, because the exam allows only basic calculators and your handwriting is part of the answer the marker reads.

Good luck.

---
