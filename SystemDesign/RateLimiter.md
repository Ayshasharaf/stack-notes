# Rate Limiter — Deep System Design Guide

---

## The Problem

Imagine you run an API. Anyone can call it.

Without a rate limiter:
- A single user sends 1,000,000 requests per second
- Your server crashes
- Every other user is blocked

A rate limiter says:

```
"You can make 100 requests per minute. That's it."
```

After 100 — you get blocked until the next window.

**Analogy:** A nightclub bouncer. There's a limit of 200 people inside. When it's full — nobody new gets in until someone leaves.

---

## How to Think About It

```
1. What are we protecting?        → an API, a service, a database
2. Who are we limiting?           → by IP, by user, by API key
3. What is the limit?             → 100 req/min, 1000 req/day
4. What happens when exceeded?    → return 429 Too Many Requests
5. Where does the limiter live?   → at the API gateway, or inside each service
```

---

## Step 1 — Core Data Model

For every user (or IP), we track:

```
user_id    |    request_count    |    window_start
u123       |    87               |    14:00:00
```

Every request: check the count. If under limit — allow and increment. If over — reject.

Simple. But the *how* of counting matters a lot.

---

## Step 2 — The Algorithms

This is the heart of rate limiting. There are four main algorithms.

---

### Algorithm 1 — Fixed Window Counter

Divide time into fixed buckets (e.g. every minute).

Count requests in the current bucket.

```
14:00:00 → 14:00:59   =  window 1  →  max 100 req
14:01:00 → 14:01:59   =  window 2  →  max 100 req
```

**Problem — the boundary attack:**

```
14:00:59  →  user sends 100 requests  (last second of window 1)
14:01:00  →  user sends 100 requests  (first second of window 2)
```

In 2 seconds: 200 requests got through. Double the limit. The window reset helped them cheat.

**Analogy:** An all-you-can-eat restaurant that resets at midnight. Show up at 11:59pm, eat. Come back at 12:01am, eat again.

---

### Algorithm 2 — Sliding Window Log

Store a timestamp for every request in a list.

When a new request comes in:

```
1. Remove all timestamps older than 1 minute
2. Count remaining timestamps
3. If count < limit → allow, add new timestamp
4. If count >= limit → reject
```

**No boundary attack.** Always looks at the last 60 real seconds.

**Problem:** Stores every timestamp. High memory usage at scale. 1 million users × 100 requests = 100 million log entries.

**Analogy:** A bouncer who keeps a handwritten list of every person who entered in the last hour, with timestamps. Accurate — but a lot of paperwork.

---

### Algorithm 3 — Sliding Window Counter (Best for most cases)

A hybrid. Less memory than log, more accurate than fixed window.

```
Current window count  +  (previous window count × overlap fraction)
```

Example:

```
Limit: 100 req/min
Previous window (14:00): 80 requests
Current window (14:01): 30 requests, we're 25% through this window

Estimated count = 30 + (80 × 0.75) = 30 + 60 = 90
```

Under 100. Allow.

**Analogy:** Estimating how busy a road was in the last hour by combining what you know from the current hour and the previous hour, weighted by time.

---

### Algorithm 4 — Token Bucket (Most flexible)

Imagine a bucket that holds tokens.

```
- Bucket capacity: 10 tokens
- Tokens added: 1 per second
- Each request costs 1 token
```

Request comes in:
- Tokens available → take one, allow request
- No tokens → reject

Unused tokens accumulate up to the bucket capacity.

**This allows bursts.** A user who didn't use the API for 10 seconds has 10 tokens — can fire 10 requests instantly.

**Analogy:** A prepaid phone plan. You get 100 texts/month. Unused texts don't roll over past 100. But you can send all 100 in one day if you want.

---

### Algorithm 5 — Leaky Bucket (Smoothest output)

Requests go into a queue (the bucket). They leak out at a fixed rate.

```
Queue: [req1, req2, req3, req4, req5]
         │
         ▼ (process 1 per 10ms)
```

No matter how many requests arrive — they're processed at a steady rate.

Overflow: if the bucket is full, new requests are dropped.

**Good for:** payment processors, billing systems — where you need a consistent, predictable output rate.

**Analogy:** A funnel. Pour water in fast or slow — it always drips out at the same rate. Overflow spills.

---

### Which algorithm to use?

| Algorithm | Best for | Trade-off |
|---|---|---|
| Fixed window | Simple counters, low precision needed | Boundary attack vulnerability |
| Sliding window log | High accuracy required | High memory |
| Sliding window counter | General purpose APIs | Slight approximation |
| Token bucket | APIs that allow bursts | Slightly complex |
| Leaky bucket | Smooth, consistent output | No burst allowed |

**Default choice for most APIs: Token bucket or Sliding window counter.**

---

## Step 3 — Where to Put the Rate Limiter

### Option A — API Gateway (Recommended)

```
Client → API Gateway (rate limit here) → Backend Services
```

One place. All services are protected. Clients get rejected before wasting backend resources.

**Analogy:** Security check at the airport entrance — not at every gate.

### Option B — Inside each service

Every service checks limits itself.

More flexible. But duplicated logic. Harder to maintain.

**Analogy:** Every shop inside a mall doing its own security check.

### Option C — Client-side

Never rely on this. Clients can bypass it easily.

---

## Step 4 — Storage: Where to Store the Counters

### In-memory (per server)

Each server tracks its own counters.

**Problem:** You have 10 servers. Each server only sees 1/10 of the requests. User sends 1000 requests spread across 10 servers — each server sees 100. Nobody blocks them.

**Analogy:** 10 bouncers each counting separately. The user walks between them. Each bouncer only counts their own entries.

### Redis (Centralized — correct approach)

All servers share one Redis instance. All counters are centralized.

```
Server 1 ──┐
Server 2 ──┼──→ Redis (single source of truth) → user:123 = 87 requests
Server 3 ──┘
```

Every server checks and updates the same counter. Consistent.

Redis operations used:

```
INCR user:123          → atomically increment counter
EXPIRE user:123 60     → reset after 60 seconds
```

**Atomic** means: increment and check happen as one operation. No race condition where two servers both see "99" and both allow the 100th request.

**Analogy:** One shared scoreboard all bouncers look at and update. Everyone stays in sync.

---

## Step 5 — The Request Flow

```
Client sends request
        │
        ▼
   API Gateway
        │
        ▼
   Rate Limiter middleware
        │
        ├── Get user ID / IP from request
        │
        ├── Check Redis counter for this user
        │
        ├── Counter < limit?
        │       │
        │       ├── YES → increment counter → forward request to backend
        │       │
        │       └── NO  → return 429 Too Many Requests
        │                  + header: Retry-After: 43
        │
        ▼
   Backend Service
```

### Response headers (good practice)

Always tell the client what's happening:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 13
X-RateLimit-Reset: 1704067200
Retry-After: 43
```

**Why?** Good clients (and developers) use these headers to back off gracefully instead of hammering your API.

---

## Step 6 — Different Limit Types

Rate limiting is not one-size-fits-all.

| Limit type | Example | Use case |
|---|---|---|
| Per IP | 100 req/min per IP | Unauthenticated endpoints |
| Per user | 1000 req/day per user | Authenticated APIs |
| Per API key | 10,000 req/hour per key | Developer API plans |
| Per endpoint | /login max 5 req/min | Sensitive endpoints |
| Global | Total 1M req/sec | Protect the whole system |

You can combine them:

```
Global limit: 1M req/sec
  AND per-user limit: 1000 req/min
    AND /login endpoint: 5 attempts/min
```

---

## Step 7 — Handling Edge Cases

### Race condition

Two requests arrive at the exact same millisecond. Both check Redis. Both see count = 99. Both increment to 100. Both get allowed. Now count is 100 but the limit is 100 — two extra slipped through.

**Fix:** Use Redis atomic operations (`INCR` + `EXPIRE` in a Lua script). Atomic means the check and increment happen as one uninterruptible step.

### Distributed Redis failure

If Redis goes down — do you block all traffic or allow all traffic?

**Fail open:** If Redis is unavailable, allow the request. Availability is prioritized.
**Fail closed:** If Redis is unavailable, reject the request. Security is prioritized.

Choose based on your system. Payment APIs: fail closed. Casual read APIs: fail open.

### Users sharing an IP (NAT)

An office of 1000 people shares one public IP. Your per-IP limit of 100 req/min blocks the whole office.

**Fix:** Always prefer per-user or per-API-key limits over per-IP when users are authenticated.

---

## Step 8 — Scaling the Rate Limiter

### Single Redis

Works up to millions of requests per second. Redis is fast — 100,000+ ops/sec on a single node.

### Redis Cluster

Shard Redis across multiple nodes. Each node handles a slice of the users.

```
user hash % 3 = node to use
```

### Local cache + Redis

To reduce Redis round-trips:

```
1. Check local in-memory cache first (< 1ms)
2. If not found — check Redis
3. Sync local cache with Redis every 100ms
```

Slight accuracy trade-off for much lower latency. Acceptable for most APIs.

---

## Full Architecture Summary

```
                         Client
                            │
                            ▼
                    ┌───────────────┐
                    │  API Gateway  │
                    │ (rate limiter │
                    │  middleware)  │
                    └───────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
       ┌──────▼──────┐      │      ┌──────▼──────┐
       │  Check      │      │      │  Return 429  │
       │  Redis      │      │      │  Too Many    │
       │  counter    │      │      │  Requests    │
       └──────┬──────┘      │      └─────────────┘
              │             │
          Under limit?      │
              │             │
           YES │          NO │
              │             └──────────────────┐
              ▼                                │
       ┌──────────────┐                ┌───────▼──────┐
       │  Increment   │                │   Redis      │
       │  counter in  │                │   counter    │
       │  Redis       │                │   at limit   │
       └──────┬───────┘                └──────────────┘
              │
              ▼
       ┌──────────────┐
       │   Backend    │
       │   Service    │
       └──────────────┘

── Redis layer ─────────────────────────────────────

All API Gateway nodes → shared Redis cluster
  user:123 → { count: 87, window: 14:00 }
  user:456 → { count: 12, window: 14:00 }
```

---

## Key Terms Glossary

| Term | Simple meaning | Analogy |
|---|---|---|
| **Rate limiter** | Caps how many requests a user can make | Nightclub bouncer with a headcount limit |
| **Fixed window** | Count resets at fixed time intervals | All-you-can-eat that resets at midnight |
| **Sliding window** | Always looks at the last N real seconds | Rolling 60-second view of activity |
| **Token bucket** | Earn tokens over time, spend them on requests | Prepaid phone plan with rollover |
| **Leaky bucket** | Process requests at a fixed rate, queue the rest | A funnel — output is always steady |
| **429** | HTTP status meaning "too many requests" | Bouncer turning you away |
| **Redis** | Fast in-memory store for shared counters | Shared scoreboard all bouncers read |
| **Atomic operation** | Check + update happen as one step, no interruption | One teller handles your whole transaction |
| **API Gateway** | Entry point that handles all incoming requests | Airport security check before any gate |
| **Fail open** | If limiter breaks, allow traffic through | Barrier gate stuck open — cars pass |
| **Fail closed** | If limiter breaks, block all traffic | Barrier gate stuck closed — cars stop |
| **TTL** | Time-to-live — counter resets after this time | Parking meter that resets every hour |
| **Retry-After** | Header telling client when to try again | "Come back in 5 minutes" sign on a shop |
| **Race condition** | Two operations collide and produce wrong result | Two people grabbing the last item simultaneously |

---



### The framework

```
Step 1 — Clarify requirements (2 min)
  "What are we rate limiting? Per user, per IP, per API key?
   What's the limit? Per second, per minute, per day?
   Is consistency critical or can we tolerate slight inaccuracy?"

Step 2 — Pick the algorithm (2 min)
  State the trade-offs.
  "I'd use token bucket because it allows natural bursts
   and is widely used in production systems like AWS."

Step 3 — Choose storage (2 min)
  "Single Redis node for simplicity.
   Redis cluster if we need to shard at massive scale."

Step 4 — Design the request flow (5 min)
  Walk through what happens on every request.
  Allow path. Reject path. Headers returned.

Step 5 — Handle edge cases (3 min)
  Race conditions. Redis failure. Shared IPs.

Step 6 — Scaling (3 min)
  How does this hold up at 10x, 100x traffic?
```




**Q1: How do you rate limit in a distributed system with 20 servers?**

Per-server counters don't work. Server 1 only sees its share of traffic.

Solution: centralized Redis. Every server reads and writes to the same counter.

Use Redis `INCR` (atomic increment) and `EXPIRE` (auto-reset after window).

```
INCR user:123:1704067200    → increment counter for this time window
EXPIRE user:123:1704067200 60  → expire after 60 seconds
```

Key includes the time window timestamp — counter automatically scoped to the window.

For very high scale: Redis cluster with consistent hashing routes each user to the same Redis shard. Same counter, same node, always.

---

**Q2: How does the token bucket algorithm work in code?**

```
last_refill = stored timestamp of last token refill
tokens      = stored token count for user

time_passed    = now - last_refill
tokens_to_add  = time_passed × refill_rate
tokens         = min(tokens + tokens_to_add, bucket_capacity)
last_refill    = now

if tokens >= 1:
    tokens -= 1
    allow request
else:
    reject request
```

Store `tokens` and `last_refill` in Redis per user. Each request recalculates how many tokens to add based on elapsed time.

---

**Q3: How do you prevent a user from being rate limited unfairly during a Redis outage?**

Two strategies:

**Fail open:** If Redis is unreachable, allow the request. Business continues. Some abuse may slip through during the outage. Acceptable for non-critical APIs.

**Local fallback:** Each server keeps a local counter. If Redis is down, fall back to local counting. Less accurate (distributed) but better than full outage.

**Circuit breaker pattern:** If Redis fails N times in a row — stop trying for X seconds. Fall back to local. Retry Redis after X seconds. Prevents thundering herd when Redis recovers.

---

**Q4: How would you implement different rate limit tiers — free vs pro vs enterprise?**

Store the user's tier alongside their ID.

```
user:123:tier  = "free"
user:456:tier  = "pro"
user:789:tier  = "enterprise"
```

When a request comes in:

```
1. Get user ID from auth token
2. Look up their tier
3. Apply tier's limit

free:       100 req/hour
pro:        10,000 req/hour
enterprise: 1,000,000 req/hour
```

Store tier config in a simple config file or database — not hardcoded. Easy to change without a deployment.

---

**Q5: How do you handle rate limiting for an API that has both cheap and expensive endpoints?**

Flat request counting is unfair.

```
GET /user/profile    →  cheap — just a DB read
POST /generate-video →  expensive — 30 seconds of compute
```

Solution: **weighted rate limiting** (cost-based).

Assign a cost to each endpoint:

```
GET  /user/profile     →  cost: 1
POST /send-email       →  cost: 5
POST /generate-video   →  cost: 50
```

User has a budget of 100 units per minute, not 100 requests.

Each request deducts its cost from the budget instead of a flat count.

```
User calls /generate-video twice: 50 + 50 = 100. Budget exhausted.
User calls /user/profile 100 times: 1 × 100 = 100. Same budget.
```

This is how OpenAI, Stripe, and AWS actually bill and limit — by tokens or compute units, not raw request count.

---

## One-Line Summary

> A rate limiter counts requests per user in a shared Redis store, uses token bucket or sliding window to track limits, rejects excess requests with 429, and scales via Redis cluster with atomic operations to prevent race conditions.