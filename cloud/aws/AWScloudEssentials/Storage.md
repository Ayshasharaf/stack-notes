# ☁️ AWS Storage — Master Reference Guide

> **How to use this:** Read top to bottom once. Then use the decision trees and scenario tables to answer any exam question.

---

# PART 1 — The Big Picture

## What are the 3 storage types?

```
┌─────────────────────────────────────────────────────────────┐
│                    WHAT ARE YOU STORING?                    │
└─────────────────────────────────────────────────────────────┘
         │                    │                    │
    Hard drive?           Files/Objects?      Shared folder?
    (OS, databases)      (photos, videos,    (many servers
         │                backups, data)      accessing same
         ▼                    │               files at once)
   BLOCK STORAGE              ▼                    │
   • EBS                 OBJECT STORAGE            ▼
   • Instance Store      • Amazon S3          FILE STORAGE
                                              • EFS (Linux)
                                              • FSx (Windows/other)
```

---

# PART 2 — Block Storage (EC2 Hard Drives)

## Instance Store vs EBS — at a glance

```
                  Instance Store        Amazon EBS
                  ──────────────        ──────────
 Data persists?   ❌ LOST on stop        ✅ KEPT after stop
 Performance      ★★★★★ Very High        ★★★★☆ High
 Setup            Auto-available         Manual attach
 Snapshots        ❌ No                  ✅ Yes
 Best for         Cache, temp data       Databases, apps
```

## EBS Snapshot Lifecycle

```
EBS Volume ──► Take Snapshot ──► Stored in S3
                    │
                    ├── Restore → new EBS volume (exact copy)
                    ├── Copy    → different Region
                    └── Share   → different AWS account
```

**Incremental backups:** First = full copy. After that = only changed blocks saved. Cheaper + faster.

## Amazon Data Lifecycle Manager (DLM)

> Automates snapshot create → retain → delete. Set it once, forget it.

```
Step 1: Create policy  →  Step 2: Pick target (volume or EC2)
Step 3: Set schedule   →  Step 4: Set retention period
Step 5: Auto-delete old snapshots  ✅
```

**Your job (not AWS's):** Schedule snapshots · Encrypt sensitive data · Test restores · Delete old snapshots

---

# PART 3 — Object Storage (Amazon S3)

## Core concept

```
BUCKET  =  container (like a folder)    — globally unique name
OBJECT  =  file stored inside           — identified by a KEY
```

- Durability: **11 nines (99.999999999%)** — data is virtually impossible to lose
- Scale: **unlimited**
- Default: **private** — you must explicitly grant access

## S3 Storage Classes — Pick by access frequency

```
HOW OFTEN DO YOU NEED THIS DATA?
         │
         ├── All the time ──────────────────► S3 Standard
         │                                    (websites, apps, gaming)
         │
         ├── All the time + need max speed ─► S3 Express One Zone
         │                                    (AI/ML, HPC — 10x faster)
         │
         ├── Rarely, but need it INSTANTLY ─► S3 Standard-IA
         │                                    (backups, DR)
         │
         ├── Rarely + can recreate if lost ─► S3 One Zone-IA
         │                                    (cheaper, single AZ)
         │
         ├── Very rarely + instant access ──► S3 Glacier Instant Retrieval
         │                                    (medical images, news archives)
         │
         ├── 1–2x per year, can wait ───────► S3 Glacier Flexible Retrieval
         │                                    (minutes–hours to retrieve)
         │
         ├── Legal/compliance archives ─────► S3 Glacier Deep Archive
         │                                    (cheapest, ~12hr retrieval)
         │
         ├── Not sure of access pattern ────► S3 Intelligent-Tiering
         │                                    (autopilot — moves data auto)
         │
         └── Data must stay on-premises ────► S3 Outposts
                                              (local legal requirements)
```

## S3 Lifecycle Policies

```
Upload → [30 days no access] → Move to IA
       → [90 days no access] → Move to Glacier
       → [365 days]          → Delete permanently
```

## S3 Security

| Tool | What it does |
|---|---|
| **Bucket policy** | Controls who accesses the bucket + its objects |
| **IAM identity policy** | Controls what users/roles can do in S3 |
| **Encryption at rest** | Protects stored objects |
| **Encryption in transit** | Protects data moving to/from S3 (HTTPS) |

---

# PART 4 — File Storage (Shared Folders)

## Amazon EFS — Linux shared file system

> Many EC2 instances read/write the **same files** at the same time. Grows and shrinks automatically. No capacity planning.

**Protocol:** NFSv4 (Linux only)

### EFS Storage Tiers

| Tier | Use when... | Availability |
|---|---|---|
| Standard | Accessed frequently | Multi-AZ |
| Infrequent Access (IA) | Accessed a few times/month | Multi-AZ |
| Archive | Accessed a few times/year | Multi-AZ (cheapest) |
| One Zone | Lower cost, single location ok | Single AZ |

### EFS Lifecycle auto-moves files

```
[Not touched 30 days]  → moves to IA
[Not touched 90 days]  → moves to Archive
[Someone opens it]     → moves back to Standard instantly
```

---

## Amazon FSx — Specialist file systems

> EFS is for Linux. FSx is for everything else — Windows, high-performance, enterprise.

| FSx Option | Pick this when... | Protocol |
|---|---|---|
| **FSx for Windows File Server** | Windows apps, Active Directory, SQL Server | SMB |
| **FSx for Lustre** | HPC, Machine Learning, video rendering | Lustre (parallel) |
| **FSx for NetApp ONTAP** | Already using NetApp on-premises | Multi-protocol |
| **FSx for OpenZFS** | Moving from ZFS servers, dev/test | NFS |

> **Exam tip:** See "SMB" or "Active Directory"? → Always **FSx for Windows**. EFS won't work there.

---

# PART 5 — Hybrid & Special Services

## AWS Storage Gateway — Bridge to cloud

> Your office thinks it's saving to a local drive. It's actually going to AWS. No app changes needed.

| Gateway Type | Looks like... | Best for... |
|---|---|---|
| **S3 File Gateway** | A normal file share (folder) | Backing files to S3, local cache of recent files |
| **Volume Gateway** | A virtual hard drive (iSCSI) | Cloud-backed disk volumes for local servers |
| **Tape Gateway** | A virtual tape library | Replacing physical backup tapes |

### Volume Gateway modes

```
Cached Mode  → popular files stay local, rest in cloud  (saves local space)
Stored Mode  → everything local + backed up to AWS      (needs more local disk)
```

---

## AWS Elastic Disaster Recovery (DRS) — Server insurance

> Continuously copies your servers to AWS. If disaster strikes, spin them up in minutes.

```
Your server ──► [continuous block-level replication] ──► AWS

Disaster happens → flip switch → servers live in AWS in minutes
```

| Metric | What it means | DRS result |
|---|---|---|
| **RPO** (Recovery Point Objective) | How much data you lose | Seconds |
| **RTO** (Recovery Time Objective) | How long to get back online | Minutes |

### Don't confuse these:

| Service | Primary purpose | Use when... |
|---|---|---|
| **AWS Backup** | Point-in-time snapshots | Accidental deletions, compliance |
| **Storage Gateway** | Connect on-prem apps to cloud | Hybrid storage |
| **Elastic DRS** | Launch entire servers in AWS | Total site failure, business continuity |

---

# PART 6 — Shared Responsibility Model

```
Fully Managed   AWS ████████████████ You ░░
  (e.g. S3)

Managed         AWS ████████ You ████████
  (e.g. EBS)

Unmanaged       AWS ░░ You ████████████████
  (e.g. EC2 + Instance Store)
```

---

# PART 7 — Scenario Cheat Sheet (Answer Any Question)

| Scenario keyword | Answer |
|---|---|
| Active website / app data | S3 Standard |
| AI/ML, millisecond speed | S3 Express One Zone |
| Backup, rarely accessed, instant when needed | S3 Standard-IA or Glacier Instant Retrieval |
| Legal records 7–10 years | S3 Glacier Deep Archive |
| Don't know access pattern | S3 Intelligent-Tiering |
| Data must stay on-premises | S3 Outposts |
| Many Linux servers share files | Amazon EFS |
| Windows + Active Directory + SMB | FSx for Windows File Server |
| HPC / Machine Learning file system | FSx for Lustre |
| Already using NetApp on-prem | FSx for NetApp ONTAP |
| On-prem + cloud hybrid (no app changes) | AWS Storage Gateway |
| Replace physical backup tapes | Storage Gateway — Tape Gateway |
| Entire server recovery after site failure | AWS Elastic Disaster Recovery |
| Fast temp storage, lost on stop | EC2 Instance Store |
| Persistent disk for EC2 | Amazon EBS |
| Automate snapshot scheduling | Amazon DLM |
| Cross-region data migration | EBS Snapshots |
| Dev team needs prod clone fast | EBS Snapshot → new volume |

---

# PART 8 — One-line Reminders

- **S3** = object storage, 11 nines durability, private by default
- **EBS** = persistent hard drive for EC2, supports snapshots
- **Instance Store** = fastest, but data dies when instance stops
- **EFS** = shared Linux folder, auto-scales, NFS protocol
- **FSx** = specialist file systems (Windows, HPC, enterprise)
- **Storage Gateway** = on-prem apps → cloud, no rewrites needed
- **DRS** = continuous replication, recover entire servers in minutes
- **DLM** = automates EBS snapshot create/retain/delete
- **Intelligent-Tiering** = S3 autopilot when you're unsure of access patterns
- **Deep Archive** = cheapest storage in all of AWS, 12hr retrieval