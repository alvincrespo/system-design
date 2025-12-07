# Consistent Hashing

## Table of Contents

- [Overview](#overview)
- [The Problem It Solves](#the-problem-it-solves)
- [How Consistent Hashing Works](#how-consistent-hashing-works)
  - [The Hash Ring Concept](#the-hash-ring-concept)
  - [The Algorithm](#the-algorithm)
  - [Visual Example](#visual-example)
- [Adding or Removing Servers](#adding-or-removing-servers)
  - [Adding a Server](#adding-a-server)
  - [Removing a Server](#removing-a-server)
- [Virtual Nodes](#virtual-nodes)
  - [The Problem: Uneven Distribution](#the-problem-uneven-distribution)
  - [The Solution: Virtual Nodes](#the-solution-virtual-nodes)
- [Scenarios](#scenarios)
- [Key Implementation Details](#key-implementation-details)
- [Limitations and Extensions](#limitations-and-extensions)
- [Key Takeaway](#key-takeaway)
- [References](#references)

## Overview

Consistent hashing is a distributed hashing technique that minimizes data redistribution when servers are added or removed from a distributed system. Unlike modulo hashing, which remaps nearly all keys when the cluster changes, consistent hashing ensures that only a small fraction of keys need to be remapped. This technique is fundamental to building scalable, fault-tolerant systems.

## The Problem It Solves

Consistent hashing solves the **cache invalidation problem** created by modulo hashing. When you use `serverIndex = hash(key) % N`, adding or removing a server changes `N`, causing most keys to remap to different servers and destroying your cache's effectiveness.

Consistent hashing decouples the mapping from the number of servers, so server topology changes only affect a small portion of keys.

## How Consistent Hashing Works

### The Hash Ring Concept

Instead of using modulo to map keys to a fixed number of servers, consistent hashing uses a **hash ring** (also called a hash circle). Here's the mental model:

1. Imagine all possible hash values arranged in a circle (like a clock face)
2. Hash values range from 0 to 2^32 - 1 (or another large number)
3. The minimum value (0) connects back to the maximum value, forming a ring
4. Both keys and servers are hashed onto this same ring

### The Algorithm

**Step 1: Hash the servers**
- Take each server's identifier (IP address, name, etc.)
- Compute its hash value
- Place it at that position on the ring

**Step 2: Hash the keys**
- Take each key (user ID, request, etc.)
- Compute its hash value
- Place it at that position on the ring

**Step 3: Map keys to servers**
- For any key, walk clockwise from its position on the ring
- Find the first server you encounter
- That server is responsible for that key

### Visual Example

Imagine a ring with positions 0-360 (degrees). You have 3 servers:
- Server A hashes to position 30
- Server B hashes to position 120
- Server C hashes to position 240

Now you have keys:
- Key X hashes to position 45 → walk clockwise → find Server B (at 120)
- Key Y hashes to position 150 → walk clockwise → find Server C (at 240)
- Key Z hashes to position 280 → walk clockwise → find Server A (at 30, wrapping around)

## Adding or Removing Servers

### Adding a Server

When you add Server D hashing to position 200:
- Only keys between Server C (240) and Server D (200, going backwards) get remapped
- All other keys stay on their original servers
- In a typical ring with many servers, this affects roughly 1/N of the keys

**Before:** Key at 180 walks clockwise → hits Server C (240)
**After:** Key at 180 walks clockwise → now hits Server D (200) instead

Keys at position 250 still hit Server A (30). Keys at 100 still hit Server B (120). Stability maintained.

### Removing a Server

When Server B (at 120) fails:
- Only keys that were mapped to Server B get remapped
- They now walk clockwise and hit the next server (Server C)
- Everything else stays put

**Before:** Key at 100 walks clockwise → hits Server B (120)
**After:** Key at 100 walks clockwise → now hits Server C (240) instead

Keys at 150 still hit Server C (240). Keys at 280 still hit Server A (30). Minimal disruption.

## Virtual Nodes

### The Problem: Uneven Distribution

If you only hash servers once on the ring, you might get uneven distribution. Imagine 2 servers that happen to hash to positions 10 and 300. All keys between 300-10 go to one server, while keys between 10-300 go to the other. Imbalanced load.

### The Solution: Virtual Nodes

Each physical server gets multiple positions on the ring (usually 100-150 virtual nodes per physical server). This is done by appending suffixes to the server identifier before hashing:

- Server A becomes: `Server A#1`, `Server A#2`, `Server A#3`, ..., `Server A#150`
- Server B becomes: `Server B#1`, `Server B#2`, `Server B#3`, ..., `Server B#150`

Each variant gets hashed separately, creating 150 different positions on the ring for Server A.

**Result:** Keys are spread much more evenly across physical servers. If one server fails, the redistributed keys go to multiple servers instead of one, balancing the load.

## Scenarios

View real-world scenarios where consistent hashing shines [here](../SCENARIOS/CONSISTENCY_HASHING.md).

For a visualization of consistent hashing, check out this [interactive demo](https://codepen.io/alvincrespo/full/xbVmxmj).

## Key Implementation Details

### Hash Function
Commonly MD5 or SHA-1 (historically) or SHA-256 (modern). Must be:
- Deterministic (same input always produces same output)
- Uniformly distributed
- Fast to compute

### Ring Size
Usually 2^32 (0 to 4,294,967,295) or 2^64. Larger ring = lower collision risk.

### Clockwise Walk
When looking up a key, you traverse clockwise until you find a server. This is O(log N) with proper implementation using binary search on server positions.

## Limitations and Extensions

**Skewed Distribution:** If servers hash unevenly, load becomes unbalanced. Virtual nodes solve this.

**Weighted Hashing:** Some systems give certain servers more virtual nodes if they have more capacity.

**Weighted Consistent Hashing:** Servers can have different weights in the ring, proportional to their capacity.

## Key Takeaway

Consistent hashing is a cornerstone technique in distributed systems. It transforms the server scaling problem from "lose most of your cache" to "lose a small, manageable portion." This small difference enables systems to:
- Handle server failures gracefully
- Scale capacity up or down without downtime
- Maintain cache hit rates even as infrastructure changes
- Distribute load fairly across servers

If you're designing any distributed system that needs fault tolerance or elastic scaling, consistent hashing should be in your toolkit.

## References

[Toptal - A Guide to Consistent Hashing](https://www.toptal.com/big-data/consistent-hashing)

[HellowInteview - Consistent Hashing](https://www.hellointerview.com/learn/system-design/core-concepts/consistent-hashing)

[GeeksforGeeks - Hashing in Distributed Systems](https://www.geeksforgeeks.org/cloud-computing/hashing-in-distributed-systems/)

[High Scalability - Consistent hashing algorithm](https://highscalability.com/consistent-hashing-algorithm/)

[P2P Systems and Distributed Hash Tables (Section 9.4.2)](https://www.cs.princeton.edu/courses/archive/spr11/cos461/docs/lec22-dhts.pdf)

[JumpBackHash Research Paper](https://arxiv.org/html/2403.18682)

[CS 168: The Modern Algorithmic Toolbox, Spring 2024](https://web.stanford.edu/class/cs168/)

[Consistent Hashing](https://tom-e-white.com/2007/11/consistent-hashing.html)
