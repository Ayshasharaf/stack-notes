# 🏗️ 30 System Design Concepts 

## 1. Client-Server Architecture
> **"The client asks. The server answers."**

```
[Browser / App]  ──── request ────▶  [Server]
     CLIENT       ◀──── response ───  (always running)
```

- **Client** = any app that sends requests (browser, mobile app)
- **Server** = machine that processes requests & sends back data
- Foundation of every web application

---

## 2. DNS (Domain Name System)
> **"The internet's phone book."**

```
You type:  google.com
               │
               ▼
            [DNS]  →  "That's 142.250.80.46"
               │
               ▼
         Connects to server at that IP
```

- Humans use domain names → computers use IP addresses
- DNS translates one to the other automatically

---

## 3. Proxy & Reverse Proxy
> **"A middleman — either for you, or for the server."**

```
FORWARD PROXY (hides you):
  [You] → [Proxy] → [Internet]
  (Server never sees your real IP)

REVERSE PROXY (hides the server):
  [Internet] → [Reverse Proxy] → [Backend Server]
  (You never know which server handled it)
```

| Type | Protects | Used By |
|------|----------|---------|
| Forward Proxy | Client identity | VPNs, corporate networks |
| Reverse Proxy | Server infrastructure | Nginx, Cloudflare |

---

## 4. Latency
> **"The delay between asking and receiving."**

```
You (Oman) ────────────────────────────▶ Server (USA)
           ◀────────────────────────────
                ~200ms round trip 😞

You (Oman) ──▶ Nearby Server (Dubai) ──▶ Response
                    ~20ms 😊
```

- Caused by physical distance data must travel
- **Fix:** Deploy servers in multiple regions worldwide
- Users connect to the **nearest** server

---

## 5. HTTP & HTTPS
> **"The rules for how data travels — and how to keep it safe."**

```
HTTP  (unsecured):
  Client ──[plain text]──▶ Server   ← anyone can read this!

HTTPS (secured):
  Client ──[🔒 encrypted]──▶ Server  ← safe, even if intercepted
```

- **HTTP** = communication rules (request has header + body, server returns response)
- **HTTPS** = HTTP + SSL/TLS encryption
- Always prefer HTTPS for any real application

---

## 6. APIs
> **"A menu for your server — tells clients what they can order."**

```
  Client                    Server
    │                          │
    │── POST /createUser ──────▶│
    │                          │  (talks to DB,
    │                          │   runs logic)
    │◀── { "id": 123 } ────────│
```

- API = structured interface between client and server
- Returns data in **JSON** or **XML** format
- Client doesn't need to know internal server details

---

## 7. REST vs GraphQL
> **"REST gives a fixed menu. GraphQL lets you customize your order."**

```
REST — fixed endpoints:
  GET /user/1         → returns ALL user data (even what you don't need)
  GET /user/1/posts   → separate call needed

GraphQL — one flexible query:
  query {
    user(id: 1) {
      name          ← only what you ask for
      recentPosts { title }
    }
  }                  → one call, exact data ✅
```

| | REST | GraphQL |
|-|------|---------|
| Data fetching | Fixed per endpoint | Exactly what you need |
| Caching | Easy | Harder |
| Complexity | Simple | More setup |

---

## 8. Databases — SQL vs NoSQL
> **"SQL is a spreadsheet. NoSQL is a filing cabinet."**

```
SQL (structured):             NoSQL (flexible):
┌────────────────────┐        { "user": "Ali",
│ id │ name │ email  │          "prefs": [...],
│  1 │ Ali  │ a@b.c  │          "history": [...] }
└────────────────────┘
```

| | SQL | NoSQL |
|-|-----|-------|
| Schema | Fixed | Flexible |
| Consistency | Strong (ACID) | Eventual |
| Scaling | Vertical | Horizontal |
| Best for | Banking, ERP | Social media, logs, real-time |

---

## 9. Vertical vs Horizontal Scaling
> **"Scale Up = bigger machine. Scale Out = more machines."**

```
VERTICAL (Scale Up):          HORIZONTAL (Scale Out):
                               
  [💻 Tiny Server]              [💻] [💻] [💻]
        ↓ upgrade                 Add more servers
  [🖥️ Big Server]               Spread the load
  
  ⚠️ Has a hard limit            ✅ Near-unlimited
  ⚠️ Single point of failure     ✅ Fault tolerant
```

- Start vertical (simple), move horizontal (scalable)
- Horizontal scaling requires a **Load Balancer**

---

## 10. Load Balancer
> **"A traffic cop that keeps servers from getting overwhelmed."**

```
                    ┌──▶ [Server 1]
Clients ──▶ [LB] ───┼──▶ [Server 2]
                    └──▶ [Server 3]

If Server 2 dies → LB automatically routes to 1 & 3
```

**Algorithms:**
- **Round Robin** — take turns (1→2→3→1→2→3...)
- **Least Connections** — send to least-busy server
- **IP Hashing** — same user always hits same server

---

## 11. Database Indexing
> **"Like the index at the back of a book — jump directly to what you need."**

```
WITHOUT index:          WITH index:
Scan row 1... ❌        Index: "Ali" → row 847
Scan row 2... ❌             ↓
Scan row 3... ❌        Jump directly ✅
...
Scan row 847 ✅
```

- Create indexes on columns you **frequently search/filter**
- ✅ Faster reads
- ⚠️ Slower writes (index must update too)
- Only index what you need — don't over-index

---

## 12. Replication
> **"Make copies of your database so reads are fast and nothing is lost."**

```
           WRITES
             │
      [Primary DB] ──────────────────┐
             │ sync                  │ sync
             ▼                       ▼
      [Read Replica 1]        [Read Replica 2]
           READ                    READ
```

- **Primary** handles all writes
- **Replicas** handle reads (spread the load)
- If primary fails → promote a replica as new primary
- ✅ Better read performance + high availability

---

## 13. Sharding (Horizontal Partitioning)
> **"Split one massive database into smaller pieces across multiple servers."**

```
User IDs 1–1M     → [Shard A]
User IDs 1M–2M    → [Shard B]
User IDs 2M–3M    → [Shard C]

Query for user 1.5M → goes only to Shard B (fast!)
```

- Each shard holds a **portion** of the total data
- Routed by a **sharding key** (e.g., User ID, region)
- ✅ Both reads AND writes scale
- ⚠️ Complex to manage; hard to re-shard later

---

## 14. Vertical Partitioning
> **"Split a wide table into narrower, focused tables."**

```
BEFORE (one giant table):
┌────┬──────┬──────────────┬─────────┬──────────────┐
│ id │ name │ login_history│ address │ billing_info │
└────┴──────┴──────────────┴─────────┴──────────────┘

AFTER (split by access pattern):
┌────┬──────┐   ┌────┬──────────────┐   ┌────┬─────────────┐
│ id │ name │   │ id │ login_history│   │ id │ billing_info│
└────┴──────┘   └────┴──────────────┘   └────┴─────────────┘
 (queried often)   (queried rarely)        (sensitive data)
```

- Improves query speed — scan fewer columns
- Great for tables with **many columns** rarely used together

---

## 15. Caching
> **"Store hot data in memory so you don't hit the database every time."**

```
Request comes in
       │
       ▼
  [Cache] ──hit?──▶ Return instantly ⚡
     │ miss
     ▼
  [Database]
     │
     ▼
  Store in Cache → Return to user
```

- **Cache-Aside Pattern:** check cache → miss → DB → populate cache
- **TTL (Time To Live):** cached data expires after N seconds
- Tools: Redis, Memcached
- ✅ Dramatically reduces latency
- ⚠️ Can serve stale data if TTL is too long

---

## 16. Denormalization
> **"Store data together (even duplicated) to avoid slow joins."**

```
NORMALIZED (clean, but slow):
Users table + Posts table → JOIN to get user's posts

DENORMALIZED (faster reads):
Posts table already includes: post_id, content, author_name, author_avatar
                              (duplicated, but no join needed) ✅
```

- Trade: **more storage + complex updates** for **faster reads**
- Best for read-heavy apps (feeds, dashboards, reports)

---

## 17. CAP Theorem
> **"In a distributed system, you can only guarantee 2 of 3 things."**

```
         Consistency
              △
             / \
            /   \
           /     \
 Availability ─── Partition
                  Tolerance

⚠️ Network failures are unavoidable → P is always required
So the real choice is: CP or AP
```

| Choice | Guarantees | Sacrifices | Example |
|--------|-----------|-----------|---------|
| **CP** | Correct data always | May go offline | Banking |
| **AP** | Always responds | May return stale data | Social feeds |

---

## 18. Blob Storage
> **"A scalable bucket for large files — images, videos, PDFs."**

```
Your App
   │
   │── upload photo ──▶ [Amazon S3 / Blob Storage]
   │                          │
   │◀── unique URL ───────────┘

User visits URL → file served directly
```

- **Blob** = Binary Large Object (images, video, docs, backups)
- Stored in **buckets** in the cloud
- Each file gets a unique URL
- ✅ Infinitely scalable, pay-as-you-go
- Examples: Amazon S3, Google Cloud Storage, Azure Blob

---

## 19. CDN (Content Delivery Network)
> **"Cache your static content close to users worldwide."**

```
Origin Server (USA)
    │
    ├──▶ CDN Node (London) ──▶ serves European users ⚡
    ├──▶ CDN Node (Dubai)  ──▶ serves Middle East users ⚡
    └──▶ CDN Node (Tokyo)  ──▶ serves Asian users ⚡
```

- Distributes static assets (images, JS, CSS, videos) globally
- User gets content from the **nearest CDN edge server**
- ✅ Lower latency + less load on origin server
- Examples: Cloudflare, Akamai, AWS CloudFront

---

## 20. WebSockets
> **"A permanent two-way phone call between client and server."**

```
Regular HTTP:
Client ──request──▶ Server ──response──▶ Client (connection closed)
Client ──request──▶ Server ... (repeat every time)

WebSocket:
Client ◀══════════════════════════▶ Server
         (connection stays open)
         Server can push anytime! ⚡
```

- Perfect for: **live chat, gaming, real-time dashboards, notifications**
- Single persistent connection = low overhead
- Server can **push** data without client asking

---

## 21. Webhooks
> **"Instead of checking repeatedly, get notified automatically."**

```
POLLING (inefficient):
Your App → "Any updates?" → Provider → "No"
Your App → "Any updates?" → Provider → "No"
Your App → "Any updates?" → Provider → "Yes! Here's the data"

WEBHOOK (efficient):
Provider ──▶ POST to your URL  (only when something happens) ✅
```

- You register a URL with a provider
- Provider calls YOUR URL when an event fires
- ✅ Far fewer API calls, real-time, efficient
- Example: Stripe calls your webhook when payment succeeds

---

## 22. Microservices
> **"Break the app into small, independent services — each does one thing."**

```
MONOLITH:                    MICROSERVICES:
┌────────────────────┐       [User Service]
│  Users             │       [Order Service]
│  Orders            │  →    [Payment Service]
│  Payments          │       [Notification Service]
│  Notifications     │       (each independent, scaled separately)
└────────────────────┘
```

| | Monolith | Microservices |
|-|----------|--------------|
| Deploy | All at once | Independently |
| Scale | Entire app | Only what needs it |
| Failure | One bug can crash all | Isolated failures |
| Complexity | Low initially | Higher |

---

## 23. Message Queues
> **"A buffer between services so they don't have to talk directly."**

```
[Producer Service] ──▶ [📬 Queue] ──▶ [Consumer Service]
  "Here's a task"      (holds it)       "I'll process it
                                         when I'm ready"
```

- **Producer** = puts messages in the queue
- **Queue** = temporary storage buffer
- **Consumer** = takes and processes messages at its own pace
- ✅ Services are decoupled (don't wait for each other)
- ✅ Handles traffic bursts without crashing
- Examples: RabbitMQ, AWS SQS, Kafka

---

## 24. Rate Limiting
> **"Put a cap on how many requests a user can make."**

```
User A: 95 requests/min  ✅ fine
User A: 100 requests/min ✅ at limit
User A: 101 requests/min ❌ 429 Too Many Requests

(protects your server from bots & abuse)
```

**Common Algorithms:**
- **Fixed Window** — count resets every minute
- **Sliding Window** — rolling 60-second count
- **Token Bucket** — earn tokens over time, spend on requests

Often implemented at the **API Gateway** level

---

## 25. API Gateway
> **"One front door for all your microservices."**

```
Clients (web, mobile, IoT)
           │
           ▼
     [API Gateway]  ←── handles: auth, rate limiting, logging
      /    |    \
     ▼     ▼     ▼
 [Users] [Orders] [Payments]
 Service  Service   Service
```

- Single entry point → routes to correct service
- Centralizes: **authentication, rate limiting, logging, SSL**
- Clients don't need to know individual service URLs
- ✅ Cleaner architecture, better security

---

## 26. Idempotency
> **"Sending the same request twice should have the same effect as sending it once."**

```
User clicks "Pay" → request sent
Network hiccups → user clicks again

WITHOUT idempotency: charged TWICE 😱
WITH idempotency:    charged once ✅ (duplicate detected)

How: each request gets a unique ID
     Server checks: "seen this ID before?" → skip if yes
```

- Critical for: payments, email sending, database writes
- Implemented using **unique request IDs** (idempotency keys)
- Common in REST APIs: PUT and DELETE are naturally idempotent

---

## 27. Consistent Hashing
> **"Distribute load across servers in a way that's easy to change."**

```
Regular hashing:  server = hash(key) % N
   → change N (add/remove server) = almost everything remaps 😱

Consistent hashing:
   → add/remove a server = only nearby keys remap ✅

Used in: distributed caches, load balancers, CDNs
```

---

## 28. Heartbeat & Health Checks
> **"How systems know if a server is still alive."**

```
Load Balancer:  "Server 2, you alive?"
Server 2:       "Yes ✅" (heartbeat every few seconds)

Server 2 stops responding...
Load Balancer:  Routes traffic away from Server 2 automatically
```

- Services **ping** each other periodically
- No response = server is down → remove from rotation
- Enables **automatic failover** without human intervention

---

## 29. Circuit Breaker
> **"Stop calling a failing service — give it time to recover."**

```
CLOSED (normal):     requests flow through ✅
OPEN (failure):      requests blocked immediately ❌
                     (don't waste time waiting for a dead service)
HALF-OPEN (testing): let one request through to test recovery
```

- Prevents **cascading failures** across microservices
- Like a fuse box — trips when overloaded, prevents fire
- After timeout, tries again — if OK, closes circuit

---

## 30. Data Consistency Patterns
> **"How do you keep data the same across multiple services/databases?"**

```
STRONG CONSISTENCY:
  Write → ALL nodes updated before confirming ✅ slow
  
EVENTUAL CONSISTENCY:
  Write → confirmed immediately
  Other nodes → catch up soon after ✅ fast, but brief lag

SAGA PATTERN (for distributed transactions):
  Step 1 ✅ → Step 2 ✅ → Step 3 ❌
                              ↓
                    Compensating transactions
                    (undo Step 1 & 2) 🔄
```

---

## ⚡ Quick Reference Cheat Sheet

| Concept | One-liner |
|---------|-----------|
| Client-Server | Client asks, server answers |
| DNS | Domain → IP address |
| Proxy | Hides client; Reverse proxy hides server |
| Latency | Delay = distance; deploy globally |
| HTTP/HTTPS | Rules for data transfer; HTTPS = encrypted |
| API | Structured interface to your server |
| REST vs GraphQL | Fixed endpoints vs. ask exactly what you need |
| SQL vs NoSQL | Structured+consistent vs. flexible+scalable |
| Vertical Scaling | Bigger machine (has limits) |
| Horizontal Scaling | More machines (near-unlimited) |
| Load Balancer | Distributes traffic; removes dead servers |
| Indexing | DB lookup table = fast reads, slower writes |
| Replication | Copies for reads + failover |
| Sharding | Split DB by rows across servers |
| Vertical Partition | Split DB by columns |
| Caching | Hot data in RAM; expires via TTL |
| Denormalization | Duplicate data to avoid slow joins |
| CAP Theorem | Distributed system: pick 2 of C, A, P |
| Blob Storage | S3-style storage for large files |
| CDN | Static content served from nearest edge |
| WebSockets | Persistent two-way connection |
| Webhooks | Server calls you when events happen |
| Microservices | Small independent services, one job each |
| Message Queue | Async buffer between services |
| Rate Limiting | Cap requests per user to protect servers |
| API Gateway | Single entry point; routes + auth + limits |
| Idempotency | Same request twice = same result once |
| Consistent Hashing | Smart distribution that handles server changes |
| Heartbeat | Services ping each other to detect failures |
| Circuit Breaker | Stop calling failing services automatically |

---

*Made for fast learning — come back to any concept anytime.*