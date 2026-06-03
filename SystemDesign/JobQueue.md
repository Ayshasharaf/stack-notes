# Job Queue / Task Scheduler — Deep System Design Guide

---

## The Problem

A user clicks "Export my data as CSV."

That export takes 30 seconds to generate.

Do you make them stare at a loading spinner for 30 seconds?

No.

```
User clicks export
  → "We're generating your file. We'll email it to you."
  → User continues using the app
  → Background worker generates the CSV
  → Email sent when done
```

That background processing is a **job queue**.

**Analogy:** A restaurant. You place your order (job). Waiter writes it on a ticket (queue). Kitchen picks it up and cooks (worker). You don't stand at the counter waiting — you sit down.

---

## When Do You Need a Job Queue?

Use a job queue when a task:

- Takes too long to do synchronously (> ~1 second)
- Doesn't need to be done immediately
- Can fail and needs to be retried
- Needs to happen at a specific time (scheduled)
- Should not block the user

Examples:

```
Sending emails           → don't block the signup response
Resizing images          → happens after upload completes
Generating reports       → heavy, runs in background
Charging credit cards    → retry if payment gateway is down
Sending notifications    → fan out to millions of users
Syncing data             → run every night at 2am
```

---

## How to Think About It

```
1. What is the job?              → what work needs to be done
2. Who creates the job?          → producer
3. Where does it wait?           → the queue
4. Who does the work?            → worker (consumer)
5. What if it fails?             → retry logic
6. What if it needs scheduling?  → delay + cron
7. What if it takes too long?    → timeout + dead letter queue
```

---

## Step 1 — Core Concepts

### Producer

The part of your app that creates a job and puts it in the queue.

```
User signs up → app creates job: { type: "send_welcome_email", user_id: 123 }
```

**Analogy:** Customer placing an order.

### Queue

A list of jobs waiting to be processed. Jobs sit here until a worker picks them up.

**Analogy:** The order tickets hanging in a restaurant kitchen.

### Worker (Consumer)

A background process that picks jobs from the queue and executes them.

```
Worker picks up job → sends welcome email → marks job as done
```

**Analogy:** The chef who picks up a ticket and cooks the meal.

### Worker Pool

Multiple workers running in parallel. Each grabs a different job.

```
Worker 1 → processing job A
Worker 2 → processing job B
Worker 3 → processing job C
```

**Analogy:** Multiple chefs in the kitchen. More chefs = more orders processed simultaneously.

---

## Step 2 — The Job Lifecycle

```
CREATED → QUEUED → PROCESSING → DONE
                              → FAILED → RETRYING → DONE
                                                   → DEAD (max retries exceeded)
```

Every job has a status. This is what lets you track, retry, and debug.

---

## Step 3 — Message Queue Systems

The queue needs to be stored somewhere reliable. Options:

### Redis (Simple, fast)

Use Redis as a queue with a library like BullMQ (Node.js) or Celery (Python).

```
Producer  →  LPUSH jobs:email { user_id: 123 }
Worker    →  BRPOP jobs:email   (blocking pop — waits for jobs)
```

**Good for:** Most backend apps. Simple setup. Fast.
**Limitation:** If Redis crashes and has no persistence — jobs can be lost.

### RabbitMQ (Reliable messaging)

A dedicated message broker. Designed specifically for queues.

Features:
- Acknowledgements (worker confirms it processed the job)
- Dead letter queues (failed jobs go here)
- Routing (send different jobs to different queues)
- Persistence (jobs survive restarts)

**Good for:** When you need reliability guarantees and complex routing.

**Analogy:** A proper postal sorting office. Letters are tracked, routed to the right department, and if undeliverable — sent to a returns department (dead letter queue).

### Kafka (High-throughput event streaming)

Not a traditional queue — it's a distributed log.

Every event is stored and can be replayed. Multiple consumers can read the same events independently.

```
Producer → Kafka topic → Consumer Group A (analytics)
                       → Consumer Group B (notifications)
                       → Consumer Group C (billing)
```

**Good for:** When multiple systems need to react to the same event. Analytics pipelines. Event sourcing.

**Analogy:** A newspaper. Printed once. Many different people read their own copy. Old editions can be re-read.

### Which to use?

| Tool | Use when |
|---|---|
| Redis + BullMQ | Simple background jobs, most apps |
| RabbitMQ | Need guaranteed delivery, complex routing |
| Kafka | High volume, multiple consumers, event replay |

---

## Step 4 — Write Flow (Enqueuing a Job)

```
User action triggers a job
        │
        ▼
   API Server
   creates job object:
   {
     id: "job_abc123",
     type: "send_email",
     payload: { user_id: 123, template: "welcome" },
     status: "queued",
     created_at: now,
     attempts: 0
   }
        │
        ▼
   Push to Queue (Redis / RabbitMQ / Kafka)
        │
        ▼
   Return response to user immediately
   "Your export is being processed"
```

The user never waits. Job is handed off instantly.

---

## Step 5 — Read Flow (Processing a Job)

```
Worker is running, polling the queue
        │
        ▼
   Picks up next job
        │
        ▼
   Marks job as "processing"
   (locks it so no other worker grabs the same job)
        │
        ▼
   Executes the job
   (sends email, resizes image, generates report...)
        │
        ├── Success → mark job as "done" → acknowledge to queue
        │
        └── Failure → mark job as "failed" → trigger retry logic
```

---

## Step 6 — Retries

Jobs fail. Networks go down. Third-party APIs time out.

A good job queue retries automatically.

### Retry with exponential backoff

Don't retry immediately — wait longer each time.

```
Attempt 1 fails → wait 5 seconds  → retry
Attempt 2 fails → wait 25 seconds → retry
Attempt 3 fails → wait 125 seconds → retry
Attempt 4 fails → wait 625 seconds → retry
Attempt 5 fails → give up → send to Dead Letter Queue
```

Formula: `wait = base_delay × multiplier^attempt`

**Why exponential?** If a service is down, hammering it every second makes it worse. Backing off gives it time to recover.

**Analogy:** Calling a friend who isn't answering. You don't call every second. You wait 5 min, then 15 min, then an hour. You don't spam them.

### Jitter

Add randomness to the wait time.

Without jitter: all 10,000 failed jobs retry at the exact same second. Thundering herd.

With jitter: they retry spread across a time window. Traffic smoothed out.

```
wait = (base_delay × multiplier^attempt) + random(0, 1000ms)
```

**Analogy:** If everyone leaves a concert at the same time — traffic jam. Staggered exits = smooth flow.

---

## Step 7 — Dead Letter Queue (DLQ)

A job that fails after all retry attempts goes to the Dead Letter Queue.

```
Normal Queue → job fails 5 times → Dead Letter Queue
```

The DLQ is not deleted. It's held for:

- Manual inspection
- Debugging why it failed
- Reprocessing after a fix is deployed

**Analogy:** Undeliverable mail. Post office doesn't throw it away — sends it to the undeliverable mail department for humans to investigate.

---

## Step 8 — Job Scheduling (Delayed + Recurring)

### Delayed jobs

Run a job at a specific time in the future.

```
{
  type: "send_reminder",
  run_at: "2024-01-15 09:00:00"
}
```

Scheduler checks every few seconds: "Is there any job whose `run_at` is now or in the past?" If yes — push to queue.

**Use case:** Send a reminder email 24 hours after signup.

### Recurring jobs (Cron)

Run a job on a schedule, like cron.

```
"every day at 2am"   → cleanup expired sessions
"every hour"         → sync exchange rates
"every monday 9am"   → send weekly digest email
```

**Analogy:** A recurring calendar event. "Every Monday, send the newsletter."

### How cron works internally

```
Scheduler process runs every minute
  → checks list of cron jobs
  → if job's schedule matches current time → push to queue
  → workers pick it up like any other job
```

---

## Step 9 — Worker Pool Design

### How many workers?

Depends on the job type:

| Job type | Workers | Why |
|---|---|---|
| I/O bound (email, API calls) | Many (50-100) | Workers mostly wait for network |
| CPU bound (image resize, PDF) | Few (= CPU cores) | More workers = context switching overhead |

**Analogy:**
- I/O jobs: waiters. More waiters = more tables served simultaneously. They spend most time walking, not cooking.
- CPU jobs: chefs. More chefs than burners = chefs bumping into each other.

### Concurrency control

Sometimes you need to limit how many of a specific job type run simultaneously.

```
Max 5 video encoding jobs at once   (CPU intensive)
Max 100 email sending jobs at once  (I/O, fast)
```

Use job-type-specific queues with different worker counts:

```
queue: video-encoding → 5 workers
queue: email-sending  → 100 workers
queue: reports        → 10 workers
```

---

## Step 10 — Idempotency

What if a worker processes a job, completes it, but crashes before acknowledging?

Queue thinks the job failed. Retries it. Job runs twice.

**Problem:** User gets charged twice. Email sent twice.

**Solution:** Make every job idempotent — safe to run multiple times.

```
Before processing: check if this job_id was already completed
  → Already done? Skip it.
  → Not done? Process and record completion.
```

Store completed job IDs in a database or Redis set.

**Analogy:** A receipt. You've already paid if you have a receipt. Showing the receipt again doesn't charge you again.

---

## Step 11 — Monitoring a Job Queue

What to track:

```
Queue depth        → how many jobs waiting? (spikes = problem)
Processing rate    → jobs completed per second
Failure rate       → % of jobs failing
Job latency        → time from queued to done
DLQ size           → how many jobs gave up? (growing = systemic problem)
Worker utilization → are workers idle or overwhelmed?
```

**Alert on:**

- Queue depth growing continuously (workers can't keep up)
- DLQ size growing (something is broken)
- Job latency exceeding SLA

---

## Full Architecture Summary

```
                    User Action
                         │
                         ▼
                    API Server
                    creates job
                         │
                         ▼
              ┌──────────────────────┐
              │       Queue          │
              │   (Redis/RabbitMQ)   │
              │                      │
              │  [job1][job2][job3]  │
              └──────────┬───────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌────▼──────┐
    │  Worker 1 │  │  Worker 2 │  │  Worker 3 │   ← worker pool
    └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
          │              │               │
          ▼              ▼               ▼
       SUCCESS        FAILURE          SUCCESS
          │              │               │
          ▼              ▼               ▼
       Mark done     Retry with      Mark done
                   backoff (×5)
                        │
                  Max retries hit?
                        │
                        ▼
               Dead Letter Queue
               (manual review)

── Scheduler ──────────────────────────────────────

Cron / Delayed jobs
        │
        ▼
  Scheduler process
  checks every minute
        │
        ▼
  Pushes to Queue when due
        │
        ▼
  Workers process normally
```

---

## Key Terms Glossary

| Term | Simple meaning | Analogy |
|---|---|---|
| **Job / Task** | A unit of work to be done | A restaurant order |
| **Queue** | Waiting list of jobs | Order tickets in a kitchen |
| **Producer** | Creates and enqueues jobs | Customer placing an order |
| **Worker / Consumer** | Picks up and processes jobs | Chef cooking the meal |
| **Worker pool** | Multiple workers running in parallel | Multiple chefs in one kitchen |
| **Retry** | Automatically try again after failure | Calling back a busy number |
| **Exponential backoff** | Wait longer between each retry | Waiting 5 min, then 15, then 1 hour before calling again |
| **Jitter** | Random delay added to backoff | Staggered concert exit to avoid traffic jam |
| **Dead Letter Queue** | Where jobs go after all retries fail | Undeliverable mail department |
| **Idempotency** | Safe to run the same job twice — same result | Showing a receipt doesn't charge you again |
| **Acknowledgement** | Worker confirms job is done to the queue | Signing for a delivery |
| **Delayed job** | Job scheduled to run in the future | Calendar reminder |
| **Cron job** | Job on a recurring schedule | Weekly alarm |
| **Concurrency** | Multiple workers running at the same time | Multiple chefs cooking different meals |
| **Queue depth** | Number of jobs waiting | Length of the queue at checkout |
| **Throughput** | Jobs completed per second | Orders served per hour |
| **I/O bound** | Job spends most time waiting on network/disk | Waiter waiting for kitchen |
| **CPU bound** | Job spends most time on computation | Chef actively cooking |

---

## How to Think in Interviews

### What interviewers want

- Do you understand why async processing exists?
- Do you know what happens when jobs fail?
- Can you reason about ordering, guarantees, and idempotency?
- Do you think about scale — what breaks at 1M jobs/day?

### The framework

```
Step 1 — Clarify (2 min)
  "What type of jobs? I/O or CPU heavy?
   How many jobs per day?
   Do they need ordering guarantees?
   What's the acceptable delay?"

Step 2 — Choose the queue (2 min)
  Redis for simplicity.
  RabbitMQ for guaranteed delivery.
  Kafka for high volume + multiple consumers.

Step 3 — Design the job lifecycle (5 min)
  Created → Queued → Processing → Done / Failed
  Walk through what happens on success and failure.

Step 4 — Retry + DLQ (3 min)
  Exponential backoff. Max attempts. Dead letter queue.

Step 5 — Idempotency (2 min)
  How do you handle double-processing?

Step 6 — Scaling (3 min)
  More workers. Separate queues per job type.
  What's the bottleneck?
```


**Q1: How do you ensure a job is never processed twice?**

In distributed systems, queues offer "at-least-once delivery." A worker can crash after finishing but before acknowledging. The queue retries. The job runs twice.

Solution: idempotency keys.

```
Before processing:
  check DB/Redis: has job_id "abc123" been completed?
  → YES: skip, acknowledge, done
  → NO: process, then mark as completed atomically
```

Store completed job IDs with a TTL matching your retry window (e.g. 24 hours). After TTL expires, the record is cleaned up automatically.

The key insight: **design your jobs so running them twice has no side effect.** If you can't — use idempotency checks.

---

**Q2: How would you design a job queue that handles 10 million jobs per day?**

```
10M jobs/day = ~115 jobs/second average
```

With bursts likely 5-10x higher = 500-1000 jobs/second peak.

Design:

- **Redis** with BullMQ or similar — handles 100,000+ ops/second easily
- **Worker pool** — auto-scale based on queue depth. Queue growing? Spin up more workers. Queue draining? Scale down.
- **Separate queues** per job type — critical jobs (payments) never starved by bulk jobs (reports)
- **Horizontal scaling** — workers are stateless. Add more machines = more throughput linearly.
- **Monitoring** — alert when queue depth exceeds N. Auto-scale trigger.

Bottleneck at this scale is usually the downstream service (email provider, database writes) — not the queue itself.

---

**Q3: How do you handle a job that takes 10 minutes to complete?**

Problems with long-running jobs:

- Worker timeout kills it mid-way
- Queue thinks it failed and retries — now two copies running
- Worker machine could restart

Solutions:

1. **Heartbeats** — worker sends "I'm still alive" signals every 30 seconds. Queue resets timeout on each heartbeat.

2. **Checkpointing** — job saves progress to DB as it works. If it dies and restarts, it resumes from the last checkpoint instead of starting over.

3. **Break it up** — split the 10-minute job into smaller chained jobs. Job A finishes → enqueues Job B → Job B finishes → enqueues Job C.

```
GenerateReport → chunk 1 of 10
              → chunk 2 of 10
              → ...
              → chunk 10 of 10
              → MergeChunks job
              → EmailReport job
```

Each small job is fast, retryable, and safe.

---

**Q4: How do you prioritize some jobs over others?**

Not all jobs are equal. A password reset email must go out in seconds. A weekly digest can wait.

Solution: **multiple queues with priority.**

```
queue: critical   → 50 workers  (password resets, payment confirmations)
queue: high       → 20 workers  (welcome emails, notifications)
queue: low        → 5 workers   (reports, digests, analytics)
```

Workers assigned to `critical` only pull from that queue. They're never blocked by low-priority work.

Some systems support a single priority queue where each job has a numeric priority. Worker always picks the highest-priority job first. Redis sorted sets (`ZADD` / `ZPOPMIN`) work well for this.

---

**Q5: What happens if your queue system (Redis) goes down?**

Jobs produced while Redis is down are lost unless you handle this.

Strategies:

1. **Persistence** — enable Redis AOF (append-only file) or RDB snapshots. Jobs survive a restart.

2. **Producer retry** — if the queue is unavailable, producer retries the enqueue with backoff. Job is not dropped — just delayed.

3. **Fallback queue** — write to a database table as a fallback queue. A recovery process drains the table into Redis when it comes back.

```
try:
  push to Redis
except Redis unavailable:
  insert to jobs_fallback table
  recovery_worker polls table → pushes to Redis when back
```

4. **Use RabbitMQ or Kafka** — both are designed for durability. Data is persisted to disk and replicated. Much harder to lose.

Key principle: **the queue should be the most reliable component in the system.** Treat it like a database — not a cache.

---

## One-Line Summary

> A job queue decouples producers from workers — jobs are created instantly, stored in a reliable queue (Redis/RabbitMQ/Kafka), processed asynchronously by a worker pool, retried with exponential backoff on failure, and dead-lettered when all retries are exhausted — keeping the user experience fast and the system resilient.