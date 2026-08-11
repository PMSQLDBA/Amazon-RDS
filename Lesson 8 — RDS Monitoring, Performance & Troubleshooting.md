# ☁️ Amazon RDS — Lesson 8/10

## Monitoring, Performance & Troubleshooting

Lesson 8 is strictly about **Amazon RDS monitoring, performance analysis, bottleneck identification, and production troubleshooting**.

The key mental model:

> **Don't guess the bottleneck. Measure the database, identify the resource under pressure, then find the workload causing it.**

---

# 1. RDS Performance Model

Think of an RDS database like this:

```text
Application
    │
    ▼
Connections
    │
    ▼
Queries
    │
    ▼
Database Engine
    │
 ┌──┼─────────────┐
 ▼  ▼             ▼
CPU Memory       Storage/I/O
    │
    ▼
Performance
```

When an application becomes slow, the database isn't automatically the problem.

You need evidence.

---

# 2. The First Things to Check

When production reports:

> "The application is slow."

Check:

```text
1. RDS availability
2. CPU utilization
3. Memory pressure
4. Storage capacity
5. I/O activity
6. Database connections
7. Query latency
8. Locks/waits
9. Database load
10. Recent configuration/deployment changes
```

Don't immediately increase the DB instance size.

---

# 3. Amazon CloudWatch

**Amazon CloudWatch** is one of the primary monitoring services for RDS.

Conceptually:

```text
RDS
 │
 ├── CPU
 ├── Storage
 ├── Connections
 ├── I/O
 └── Other metrics
       │
       ▼
   CloudWatch
       │
       ▼
     Alarm
```

You can use these metrics to identify trends and trigger alerts.

---

# 4. CPUUtilization

One of the first metrics people check:

```text
CPUUtilization
```

Example:

```text
10% → 20% → 30%
```

Normal.

But:

```text
80% → 90% → 95% → 99%
```

could indicate CPU pressure.

Possible causes:

```text
Expensive queries
Too many concurrent queries
Poor query plans
Large scans
Increased application traffic
Maintenance operations
```

---

# 5. High CPU Does Not Automatically Mean Resize

Suppose:

```text
CPU = 95%
```

Bad response:

> "Increase the instance size immediately."

Better:

```text
CPU high
   ↓
Identify workload
   ↓
Find expensive queries
   ↓
Check query plans
   ↓
Optimize query/index/schema
   ↓
Resize if capacity is genuinely insufficient
```

Sometimes scaling is correct.

But optimization may be the better solution.

---

# 6. FreeableMemory

Memory pressure is another important area.

Conceptually:

```text
Instance Memory
      │
      ├── Database
      ├── Connections
      ├── Cache
      └── OS/other processes
```

Monitor:

```text
FreeableMemory
```

Persistently low available memory can indicate memory pressure.

Potential causes:

```text
Too many connections
Large queries
Memory-intensive operations
Insufficient instance size
Workload changes
```

---

# 7. Database Connections

Monitor:

```text
DatabaseConnections
```

Suppose your application suddenly opens:

```text
500
1000
2000
```

connections.

You may encounter connection exhaustion or increased resource pressure.

Remember:

> **More connections do not automatically mean more performance.**

Too many concurrent sessions can actually make performance worse.

---

# 8. Connection Pooling

A common production architecture is:

```text
Application
     │
     ▼
Connection Pool
     │
     ▼
RDS
```

Instead of:

```text
1000 requests
     │
     ▼
1000 database connections
```

you may have:

```text
1000 requests
     │
     ▼
Connection Pool
     │
     ▼
Controlled DB connections
```

This reduces unnecessary connection creation and database pressure.

---

# 9. Database Connections vs Connection Pool Size

Suppose:

```text
Application instances = 20
Pool size = 100 each
```

Potential maximum:

```text
20 × 100 = 2,000 connections
```

That's important.

Scaling application servers can unintentionally multiply database connections.

---

# 10. FreeStorageSpace

Monitor:

```text
FreeStorageSpace
```

Example:

```text
500 GB
 ↓
300 GB
 ↓
100 GB
 ↓
20 GB
```

You have a storage problem developing.

Don't wait until:

```text
0 GB
```

before responding.

---

# 11. Storage Pressure

Low storage can result from:

```text
Growing tables
Large indexes
Logs
Temporary data
Unexpected workload
Long-running transactions
Database maintenance
```

Investigate the actual cause.

Increasing storage may solve the immediate capacity problem, but you should understand why growth occurred.

---

# 12. Read IOPS and Write IOPS

I/O metrics help determine storage workload.

Conceptually:

```text
Queries
  │
  ├── Reads ──→ Read IOPS
  │
  └── Writes ─→ Write IOPS
```

High IOPS isn't automatically bad.

You need to compare:

```text
IOPS
+
Latency
+
Throughput
+
CPU
+
Query workload
```

---

# 13. Read Latency and Write Latency

Latency tells you how long I/O operations take.

For example:

```text
Read latency ↑
```

could indicate storage pressure or workload behavior.

Similarly:

```text
Write latency ↑
```

may indicate heavy write activity or storage constraints.

Always correlate latency with workload.

---

# 14. Throughput

Storage throughput measures how much data is being transferred.

Think:

```text
IOPS
→ Number of I/O operations

Throughput
→ Amount of data transferred
```

Both matter.

A workload doing many small operations behaves differently from one doing fewer large operations.

---

# 15. Burst Balance

Some storage configurations have burst-related behavior.

If burst capacity becomes depleted, performance characteristics can change.

When investigating storage performance, don't only ask:

> "Is IOPS high?"

Also investigate the storage type and its performance characteristics.

---

# 16. Performance Insights

**Amazon RDS Performance Insights** helps analyze database load and identify the database activity consuming resources.

Conceptually:

```text
Application
    │
    ▼
Database workload
    │
    ▼
Performance Insights
    │
    ├── SQL
    ├── Waits
    └── Database load
```

This is extremely useful for query-level performance investigations.

---

# 17. Database Load

Think:

```text
Database Load
      │
      ├── CPU
      ├── I/O waits
      ├── Lock waits
      └── Other waits
```

If database load suddenly increases, you want to determine:

```text
Which queries?
Which users?
Which waits?
Which resources?
```

---

# 18. Top SQL

Suppose Performance Insights shows:

```text
Query A → 70% of DB load
Query B → 15%
Query C → 5%
```

You should investigate Query A first.

This is much more effective than randomly optimizing queries.

---

# 19. Query Optimization

Suppose:

```sql id="j0r1s7"
SELECT *
FROM orders
WHERE customer_id = 100;
```

If `customer_id` isn't indexed and the table is huge, the database may need to inspect many rows.

An appropriate index may dramatically improve the query.

But:

> **Don't blindly create indexes.**

Indexes have costs.

---

# 20. Index Trade-Off

Indexes improve reads:

```text id="a5p44f"
SELECT
   ↓
Index
   ↓
Faster lookup
```

But indexes can increase:

```text id="k6x4a3"
INSERT cost
UPDATE cost
DELETE cost
Storage usage
Maintenance overhead
```

Therefore:

> Index based on workload and query plans.

---

# 21. EXPLAIN

For PostgreSQL, a key troubleshooting tool is:

```sql id="3z4p7c"
EXPLAIN
SELECT ...
```

It helps you understand the query plan.

For actual execution behavior, PostgreSQL also supports:

```sql id="o6tya0"
EXPLAIN ANALYZE
```

Use it carefully on production workloads because it actually executes the query.

---

# 22. Sequential Scan

Suppose you see:

```text id="wqbyl5"
Seq Scan
```

That means PostgreSQL is scanning the table sequentially.

This is not automatically bad.

For a tiny table:

```text id="9txe5v"
100 rows
```

a sequential scan can be perfectly reasonable.

For:

```text id="8h0zv0"
500 million rows
```

it may be expensive depending on the query.

Context matters.

---

# 23. Index Scan

An index can allow PostgreSQL to locate relevant rows more efficiently.

Conceptually:

```text id="e6xkjo"
Query
  ↓
Index
  ↓
Matching rows
```

But again:

> The optimizer chooses the plan based on statistics and cost estimates.

Don't assume an index will always be used.

---

# 24. Query Plan Problem

Suppose a query was fast yesterday:

```text id="6s7iop"
100 ms
```

Today:

```text id="8u8v7a"
8 seconds
```

Possible causes include:

```text
Data growth
Statistics changes
Different parameter values
Plan changes
Missing index
Locking
I/O pressure
CPU pressure
```

Don't immediately blame the application.

---

# 25. Locks

A database can be slow even when CPU isn't high.

Example:

```text id="n1e6j4"
Transaction A
      │
      ▼
Locks row
      │
      X
Transaction B
      │
      ▼
Waiting
```

Application sees:

```text id="w0n9s8"
Query taking 30 seconds
```

But CPU might be:

```text id="v8w1tq"
20%
```

Why?

Because the query is **waiting**, not computing.

---

# 26. Waits Are Extremely Important

When investigating performance, ask:

> **What is the database waiting for?**

Common categories:

```text id="0n8xqd"
CPU
I/O
Locks
Transactions
Network
Other database waits
```

Performance analysis becomes much easier when you identify the dominant wait.

---

# 27. Long-Running Transactions

A transaction that stays open for a long time can cause problems.

Conceptually:

```text id="6kn1kr"
BEGIN
  │
  ├── Work
  │
  ├── Work
  │
  ├── Work
  │
  └── COMMIT
```

If it remains open too long, it can:

```text
Hold locks
Prevent cleanup
Increase table bloat
Increase resource usage
Block other operations
```

---

# 28. PostgreSQL Connections

For PostgreSQL, inspect active sessions and transaction state using appropriate PostgreSQL system views.

For example:

```sql id="c3osjv"
SELECT pid,
       usename,
       state,
       query
FROM pg_stat_activity;
```

This can help identify:

```text
Active sessions
Idle sessions
Long-running queries
Users
Current SQL
```

---

# 29. Finding Long-Running Queries

Conceptually:

```text id="u1e1qk"
pg_stat_activity
      │
      ▼
Active sessions
      │
      ▼
Query duration
      │
      ▼
Long-running queries
```

Then investigate:

```text id="b38o1n"
What query?
Who started it?
Why is it slow?
Is it blocked?
Is it doing expensive I/O?
Is the query plan appropriate?
```

---

# 30. Slow Query Scenario

Application:

> "Orders page takes 15 seconds."

Start:

```text id="9ny5kj"
CloudWatch
   ↓
CPU normal
Memory normal
Storage normal
```

So infrastructure isn't obviously saturated.

Next:

```text id="j0k8lo"
Performance Insights
   ↓
Top SQL
```

Find:

```text id="8d0g7r"
SELECT ...
```

Then:

```text id="z9xj3p"
EXPLAIN
   ↓
Query plan
   ↓
Missing/inefficient index
```

Now you have evidence.

---

# 31. Real-Time Scenario #1 — CPU at 95%

```text id="2z8w8v"
CPU 95%
```

Investigation:

```text id="p6sp6q"
Performance Insights
        ↓
Top SQL
        ↓
Expensive query
        ↓
EXPLAIN
        ↓
Optimize
```

If workload legitimately exceeds capacity:

```text id="b4q4ca"
Scale instance
```

Optimization and scaling aren't mutually exclusive.

---

# 32. Real-Time Scenario #2 — CPU 20%, Application Slow

This is a classic interview scenario.

Don't say:

> "Increase the instance size."

Instead:

```text id="2j8j7p"
CPU = 20%
    ↓
Check waits
    ↓
Check locks
    ↓
Check I/O latency
    ↓
Check connections
    ↓
Check query latency
```

The database may be waiting on locks or storage.

---

# 33. Real-Time Scenario #3 — Connections Near Maximum

Suppose:

```text id="s6c5j7"
DatabaseConnections
        ↓
Near configured/engine limit
```

Possible causes:

```text
Connection leak
Pool too large
Too many application instances
Traffic spike
Long-running sessions
Idle connections
```

Fix the cause rather than blindly increasing the database size.

---

# 34. Real-Time Scenario #4 — Storage Almost Full

```text id="h1f3td"
FreeStorageSpace ↓↓↓
```

Immediate actions:

```text
1. Confirm growth trend.
2. Determine what is consuming storage.
3. Check whether storage autoscaling/configuration is appropriate.
4. Increase capacity if required.
5. Investigate abnormal growth.
```

Don't wait for the database to hit zero free space.

---

# 35. Real-Time Scenario #5 — Sudden Write Latency

Suppose:

```text id="k2c8b6"
Write latency
     ↑↑↑
```

Check:

```text id="k3q9as"
Write workload
IOPS
Throughput
Storage configuration
Long transactions
Lock contention
Recent application changes
```

Find the bottleneck before changing architecture.

---

# 36. Real-Time Scenario #6 — Deadlocks

A deadlock can occur when:

```text id="4h2vca"
Transaction A
locks Row 1
   ↓
waits for Row 2

Transaction B
locks Row 2
   ↓
waits for Row 1
```

Result:

```text id="8a0h5b"
A waits for B
B waits for A
```

Database detects the deadlock and resolves it according to engine behavior.

Your application should be designed to handle retryable transaction failures appropriately.

---

# 37. Real-Time Scenario #7 — Query Suddenly Became Slow

Timeline:

```text id="e9y6az"
10:00 → 100 ms
11:00 → 100 ms
12:00 → 5 sec
```

Investigate:

```text id="tqfl5q"
Data volume
Query plan
Statistics
Indexes
Locks
I/O
CPU
Recent deployments
Parameter values
```

The key is comparing the **before vs after** state.

---

# 38. Real-Time Scenario #8 — Application Traffic Doubles

Before:

```text id="n40l2m"
1,000 requests/min
```

After:

```text id="1o6e7g"
2,000 requests/min
```

Database impact:

```text id="o2x1e9"
Connections ↑
CPU ↑
Queries ↑
IOPS ↑
Latency ↑
```

Possible responses:

```text id="8v8vxx"
Query optimization
Connection pooling
Caching
Read scaling
Instance scaling
Application optimization
```

Choose based on the actual bottleneck.

---

# 39. Real-Time Scenario #9 — One Query Consumes Most DB Load

Performance Insights:

```text id="h4v3xq"
Query A → 80%
Query B → 8%
Query C → 4%
```

Don't spend hours optimizing Query C.

Start with:

```text id="1m1b1h"
Query A
   ↓
EXPLAIN
   ↓
Indexes / joins / filters
   ↓
Optimization
```

Focus on the biggest contributors first.

---

# 40. Real-Time Scenario #10 — Database Works but Application Times Out

Architecture:

```text id="u0b2y7"
Application
    │
    ▼
Connection
    │
    ▼
Query
    │
    ▼
Database
```

Possible issue:

```text id="3byd1h"
Database query = 20 seconds
Application timeout = 5 seconds
```

The database may be functioning normally from its perspective.

But the application considers the request failed.

Therefore performance troubleshooting must consider **end-to-end latency**.

---

# 41. CloudWatch vs Performance Insights

Remember the difference.

### CloudWatch

Good for:

```text id="x0n3uk"
CPU
Memory-related metrics
Storage
Connections
I/O
Network
Alarms
Infrastructure trends
```

### Performance Insights

Good for:

```text id="c2h4jz"
Database load
SQL activity
Waits
Query-level investigation
Database performance diagnosis
```

Use them together.

---

# 42. Enhanced Monitoring

RDS Enhanced Monitoring provides more granular OS-level metrics than standard CloudWatch RDS metrics.

Conceptually:

```text id="5p0tbd"
RDS
 │
 ├── CloudWatch metrics
 │
 └── Enhanced Monitoring
        │
        ▼
    OS-level detail
```

This can help investigate CPU, memory, processes, and other operating-system-level behavior.

---

# 43. Monitoring Layers

Use this model:

```text id="v2k0jv"
Application
    │
    ▼
Application Metrics
    │
    ▼
CloudWatch
    │
    ▼
RDS Metrics
    │
    ▼
Performance Insights
    │
    ▼
Database / SQL
```

The deeper you investigate, the more specific your diagnosis becomes.

---

# 44. Alarms

Don't just collect metrics.

Create meaningful alarms.

Examples:

```text id="g4l1wb"
CPU unusually high
Free storage critically low
Connection count unusually high
Replication lag high where applicable
```

But avoid creating hundreds of noisy alerts.

An alert should trigger an actionable response.

---

# 45. Baselines

You need to know what "normal" means.

Example:

```text id="t1f8t3"
Normal CPU:
30–50%

Normal connections:
100–200

Normal latency:
20–50 ms
```

Then:

```text id="g2s8p7"
CPU = 95%
Connections = 900
Latency = 2 sec
```

becomes meaningful.

Without a baseline, a metric alone tells you less.

---

# 46. Performance Troubleshooting Framework

Use this every time:

```text id="c4px5c"
Symptom
   ↓
Metric
   ↓
Resource
   ↓
Workload
   ↓
Query
   ↓
Root Cause
   ↓
Fix
   ↓
Validate
```

Example:

```text id="v2l7cs"
Slow API
   ↓
High DB latency
   ↓
I/O wait
   ↓
Expensive SQL
   ↓
Missing index
   ↓
Add/optimize index
   ↓
Measure again
```

---

# 47. Don't Confuse Symptoms With Causes

Example:

```text id="9a8m3j"
CPU = 95%
```

CPU is a **symptom**.

Possible root cause:

```text id="6f2m9a"
Bad query
```

Similarly:

```text id="l2o3b5"
High latency
```

could be caused by:

```text
Locks
I/O
Network
CPU
Query plan
Connection exhaustion
```

Always investigate the chain.

---

# 48. Production Performance Checklist

```text id="9by6fr"
☐ CloudWatch metrics monitored
☐ CPU monitored
☐ Memory monitored
☐ Storage monitored
☐ Connections monitored
☐ I/O monitored
☐ Performance Insights enabled where appropriate
☐ Slow SQL investigated
☐ Query plans reviewed
☐ Locks/waits investigated
☐ Connection pooling configured appropriately
☐ Baselines established
☐ Meaningful alarms configured
☐ Capacity trends reviewed
☐ Performance tests performed
```

---

# 49. Interview Question — CPU Is 95%. What Do You Do?

Strong answer:

1. Confirm the duration and whether it is a sustained condition.
2. Check Performance Insights/database load.
3. Identify top SQL contributing to load.
4. Inspect query plans and waits.
5. Check whether recent deployments or traffic changes caused it.
6. Optimize queries where possible.
7. Scale the RDS instance if the workload genuinely requires more capacity.
8. Validate performance after the change.

---

# 50. Interview Question — CPU Is Low but Database Is Slow

Strong answer:

> I would investigate waits rather than assuming CPU capacity is the issue.

Check:

```text
Locks
I/O latency
Storage
Connections
Long-running transactions
Query plans
Network/application timeouts
```

---

# 51. Interview Question — How Do You Find the Problematic Query?

Answer:

```text id="2s8f4m"
Performance Insights
      ↓
Database load
      ↓
Top SQL
      ↓
Identify expensive query
      ↓
EXPLAIN / EXPLAIN ANALYZE
      ↓
Optimize
      ↓
Measure again
```

---

# 52. Interview Question — CloudWatch vs Performance Insights?

Answer:

> CloudWatch provides infrastructure/service-level metrics and alarms, while Performance Insights provides deeper database workload visibility, including database load, SQL activity, and waits. They complement each other.

---

# 53. The Complete RDS Performance Architecture

```text id="1x8h3k"
                    APPLICATION
                         │
                         ▼
                 Request Latency
                         │
                         ▼
                  Connection Pool
                         │
                         ▼
                       RDS
                         │
          ┌──────────────┼──────────────┐
          │              │              │
         CPU           Memory          I/O
          │              │              │
          └──────────────┼──────────────┘
                         │
                         ▼
                  Database Load
                         │
                         ▼
                Performance Insights
                         │
                  ┌──────┴──────┐
                  ▼             ▼
                 SQL           Waits
                  │             │
                  ▼             ▼
              Query Plan     Locks/I/O
                  │
                  ▼
               Optimize
```

---

# 🧠 Lesson 8 — Must Remember

```text
1. Don't troubleshoot performance by guessing.
2. CloudWatch gives important RDS metrics.
3. Performance Insights helps identify database load and SQL.
4. High CPU does not automatically mean resize.
5. Low CPU does not mean the database cannot be slow.
6. Connections can become a bottleneck.
7. Storage capacity and I/O latency both matter.
8. Locks and waits can cause severe latency.
9. EXPLAIN helps understand PostgreSQL query plans.
10. Always measure before and after a performance change.
```

# 🔥 Golden Rule

> **Find the bottleneck first; then choose the fix.**

---

## 🎯 Lesson 8 Complete

**Lesson 9 — RDS Scaling, Read Replicas, High Availability & Disaster Recovery**

**Lesson 10 — RDS Production Architecture, Migration, Operations & Real-World Interview Scenarios**
