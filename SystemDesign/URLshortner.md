# URL Shortener — Deep System Design Guide

---

## The Problem

You have a long URL:

```
https://amazon.com/gaming-laptop-intel-i7-16gb-rtx4060-ssd-512gb
```

You want a short, shareable link:

```
tiny.ly/aB9xK2Q
```

When someone clicks it — they land on the original page.

That's it. Simple idea. Hard to scale.

---

## How to Think About Any System 

Before touching technology, ask these questions:

```
1. What does this system DO?
2. What data does it need to store?
3. What happens most — reads or writes?
4. What breaks when 1 million people use it?
5. How do we fix what breaks?
```

For a URL shortener:

- It **creates** short codes (write)
- It **redirects** visitors (read)
- Reads happen way more than writes — 100x more
- So the read path must be extremely fast

---

## Step 1 — Core Data Model

Strip everything away. What's the minimum data we need?

```
short_code    →    original_url
aB9xK2Q      →    amazon.com/laptop
```

That's a key-value pair. One input, one output.

We also want to know:

```
short_code    |    original_url              |    created_at    |    click_count
aB9xK2Q      |    amazon.com/laptop         |    2024-01-01    |    48302
```

Simple table. Nothing complex yet.

---

## Step 2 — Generating the Short Code

We need a short code that is:

- Unique (no two URLs get the same code)
- Short (7 characters is enough)
- Not guessable

### The wrong way

Use auto-increment numbers: 1, 2, 3, 4...

Problem: anyone can guess the next URL.

```
tiny.ly/1
tiny.ly/2
tiny.ly/3   ← anyone can discover your private links
```

### The right way — Base62

Take a unique number. Encode it using 62 characters:

```
a-z   (26)
A-Z   (26)
0-9   (10)
────────────
Total: 62 characters
```

A 7-character Base62 code gives you:

```
62⁷ = 3,578,414,841,792 combinations
```

That's 3.5 trillion short URLs. Enough.

**Analogy:** Think of it like a license plate. Instead of only digits (0-9), you use letters AND digits , so you get way more combinations from fewer characters.

---

## Step 3 — Write Flow (Creating a Short URL)

```
User submits long URL
        │
        ▼
   API Server
   (validates the URL, checks it's not spam)
        │
        ▼
   ID Generator
   (generates a unique number → converts to Base62 → aB9xK2Q)
        │
        ▼
   Database
   (stores: aB9xK2Q → amazon.com/laptop)
        │
        ▼
   Returns short URL to user
```

**Analogy:** Like a post office. You bring a package (long URL). They give it a tracking number (short code). They store the package. When someone gives back the tracking number, they find the package.

---

## Step 4 — Read Flow (Clicking a Short URL)

This is the critical path. It must be fast.

```
User clicks tiny.ly/aB9xK2Q
        │
        ▼
   Load Balancer
   (decides which server handles this request)
        │
        ▼
   Redirect Service
   (looks up aB9xK2Q)
        │
        ├──→ Cache (Redis) ──→ FOUND → return URL instantly
        │
        └──→ Database ───────→ NOT IN CACHE → fetch → store in cache → return URL
        │
        ▼
   302 Redirect
   (browser goes to original URL)
```

**Analogy:** Like a librarian. First they check the sticky notes on their desk (cache). If it's there — instant answer. If not, they walk to the shelves (database), find it, then write a sticky note for next time.

---

## Step 5 — Storage Design

### Which database?

For URL shorteners, use a **relational database** (like PostgreSQL) or a **key-value store** (like DynamoDB).

Why?

- The data structure is simple — just key → value
- We need fast lookups by short code
- We don't need complex joins

**Add an index on short_code.** This makes lookups almost instant, like a book's index page.

**Analogy:** A phone book sorted by name. You don't read every page — you jump straight to the right letter.

### SQL vs NoSQL

| Situation | Use |
|---|---|
| Small-medium scale, need analytics | PostgreSQL |
| Massive scale, billions of URLs | DynamoDB / Cassandra |

---

## Step 6 — Caching

### What is a cache?

Temporary memory that stores the most recently used data.

Instead of hitting the database every time — you hit memory first.

Memory is 100-1000x faster than a database disk read.

**Analogy:** Sticky notes on your desk. Instead of opening the filing cabinet every time someone asks for a phone number — you write the most common ones on sticky notes. Fast, convenient.

### Cache hit vs cache miss

- **Cache hit** — data was in cache. Return instantly.
- **Cache miss** — not in cache. Go to DB. Store result in cache. Return.

### Cache-aside pattern

```
1. Check cache
2. If found (HIT) → return it
3. If not found (MISS) → query DB → store in cache → return it
```

### What to cache?

The most popular short codes. 20% of links get 80% of the clicks (Pareto principle). Caching just those 20% eliminates most DB load.

### Cache expiry (TTL)

Cached data gets a time limit. After the limit — it expires and the next request fetches fresh data.

**Why?** If someone updates the destination URL, you don't want users stuck with the old cached version forever.

---

## Step 7 — Redirect Types

When someone clicks a short URL, the server sends back a redirect code.

### 301 — Permanent Redirect

- Browser saves this permanently
- Future clicks go directly to the original URL (skip our server)
- We never see the request again
- **Problem:** We can't track clicks

### 302 — Temporary Redirect

- Browser checks our server every single time
- We can count every click
- We can track country, device, browser
- **Use this for URL shorteners**

**Analogy:**
- 301 is like giving someone your new home address permanently — they never need to ask again.
- 302 is like telling them to call your receptionist each time — who then directs them. The receptionist logs every call.

---

## Step 8 — Analytics (Async Processing)

### The wrong approach

```
User clicks
  → Record analytics
  → Write to database
  → THEN redirect
```

Problem: user waits for all that work before seeing the destination.

### The right approach

```
User clicks
  → Redirect immediately  (user is gone, happy)
  → Fire event to queue   (background, no wait)
        │
        ▼
   Kafka (event queue)
        │
        ▼
   Analytics Workers
   (process at their own pace)
        │
        ▼
   Analytics Database
   (clicks, country, device, browser, timestamp)
```

**This is called async processing.** Do the important thing first (redirect). Do the non-urgent thing in the background.

### What is Kafka?

A system that receives events and queues them up for workers to process.

**Analogy:** A restaurant order system. Waiter takes your order (event). Passes it to the kitchen on a ticket (Kafka). Kitchen processes tickets in order. You don't wait at the counter while they cook — you sit down.

### What data do we store?

```
short_code    |    timestamp          |    country    |    device     |    browser
aB9xK2Q      |    2024-01-01 14:22   |    OM         |    mobile     |    Chrome
```

---

## Step 9 — Scaling

### What breaks first?

With 1 million users per day:

1. **Single server** gets overwhelmed → add more servers
2. **Single database** becomes the bottleneck → add cache + read replicas
3. **Database too large** → shard it

### Load Balancer

Distributes incoming traffic across multiple servers. If one server dies, others keep running.

**Analogy:** A supermarket with multiple checkout counters. One cashier per person = queue disaster. One cashier per lane = fast.

### Replication

Create copies of your database. One copy is the primary (handles writes). Others are replicas (handle reads).

**Analogy:** Photocopies of a notebook. Multiple people can read different copies at the same time. One person updates the original.

### Sharding

Split your database across multiple machines.

```
Users A-M  →  Database 1
Users N-Z  →  Database 2
```

Each machine handles a fraction of the data.

**Analogy:** A library split into buildings. Building 1 for fiction. Building 2 for non-fiction. Smaller buildings, faster to search.

---

## Full Architecture Summary

```
                        ┌─────────────┐
                        │    Client   │
                        └──────┬──────┘
                               │
                        ┌──────▼──────┐
                        │Load Balancer│  ← spreads traffic
                        └──────┬──────┘
                               │
               ┌───────────────┼───────────────┐
               │               │               │
        ┌──────▼─────┐  ┌──────▼─────┐  ┌──────▼─────┐
        │ API Server │  │ API Server │  │ API Server │  ← horizontal scale
        └──────┬─────┘  └────────────┘  └────────────┘
               │
       ┌───────┴────────┐
       │                │
┌──────▼─────┐   ┌──────▼──────┐
│   Cache    │   │  Database   │  ← cache-aside pattern
│  (Redis)   │   │ (PostgreSQL)│
└────────────┘   └──────┬──────┘
                        │
               ┌────────┴────────┐
               │                 │
        ┌──────▼──────┐  ┌───────▼──────┐
        │  Primary DB │  │  Replica DB  │  ← replication
        └─────────────┘  └──────────────┘

── Analytics path ──────────────────────────

Click Event → Kafka → Analytics Workers → Analytics DB
```

---

## Key Terms Glossary

| Term | Simple meaning | Analogy |
|---|---|---|
| **Base62** | Encode numbers using letters + digits | License plate with letters AND numbers |
| **Cache** | Fast memory storage for common data | Sticky notes on your desk |
| **Cache hit** | Data found in cache — instant return | Found the sticky note |
| **Cache miss** | Not in cache — go to database | No sticky note, open the filing cabinet |
| **Cache-aside** | Check cache first, query DB on miss | Check sticky notes, then filing cabinet |
| **302 redirect** | Temporary redirect — checks server every time | Hotel stays — receptionist re-routes every visit |
| **301 redirect** | Permanent redirect — browser skips server | New home address — no need to ask again |
| **Load balancer** | Distributes traffic across servers | Supermarket with multiple checkout lanes |
| **Replication** | Copies of the database | Photocopies of a notebook |
| **Sharding** | Split database across machines | Library split into multiple buildings |
| **Async** | Do work in background without blocking the user | Restaurant — sit down while kitchen cooks |
| **Kafka** | Event queue system | Restaurant ticket system between waiter and kitchen |
| **Rate limiting** | Cap requests per user per time window | One free refill per customer per visit |
| **Idempotency** | Same request → same result, safe to retry | Pressing elevator button twice still goes to same floor |
| **TTL** | Time-to-live — how long cached data lives | Sticky note with an expiry date |
| **Index** | Makes database lookups fast | Book index page — jump straight to the right section |

---

```
Step 1 — Clarify requirements (2 minutes)
  "How many users? Read-heavy or write-heavy?
   Do we need analytics? Any expiry on links?"

Step 2 — Estimate scale (2 minutes)
  "100M URLs created per day = ~1200 writes/sec
   10B clicks per day = ~115,000 reads/sec"

Step 3 — Core data model (2 minutes)
  "What's the simplest data structure that solves this?"

Step 4 — API design (2 minutes)
  "POST /shorten  →  returns short URL
   GET /:code    →  redirects to original"

Step 5 — High-level architecture (5 minutes)
  Draw: Client → LB → API → Cache → DB

Step 6 — Deep dive (5 minutes)
  Pick the hardest part and go deep
  (short code generation, cache strategy, analytics)

Step 7 — Scaling discussion (3 minutes)
  "Here's what breaks first and how I'd fix it"
```



**Q1: How do you guarantee short codes are unique across multiple servers?**

Naive approach: each server generates its own IDs — collisions possible.

Better approach: use a **centralized ID generator** (like Twitter's Snowflake).

Snowflake generates unique IDs using:
- Timestamp (milliseconds)
- Machine ID
- Sequence number within that millisecond

Result: every ID is globally unique without coordination between servers.

Alternative: use a database sequence (auto-increment) as the source of truth. Each server requests a range of IDs (e.g. server 1 gets 1–1000, server 2 gets 1001–2000). No collisions. Each server converts its allocated number to Base62.

---

**Q2: What happens when the cache is full?**

Cache has limited memory. When full, it must evict (remove) old entries.

Common eviction policies:

- **LRU (Least Recently Used)** — remove the entry not accessed for the longest time
- **LFU (Least Frequently Used)** — remove the entry accessed the fewest times
- **TTL expiry** — entries expire after a set time and are removed automatically

For URL shorteners: LRU makes sense. Popular links stay cached. Links nobody clicks get evicted.

---

**Q3: How would you handle 100,000 redirects per second?**

At this scale, a single server and database cannot handle the load.

Solution stack:

1. **Multiple API servers** behind a load balancer
2. **Redis cache** handles ~90% of reads (most popular links)
3. **Read replicas** for the remaining DB reads
4. **CDN** for the most viral links (cache at edge, closest to user)

The database barely gets touched. Almost everything is served from cache or CDN.

---

**Q4: How do you prevent abuse — someone creating millions of spam URLs?**

Multiple layers:

- **Rate limiting** — max N URLs created per IP per minute
- **Authentication** — require an account to create URLs
- **CAPTCHA** — stop automated scripts
- **URL validation** — scan destination URL against blocklists (malware, phishing)
- **Blacklist** — ban IPs or accounts that abuse the system

---

**Q5: How would you design link expiry — short URLs that stop working after 30 days?**

Store an `expires_at` column in the database:

```
short_code  |  original_url     |  expires_at
aB9xK2Q    |  amazon.com/...   |  2024-02-01 00:00:00
```

On every redirect request:

```
1. Look up short_code
2. Check if expires_at < now
3. If expired → return 410 Gone
4. If valid → redirect
```

For cleanup: run a background job nightly that deletes expired rows. This keeps the database lean. Also cache the expiry check result so expired links don't hammer the DB.

---

## One-Line Summary

> Generate a Base62 short code → store the mapping → serve reads via Redis cache →
> redirect with 302 → track clicks async via Kafka → scale with sharding and replication.