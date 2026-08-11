# ☁️ Amazon RDS — Lesson 5/10

## Read Replicas & Read Scaling

Lesson 5 is strictly about **Amazon RDS Read Replicas** and how to scale read-heavy workloads.

The most important distinction:

> **Multi-AZ is primarily for high availability. Read Replicas are primarily for scaling read workloads.**

---

# 1. Why Do We Need Read Replicas?

Imagine an application receiving:

```text
100,000 database operations
        │
        ├── 90,000 reads
        └── 10,000 writes
```

If everything goes to one database:

```text
                 Application
                      │
                      ▼
                  RDS Primary
                 /     |      \
              Read   Read    Write
```

The primary may become overloaded by read traffic.

Read replicas allow you to distribute read workloads.

---

# 2. Basic Read Replica Architecture

Conceptually:

```text
                       Application
                    /      |       \
                   /       |        \
                  ▼        ▼         ▼
              Primary   Replica-1  Replica-2
                │           ▲          ▲
                │           │          │
                └───────────┴──────────┘
                   Replication
```

The primary handles writes.

Replicas can handle eligible read workloads.

---

# 3. What Is a Read Replica?

A Read Replica is a separate RDS DB instance that receives replicated data from a source database.

Conceptually:

```text
Source DB
   │
   │ Replication
   ▼
Read Replica
```

The replica maintains a copy of the source data for read-oriented workloads.

---

# 4. Why Use Read Replicas?

Common reasons include:

```text id="6t2yfp"
Read scaling
Reporting
Analytics
BI workloads
Read-heavy applications
Reducing read load on primary
Geographically closer reads in supported architectures
```

The core use case:

> **Move eligible read workloads away from the primary.**

---

# 5. Read Replica vs Multi-AZ

This is one of the most important interview comparisons.

| Multi-AZ                        | Read Replica                       |
| ------------------------------- | ---------------------------------- |
| High availability               | Read scaling                       |
| Standby                         | Readable replica                   |
| Failover-focused                | Read workload-focused              |
| Primarily protects availability | Primarily increases read capacity  |
| Not intended for normal reads   | Designed for read workloads        |
| Replication is HA-oriented      | Generally asynchronous replication |

Remember:

```text id="r7u2c8"
HA requirement
     ↓
Multi-AZ

Read scaling requirement
     ↓
Read Replica
```

---

# 6. Read Replicas Are Usually Asynchronous

Conceptually:

```text id="8p5w4k"
Primary
   │
   │ write
   ▼
Replication
   │
   ▼
Replica
```

The replica may temporarily be behind the primary.

This creates an important concept:

> **Replica lag.**

---

# 7. What Is Replica Lag?

Suppose the primary has:

```text id="k0j4t7"
Order #1001
Order #1002
Order #1003
Order #1004
```

The replica currently has:

```text id="p3m9x1"
Order #1001
Order #1002
```

The replica hasn't caught up yet.

That difference is replication lag.

Conceptually:

```text id="3h8s6f"
Primary
Data position
      │
      │
      │  lag
      ▼
Replica
Data position
```

---

# 8. Why Does Replica Lag Matter?

Imagine a user submits:

```text id="x5k9z2"
Create Order #500
```

The application immediately queries a read replica:

```text id="q2c7w8"
"Show my latest order."
```

If replication hasn't caught up:

```text id="8s1h5v"
Primary → Order exists
Replica → Order doesn't exist yet
```

The user might see stale data.

---

# 9. Read-After-Write Consistency Problem

This is a classic production scenario.

```text id="w9c1k4"
WRITE
  │
  ▼
Primary
  │
  │ asynchronous replication
  ▼
Replica
```

Immediately:

```text id="b6m0v3"
READ
  │
  ▼
Replica
```

The read may not see the recent write.

This is called a **read-after-write consistency challenge**.

---

# 10. How Do We Handle Read-After-Write?

One common application strategy is:

```text id="y8r2w6"
Write request
     │
     ▼
Primary
     │
     ▼
For immediate reads
     │
     ▼
Primary
```

Then:

```text id="n7x4j1"
Normal read-only traffic
        │
        ▼
Read Replica
```

Conceptually:

```text id="q9f2k5"
               Application
              /           \
             ▼             ▼
          Writes       Read-heavy
             │             │
             ▼             ▼
          Primary       Replica
```

The application decides where each request goes.

---

# 11. Read/Write Splitting

This pattern is called **read/write splitting**.

```text id="z4g6n8"
Application
     │
     ├── WRITE ───────► Primary
     │
     └── READ ────────► Replica
```

This can significantly reduce read pressure on the primary.

But the application must understand consistency requirements.

---

# 12. Not Every Read Should Go to a Replica

Suppose:

```text id="h2v8s5"
User creates payment
```

Immediately after:

```text id="m6x1q4"
Application checks payment status
```

If that read is sent to a lagging replica:

```text id="p5c8r2"
Payment = "Not found"
```

That could be unacceptable.

So:

```text id="s3d7k9"
Critical immediate read
       ↓
Primary

Eventually consistent read
       ↓
Replica
```

---

# 13. Good Read Replica Workloads

Good candidates often include:

```text id="u4n8x2"
Product catalog browsing
Search
Reporting
Dashboards
Analytics
Historical queries
Read-heavy APIs
```

provided the application's consistency requirements allow replica reads.

---

# 14. Bad Read Replica Workloads

Be careful with:

```text id="w7k3p1"
Immediate post-write validation
Strong consistency requirements
Critical transaction verification
Workloads requiring latest data
```

unless the application explicitly handles replica lag.

---

# 15. Multiple Read Replicas

You can have multiple replicas where supported.

Conceptually:

```text id="b3v9m6"
                  Primary
                 /   |    \
                /    |     \
               ▼     ▼      ▼
            Replica Replica Replica
               1       2       3
```

Applications can distribute read workloads among them.

---

# 16. Scaling Reads

Suppose:

```text id="g8x1s5"
Primary
Read load = 90%
Write load = 10%
```

Add replicas:

```text id="q2m7r9"
                Primary
               /       \
              ▼         ▼
          Replica-1   Replica-2
```

Then:

```text id="c5n4k8"
Read traffic
   ├── Replica-1
   └── Replica-2

Write traffic
   └── Primary
```

This can increase read capacity.

---

# 17. Vertical vs Horizontal Read Scaling

### Vertical scaling

```text id="j1f8q4"
Small RDS
    ↓
Larger RDS
```

You increase compute capacity.

### Read scaling

```text id="r7x2m9"
One DB
  ↓
Multiple read replicas
```

You distribute read workloads.

This is a fundamental architectural distinction.

---

# 18. Replica Lag

Replica lag can occur because replication can't keep up with the source workload.

Possible causes:

```text id="m8v4z2"
High write volume
Heavy transactions
Replica under-sized
Long-running queries
Resource contention
Network/replication conditions
```

The exact behavior depends on the RDS engine.

---

# 19. What Should You Do When Replica Lag Increases?

Don't immediately add another replica.

First investigate:

```text id="x3q6k1"
1. Replica CPU
2. Replica memory
3. Replica I/O
4. Replication workload
5. Long-running queries
6. Write volume on source
7. Replica configuration
```

Then determine the root cause.

---

# 20. Real-Time Scenario #1 — Replica Lag = 0

Suppose:

```text id="k6v1q8"
Replica lag = 0
```

Excellent.

It means the replica is currently keeping up with the source according to the relevant engine metric.

But it doesn't mean:

> "The replica can never fall behind."

Lag is dynamic.

---

# 21. Real-Time Scenario #2 — Replica Lag Suddenly Increases

Example:

```text id="f3z9p2"
10:00 → 0 sec
10:10 → 2 sec
10:20 → 15 sec
10:30 → 60 sec
```

Investigate:

```text id="u8c5m4"
Did write traffic increase?
Did a large transaction start?
Is replica CPU high?
Is replica I/O saturated?
Are long-running queries consuming resources?
```

Use metrics and database-level investigation.

---

# 22. Real-Time Scenario #3 — Reporting Query Hurts Production

Suppose a BI team runs:

```sql id="0o7qk4"
SELECT ...
FROM huge_orders_table
JOIN ...
GROUP BY ...
```

on the primary.

The application slows down.

A read replica can be a better place for appropriate reporting workloads:

```text id="a9f5x2"
Production writes
      │
      ▼
   Primary
      │
      │ replication
      ▼
   Replica
      │
      ▼
Reporting
```

This isolates some reporting workload from the primary.

---

# 23. Real-Time Scenario #4 — User Doesn't See Newly Created Record

User:

```text id="n4j8x1"
Creates order
```

Application:

```text id="q7p2m6"
Writes → Primary
Reads → Replica
```

Replica is behind.

Result:

```text id="d5r9w3"
"Order not found"
```

Fix options include:

```text id="x2k8v4"
Route immediate read to primary
OR
Implement consistency-aware routing
OR
Wait until replica catches up where appropriate
```

---

# 24. Real-Time Scenario #5 — Primary CPU Is High Due to Reads

Metrics:

```text id="s8w3m7"
CPU = 90%
Reads = 85%
Writes = 15%
```

Investigate:

```text id="p1q6x9"
Can reads be routed to replicas?
Are queries optimized?
Are indexes appropriate?
Would caching help?
```

If the workload is suitable:

```text id="r4v7c2"
Primary
  │
  ├── Writes
  │
  └── Some reads
       ↓
   Read Replica
       ↓
   More reads
```

---

# 25. Real-Time Scenario #6 — Replica Is Too Small

Suppose:

```text id="w6h2j9"
Primary:
CPU = 40%

Replica:
CPU = 95%
```

The replica may be under-sized for its workload.

Remember:

> **The replica is its own DB instance with its own compute resources.**

You may need to scale the replica appropriately.

---

# 26. Real-Time Scenario #7 — Multiple Replicas Still Don't Help

Suppose:

```text id="c8k5x3"
Replica-1
Replica-2
Replica-3
```

but application latency remains high.

Don't assume:

> "We need Replica-4."

Investigate:

```text id="y2v6m8"
Slow SQL
Missing indexes
Application bottlenecks
Network latency
Connection pooling
Caching opportunities
Replica lag
```

Replication doesn't fix inefficient queries.

---

# 27. Real-Time Scenario #8 — Replica Used for Analytics

Suppose analytics runs heavy queries:

```text id="r1x9p5"
SELECT ...
JOIN ...
GROUP BY ...
ORDER BY ...
```

If the replica is shared with application reads:

```text id="k7m3z8"
Analytics
    │
    ▼
Replica
    ▲
    │
Application Reads
```

Analytics could consume the replica's resources.

You may need workload isolation through additional replicas or a different analytics architecture.

---

# 28. Real-Time Scenario #9 — Replica Has High I/O

Metrics:

```text id="m5q8v2"
CPU = 35%
I/O = High
Latency = High
```

Investigate:

```text id="n3c7x9"
Reporting queries
Large scans
Indexes
Storage performance
Query execution plans
```

Again:

> **CPU is only one dimension of database performance.**

---

# 29. Real-Time Scenario #10 — Replica Is Used for Disaster Recovery

A Read Replica can sometimes play a role in recovery architectures because it can potentially be promoted.

Conceptually:

```text id="e8x4k1"
Primary
   │
   ▼
Read Replica
   │
   │ Promotion
   ▼
Independent Primary
```

But don't confuse this with the standard Multi-AZ standby architecture.

For production DR planning, consider:

```text id="h6v2m9"
RPO
RTO
Region
Replication
Backups
Promotion
Application recovery
```

---

# 30. What Happens If the Primary Fails?

This is where people confuse Read Replicas and Multi-AZ.

With Multi-AZ:

```text id="b2m7x4"
Primary
   ↓
Standby
   ↓
Automatic failover
```

With a Read Replica:

```text id="q9k3v6"
Primary
   ↓
Read Replica
   ↓
Potential promotion
```

A Read Replica isn't simply an automatic Multi-AZ standby.

---

# 31. Promotion of a Read Replica

A Read Replica can be promoted in supported RDS scenarios.

Conceptually:

```text id="f4x8z1"
Read Replica
      │
      ▼
  Promotion
      │
      ▼
Independent DB instance
```

After promotion, it becomes a standalone database instance rather than continuing as the same replica relationship.

---

# 32. Why Promote a Read Replica?

Potential reasons:

```text id="s7v1q5"
Disaster recovery
Migration
Workload isolation
Testing
Creating an independent database
```

The exact operational implications depend on the engine and architecture.

---

# 33. Read Replica and Backup

Read replicas and backups solve different problems.

```text id="x3m8v2"
Read Replica
      ↓
Read scaling / replication
```

```text id="q6p4n9"
Backup
      ↓
Data recovery
```

Don't treat a replica as a replacement for backups.

---

# 34. Read Replica and Data Protection

Suppose:

```text id="w1r5c7"
DELETE FROM orders;
```

If the change is replicated:

```text id="m9x2k4"
Primary
   ↓
Delete replicated
   ↓
Replica
```

The replica doesn't provide a historical copy of the pre-delete state.

For that, you need backups/PITR.

---

# 35. Read Routing Architecture

A more mature application might use:

```text id="j8q3v6"
                 Application
                      │
              ┌───────┴────────┐
              │                │
             WRITE            READ
              │                │
              ▼                ▼
           Primary         Read Router
                               │
                     ┌─────────┼─────────┐
                     ▼         ▼         ▼
                   Rep-1     Rep-2     Rep-3
```

The application or database access layer determines where queries go.

---

# 36. Read Replica Selection

If you have multiple replicas, routing might consider:

```text id="v4n7x2"
Replica health
Replica lag
Latency
Geographic location
Current load
Workload type
```

A sophisticated application doesn't blindly send every read to the same replica.

---

# 37. Replica Lag and User Experience

Suppose:

```text id="a2c8m5"
Replica lag = 30 seconds
```

A user might see:

```text
"I uploaded my document."
```

Then immediately:

```text
"My document isn't there."
```

The backend might actually be correct:

```text
Primary → document exists
Replica → document hasn't arrived yet
```

The issue is **consistency**, not necessarily database corruption.

---

# 38. Eventual Consistency

With asynchronous replication, a replica can temporarily represent an earlier state.

Conceptually:

```text id="f9v3k6"
Time T0
Primary = A
Replica = A

Time T1
Primary = B
Replica = A

Time T2
Primary = B
Replica = B
```

Between T1 and T2:

```text id="w5x1r8"
Replica is stale.
```

This is why applications must understand consistency requirements.

---

# 39. Read Replica Sizing

Don't assume the replica should always have the same size as the primary.

It depends on:

```text id="k3m7q9"
Read workload
Query complexity
Concurrency
Reporting workload
Storage requirements
CPU
Memory
I/O
```

Example:

```text id="d8x2v5"
Primary
Heavy writes + moderate reads

Replica
Heavy analytical reads
```

The optimal sizing may differ.

---

# 40. Cost Consideration

Every replica is another DB instance with its own costs.

Therefore:

```text id="q4m9x7"
Replica-1
Replica-2
Replica-3
Replica-4
```

shouldn't be added just because:

> "More replicas = better."

Instead:

```text id="v8c3n2"
Measure
  ↓
Identify bottleneck
  ↓
Estimate required read capacity
  ↓
Add appropriate replicas
```

---

# 41. Read Replicas vs Caching

Not every read-scaling problem needs a replica.

Suppose:

```text id="p2k7x4"
Same product requested
1 million times
```

A cache may be much more efficient:

```text id="g6v9m1"
Application
    │
    ▼
Cache
    │
    └── Cache miss
           ↓
          RDS
```

A good architecture chooses the appropriate layer.

---

# 42. Read Replicas vs Query Optimization

Suppose one query performs a huge table scan.

Adding replicas:

```text id="n5x8q2"
Primary
 ├── Replica
 ├── Replica
 └── Replica
```

may distribute the bad query but doesn't fix it.

Better:

```text id="r7m3v1"
Bad SQL
  ↓
Execution plan
  ↓
Index/query optimization
  ↓
Lower resource consumption
```

---

# 43. Read Replica Decision Tree

Use this:

```text id="k8x2m4"
Read workload too high?
        │
        ▼
Optimize SQL first
        │
        ▼
Consider caching
        │
        ▼
Still insufficient?
        │
        ▼
Read Replica
        │
        ▼
Monitor replica lag
        │
        ▼
Add replicas if justified
```

---

# 44. Production Read Scaling Architecture

A mature design might look like:

```text id="q6v9x3"
                         Users
                           │
                           ▼
                     Application
                      /       \
                     /         \
                Writes          Reads
                  │               │
                  ▼               ▼
              RDS Primary     Read Router
                                  │
                     ┌────────────┼────────────┐
                     ▼            ▼            ▼
                  Replica-1    Replica-2    Replica-3
                     │            │            │
                  Reads         Reads         Reads
```

Alongside:

```text id="m4z7p1"
Primary
  │
  └── Multi-AZ standby
```

And:

```text id="w2c8k5"
Backups
```

Now you have separate mechanisms for:

```text id="y5n1r9"
HA
Read scaling
Recovery
```

---

# 45. Interview Question — What is a Read Replica?

A strong answer:

1. A Read Replica is a separate RDS DB instance that replicates data from a source database.
2. Its primary purpose is to scale read workloads.
3. Replication is generally asynchronous.
4. This means replica lag can occur.
5. Applications can route suitable read-only workloads to replicas.
6. A replica can potentially be promoted for certain recovery or migration scenarios.

---

# 46. Interview Question — What is replica lag?

1. Replica lag is the delay between changes on the source and their availability on the replica.
2. It occurs with asynchronous replication.
3. High write volume can increase lag.
4. Resource contention on the replica can increase lag.
5. Applications must account for possible stale reads.
6. Monitoring replica lag is important for production read routing.

---

# 47. Interview Question — Why not send all reads to Read Replicas?

Because some reads require the latest committed state.

For example:

```text id="q1v8m4"
Write order
   ↓
Immediately read order
```

If the read goes to a lagging replica:

```text id="c7x3k9"
Order may not appear yet.
```

Therefore, consistency requirements must determine routing.

---

# 48. Interview Question — Multi-AZ vs Read Replica?

Use this concise answer:

> **Multi-AZ is primarily for high availability and failover. Read Replicas are primarily for scaling read workloads. Multi-AZ uses a standby architecture, while Read Replicas are readable copies that generally use asynchronous replication and can experience replica lag.**

---

# 49. The Golden Architecture

Memorize this:

```text id="u5m8x2"
                         APPLICATION
                       /             \
                      /               \
                  WRITE               READ
                    │                  │
                    ▼                  ▼
               RDS PRIMARY        READ ROUTER
                    │              /    |    \
                    │             ▼     ▼     ▼
                    │           Rep-1 Rep-2 Rep-3
                    │
                    ▼
               Multi-AZ
                 Standby
```

And separately:

```text id="j7c3v9"
Backup
  ↓
Recovery
```

So:

```text id="e2x8m5"
Multi-AZ    → Availability
Replica     → Read Scaling
Backup      → Recovery
```

---

# 🧠 Lesson 5 — Must Remember

```text
1. Read Replicas are primarily for read scaling.
2. They are separate RDS DB instances.
3. Replication is generally asynchronous.
4. Replica lag is therefore possible.
5. Read-after-write consistency must be considered.
6. Writes normally go to the primary.
7. Suitable reads can go to replicas.
8. Read Replicas are not backups.
9. Read Replicas are not the same as Multi-AZ.
10. Optimize SQL and consider caching before blindly adding replicas.
```

## 🔥 Golden Rule

> **Scale reads with Read Replicas, protect availability with Multi-AZ, and protect data with backups.**

---

## 🎯 Lesson 5 Complete

### Next: **Lesson 6 — RDS Backups, Snapshots & Point-in-Time Recovery**

We'll cover **automated backups, manual snapshots, transaction-log-based recovery, retention, PITR, restore scenarios, accidental deletion, regional disaster scenarios, RPO/RTO, and real production recovery procedures.**
