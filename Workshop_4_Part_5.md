# Workshop 4 — NTP & Syslog Logging: Part 5 — Research Questions

## About This Part

The workshop includes three research questions for your
portfolio. These are **your own work** — the answers are not
provided here or anywhere in the companion guide.

Each question below is followed by some thinking prompts to
help you get started. These are not answers — they are
directions to explore. Your final answers must be written in
your own words, fully referenced, and submitted as part of
your portfolio.

> **Fully referenced** means citing your sources. If you use
> information from a Cisco document, an RFC, a textbook, or a
> website, include the reference. Your tutor is looking for
> evidence that you researched the topic, not that you memorized
> the companion guide.

---

## Question 1

*"What are the advantages and disadvantages of setting the
warning level to a high value?"*

> **Terminology note:** The "warning level" in this question
> refers to the syslog trap level — the same setting you
> configured in Part 3 with `logging trap debugging`. The
> worksheet uses "warning level" as a general term; Cisco calls
> it the "trap level."

**Think about:**
- What does "high value" mean here — a high number (like 7) or
  a high severity (like 0)? The phrasing is ambiguous, so
  consider both interpretations and be clear in your answer
  about which direction you are discussing.
- What happens to the volume of messages at each end of the
  scale?
- How does message volume affect the syslog server's storage
  and the network administrator's ability to find important
  events?
- What might you miss if you only capture the most critical
  messages?
- What might you drown in if you capture everything?
- Think about real scenarios: a security audit vs daily
  monitoring — would you use the same trap level for both?

**Where to look:**
- Cisco documentation on `logging trap` and syslog severity
  levels
- Search for "syslog severity levels best practices"
- RFC 5424 (The Syslog Protocol) — Section 6.2.1 covers
  severity values

---

## Question 2

*"With respect to NTP server what is meant by the stratum
level? Why is it important to have the timestamping?"*

This is two questions in one. Address both parts.

**For stratum level, think about:**
- What is the stratum hierarchy and where does it start?
- Why does stratum number increase with each hop?
- What happens if two NTP sources disagree — how does a client
  decide which one to trust?
- In this lab, R1 was stratum 5 and R2 became stratum 6 — what
  does that tell you about accuracy?
- What would happen if someone set their router to stratum 1
  when it had no real time source?

**For timestamping, think about:**
- What problem does timestamping solve that you observed in
  Part 1 (before NTP was configured)?
- If two devices report a network failure at different times,
  how do you determine the sequence of events?
- Think beyond networking — why do banks, hospitals, and stock
  exchanges care about precise time?
- What is the relationship between NTP and the usefulness of
  syslog?

**Where to look:**
- Cisco documentation on NTP and `ntp master`
- RFC 5905 (Network Time Protocol Version 4) — look at the
  stratum definitions
- Search for "why NTP matters network management"

---

## Question 3

*"What are the strengths and weaknesses of logging events in
the computing and networking environment?"*

This is the broadest question. Think beyond just syslog on
Cisco routers — the question says "computing and networking
environment."

**For strengths, think about:**
- Troubleshooting and root cause analysis
- Security monitoring and forensics
- Compliance and audit requirements
- Capacity planning and trend analysis
- How logs helped you in this lab (Part 4) to understand what
  happened on the network

**For weaknesses, think about:**
- Storage costs and retention policies
- Performance impact on the devices generating logs
- Privacy concerns — what sensitive information might logs
  contain?
- The problem of too much data — can you find a needle in a
  haystack of millions of log lines?
- What happens if the logging system itself fails or is
  compromised?
- UDP 514 (syslog) is unencrypted and connectionless — what
  are the implications?

**Where to look:**
- Search for "advantages disadvantages network logging"
- NIST guidelines on log management (SP 800-92)
- Compare syslog with more modern approaches (SIEM systems,
  structured logging)
- Think about GDPR or data protection regulations and how they
  affect what you can log

---

## Writing Tips

- Answer each question in **2–3 paragraphs** minimum. One-line
  answers will not earn full marks.
- **Reference everything.** Even if you think something is
  common knowledge, citing a source shows you verified it.
- Use the lab you just completed as a concrete example — "In
  this lab, we configured logging trap debugging, which
  means..." is stronger than a purely theoretical answer.
- If a question is ambiguous (like Question 1), acknowledge the
  ambiguity and explain your interpretation before answering.

---

## What You Have Built — Full Workshop Summary

Over Parts 0 through 4, you built a complete network monitoring
system:

```
 Part │ What You Did                     │ Why It Matters
 ═════╪══════════════════════════════════╪═══════════════════════════════
  0   │ Learned the concepts             │ Foundation for everything
  1   │ Built the topology, configured   │ Network must work before
      │ IPs and RIPv2                    │ you can monitor it
  2   │ Set up NTP (R1 master, R2 client)│ Synchronized timestamps
  3   │ Configured syslog (timestamps,   │ Central log collection
      │ logging host, trap level)        │
  4   │ Generated and analyzed events    │ Proved the system works
  5   │ Research questions               │ Deeper understanding
```

Good luck with the research. Use your search engine, cite your
sources, and write in your own words.
