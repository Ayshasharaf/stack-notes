# Kafka — Just the Important Stuff


---

## What is Kafka?

A **durable message pipe** between apps.

One app sends messages into it. Other apps read from it. Messages stay on disk — they don't disappear after being read.

> Think of it like YouTube. The creator (producer) uploads videos. Viewers (consumers) watch whenever they want. Old videos stay available. Multiple viewers watch independently.

---

## Why use it?

Use Kafka when a regular queue isn't enough:

- Multiple services need the **same data**
- You need to **replay** past events
- You need to handle **millions of messages/sec**
- Services need to be **decoupled** — producer doesn't care who's consuming

Real uses: fraud detection, order tracking, analytics pipelines, microservice communication.

---

## The Core Flow

```
Producer  →  Topic (with partitions)  →  Consumer Group A
                                      →  Consumer Group B
```

Both groups get ALL messages — independently. Group A reading doesn't remove anything for Group B.

---

## 7 Things You Must Know

### 1. Topic — a named stream

A topic is like a folder or a database table. You publish to it, consumers subscribe to it.

```
payment-events
user-signups
order-updates
```

One Kafka cluster can have many topics.

---

### 2. Partition — what makes Kafka fast

Each topic is split into partitions. Each partition is an ordered, append-only log.

```
Topic: payment-events
  ├── Partition 0: [msg0, msg3, msg6 ...]
  ├── Partition 1: [msg1, msg4, msg7 ...]
  └── Partition 2: [msg2, msg5, msg8 ...]
```

Multiple consumers can read different partitions at the same time. That's the parallelism.

Messages within one partition are always in order. Across partitions — no global order.

---

### 3. Offset — a bookmark

Every message in a partition has an offset — just a number: 0, 1, 2, 3...

Kafka remembers which offset each consumer group reached. If your app crashes and restarts, it picks up from where it left off.

> Like a Kindle bookmark. Close the book, reopen it — you resume from the exact page, not the beginning.

---

### 4. Producer — the sender

Any app that writes messages to a topic. It picks which partition to write to:

- **With a key** → `hash(key) % partitions` → same key = same partition
- **No key** → round-robin across partitions

---

### 5. Consumer & Consumer Group — the reader(s)

A consumer reads messages from a topic. It pulls at its own pace — Kafka never pushes.

A **consumer group** is multiple consumers sharing the work:

```
Topic has 3 partitions, group has 3 consumers:
  Consumer 1 → Partition 0
  Consumer 2 → Partition 1
  Consumer 3 → Partition 2
```

Two separate groups both receive all messages independently.

---

### 6. Message Key — how to guarantee order

Attach a key (e.g. userId) to a message. Same key always lands in the same partition → all events for that entity are in order.

No key = round-robin = no ordering guarantee.

> Like sorting mail by postcode. All mail for the same postcode lands in the same tray, processed in order.

---

### 7. Retention — messages don't disappear

Kafka keeps messages on disk for a set time (default: 7 days). Reading a message does NOT delete it. Your consumer can re-read old messages or catch up if it was offline.

---

## The 5 Golden Rules

```
1. Kafka does not push. Consumers pull at their own pace.

2. Order is per-partition, not across the whole topic.

3. Consumers in a group can't exceed partitions.
   Extra consumers sit idle.

4. Reading a message does not delete it.
   Other groups can still read it.

5. Use a message key when order matters
   for a specific entity (user, order, account).
```

---

## Terms at a Glance

| Term | One line |
|---|---|
| Topic | Named stream — like a folder |
| Partition | Ordered sub-log. Enables parallelism. |
| Offset | Message's position number in a partition |
| Producer | App that writes messages |
| Consumer | App that reads messages |
| Consumer group | Pool of consumers sharing partitions |
| Key | Routes message to a specific partition. Same key = same partition = ordered. |
| Broker | A Kafka server |
| Retention | How long messages are kept (default 7 days) |
| Replication | Copies of partitions across brokers for fault tolerance |