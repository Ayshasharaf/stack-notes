# Designing a Key-Value Store

Core API:

```
put(key, value)   // insert value for key
get(key)          // retrieve value for key
```

**Goals:** store large datasets, low latency, high availability, tunable consistency.

---

## 1. CAP Theorem

- **C**onsistency, **A**vailability, **P**artition tolerance : pick 2.
- Network failures are inevitable in distributed systems, so **P is non-negotiable**.
- Real choice is between **CP** (consistent, may reject requests) and **AP** (available, may return stale data).

---

## 2. Scaling: Consistent Hashing

- Data is distributed across nodes using consistent hashing (minimizes reshuffling when nodes join/leave).
- Each key is replicated to **N** nodes for durability and availability.

---



## 3. Tunable Consistency: Quorum

With replication factor **N**, define:

- **W** = nodes that must ack a write
- **R** = nodes that must ack a read


| Setting                 | Effect                                              |
| ----------------------- | --------------------------------------------------- |
| `W = 1`                 | Fast writes : only 1 node needs to ack              |
| `R = 1`                 | Fast reads : only 1 node needs to ack               |
| `W + R > N`             | Strong consistency : read/write sets always overlap |
| e.g. `W = R = 2, N = 3` | Common strong-consistency config                    |


With `W + R > N`, every read overlaps with the last write, so it sees the latest version. `W + R ≤ N` favors speed/availability over consistency.

---



## 4. Resolving Conflicts: Vector Clocks

Each version is tagged with `[server, counter]` pairs so concurrent edits can be detected instead of silently overwritten.


| Step | Event                                                                           | Vector Clock                                   |
| ---- | ------------------------------------------------------------------------------- | ---------------------------------------------- |
| 1    | Client writes D1 via server `Sx`                                                | `D1[(Sx,1)]`                                   |
| 2    | D1 → D2, written via `Sx`                                                       | `D2[(Sx,2)]`                                   |
| 3    | D2 → D3, written via `Sy`                                                       | `D3[(Sx,2),(Sy,1)]`                            |
| 4    | D2 → D4, written via `Sz` (concurrently)                                        | `D4[(Sx,2),(Sz,1)]`                            |
| 5    | Client reads D3 & D4 → **conflict detected** (both descend from D2 but diverge) | resolved manually → `D5[(Sx,3),(Sy,1),(Sz,1)]` |


**Key idea:** if one clock's history is a strict superset of another's, it's a clean overwrite. If neither contains the other, it's a conflict the client (or app logic) must resolve.

---



## 5. Handling Failures

**Detecting failures**

- **Gossip protocol** : nodes exchange heartbeats; if a node stops responding, peers mark it down.

**Temporary failures**

- **Sloppy quorum** : a healthy neighboring node temporarily accepts reads/writes on behalf of the down node.
- Once the original node recovers, changes are synced back via **hinted handoff**.

**Permanent failures**

- **Anti-entropy via Merkle trees** : each node builds a tree of hashes over its data.
- Compare root hashes between replicas: match → data identical; mismatch → walk down the tree to find just the differing chunks, instead of diffing everything.

**Data center failures**

- Power outages, network partitions, natural disasters can take out an entire DC.
- Mitigate by replicating data **across multiple data centers**, not just multiple nodes.

---



## Summary


| Requirement          | Solution                                  |
| -------------------- | ----------------------------------------- |
| Store large datasets | Consistent hashing + replication (N)      |
| Low latency          | Small W / R (fast quorum)                 |
| High availability    | Sloppy quorum, multi-DC replication       |
| Tunable consistency  | W + R vs N                                |
| Conflict resolution  | Vector clocks                             |
| Failure detection    | Gossip protocol                           |
| Node recovery        | Hinted handoff + Merkle tree anti-entropy |


[https://bytebytego.com/courses/system-design-interview/design-a-key-value-store](https://bytebytego.com/courses/system-design-interview/design-a-key-value-store)