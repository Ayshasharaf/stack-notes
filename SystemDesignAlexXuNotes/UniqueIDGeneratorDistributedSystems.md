# Designing a Unique ID Generator in Distributed Systems

## Why this matters : where unique IDs show up


| Use case     | What the ID identifies         |
| ------------ | ------------------------------ |
| Social media | Post ID, comment ID            |
| YouTube      | Video ID                       |
| Spotify      | Song ID, playlist ID           |
| E-commerce   | Order ID                       |
| Banking      | Transaction ID                 |
| IoT          | Device ID, event ID            |
| Kafka        | Message offset / partition key |


The common thread: a single machine can't generate every ID (too much load, single point of failure), so many machines must generate IDs **independently** while guaranteeing no two ever collide.

`auto_increment` (like MySQL's `AUTO_INCREMENT`) only works when there's **one** database/server owning the counter. The moment you scale horizontally, you need a distributed strategy.

---



## Requirements

- **Unique** : no collisions, ever.
- **Numeric only** : no letters/symbols.
- **Fits in 64 bits** : compact, index-friendly, sortable as an integer.
- **Ordered by time (roughly)** : newer IDs > older IDs, so you can sort by ID instead of a separate timestamp column.
- **High throughput** : support 10,000+ IDs/second.

> `[IMAGE PLACEHOLDER: table summarizing the 5 requirements above as icons]`

---



## Approach 1 : Multi-master replication (range-based auto-increment)

Each DB server increments by `k` (the total number of servers) instead of by 1.

Example with 4 servers:

- Server 1 → 1, 5, 9, 13...
- Server 2 → 2, 6, 10, 14...
- Server 3 → 3, 7, 11, 15...
- Server 4 → 4, 8, 12, 16...

> `[IMAGE PLACEHOLDER: diagram : 4 DB servers, each labeled with its increment sequence, client writes routed to different servers]`

**Pros**

- Numeric, simple to understand, no external coordinator needed for the increment itself.

**Cons**

- Hard to scale across multiple data centers.
- IDs don't increase monotonically with time *across* servers (server 4's ID 8 can be created before server 1's ID 9 that comes later in real time).
- Adding/removing a server breaks the increment pattern : you have to recompute the offsets.

---



## Approach 2 : UUID (Universally Unique Identifier)

Each server generates its own 128-bit random/pseudo-random ID locally : no coordination between servers at all.

Example: `09c93e62-50b4-468d-bf8a-c07e1040bf47`

> `[IMAGE PLACEHOLDER: diagram : multiple web servers each independently generating a UUID, no communication between them]`

**Pros**

- Dead simple to generate.
- No synchronization between servers → no synchronization bugs.
- Scales linearly with the number of servers generating IDs.

**Cons**

- 128 bits : we need 64 bits.
- Not time-ordered (random UUIDs don't sort chronologically).
- Can be non-numeric (contains letters).

---



## Approach 3 : Ticket server (Flickr's approach)

One centralized server owns a database table with an `auto_increment` column. Every ID request goes to this server, which hands out the next number.

> `[IMAGE PLACEHOLDER: diagram : many app servers all requesting IDs from a single centralized "ticket server", ticket server backed by a DB with auto_increment]`

**Pros**

- Numeric IDs.
- Easy to implement; fine for small/medium-scale systems.

**Cons**

- **Single point of failure.** If it goes down, nothing that depends on it can generate IDs.
- Running multiple ticket servers to fix this reintroduces the original problem: keeping them in sync.

---



## Approach 4 : Twitter Snowflake (the go-to answer in interviews)

Instead of coordinating between machines, **encode enough information into the ID itself** so each machine can generate IDs independently, yet the IDs still stay roughly time-ordered and globally unique.

A Snowflake ID is a 64-bit integer split into fields:


| Field           | Bits | Purpose                                                                                                         |
| --------------- | ---- | --------------------------------------------------------------------------------------------------------------- |
| Sign bit        | 1    | Always 0. Reserved / keeps the number non-negative.                                                             |
| Timestamp       | 41   | Milliseconds since a custom epoch (Twitter's default: Nov 04, 2010, 01:42:54 UTC). Gives ordering by time.      |
| Datacenter ID   | 5    | Up to 2⁵ = 32 datacenters.                                                                                      |
| Machine ID      | 5    | Up to 2⁵ = 32 machines per datacenter.                                                                          |
| Sequence number | 12   | Increments per ID generated **within the same millisecond** on that machine. Resets to 0 every new millisecond. |


**Total: 1 + 41 + 5 + 5 + 12 = 64 bits.**

Twitter Snowflake Architecture

**Why it works**

- Timestamp bits ⇒ IDs are ordered by time.
- Datacenter + machine ID ⇒ every machine has its own "namespace," so two machines can never generate the same ID even at the exact same millisecond.
- Sequence number ⇒ handles bursts : one machine can emit up to 4096 (2¹²) unique IDs within a single millisecond.

**Capacity check:** 1 machine × 4096 IDs/ms = 4,096,000 IDs/sec on a *single* machine : comfortably clears the 10,000/sec requirement, and it scales horizontally across up to 1024 machines (32 datacenters × 32 machines each).

**The one critical dependency: clock synchronization**

Because the timestamp field is what gives ordering and (partly) uniqueness, all machines must agree on time. If a machine's clock drifts backward (e.g., after an NTP correction), it could generate an ID that collides with : or falls before : one it already issued.

Mitigations:

- Use **NTP (Network Time Protocol)** to keep clocks synced across machines.
- Detect clock drift/rollback and refuse to generate IDs (or wait) until the clock catches back up.



## Twitter Snowflake Architecture



## Comparing the four approaches


| Approach                       | Numeric | Time-ordered       | Fits 64-bit | Single point of failure    | Scales well                 |
| ------------------------------ | ------- | ------------------ | ----------- | -------------------------- | --------------------------- |
| Multi-master (range increment) | ✅       | ❌ (across servers) | ✅           | ❌                          | ⚠️ Hard when servers change |
| UUID                           | ❌       | ❌                  | ❌ (128-bit) | ✅ None                     | ✅                           |
| Ticket server                  | ✅       | ✅                  | ✅           | ⚠️ Yes (unless replicated) | ⚠️ Limited                  |
| Twitter Snowflake              | ✅       | ✅                  | ✅           | ✅ None                     | ✅                           |


**Snowflake is the standard answer** because it's the only approach that satisfies *all* the stated requirements (numeric, 64-bit, time-ordered, high throughput, no single point of failure) without needing servers to talk to each other on every ID request.

---



## Quick recap

- Auto-increment breaks down once you go distributed : no single owner of "the next number."
- Multi-master range increments and UUIDs solve uniqueness but fail time-ordering or bit-size requirements.
- A ticket server is simple but centralizes risk.
- Snowflake solves it by encoding *time + location + sequence* directly into the ID, trading a coordination problem for a clock-synchronization problem.

