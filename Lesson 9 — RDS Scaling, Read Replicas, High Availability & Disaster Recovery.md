# ☁️ Amazon RDS — Lesson 9/10

## Scaling, Read Replicas, High Availability & Disaster Recovery

Lesson 9 is strictly about **how to scale Amazon RDS, how Read Replicas differ from Multi-AZ, and how to design RDS for HA and DR**.

The most important mental model:

> **Scale up for more database capacity, Read Replicas for read scaling, Multi-AZ for high availability, and cross-Region DR for larger disaster scenarios.**

---

# 1. The RDS Scaling Problem

Imagine your application grows:

```text
Year 1
100 requests/sec
      ↓
RDS performs well

Year 2
1,000 requests/sec
      ↓
RDS under pressure

Year 3
10,000 requests/sec
      ↓
Architecture must evolve
```

You need to determine **what exactly is limiting performance**.

Possible bottlenecks:

```text
CPU
Memory
Storage
IOPS
Connections
Read workload
Write workload
Database design
```

---

# 2. Two Major Types of Scaling

RDS scaling broadly falls into:

### Vertical scaling

```text
Small instance
     ↓
Larger instance
```

### Horizontal/read scaling

```text
              RDS Primary
             /     |      \
            /      |       \
       Replica   Replica   Replica
```

The right approach depends on the workload.

---

# 3. Vertical Scaling

Suppose you have:

```text
RDS instance
4 vCPU
16 GB memory
```

and the workload needs more capacity.

You can move to a larger supported DB instance class:

```text
4 vCPU / 16 GB
       ↓
8 vCPU / 32 GB
       ↓
16 vCPU / 64 GB
```

This is **vertical scaling**.

---

# 4. When Should You Scale Up?

Scale up when the database is genuinely constrained by:

```text
CPU
Memory
Connection capacity
Database workload
Storage performance
```

Example:

```text
CPU = 95%
Memory pressure = high
Top SQL already optimized
Traffic continues growing
```

A larger instance may be appropriate.

---

# 5. Don't Scale Before Investigating

Bad approach:

```text
Database slow
     ↓
Double instance size
```

Better:

```text
Database slow
     ↓
Measure
     ↓
Find bottleneck
     ↓
Optimize
     ↓
Scale if necessary
```

Otherwise you may spend significantly more money while the actual problem remains.

---

# 6. Storage Scaling

Database growth can also require additional storage.

Example:

```text
500 GB
  ↓
800 GB
  ↓
1 TB
```

Depending on the RDS engine, storage configuration, and supported features, storage scaling can be handled differently.

Some configurations support **storage autoscaling**, which can automatically increase allocated storage when defined thresholds are reached.

---

# 7. Storage Autoscaling

Conceptually:

```text
Available Storage
      │
      ▼
Threshold reached
      │
      ▼
RDS increases storage
      │
      ▼
More capacity
```

This helps reduce the risk of unexpectedly running out of storage.

But:

> Storage autoscaling solves storage capacity. It does not automatically solve CPU, memory, poor queries, or connection bottlenecks.

---

# 8. Read Scaling

Now imagine:

```text
90% reads
10% writes
```

Scaling the primary may not be the best architectural answer.

Instead:

```text
                 Primary
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       Replica   Replica   Replica
          │         │         │
          ▼         ▼         ▼
        Reads     Reads     Reads
```

This is where **Read Replicas** become useful.

---

# 9. What Is a Read Replica?

A Read Replica is a separate DB instance that receives replicated data from a source database and can serve read traffic.

Conceptually:

```text
Primary
  │
  │ Replication
  ▼
Read Replica
  │
  ▼
Read queries
```

The application can direct appropriate read workloads to replicas.

---

# 10. Read Replica ≠ Multi-AZ Standby

This is one of the **most important RDS interview questions**.

### Read Replica

Purpose:

```text
Read scaling
```

### Multi-AZ standby

Purpose:

```text
High availability / failover
```

Don't confuse them.

---

# 11. Multi-AZ Architecture

Conceptually:

```text
                 RDS
                  │
        ┌─────────┴─────────┐
        │                   │
      AZ-A                 AZ-B
        │                   │
     Primary             Standby
        │                   │
        └──── HA mechanism ─┘
```

The standby exists primarily for availability/failover rather than application read scaling.

---

# 12. Read Replica Architecture

```text
                 Primary
                    │
               Replication
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       Replica   Replica   Replica
```

The replicas can be used to distribute eligible read traffic.

---

# 13. Read Replica — The Core Idea

Suppose:

```text
Application
   │
   ├── INSERT ────────→ Primary
   │
   ├── UPDATE ────────→ Primary
   │
   ├── SELECT ────────→ Replica
   │
   └── SELECT ────────→ Replica
```

This reduces read pressure on the primary.

---

# 14. Read Replica Is Not Automatically Strongly Consistent

This is critical.

Replication can involve **replication lag**.

Example:

```text
Primary
   │
   │ t = 10:00:00
   ▼
Write committed

Replica
   │
   │ t = 10:00:02
   ▼
Write arrives
```

For those two seconds, a replica may not reflect the latest primary state.

Therefore:

> **Don't send every read to a replica without understanding consistency requirements.**

---

# 15. Replication Lag

Suppose:

```text
Replication Lag
     ↑
  0 sec
  1 sec
  5 sec
 30 sec
```

The application may read stale data from a replica.

Possible causes:

```text
Heavy write workload
Replica resource constraints
Network conditions
Long-running operations
Replication workload
```

The exact mechanics depend on the RDS engine.

---

# 16. Read-After-Write Problem

Example:

```text
User places order
       │
       ▼
Write → Primary
       │
       ▼
Immediately request orders
       │
       ▼
Read → Replica
```

If replication hasn't caught up:

```text
Order appears missing
```

The order exists on the primary.

The replica is simply behind.

---

# 17. Solving Read-After-Write

Possible application strategies include:

```text
Recent writes
     ↓
Read from Primary

Older/general reads
     ↓
Read from Replica
```

Conceptually:

```text
             Application
                 │
        ┌────────┴────────┐
        │                 │
      Writes           Reads
        │                 │
        ▼           ┌─────┴─────┐
     Primary        │           │
                 Strong       Scalable
                 read          reads
                  │             │
                  ▼             ▼
               Primary       Replica
```

The correct strategy depends on business consistency requirements.

---

# 18. Read Replica Use Cases

Common use cases:

```text
Read-heavy applications
Reporting
Analytics workloads
Dashboards
Search-like workloads
Read scaling
Reducing primary read pressure
```

But make sure the workload is appropriate for a replica.

---

# 19. Reporting Workload Example

Suppose production application generates:

```text
90% transactional queries
10% reporting
```

A huge reporting query might compete with transactional traffic.

Instead:

```text
Production Primary
       │
       ├── Application writes
       └── Application reads

Read Replica
       │
       └── Reporting
```

This can isolate the reporting workload.

---

# 20. Read Replica Is Not a Backup

Another common interview mistake:

> "We have Read Replicas, so we're protected."

No.

A Read Replica is not a substitute for backups.

Why?

If an application executes:

```sql
DELETE FROM orders;
```

that logical change can propagate to the replica.

So:

```text
Primary → DELETE
Replica → DELETE
```

You still need backups/PITR.

---

# 21. Read Replica Is Not a Backup — Memorize

```text
Backup
   ↓
Historical recovery

Read Replica
   ↓
Read scaling / workload distribution

Multi-AZ
   ↓
High availability
```

Three different purposes.

---

# 22. Multi-AZ for High Availability

Suppose the primary DB instance has an infrastructure failure.

With Multi-AZ:

```text
Primary
   ❌
   │
   ▼
Standby
   │
   ▼
Failover
   │
   ▼
New Primary
```

The objective is to restore database availability with minimal manual intervention.

---

# 23. Multi-AZ Is Not Read Scaling

Suppose:

```text
Primary
   │
   └── Standby
```

You should not think:

```text
Application
   ├── Reads → Primary
   └── Reads → Standby
```

The Multi-AZ standby is not intended to be your normal read-scaling target.

For read scaling, use an appropriate Read Replica architecture.

---

# 24. High Availability vs Disaster Recovery

These are related but different.

### High Availability

Question:

> What happens if one component/Availability Zone fails?

Example:

```text
AZ-A failure
     ↓
Multi-AZ failover
```

### Disaster Recovery

Question:

> What happens if a much larger failure occurs?

Example:

```text
Region unavailable
     ↓
DR strategy
     ↓
Another Region
```

---

# 25. Regional Disaster Recovery

For major disaster scenarios, you may design:

```text
Region A
   │
   │ Replication / backup
   ▼
Region B
```

Region B can be prepared for recovery depending on your RTO/RPO requirements.

---

# 26. Cross-Region Read Replica

For supported RDS engines/configurations, Read Replicas can be created in another AWS Region.

Conceptually:

```text
Region A
   │
   ▼
Primary
   │
   │ Cross-Region replication
   ▼
Region B
   │
   ▼
Read Replica
```

This can support:

```text
Disaster recovery
Regional read access
Data locality requirements
```

---

# 27. Cross-Region DR Trade-Off

Cross-Region DR introduces:

```text
Replication lag
Network considerations
Cost
Operational complexity
Failover procedures
Application DNS/endpoint changes
Security/KMS considerations
```

Don't create cross-Region replicas just because they sound impressive.

Design them against actual business requirements.

---

# 28. RPO and DR

Suppose the business says:

```text
RPO = 5 minutes
```

Meaning:

> The business can tolerate losing at most about five minutes of data.

Your DR replication and recovery process must be capable of meeting that target.

---

# 29. RTO and DR

Suppose:

```text
RTO = 30 minutes
```

Meaning:

> The service should be restored within about 30 minutes.

It's not enough to say:

> "We have a replica."

You need to test the complete process:

```text
Failure
  ↓
Detection
  ↓
Promotion/recovery
  ↓
Application redirection
  ↓
Validation
  ↓
Service restored
```

---

# 30. DR Is a Process, Not a Checkbox

A DR architecture is incomplete if you don't know:

```text
Who declares disaster?
Who performs failover?
How is application traffic redirected?
How are secrets handled?
How is DNS updated?
How is the recovered DB validated?
How do you return to normal operations?
```

Document these steps.

---

# 31. RDS Scaling Decision Tree

Use this:

```text
                 Database Slow
                      │
                      ▼
                 Find bottleneck
                      │
        ┌─────────────┼──────────────┐
        │             │              │
       CPU          Reads          Storage
        │             │              │
        ▼             ▼              ▼
    Scale up      Read Replica    Add capacity
                      │
                      ▼
                 Read scaling
```

If the problem is:

```text
Bad SQL
```

then:

```text
Optimize SQL
```

not necessarily:

```text
Buy bigger hardware
```

---

# 32. Scaling Scenario #1 — CPU Saturation

Current:

```text
CPU = 95%
Read workload = moderate
Queries optimized
```

Potential solution:

```text
Vertical scaling
```

Move to a larger instance class if supported and justified.

---

# 33. Scaling Scenario #2 — Read Traffic Explosion

Current:

```text
CPU = 80%
Reads = 90%
Writes = 10%
```

Instead of only scaling the primary:

```text
Primary
   │
   ├── Replica 1
   ├── Replica 2
   └── Replica 3
```

Route eligible read traffic to replicas.

---

# 34. Scaling Scenario #3 — Storage Growth

Current:

```text
FreeStorageSpace ↓
```

Investigate:

```text
Data growth
Indexes
Logs
Unexpected workload
```

Then:

```text
Storage scaling/autoscaling
```

where appropriate.

---

# 35. Scaling Scenario #4 — Reporting Hurts Production

Problem:

```text
Production
   │
   ├── Transactions
   └── Huge reports
```

Solution:

```text
Production Primary
       │
       ▼
Read Replica
       │
       ▼
Reporting
```

This separates workloads.

---

# 36. Scaling Scenario #5 — Read Replica Is Behind

Problem:

```text
Replica lag = high
```

Investigate:

```text
Write rate
Replica instance capacity
Query workload
Network/replication conditions
Long-running operations
```

Don't simply add more replicas.

If the source is generating writes faster than a replica can process them, more replicas may not solve the underlying bottleneck.

---

# 37. Scaling Scenario #6 — Application Reads Stale Data

Problem:

```text
User updates profile
      ↓
Immediate read
      ↓
Old profile returned
```

Likely cause:

```text
Read routed to lagging replica
```

Possible solution:

```text
Recent write
   ↓
Read primary

Normal reads
   ↓
Read replicas
```

This is an application architecture decision.

---

# 38. HA Scenario #1 — Primary Instance Failure

Architecture:

```text
AZ-A
Primary
   ❌

AZ-B
Standby
   │
   ▼
Failover
```

Application should reconnect using the appropriate RDS endpoint rather than hardcoding a specific underlying host/IP.

---

# 39. HA Scenario #2 — Availability Zone Failure

If an entire Availability Zone becomes unavailable:

```text
AZ-A
 ❌
```

a correctly configured Multi-AZ deployment can fail over to another AZ.

This is why Multi-AZ architecture matters.

---

# 40. DR Scenario #1 — Region Failure

Suppose:

```text
us-east-1
     ❌
```

and the business requires regional recovery.

A possible architecture:

```text
Primary Region
      │
      │
      ▼
DR Region
      │
      ▼
Recovery database
```

The actual DR mechanism depends on the RDS engine and required RPO/RTO.

---

# 41. DR Scenario #2 — Accidental Data Deletion

This is not primarily a Multi-AZ problem.

Example:

```text
DELETE
  ↓
Primary
  ↓
Replication
  ↓
Standby/replica
```

The deletion can propagate.

Use:

```text
Backup
+
PITR
```

for historical recovery.

---

# 42. Four Concepts You Must Never Mix Up

Memorize this table:

| Requirement         | Primary RDS Tool         |
| ------------------- | ------------------------ |
| More CPU/memory     | Vertical scaling         |
| More read capacity  | Read Replicas            |
| Instance/AZ failure | Multi-AZ                 |
| Historical recovery | Backups/PITR             |
| Regional disaster   | Cross-Region DR strategy |

This is one of the highest-value RDS interview concepts.

---

# 43. Scaling + HA + DR Architecture

A mature architecture can look like:

```text
                         Application
                              │
                    ┌─────────┴─────────┐
                    │                   │
                  Writes              Reads
                    │                   │
                    ▼                   ▼
              ┌──────────┐       ┌───────────┐
              │ RDS      │       │ Replicas  │
              │ Primary  │──────▶│ 1 / 2 / 3 │
              └────┬─────┘       └───────────┘
                   │
              Multi-AZ
                   │
             ┌─────┴─────┐
             │           │
           AZ-A         AZ-B
          Primary      Standby
           
                   │
            DR strategy
                   │
                   ▼
              Region B
```

And surrounding it:

```text
Backups
PITR
Monitoring
Security
Secrets
```

---

# 44. Scaling Doesn't Fix Everything

Suppose:

```text
Query = terrible
```

and you upgrade:

```text
8 vCPU
   ↓
32 vCPU
```

The query might still be terrible.

Example:

```text
10-second query
     ↓
8 vCPU

9-second query
     ↓
32 vCPU
```

You spent more money but didn't solve the root cause.

Always investigate SQL first where appropriate.

---

# 45. Read Replicas Don't Fix Writes

Suppose:

```text
Write workload = 90%
Read workload = 10%
```

Adding:

```text
10 Read Replicas
```

doesn't magically distribute writes.

Writes still go to the primary in a typical read-replica architecture.

For write scaling, you may need fundamentally different database/application architecture.

---

# 46. Read Replicas and Failover

A Read Replica can potentially be promoted to become a standalone primary in supported scenarios.

This can be useful for:

```text
Disaster recovery
Migration
Planned architecture changes
```

But promotion is an operational action and should be tested.

---

# 47. Planned Failover vs Disaster

### Planned

```text
Maintenance
   ↓
Controlled failover
   ↓
Validate
```

### Unplanned

```text
Infrastructure failure
   ↓
Automatic/managed failover
   ↓
Recovery
```

The operational procedures differ.

---

# 48. Endpoint Considerations

Applications should generally connect through the appropriate RDS endpoint rather than relying on an underlying instance IP.

Conceptually:

```text
Application
    │
    ▼
RDS Endpoint
    │
    ▼
Current database target
```

This is especially important for failover scenarios.

---

# 49. Application Retry Logic

A highly available database isn't enough if the application cannot recover from transient connection failures.

Application should appropriately handle:

```text
Connection interruption
Transient failures
Failover
Retryable errors
Connection pool reset
```

But don't implement unlimited retries.

Otherwise you can create:

```text
Failure
  ↓
Retry storm
  ↓
More load
  ↓
Larger failure
```

---

# 50. The Retry Storm Problem

Suppose RDS temporarily fails over.

1000 application requests fail.

Each request retries 10 times immediately:

```text
1000 × 10
=
10,000 attempts
```

This can make recovery harder.

Use appropriate:

```text
Exponential backoff
Jitter
Retry limits
Circuit breakers where appropriate
```

---

# 51. DR Testing

A DR plan isn't real until tested.

Test:

```text id="t8jv3u"
Failover
Recovery
Application reconnection
DNS/endpoint handling
Secrets
Permissions
Data validation
RTO
RPO
```

Document the result.

---

# 52. Interview Question — Multi-AZ vs Read Replica?

Strong answer:

> Multi-AZ is primarily for high availability and failover, while Read Replicas are primarily for read scaling and workload distribution. Read Replicas can have replication lag and are not a replacement for backups.

---

# 53. Interview Question — How Do You Scale RDS?

Strong answer:

1. Identify the bottleneck first.
2. Scale the DB instance vertically when CPU/memory capacity is insufficient.
3. Increase storage when storage capacity/performance requires it.
4. Use Read Replicas for read-heavy workloads.
5. Optimize queries and indexes before blindly scaling infrastructure.
6. Use caching/application optimization where appropriate.
7. Monitor after the change.

---

# 54. Interview Question — How Do You Design RDS for HA?

Strong answer:

```text
Multi-AZ
   +
Private networking
   +
Automated backups
   +
PITR
   +
Monitoring
   +
Application retry/reconnection
```

The exact architecture depends on the database engine and business RTO/RPO.

---

# 55. Interview Question — How Do You Design RDS for DR?

Strong answer:

1. Define RPO and RTO.
2. Enable appropriate backups and PITR.
3. Evaluate cross-Region recovery requirements.
4. Use an appropriate cross-Region replication/backup strategy.
5. Document failover and recovery procedures.
6. Test the complete DR process regularly.
7. Validate application, DNS/endpoint, credentials, and data recovery.

---

# 56. Interview Question — Can Read Replicas Protect Against DELETE?

Answer:

> No. A logical change such as DELETE can be replicated to the replica. Read Replicas are primarily for read scaling and certain DR/migration scenarios; backups and PITR provide historical recovery.

---

# 57. Interview Question — Why Is Replica Lag Dangerous?

Because the application may read stale data.

Example:

```text
Primary:
Order = CREATED

Replica:
Order = NOT YET VISIBLE
```

Therefore applications need to understand consistency requirements when routing reads to replicas.

---

# 58. Complete RDS Architecture

Memorize:

```text
                       USERS
                         │
                         ▼
                    APPLICATION
                         │
              ┌──────────┴──────────┐
              │                     │
            WRITE                  READ
              │                     │
              ▼                     ▼
          RDS PRIMARY          READ REPLICA
              │                 /        \
              │                /          \
              ▼               ▼            ▼
           MULTI-AZ        Replica       Replica
              │
              ▼
           STANDBY

              │
              ▼
         AUTOMATED BACKUPS
              │
              ▼
             PITR

              │
              ▼
       CROSS-REGION DR
```

---

# 59. The Ultimate RDS Decision Matrix

| Problem             | Think First                             |
| ------------------- | --------------------------------------- |
| CPU saturation      | Vertical scaling / query optimization   |
| Memory pressure     | Larger instance / workload optimization |
| Storage nearly full | Storage scaling/autoscaling             |
| Too many reads      | Read Replicas                           |
| Primary/AZ failure  | Multi-AZ                                |
| Accidental DELETE   | PITR                                    |
| Long-term recovery  | Snapshots/backup strategy               |
| Region failure      | Cross-Region DR                         |
| Slow SQL            | Query plan/index/query optimization     |
| Replica stale       | Replication lag/consistency strategy    |

---

# 60. Production Checklist

```text
☐ Identify CPU/memory/storage bottlenecks
☐ Monitor database connections
☐ Use Read Replicas for appropriate read-heavy workloads
☐ Understand replica lag
☐ Use Multi-AZ for HA
☐ Maintain automated backups
☐ Maintain PITR capability
☐ Define RPO
☐ Define RTO
☐ Evaluate cross-Region DR
☐ Test failover
☐ Test application reconnection
☐ Test DR
☐ Monitor after scaling
☐ Don't use Read Replicas as backups
☐ Don't use Multi-AZ as read scaling
```

---

# 🧠 Lesson 9 — Must Remember

```text
1. Vertical scaling = bigger DB instance.
2. Read Replicas = read scaling.
3. Multi-AZ = high availability/failover.
4. Backups/PITR = historical recovery.
5. Read Replicas can have replication lag.
6. Read Replicas are not backups.
7. Multi-AZ standby isn't your normal read-scaling target.
8. Cross-Region architecture is for appropriate DR requirements.
9. RPO defines acceptable data loss.
10. RTO defines acceptable recovery time.
11. Application retry/reconnection matters during failover.
12. Always test HA and DR rather than assuming they work.
```

# 🔥 Golden Rule

> **Scale up for capacity, scale out reads with replicas, use Multi-AZ for availability, and use backups/PITR plus an appropriate cross-Region strategy for disaster recovery.**

---

## 🎯 Lesson 9 Complete

**Lesson 10 — RDS Production Architecture, Migration, Operations & Real-World Scenarios**

This final lesson will bring **Lessons 1–9 together** into a production-ready RDS architecture and cover **migration, maintenance, parameter/option groups, upgrades, patching, operational runbooks, cost optimization, troubleshooting scenarios, and final interview questions.**
