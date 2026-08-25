# 🗄️ AWS Databases — Master Reference Guide

> **How to use this:** Read top to bottom once. Then use the decision tree and scenario table to answer any exam question instantly.

---

# PART 1 — The Big Picture

## How much do you want AWS to manage?

```
┌─────────────────────────────────────────────────────────────────┐
│              HOW MUCH CONTROL DO YOU NEED?                      │
└─────────────────────────────────────────────────────────────────┘
         │                    │                    │
    Total control         Some control         No servers at all
         │                    │                    │
         ▼                    ▼                    ▼
    UNMANAGED            MANAGED              FULLY MANAGED
   MySQL on EC2          Amazon RDS           Amazon DynamoDB
                         Amazon Aurora        Amazon DocumentDB
                                              Amazon Neptune
                                              Amazon ElastiCache

   You do: everything    You do: settings,    You do: just write
                         queries, scaling     data and build app
   AWS does: nothing     AWS does: hardware,  AWS does: everything
                         patching, backups    else
```

---

# PART 2 — Relational Databases (SQL / Structured Data)

> Think: **Excel spreadsheet** — strict rows, columns, tables. Great for structured data like bank transactions, inventory, orders.

## Amazon RDS — Managed SQL database

**Supported engines:** MySQL · PostgreSQL · MariaDB · Oracle · SQL Server

| Feature | What it does | Think of it as... |
|---|---|---|
| **Multi-AZ** | Standby copy in another zone, auto-failover | Insurance policy |
| **Read Replicas** | Extra copies to handle read traffic | Extra cashiers at checkout |
| **Automatic backups** | AWS handles it | Built-in safety net |

> **Use RDS when:** You need a traditional relational database without managing the server yourself.

---

## Amazon Aurora — AWS's premium SQL database

> Same as RDS but **faster, smarter, self-healing.**

```
vs MySQL:      5x faster
vs PostgreSQL: 3x faster

Storage:  starts at 10GB → grows automatically → up to 128TB
Copies:   6 copies across 3 Availability Zones
Healing:  corrupted data? Aurora fixes itself automatically
```

> **Use Aurora when:** You need max performance and resilience from a relational database.

---

## RDS vs Aurora — Quick comparison

| | RDS | Aurora |
|---|---|---|
| Engine | MySQL, PostgreSQL, etc. | MySQL or PostgreSQL compatible |
| Speed | Standard | Up to 5x faster |
| Storage | Fixed, manual resize | Auto-scales to 128TB |
| Self-healing | ❌ | ✅ |
| Best for | Standard relational workloads | High-performance, large-scale apps |

---

# PART 3 — NoSQL Databases (Flexible / Fast)

> Think: **Collection of folders** — no fixed structure. Each item can have different fields. Built for speed and scale.

## Relational vs NoSQL — core difference

```
RELATIONAL (RDS)                    NOSQL (DynamoDB)
────────────────                    ────────────────
user_id | name  | email             { "user_id": "123",
──────────────────────               "name": "Sara",
123     | Sara  | sara@x.com         "fav_color": "blue" }
124     | Ahmed | ahmed@x.com
                                    { "user_id": "124",
Every row same columns.              "name": "Ahmed",
                                     "emergency_contact": "Mom" }

                                    Items can have different fields. ✅
```

---

## Amazon DynamoDB — Fully managed NoSQL

> Serverless. No patching. No servers. Just create a table and go.

```
Speed:        Single-digit millisecond — always, at any scale
Scale:        100 users or 100 million users — same speed
Availability: Data replicated across 3 facilities in a region
Encryption:   On by default — data encrypted before hitting disk
```

| Feature | What it does |
|---|---|
| **Auto Scaling** | Adjusts capacity up/down based on traffic automatically |
| **Global Tables** | Sync data across countries — Tokyo + New York same speed |
| **Encryption** | Default, automatic, no setup needed |

> **Use DynamoDB when:** Mobile apps · gaming leaderboards · real-time bidding · any app needing massive scale + speed

---

# PART 4 — In-Memory Caching (Ultra-Fast Layer)

> Think: Moving your most-used tools from a storage shed (disk) to your work belt (RAM). Microsecond speed.

## How caching works

```
User requests data
        │
        ▼
  Check CACHE first
        │
   ┌────┴────┐
   │         │
Cache HIT  Cache MISS
   │         │
   │         ▼
   │    Go to main database → get data → store copy in cache
   │         │
   └────┬────┘
        ▼
  Return data instantly ⚡
```

**Why it matters:** Reduces load on your main database. Makes apps feel instant.

---

## Amazon ElastiCache — Managed in-memory cache

**Supported engines:** Redis · Memcached · Valkey

| Feature | What it does |
|---|---|
| **Speed** | Sub-millisecond latency |
| **Session management** | Remember logged-in users across pages |
| **Leaderboards** | Real-time scores updating every second |
| **High availability** | Replicas across AZs, auto-failover if node dies |
| **Scalability** | Add/remove nodes as traffic spikes |
| **Security** | Encryption at rest + in transit |

> **Use ElastiCache when:** Your database is the bottleneck · you need microsecond reads · real-time apps · session storage

---

# PART 5 — Purpose-Built Databases

> When standard SQL or NoSQL doesn't fit, AWS has a specialist for it.

## Amazon DocumentDB — JSON document store

> MongoDB-compatible. Great for flexible, document-style data.

```
Perfect for:  Content management (blog posts)
              Product catalogs (different fields per product)
              Any app already using MongoDB
```

- Schema-flexible — no need to define tables upfront
- Fully managed, auto-scaling storage
- Migrate from MongoDB **without changing your code**

---

## Amazon Neptune — Graph database

> Built for **relationships between data**, not just the data itself.

```
Traditional DB asks:  "Who is Ahmed?"
Neptune asks:         "Who are Ahmed's friends, who are THEIR
                       friends, and which ones like Sushi?"
                       → answered in milliseconds
```

| Use case | Example |
|---|---|
| **Social networks** | Map user connections |
| **Fraud detection** | Find bank accounts secretly linked to same person |
| **Recommendation engines** | "Customers who bought X also bought Y" |

---

# PART 6 — Backup & Management

## AWS Backup — Central backup command center

> One dashboard to back up **everything** — S3, RDS, DynamoDB, EFS, EC2, and more.

```
Without AWS Backup:  RDS console for DB backups
                     S3 console for S3 backups
                     EFS console for file backups
                     ... one console per service 😩

With AWS Backup:     One place. All services. One policy. ✅
```

**Key capability:** Tag resources as "Production" → auto-backup every night at 2AM → copy to another region automatically.

> **Use when:** Compliance audits · multi-service backup policies · cross-region backup automation

---

# PART 7 — Decision Tree (Which DB to pick?)

```
What kind of data are you storing?
          │
          ├── Structured, tables, SQL needed?
          │         │
          │         ├── Need max speed + self-healing? ──► Aurora
          │         └── Standard SQL workload? ──────────► RDS
          │
          ├── Flexible structure, need massive scale?
          │         └── ──────────────────────────────────► DynamoDB
          │
          ├── JSON documents, MongoDB compatible?
          │         └── ──────────────────────────────────► DocumentDB
          │
          ├── Relationships between data (social, fraud)?
          │         └── ──────────────────────────────────► Neptune
          │
          ├── Need microsecond speed on top of existing DB?
          │         └── ──────────────────────────────────► ElastiCache
          │
          ├── Need full control over every setting?
          │         └── ──────────────────────────────────► MySQL/DB on EC2
          │
          └── Need to back up multiple AWS services centrally?
                    └── ──────────────────────────────────► AWS Backup
```

---

# PART 8 — Scenario Cheat Sheet

| Scenario keyword | Answer |
|---|---|
| Bank transactions, inventory, orders (structured) | RDS |
| Max performance relational DB, self-healing | Aurora |
| MongoDB migration, JSON documents, product catalog | DocumentDB |
| Gaming leaderboard, mobile app, real-time, massive scale | DynamoDB |
| Social network connections, fraud detection, recommendations | Neptune |
| App is too slow, database overloaded, need microsecond reads | ElastiCache |
| Remember user login session across pages | ElastiCache |
| Need full control, specific OS, custom DB config | MySQL/DB on EC2 |
| Back up RDS + S3 + EFS all in one place | AWS Backup |
| Compliance — prove all data is backed up | AWS Backup |
| Database failover, high availability, auto-switch | RDS Multi-AZ or Aurora |
| Handle read-heavy traffic, reduce DB load | RDS Read Replicas |
| Data in multiple countries, same speed everywhere | DynamoDB Global Tables |
| Don't know scale yet, serverless, no server management | DynamoDB |

---

# PART 9 — One-line Reminders

- **RDS** = managed SQL (MySQL, PostgreSQL etc.), you set it up, AWS maintains it
- **Aurora** = AWS's own SQL engine — faster, self-healing, auto-scales to 128TB
- **DynamoDB** = serverless NoSQL, millisecond speed at any scale, no servers
- **DocumentDB** = MongoDB-compatible, flexible JSON documents
- **Neptune** = graph database — think relationships, not rows
- **ElastiCache** = RAM-speed cache layer on top of your database
- **AWS Backup** = one dashboard to rule all backups across all services
- **Multi-AZ** = standby copy, auto-failover = high availability
- **Read Replicas** = extra copies for read traffic = performance
- **Unmanaged** = full control, full responsibility (EC2 + your own DB)