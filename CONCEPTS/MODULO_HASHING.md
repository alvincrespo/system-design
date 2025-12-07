# Modulo Hashing

## Table of Contents

- [Overview](#overview)
- [Basic Formula](#basic-formula)
- [How It Works](#how-it-works)
- [The Critical Problem: Cache Invalidation](#the-critical-problem-cache-invalidation)
  - [What Breaks?](#what-breaks)
  - [The Cascade Effect](#the-cascade-effect)
- [Why This Matters](#why-this-matters)
- [The Solution](#the-solution)
- [Real-World Scenario: Cache Collapse](#real-world-scenario-cache-collapse)
- [Key Takeaway](#key-takeaway)
- [References](#references)

## Overview

Modulo hashing is a naive, straightforward approach to distribute keys (requests, user IDs, data items) across a pool of servers. It's the simplest distribution technique but has critical limitations in distributed systems.

## Basic Formula

```
serverIndex = hash(key) % N
```

Where:
- `hash(key)` = a hash function applied to the key, returning a large number
- `%` = modulo operator
- `N` = number of servers in the pool
- `serverIndex` = the index of the server that handles this key (0 to N-1)

## How It Works

**Example:**
- You have 3 servers (N = 3)
- A user with ID `user_123` comes in
- Hash function returns: `456789`
- Calculation: `456789 % 3 = 0`
- Result: Request goes to **Server 0**

The modulo operator ensures the result always falls within the valid range (0 to N-1), so every key maps to some server.

## The Critical Problem: Cache Invalidation

### What Breaks?

When a server dies or new servers are added, `N` changes. This causes **almost every key to remap to a different server**.

**Example:**
- Same key `user_123` hashing to `456789`
- Original: `456789 % 3 = 0` (Server 0)
- One server dies, now N = 2
- New calculation: `456789 % 2 = 1` (Server 1)
- Same key now points to a **different server**

### The Cascade Effect

With millions of keys, removing one server causes massive remapping:
- Keys scatter across all new servers
- New servers receive duplicate requests they've never seen
- Cached data is lost
- Databases get hammered with repeated queries
- System experiences severe performance degradation

This is known as the **cache invalidation problem** or **thrashing problem**, related to the **thundering herd effect**.

## Why This Matters

In distributed systems, servers fail regularly. You might need to:
- Replace a failing server
- Add capacity during traffic spikes
- Perform maintenance

With modulo hashing, each of these operations invalidates most of your cache and can bring your system down.

## The Solution

**Consistent Hashing** solves this problem by remapping only a small portion of keys when servers change, keeping most mappings stable.

## Real-World Scenario: Cache Collapse

Imagine you're running a **video streaming platform** with a distributed cache layer for user session data.

**Setup:**

- 10 cache servers using modulo hashing
- 50 million active users
- Each user's session data cached (watch history, preferences, recommendations)

**What Happens:**

- 3am: One cache server fails due to a hardware issue
- Your system detects this and removes it from the pool (N goes from 10 to 9)
- Immediately, nearly 45 million users' sessions remap to different cache servers
- All those sessions cause simultaneous cache misses
- The database gets hammered with 45 million queries to rebuild session data
- Response times spike from 100ms to 10+ seconds
- Users experience buffering, slow page loads, platform appears broken
- Angry user complaints flood in, and you're in a 2-hour incident recovery

**With Consistent Hashing:**

- Same server fails
- Only ~5 million sessions (roughly 1/10th) remap to neighboring servers
- Database load increases moderately
- Response times stay acceptable
- Users barely notice

This scenario shows why system designers care about hashing strategies. It's not academic—it directly impacts whether your platform stays up or crashes under real-world hardware failures. Knowing the difference between modulo hashing and consistent hashing can be the difference between a resilient system and one that collapses at the worst times.

## Key Takeaway

Modulo hashing is intuitive and works fine for single-machine scenarios or non-distributed systems where server count never changes. But for production distributed systems, it's too brittle. This is why consistent hashing became a fundamental technique in system design.

## References

[Toptal - A Guide to Consistent Hashing](https://www.toptal.com/big-data/consistent-hashing)

[HellowInteview - Consistent Hashing](https://www.hellointerview.com/learn/system-design/core-concepts/consistent-hashing)

[GeeksforGeeks - Hashing in Distributed Systems](https://www.geeksforgeeks.org/cloud-computing/hashing-in-distributed-systems/)

[High Scalability - Consistent hashing algorithm](https://highscalability.com/consistent-hashing-algorithm/)

[P2P Systems and Distributed Hash Tables (Section 9.4.2)](https://www.cs.princeton.edu/courses/archive/spr11/cos461/docs/lec22-dhts.pdf)
