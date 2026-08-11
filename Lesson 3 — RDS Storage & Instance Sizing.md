# ☁️ Amazon RDS — Lesson 3/10

## RDS Storage & Instance Sizing

Lesson 3 is strictly about **how to size Amazon RDS compute and storage for real workloads**.

The most important concept is:

> **RDS performance is not determined by CPU alone. You must size compute, memory, storage capacity, IOPS, throughput, and network capability together.**

---

# 1. The RDS Sizing Mental Model

Think of an RDS instance as having several independent dimensions:

```text
                 RDS
                  │
        ┌─────────┼─────────┐
        │         │         │
       CPU      Memory    Network
        │         │         │
        └─────────┼─────────┘
                  │
               Storage
                  │
        ┌─────────┼─────────┐
        │         │         │
     Capacity    IOPS    Throughput
```

A database can have:

```text
High CPU
Low IOPS
```

or:

```text
Low CPU
High I/O latency
```

or:

```text
Enough CPU
Enough IOPS
Insufficient memory
```

Each situation requires a different diagnosis.

---

# 2. What Is an RDS DB Instance Class?

The **DB instance class** determines the compute characteristics available to your database.

Conceptually:

```text
DB Instance Class
       │
       ├── vCPU
       ├── Memory
       ├── Network capability
       └── Other hardware characteristics
```

So when someone says:

> "Let's move to a larger RDS instance."

They're primarily talking about increasing the compute resources available to the DB instance.

---

# 3. CPU

CPU is consumed by database workloads such as:

* Complex queries
* Joins
* Sorting
* Aggregations
* Encryption-related processing
* High concurrency
* Database background activity

Example:

```text
CPU
100% ──────────────── 🔴
 80% ──────────────── 🟠
 50% ──────────────── 🟢
 20% ────────────────
```

But:

> **High CPU doesn't automatically mean "buy a bigger instance."**

First determine *why* CPU is high.

---

# 4. Memory

Memory is extremely important for database workloads.

It can help with:

```text
Caching
Query execution
Sort operations
Connections
Working data
Database processes
```

A database with insufficient memory may repeatedly access storage instead of serving frequently accessed data efficiently from memory.

Conceptually:

```text
Frequently accessed data
          │
          ▼
       Memory
          │
      ┌───┴───┐
      │       │
   Fast     Fast
```

Compared with repeatedly going to storage:

```text
Database
   │
   ▼
Storage
   │
   ▼
Higher latency
```

---

# 5. Why CPU and Memory Must Be Considered Together

Suppose you observe:

```text
CPU = 30%
Memory pressure = High
```

Increasing CPU may not solve the problem.

You need an instance configuration with more appropriate memory.

Conversely:

```text
CPU = 95%
Memory = Healthy
```

may point toward CPU-intensive workload or insufficient compute.

Therefore:

> **Choose instance classes based on workload characteristics, not just one metric.**

---

# 6. Storage Capacity

Storage capacity answers:

> **How much data can I store?**

For example:

```text
100 GB
500 GB
1 TB
5 TB
...
```

Capacity requirements include more than today's database size.

You should consider:

```text
Current data
+
Expected growth
+
Indexes
+
Temporary/operational needs
+
Maintenance requirements
+
Future growth
```

---

# 7. Storage Performance

Storage performance is different from storage capacity.

Think:

```text
Capacity
= How much?

IOPS
= How many I/O operations?

Throughput
= How much data per second?

Latency
= How quickly does each operation respond?
```

These are different characteristics.

---

# 8. What Is IOPS?

**IOPS = Input/Output Operations Per Second.**

Suppose your workload generates many small database reads/writes.

Conceptually:

```text
Application
    │
    ▼
Database
    │
    ├── Read
    ├── Read
    ├── Write
    ├── Read
    └── Write
          │
          ▼
       Storage
```

IOPS describes how many storage operations can be handled per second.

---

# 9. What Is Throughput?

Throughput measures the amount of data transferred per unit of time.

For example:

```text
MB/s
GB/s
```

Consider two workloads:

### Workload A

Many tiny I/O operations:

```text
1000s of small reads/writes
```

IOPS may be very important.

### Workload B

Large sequential operations:

```text
Large data transfers
```

Throughput may become more important.

Therefore:

> **IOPS and throughput are not interchangeable.**

---

# 10. What Is Storage Latency?

Latency is the time required for an I/O operation.

Conceptually:

```text
Request
   │
   ▼
Storage
   │
   ▼
Response
```

If storage latency increases:

```text
Query
 ↓
I/O wait
 ↓
Longer execution time
```

This can make applications slow even when CPU utilization looks normal.

---

# 11. Common RDS Storage Options

Amazon RDS supports different EBS-backed storage configurations depending on the engine, Region, and current RDS capabilities.

Common categories you'll encounter include:

```text
General Purpose SSD
Provisioned IOPS SSD
Magnetic / legacy configurations
```

For modern production deployments, the relevant choice is usually between **general-purpose SSD** and **provisioned IOPS SSD**, subject to engine and configuration support.

---

# 12. General Purpose SSD

General-purpose SSD storage is designed to provide a balance between:

```text
Cost
+
Performance
```

It's often appropriate for:

* General applications
* Moderate database workloads
* Development/test
* Many production workloads

But don't choose it blindly.

Measure the actual workload.

---

# 13. Provisioned IOPS SSD

Provisioned IOPS storage is designed for workloads requiring more predictable/high storage performance.

Typical scenarios might include:

```text
High transaction volume
I/O-intensive applications
Consistent performance requirements
Business-critical databases
```

The important idea:

> **You're paying for storage performance characteristics as well as capacity.**

---

# 14. Storage Autoscaling

RDS can support **storage autoscaling** for supported configurations.

Conceptually:

```text
Storage
100 GB
  │
  │ data grows
  ▼
150 GB
  │
  ▼
200 GB
  │
  ▼
250 GB
```

This can reduce the operational risk of running out of storage.

But:

> **Storage autoscaling is not a substitute for capacity planning.**

You should still monitor growth.

---

# 15. What Happens If RDS Runs Out of Storage?

This is a serious production event.

Potential consequences include:

```text
Writes fail
Database operations become impaired
Application errors
Maintenance problems
Operational instability
```

Therefore:

```text
Storage Used %
      ↓
Monitoring
      ↓
Alert
      ↓
Capacity action
```

should be part of production operations.

---

# 16. Storage Growth Planning

Suppose:

```text
Current DB = 800 GB
Monthly growth = 50 GB
```

Don't provision exactly:

```text
800 GB ❌
```

You need to account for expected growth and operational headroom.

A better planning process:

```text
Current size
    +
Growth forecast
    +
Safety margin
    +
Business requirements
```

Then validate the result against RDS storage limits and cost.

---

# 17. Instance Sizing — Don't Start With Guesswork

A common beginner approach:

> "Production database = large instance."

That's not engineering.

Instead:

```text
Workload
   ↓
Metrics
   ↓
Baseline
   ↓
Capacity requirement
   ↓
Instance selection
   ↓
Load testing
   ↓
Production
```

---

# 18. What Metrics Should You Collect?

At minimum, monitor:

```text
CPU utilization
Free memory / memory pressure indicators
Database connections
Storage utilization
Read IOPS
Write IOPS
Read latency
Write latency
Network throughput
Database load
```

Depending on engine/configuration, additional database-specific metrics are useful.

---

# 19. CPU Is 95% — Should You Scale?

Not automatically.

Suppose:

```text
CPU = 95%
Database load = High
Top SQL = Expensive SELECT
```

You may have a query problem.

Instead:

```text
Optimize query
      ↓
CPU decreases
      ↓
No immediate scaling required
```

Now consider:

```text
CPU = 95%
Queries are already optimized
Traffic has doubled
```

Scaling may be appropriate.

---

# 20. Low CPU but Slow Database

This is a classic production interview scenario.

Suppose:

```text
CPU = 25%
Application latency = 5 seconds
```

What could be happening?

Potential causes:

```text
I/O latency
Lock contention
Slow SQL
Connection waits
Network latency
Database waits
```

Therefore:

> **Low CPU does not mean the database is healthy.**

---

# 21. High I/O With Normal CPU

Example:

```text
CPU = 35%
IOPS = Very High
I/O latency = High
```

This suggests you should investigate storage behavior.

Possible actions:

```text
Analyze queries
Check access patterns
Review indexes
Check storage performance
Evaluate storage configuration
```

Don't simply increase CPU.

---

# 22. High Connections

Suppose:

```text
Connections = 95% of expected capacity
CPU = 30%
```

The database may still be under pressure.

Why?

Every connection has overhead.

Potential causes:

```text
Large connection pools
Connection leaks
Too many application instances
Traffic increase
Poor connection lifecycle management
```

The correct solution may involve **connection-pool optimization**, not a larger RDS instance.

---

# 23. Connection Pool Example

Suppose:

```text
10 application servers
```

Each creates:

```text
100 DB connections
```

Total:

```text
10 × 100 = 1000 connections
```

Even if each application isn't heavily using those connections, you've created a large connection footprint.

This is why connection architecture matters.

---

# 24. Storage vs Instance Scaling

These are different actions.

### Vertical compute scaling

```text
Small Instance
      ↓
Larger Instance
```

Improves things like:

```text
CPU
Memory
Potential network capability
```

### Storage scaling

```text
100 GB
 ↓
500 GB
```

Improves:

```text
Capacity
```

And depending on the storage configuration, may affect available storage performance.

Don't confuse these operations.

---

# 25. Vertical Scaling

Vertical scaling means increasing the size/capacity of the RDS DB instance.

Example:

```text
Current
   ↓
Smaller instance

       SCALE UP

   ↓

Larger instance
```

Useful when:

```text
CPU insufficient
Memory insufficient
Network capacity insufficient
Overall compute capacity insufficient
```

---

# 26. Horizontal Read Scaling

For supported RDS engines/configurations, **read replicas** can be used to scale read workloads.

Conceptually:

```text
                  Application
                 /     |      \
                /      |       \
               ▼       ▼        ▼
            Writer   Reader   Reader
              │
             RDS
```

We'll study read replicas in **Lesson 5**.

Important:

> **Read replicas are not simply "more CPU for the same database instance."**

They are separate database instances used for read scaling and other workloads.

---

# 27. A Practical Sizing Example

Suppose your current RDS instance has:

```text
CPU:              85%
Memory:           Healthy
Read IOPS:        Moderate
Write IOPS:       Moderate
Storage:          55%
Connections:      Healthy
```

Your first question should be:

> Why is CPU high?

Suppose Performance Insights shows one query consuming most database load.

Then:

```text
Optimize query
     ↓
CPU 85% → 45%
```

No immediate instance scaling required.

---

# 28. Another Sizing Example

Now:

```text
CPU:            90%
Memory:         Under pressure
IOPS:           High
Connections:    High
```

This is not necessarily one problem.

You may have a workload that has outgrown the current instance.

Investigation should include:

```text
Query workload
Connection behavior
Storage performance
Instance capacity
Traffic growth
```

Then determine the correct scaling strategy.

---

# 29. Production Capacity Planning

Use this model:

```text
                    CURRENT
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
            CPU      Memory    Storage
             │         │         │
             └─────────┼─────────┘
                       │
                  Workload Growth
                       │
                       ▼
                Future Capacity
                       │
                       ▼
                  RDS Sizing
```

Don't size based solely on today's utilization.

---

# 30. Benchmark Before Production

For important production systems:

```text
Prototype
   ↓
Load test
   ↓
Measure
   ↓
Tune
   ↓
Repeat
   ↓
Production
```

Measure:

```text
Throughput
Latency
CPU
Memory
IOPS
Connections
Errors
```

A database that performs well with 100 users may behave very differently with 10,000 concurrent requests.

---

# 31. Real-Time Scenario #1 — Storage is 90% full

What do you do?

### Step 1

Check growth rate.

### Step 2

Determine whether storage autoscaling is configured and applicable.

### Step 3

Increase/allow sufficient storage capacity.

### Step 4

Investigate why storage is growing.

Possible causes:

```text
Data growth
Logs
Temporary objects
Indexes
Unexpected workload
```

### Step 5

Create an alert before the next capacity threshold.

---

# 32. Real-Time Scenario #2 — CPU suddenly hits 100%

Don't immediately resize.

Check:

```text
1. Top SQL
2. Database load
3. Query frequency
4. Recent deployment
5. Traffic increase
6. Execution plans
```

Then decide:

```text
Bad SQL?
   ↓
Optimize

Legitimate workload growth?
   ↓
Scale
```

---

# 33. Real-Time Scenario #3 — I/O latency increases

You observe:

```text
CPU = 30%
I/O latency = High
Application latency = High
```

Investigate:

```text
Storage configuration
IOPS
Throughput
Top SQL
Query access patterns
Indexes
Database workload
```

This is a classic example of why CPU alone is insufficient.

---

# 34. Real-Time Scenario #4 — Database is slow only during peak traffic

Suppose:

```text
09:00 → Normal
12:00 → Normal
18:00 → Very slow
```

Compare:

```text
Traffic
CPU
Memory
Connections
IOPS
Latency
Top SQL
```

If every resource reaches a predictable peak, capacity planning may be required.

If one query suddenly dominates, SQL optimization may be the better solution.

---

# 35. Real-Time Scenario #5 — Storage is fine but performance is poor

Example:

```text
Storage used = 40%
CPU = 40%
I/O latency = High
```

The problem isn't storage capacity.

Investigate **storage performance**, query behavior, and workload characteristics.

This distinction is extremely important:

```text
Storage Capacity ≠ Storage Performance
```

---

# 36. Real-Time Scenario #6 — Increasing instance size didn't fix the problem

Suppose:

```text
Before:
CPU = 85%
Latency = 5 sec

After scaling:
CPU = 35%
Latency = 5 sec
```

What does this tell you?

The original bottleneck probably wasn't CPU alone.

Investigate:

```text
I/O
Locks
SQL
Connections
Network
Database waits
```

Scaling is not a substitute for diagnosis.

---

# 37. Real-Time Scenario #7 — Connections keep increasing

Suppose:

```text
08:00 → 100
10:00 → 300
12:00 → 500
14:00 → 800
```

Ask:

```text
Are connections being closed?
Is the application pool too large?
Did application instances increase?
Is there a connection leak?
Are idle connections accumulating?
```

This is often an application architecture problem rather than an RDS compute problem.

---

# 38. Real-Time Scenario #8 — Database growth is faster than expected

Suppose:

```text
Expected: 20 GB/month
Actual:   100 GB/month
```

Don't simply increase storage.

Investigate:

```text
Which tables grew?
Which indexes grew?
Did traffic increase?
Did a logging/audit feature change?
Did an application bug create duplicate data?
```

Capacity management should include **growth analysis**.

---

# 39. Real-Time Scenario #9 — Need more read capacity

Suppose:

```text
Writes = manageable
Reads = overwhelming
```

Potential architecture:

```text
                 Application
                 /    |    \
                ▼     ▼     ▼
             Writer Reader Reader
```

Read replicas may help.

But first verify:

```text
Can reads be routed?
Is replica lag acceptable?
Is the workload read-heavy?
```

We'll go deeper in Lesson 5.

---

# 40. Real-Time Scenario #10 — Management asks "Can we use the smallest RDS instance?"

Your answer should not be:

> "Yes."

Instead:

```text
Define workload
     ↓
Measure baseline
     ↓
Load test
     ↓
Determine capacity
     ↓
Select smallest instance
that meets:
     │
     ├── Performance
     ├── Availability
     ├── Growth
     └── Business requirements
```

The goal isn't:

> **Smallest instance.**

The goal is:

> **Smallest instance that safely satisfies the requirements.**

---

# 41. RDS Cost vs Performance

Larger instances generally cost more.

So your optimization objective is:

```text
                PERFORMANCE
                     ▲
                     │
                     │
          ┌──────────┼──────────┐
          │          │          │
        Under      Optimal    Over
      provisioned             provisioned
          │          │          │
          └──────────┴──────────┘
                     │
                    COST
```

You want the **optimal point**, not maximum hardware.

---

# 42. The RDS Sizing Decision Tree

Use this in production:

```text
                 Database Slow?
                      │
                      ▼
               Measure Metrics
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
       CPU          Memory          I/O
        │             │             │
     High?          Pressure?     Latency?
        │             │             │
        ▼             ▼             ▼
    SQL/CPU        Instance       Storage/
    workload       sizing         workload
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                 Connections?
                      │
                      ▼
                Database Load
                      │
                      ▼
                Root Cause
                      │
                      ▼
              Optimize / Scale
```

---

# 43. Interview Question — What factors determine RDS instance sizing?

A strong answer:

1. CPU requirements.
2. Memory requirements.
3. Database connection requirements.
4. Network throughput requirements.
5. Workload characteristics and concurrency.
6. Growth expectations and performance requirements.

---

# 44. Interview Question — What's the difference between IOPS and throughput?

1. IOPS measures the number of I/O operations per second.
2. Throughput measures the amount of data transferred per second.
3. Small random I/O workloads may be IOPS-sensitive.
4. Large sequential workloads may be more throughput-sensitive.
5. High IOPS does not automatically mean high throughput.
6. Both should be considered alongside latency and workload characteristics.

---

# 45. Interview Question — CPU is 90%, should you scale RDS?

Best answer:

1. Not immediately.
2. First identify what is consuming CPU.
3. Use database monitoring and Performance Insights where available.
4. Check top SQL and execution plans.
5. Determine whether the workload increased or the SQL became inefficient.
6. Scale only if the workload genuinely requires additional compute capacity.

---

# 46. Interview Question — Storage is 95% full. What do you do?

1. Assess how quickly storage is growing.
2. Check whether storage autoscaling is enabled and appropriate.
3. Increase capacity before reaching a critical condition.
4. Investigate which data or objects are driving growth.
5. Establish proactive storage alerts.
6. Incorporate growth trends into capacity planning.

---

# 47. Interview Question — Why isn't a larger RDS instance always the answer?

Because:

```text
Database performance
≠ CPU alone
```

You may instead have:

```text
I/O bottleneck
Lock contention
Poor SQL
Connection exhaustion
Network latency
Storage limitations
```

Scaling the wrong resource wastes money and may not improve performance.

---

# 48. The 5 Golden Metrics

If you're troubleshooting RDS performance, remember:

```text
       ┌──────────────┐
       │     CPU      │
       └──────────────┘
              │
 ┌────────────┼────────────┐
 ▼            ▼            ▼
Memory       I/O       Connections
 │            │            │
 └────────────┼────────────┘
              ▼
        Database Load
```

Then drill into:

```text
Top SQL
Waits
Latency
Execution plans
```

---

# 🧠 Lesson 3 — Must Remember

```text
1. Instance class determines compute capacity.
2. CPU and memory are different resources.
3. Storage capacity ≠ storage performance.
4. IOPS = operations per second.
5. Throughput = amount of data transferred per second.
6. Latency = time taken for an I/O operation.
7. High CPU doesn't automatically mean scale-up.
8. Low CPU doesn't guarantee good performance.
9. Connection count is a critical capacity consideration.
10. Always measure before changing instance or storage.
```

# 🔥 The Golden Rule

> **Don't ask "How big should my RDS instance be?" Ask "What does my workload require, and which resource is actually limiting performance?"**

---

### Next:

**Lesson 4 — RDS High Availability & Multi-AZ**

This is the lesson where we'll deeply cover **primary vs standby, synchronous replication, automatic failover, failover endpoints, what applications experience during failover, RTO, and real production failure scenarios.**
