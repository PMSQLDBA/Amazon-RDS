# ☁️ Amazon RDS — Lesson 10/10

## Production Architecture, Migration, Operations & Real-World Scenarios

This is the **final RDS lesson**. It brings Lessons 1–9 together into a production-ready mental model.

The goal is not just to know RDS features, but to answer:

> **"How would you design, migrate, operate, secure, troubleshoot, scale, and recover an RDS database in a real production environment?"**

---

# 1. The Complete RDS Production Model

Think of production RDS across **10 areas**:

```text id="3j6l5p"
                 RDS PRODUCTION
                      │
 ┌────────┬───────────┼───────────┬──────────┐
 │        │           │           │          │
Network Security   HA/DR       Scaling   Performance
 │        │           │           │          │
VPC      IAM        Multi-AZ    Replicas   Monitoring
SG       TLS        Backups     Resize     SQL
 │        │           │           │          │
 └────────┴───────────┴───────────┴──────────┘
                      │
             Operations & Migration
```

---

# 2. Production RDS Reference Architecture

A typical architecture:

```text id="k9w0ru"
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │ Load Balancer │
                    └───────┬───────┘
                            │
                            ▼
                    Application Tier
                       Private Subnets
                            │
                         App-SG
                            │
                       TCP 5432/3306
                            │
                         RDS-SG
                            │
                            ▼
                 ┌─────────────────────┐
                 │     RDS Primary     │
                 │    Private Subnet   │
                 └──────────┬──────────┘
                            │
                       Multi-AZ
                            │
                            ▼
                 ┌─────────────────────┐
                 │     Standby         │
                 │    Another AZ       │
                 └─────────────────────┘
```

And separately:

```text id="3y8qf7"
RDS
 │
 ├── Automated Backups
 ├── PITR
 ├── Read Replicas
 ├── CloudWatch
 ├── Performance Insights
 ├── Secrets Manager
 └── KMS
```

---

# 3. Production Architecture Principle

A strong production RDS design normally considers:

```text id="w9v6bg"
Private networking
+
Restricted Security Groups
+
Encryption
+
Secrets management
+
Multi-AZ
+
Backups/PITR
+
Monitoring
+
Least privilege
+
Performance strategy
+
DR strategy
```

No single feature makes a database production-ready.

---

# 4. RDS Networking Design

Start with:

```text id="1y2w5f"
VPC
 │
 ├── Public Subnets
 │
 └── Private Subnets
       │
       ├── Application
       │
       └── Database
```

The database should generally reside in private networking.

The application tier can access it internally.

---

# 5. DB Subnet Group

For RDS, configure an appropriate DB subnet group using suitable subnets across Availability Zones.

Conceptually:

```text id="2a7l3x"
VPC
 │
 ├── Private Subnet AZ-A
 │
 └── Private Subnet AZ-B
          │
          ▼
     DB Subnet Group
          │
          ▼
          RDS
```

This supports the desired Multi-AZ architecture.

---

# 6. Security Group Architecture

Use separate Security Groups.

Example:

```text id="jv0qwx"
Application-SG
       │
       │ TCP 5432
       ▼
     RDS-SG
```

RDS-SG:

```text id="7ml7v8"
Inbound:
TCP 5432
Source = Application-SG
```

Avoid unnecessarily broad rules such as:

```text id="gq3j7p"
0.0.0.0/0 → TCP 5432
```

---

# 7. Public Accessibility

For a typical production application:

```text id="z4a5f1"
Internet
   X
   │
   ▼
Private RDS
   ▲
   │
Application
```

Avoid directly exposing your database to the internet unless there is a well-understood requirement and compensating controls.

---

# 8. Authentication Architecture

A production application should not use a shared administrator account.

Prefer:

```text id="1x1u6q"
Application
    │
    ▼
Application-specific DB role
```

For example:

```text id="j4c8sv"
app_user
   │
   ├── SELECT
   ├── INSERT
   ├── UPDATE
   └── required procedures
```

rather than:

```text id="kq8e4w"
app_user
   ↓
SUPERUSER
```

---

# 9. Secrets Architecture

A common production pattern:

```text id="7h7v0j"
Application
     │
     ▼
Secrets Manager
     │
     ▼
Database credentials
     │
     ▼
RDS
```

Avoid embedding production credentials in:

```text id="kwl0q7"
Source code
Git
Container images
Documentation
Chat messages
```

---

# 10. Encryption Architecture

Use two separate concepts:

```text id="0j3k0d"
At Rest
   ↓
RDS encryption / KMS

In Transit
   ↓
TLS
```

Think:

```text id="r0j5n4"
Stored data
   → encrypted

Network traffic
   → encrypted
```

Both protect different parts of the system.

---

# 11. High Availability Architecture

For production workloads requiring high availability:

```text id="0t0p9c"
                  RDS
                   │
          ┌────────┴────────┐
          │                 │
        AZ-A              AZ-B
          │                 │
       Primary            Standby
```

The objective is to survive an Availability Zone or instance-level failure with managed failover.

---

# 12. Read Scaling Architecture

For read-heavy workloads:

```text id="8ehf5e"
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

But always consider replication lag and consistency requirements.

---

# 13. Backup Architecture

Production RDS should have an explicit recovery strategy.

Conceptually:

```text id="7z2o8k"
RDS
 │
 ├── Automated Backups
 │       │
 │       └── Point-in-Time Recovery
 │
 └── Manual Snapshots
```

Backups protect against scenarios such as:

```text id="v1j9m6"
Accidental deletion
Bad deployment
Corruption
Logical errors
Operational mistakes
```

---

# 14. PITR

Point-in-time recovery allows you to recover a database to an appropriate point within the available backup/recovery window, subject to the engine and RDS configuration.

Example:

```text id="f5l7v1"
10:00 ───── 11:00 ───── 12:00 ───── 13:00
                    │
                 Bad DELETE
                    │
                    ▼
              Recover to
              11:59
```

This is fundamentally different from simply restoring the latest snapshot.

---

# 15. Snapshot vs PITR

### Snapshot

```text id="f6u4dj"
Specific point
    ↓
Restore
```

### PITR

```text id="t7q3xg"
Backup history
    ↓
Choose recovery point
    ↓
Restore
```

Use the appropriate mechanism for the incident.

---

# 16. RDS Migration

Now suppose an organization has:

```text id="q3x8cd"
On-Premises PostgreSQL
```

and wants:

```text id="z9j6m5"
AWS RDS PostgreSQL
```

A migration isn't just:

```text id="n6k8sm"
Copy database
```

You must consider:

```text id="g0t4z9"
Schema
Data
Users
Permissions
Extensions
Connectivity
Downtime
Application changes
Validation
Rollback
```

---

# 17. Migration Strategies

Broadly, migrations can be:

```text id="0v9u7s"
Offline / dump-restore
        OR
Online / minimal downtime
```

The correct approach depends on:

```text id="b3w1c8"
Database size
Downtime tolerance
Change rate
Network connectivity
Application architecture
RPO/RTO
```

---

# 18. Dump and Restore

For a smaller database, a common approach is:

```text id="l5m4m8"
Source DB
   │
   ▼
Backup / Dump
   │
   ▼
Transfer
   │
   ▼
RDS
   │
   ▼
Restore
```

Then:

```text id="0e7r8q"
Validate
   ↓
Switch application
```

This can be straightforward but may require downtime.

---

# 19. Large Database Migration

For a large production database:

```text id="r0d3zw"
Source
  │
  │ Initial load
  ▼
RDS
  │
  │ Continuous changes
  ▼
Replication
```

Then during the cutover:

```text id="d1y8ra"
Stop writes
   ↓
Synchronize
   ↓
Validate
   ↓
Switch application
   ↓
Monitor
```

The goal is to minimize downtime.

---

# 20. AWS Database Migration Service

**AWS Database Migration Service (AWS DMS)** is commonly used for database migrations and ongoing replication scenarios.

Conceptually:

```text id="9g2g4h"
Source Database
       │
       ▼
     AWS DMS
       │
       ▼
  Target RDS
```

It can support migration patterns where continuous replication helps reduce downtime.

---

# 21. DMS Migration Model

A common logical sequence:

```text id="v3u6sf"
1. Prepare source
        ↓
2. Prepare target RDS
        ↓
3. Initial full load
        ↓
4. Continuous change replication
        ↓
5. Validate
        ↓
6. Cutover
        ↓
7. Monitor
```

The exact implementation depends on source and target engines.

---

# 22. Migration Pre-Checks

Before migration, verify:

```text id="6l8z3h"
☐ Database engine/version
☐ Extensions
☐ Schema compatibility
☐ Users/roles
☐ Network connectivity
☐ Security Groups
☐ DNS/application configuration
☐ Storage requirement
☐ Performance requirements
☐ Backup strategy
☐ RPO/RTO
☐ Downtime window
☐ Rollback strategy
```

---

# 23. Migration Validation

Never assume:

> "The migration succeeded because the tool says complete."

Validate:

```text id="u9c2bx"
Table counts
Data integrity
Indexes
Constraints
Sequences
Users
Permissions
Application functionality
Query performance
```

For critical systems, perform application-level validation too.

---

# 24. Migration Cutover

A controlled cutover:

```text id="7o5x0j"
Old DB
   │
   ▼
Final synchronization
   │
   ▼
Freeze writes
   │
   ▼
Validate target
   │
   ▼
Switch application
   │
   ▼
Monitor
```

Don't switch traffic before verifying the target.

---

# 25. Rollback Planning

Always ask:

> "What happens if the new RDS database fails after cutover?"

Possible rollback process:

```text id="b5d8x4"
Detect issue
    ↓
Stop application writes
    ↓
Assess data divergence
    ↓
Redirect to source if safe
    ↓
Investigate
    ↓
Retry migration
```

Rollback becomes much harder after new writes occur on the target.

Therefore:

> **Define rollback before the migration starts.**

---

# 26. RDS Maintenance

RDS is managed, but it is not maintenance-free.

You still need to manage:

```text id="q7g7qm"
Engine versions
Patching
Parameter changes
Storage
Performance
Backups
Security
Application compatibility
```

AWS manages underlying infrastructure tasks, but you remain responsible for your database workload and configuration.

---

# 27. Maintenance Windows

RDS provides maintenance scheduling mechanisms for changes that require maintenance.

You should:

```text id="v1n4m9"
Choose an appropriate maintenance window
Monitor pending maintenance
Test upgrades
Communicate expected impact
Validate after maintenance
```

Don't treat maintenance as something that can be ignored.

---

# 28. Engine Upgrades

Example:

```text id="x8s2pp"
PostgreSQL
15
 ↓
16
```

Before upgrading:

```text id="n9r7q1"
Test application
Test extensions
Test SQL compatibility
Review release notes
Test performance
Verify backups
Prepare rollback/recovery plan
```

---

# 29. Major vs Minor Upgrades

Conceptually:

### Minor upgrade

Usually:

```text id="8yd7kr"
Bug fixes
Security fixes
Smaller compatibility impact
```

### Major upgrade

Can involve:

```text id="d3u5x8"
Feature changes
Behavior changes
Compatibility changes
Application impact
```

Always follow the specific engine/version upgrade documentation.

---

# 30. Parameter Groups

Parameter groups allow you to manage database engine parameters.

Conceptually:

```text id="6n8r9d"
RDS
 │
 ▼
Parameter Group
 │
 ▼
Database configuration
```

Examples can include settings affecting:

```text id="2q8w7n"
Memory-related behavior
Logging
Connections
Engine behavior
```

Never copy random parameter values from the internet into production.

Understand the workload and the engine.

---

# 31. Parameter Change Process

A safer process:

```text id="j5j4i0"
Understand parameter
       ↓
Check current value
       ↓
Research impact
       ↓
Test in non-production
       ↓
Measure
       ↓
Schedule production change
       ↓
Monitor
```

---

# 32. Option Groups

Some RDS engines/configurations use option groups to enable additional database functionality.

Before enabling an option:

```text id="7b0q4k"
What does it do?
Does it require restart?
Does it change security?
Does it change network behavior?
Does it add operational complexity?
```

Understand the impact first.

---

# 33. RDS Monitoring Strategy

A production monitoring stack might include:

```text id="v6d3g9"
CloudWatch
    │
    ├── CPU
    ├── Connections
    ├── Storage
    ├── I/O
    └── Alarms
         │
         ▼
Performance Insights
         │
         ├── SQL
         └── Waits
```

And:

```text id="3z7d2c"
CloudTrail
Database logs
Application logs
```

for auditing and troubleshooting.

---

# 34. Operational Runbook

A good RDS team should have runbooks for:

```text id="0w4k5v"
High CPU
Low storage
Connection exhaustion
Slow queries
Lock contention
Failover
Replica lag
Backup restore
Migration
Engine upgrade
Security incident
DR activation
```

This turns tribal knowledge into repeatable operations.

---

# 35. High CPU Runbook

```text id="2w6s8x"
CPU alert
   ↓
Check duration
   ↓
Check Performance Insights
   ↓
Identify top SQL
   ↓
Check waits
   ↓
Check recent deployments
   ↓
Optimize or scale
   ↓
Validate
```

---

# 36. Low Storage Runbook

```text id="g1v7t2"
Storage alert
   ↓
Check growth trend
   ↓
Identify consuming data
   ↓
Check storage autoscaling/configuration
   ↓
Increase capacity if needed
   ↓
Investigate abnormal growth
   ↓
Monitor
```

---

# 37. Connection Exhaustion Runbook

```text id="7p9z3m"
Connections high
      ↓
Check application instances
      ↓
Check pool size
      ↓
Check idle sessions
      ↓
Check connection leaks
      ↓
Check long transactions
      ↓
Reduce/repair connection pressure
```

---

# 38. Replica Lag Runbook

```text id="2d8v4j"
Replica lag
    ↓
Check write rate
    ↓
Check replica resources
    ↓
Check workload
    ↓
Check long-running operations
    ↓
Determine whether replica is undersized
    ↓
Fix / scale / redesign read routing
```

---

# 39. Database Failure Runbook

If the primary becomes unavailable:

```text id="z6v0z2"
Detect
  ↓
Confirm RDS status
  ↓
Check failover behavior
  ↓
Monitor new primary
  ↓
Verify application reconnection
  ↓
Validate transactions
  ↓
Monitor performance
```

Don't immediately start changing infrastructure while the managed failover process is occurring.

---

# 40. Security Incident Runbook

Suppose a production DB credential is exposed.

Immediate priorities:

```text id="v3j1k4"
1. Assess exposure
2. Rotate credential
3. Revoke unnecessary access
4. Check audit logs
5. Investigate access
6. Validate application
7. Prevent recurrence
```

Never simply delete the exposed credential from source code and assume the problem is solved.

---

# 41. Real-Time Scenario #1 — RDS Is Suddenly Slow

Application says:

> "Everything became slow."

Your investigation:

```text id="h5j8a1"
CloudWatch
   ↓
CPU?
Memory?
Connections?
Storage?
I/O?
   ↓
Performance Insights
   ↓
Top SQL
   ↓
Waits
   ↓
Query plan
```

Then identify the root cause.

---

# 42. Real-Time Scenario #2 — CPU 98%

You discover:

```text id="w8v6e3"
Query A
   ↓
Full table scan
   ↓
Huge table
```

Don't immediately scale.

First investigate:

```text id="4q9v0k"
Index
Query design
Statistics
Execution plan
```

Then scale if the remaining workload genuinely requires more capacity.

---

# 43. Real-Time Scenario #3 — Production Database Is Full

```text id="g0y8w3"
FreeStorageSpace ≈ 0
```

Immediate concern:

```text id="m8h4b6"
Database availability/performance risk
```

Actions:

```text id="n7y5q2"
Increase storage where appropriate
+
Determine why storage grew
+
Monitor recovery
```

Then investigate the underlying growth.

---

# 44. Real-Time Scenario #4 — Application Can't Connect After Failover

Possible causes:

```text id="w8q3s5"
Connection pool has stale connections
DNS/endpoint handling issue
Security Group issue
TLS issue
Application retry problem
Credentials issue
```

The database may have successfully failed over.

The application still needs to reconnect correctly.

---

# 45. Real-Time Scenario #5 — Read Replica Is Returning Old Data

Likely explanation:

```text id="q6x4p8"
Replication lag
```

Determine:

```text id="9j1y2x"
Is stale data acceptable?
```

If not:

```text id="m7g8k2"
Route consistency-sensitive reads to primary
```

Use replicas for workloads that can tolerate their consistency characteristics.

---

# 46. Real-Time Scenario #6 — Migration Finished but Application Fails

Don't immediately rollback.

Check:

```text id="x9v0l2"
DNS
Credentials
Security Groups
Database roles
Schema
Extensions
Connection strings
TLS
Parameter differences
Application compatibility
```

Migration success and application success are different validation stages.

---

# 47. Real-Time Scenario #7 — Database Upgrade Causes Errors

Application starts failing after an engine upgrade.

Investigate:

```text id="r3j6z7"
Version compatibility
Deprecated behavior
Extensions
SQL changes
Parameter changes
Driver compatibility
Execution plans
```

This is why upgrade testing is critical.

---

# 48. Real-Time Scenario #8 — RDS Is Healthy but API Is Slow

Metrics:

```text id="z4x2h8"
RDS CPU = 30%
RDS memory = normal
RDS I/O = normal
```

Don't conclude:

> "RDS is slow."

Check:

```text id="1c8z3j"
Application
Network
Connection pool
External APIs
Application code
Serialization
Caching
```

RDS is only one component of the request path.

---

# 49. Real-Time Scenario #9 — Database Cost Suddenly Increased

Investigate:

```text id="g4c7v2"
Instance size
Running instances
Read Replicas
Storage
IOPS/storage configuration
Backup retention
Snapshots
Cross-Region resources
Data transfer
```

Then determine what changed.

Cost optimization should never compromise required availability or recovery objectives.

---

# 50. Real-Time Scenario #10 — Need 24x7 Production Availability

A reasonable starting architecture:

```text id="4j7z0h"
Private RDS
    +
Multi-AZ
    +
Automated backups
    +
PITR
    +
Monitoring
    +
Alerting
    +
Application retry
    +
DR strategy
```

Then validate the actual RTO/RPO requirements.

---

# 51. Cost Optimization

RDS cost isn't only:

```text id="9f3n2c"
Instance price
```

Consider:

```text id="4p0q7s"
DB instance
Storage
Provisioned IOPS where applicable
Backup storage
Read Replicas
Multi-AZ architecture
Data transfer
Snapshots
Cross-Region resources
```

---

# 52. Cost Optimization Strategy

Use:

```text id="x1b7d9"
Right-size
   ↓
Remove unused resources
   ↓
Optimize queries
   ↓
Use appropriate storage
   ↓
Review replicas
   ↓
Review backup retention
   ↓
Monitor continuously
```

Don't reduce cost by removing critical HA/backup controls without checking business requirements.

---

# 53. RDS Production Change Process

Before changing production:

```text id="j3s9q6"
Change request
     ↓
Impact analysis
     ↓
Backup/recovery verification
     ↓
Test in non-production
     ↓
Maintenance window
     ↓
Change
     ↓
Validation
     ↓
Monitoring
     ↓
Rollback if required
```

This is a mature operational approach.

---

# 54. Blue/Green Thinking

For risky database changes, you may want a parallel environment where you can validate changes before switching production traffic, depending on the RDS engine/features and migration strategy.

Conceptually:

```text id="9p4v6y"
Production
   │
   └── Current DB

Parallel Environment
   │
   └── New DB
        │
        ▼
     Validate
        │
        ▼
      Cutover
```

The exact implementation depends on your RDS engine and AWS capabilities.

---

# 55. Production Documentation

Document:

```text id="5z7m0k"
DB endpoint
Engine/version
Instance class
Subnet group
Security Groups
Parameter group
Backup retention
Maintenance window
HA configuration
Read Replicas
Monitoring
RPO
RTO
DR procedure
Owner/team
Escalation process
```

Don't rely on someone's memory.

---

# 56. RDS Production Checklist

### Architecture

```text id="r8s5v3"
☐ Private networking
☐ Appropriate subnet group
☐ Multi-AZ where required
☐ Read Replicas where justified
```

### Security

```text id="n1q6j8"
☐ Security Groups restricted
☐ Encryption at rest
☐ TLS
☐ Secrets management
☐ IAM least privilege
☐ Database least privilege
```

### Reliability

```text id="c8z0r4"
☐ Automated backups
☐ PITR
☐ Snapshots
☐ DR strategy
☐ RPO defined
☐ RTO defined
```

### Operations

```text id="p5m2x9"
☐ CloudWatch
☐ Performance Insights
☐ Alarms
☐ Logs
☐ Runbooks
☐ Maintenance process
```

### Performance

```text id="w4j7h2"
☐ CPU monitored
☐ Memory monitored
☐ Connections monitored
☐ Storage monitored
☐ I/O monitored
☐ Query performance reviewed
☐ Indexes reviewed
```

---

# 57. The RDS Production Golden Architecture

Memorize this:

```text id="q3z7c1"
                         USERS
                           │
                           ▼
                    ┌──────────────┐
                    │ Load Balancer│
                    └──────┬───────┘
                           │
                           ▼
                    APPLICATION
                     PRIVATE SUBNET
                           │
                           │
                      App-SG
                           │
                           │ DB PORT
                           ▼
                      RDS-SG
                           │
                           ▼
                  ┌─────────────────┐
                  │   RDS PRIMARY   │
                  │  PRIVATE SUBNET │
                  └────────┬────────┘
                           │
                       Multi-AZ
                           │
                           ▼
                  ┌─────────────────┐
                  │    STANDBY      │
                  │   OTHER AZ      │
                  └─────────────────┘

                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
          Read Replica  Backups      Monitoring
                           │
                           ▼
                          PITR

              ┌────────────────────────────┐
              │ KMS + Secrets Manager      │
              │ CloudWatch + PI            │
              └────────────────────────────┘
```

---

# 58. The Ultimate RDS Mental Model

When you receive any RDS production problem, ask these questions **in order**:

```text id="k2r9z0"
1. Is the database reachable?
             ↓
2. Is authentication working?
             ↓
3. Is authorization correct?
             ↓
4. Is the database healthy?
             ↓
5. What resource is constrained?
             ↓
6. What SQL/workload causes it?
             ↓
7. Is this a scaling problem?
             ↓
8. Is this an HA problem?
             ↓
9. Is this a DR/recovery problem?
             ↓
10. How do we prevent recurrence?
```

This framework is extremely useful in real production incidents.

---

# 59. Top RDS Interview Questions — Final Set

### Q1. Multi-AZ vs Read Replica?

**Answer:**

```text id="4n2c7v"
Multi-AZ
→ High availability/failover

Read Replica
→ Read scaling/workload distribution
```

---

### Q2. Is Read Replica a backup?

**Answer:**

No.

```text id="3m7q1a"
Read Replica → read scaling
Backup/PITR → recovery
```

---

### Q3. How do you scale RDS?

**Answer:**

```text id="p4v9n1"
Vertical scaling
Read Replicas
Storage scaling
Query optimization
Connection optimization
Caching/application optimization
```

Choose based on the bottleneck.

---

### Q4. How do you secure RDS?

**Answer:**

```text id="d8j3x0"
Private networking
Security Groups
TLS
Encryption/KMS
Secrets Manager
IAM
Database roles
Least privilege
Auditing
```

---

### Q5. How do you troubleshoot slow RDS?

**Answer:**

```text id="q9w5e3"
CloudWatch
    ↓
Performance Insights
    ↓
Top SQL
    ↓
Waits
    ↓
EXPLAIN
    ↓
Root cause
    ↓
Fix
    ↓
Validate
```

---

### Q6. What is RPO?

> Maximum acceptable amount of data loss measured in time.

Example:

```text id="8n6v2m"
RPO = 5 minutes
```

The business can tolerate approximately five minutes of data loss.

---

### Q7. What is RTO?

> Maximum acceptable time to restore the service.

Example:

```text id="w3r7k2"
RTO = 30 minutes
```

The service should be restored within about 30 minutes.

---

### Q8. How would you migrate a large production database?

```text id="9q5h1x"
Assess
 ↓
Prepare target
 ↓
Initial load
 ↓
Continuous replication
 ↓
Validate
 ↓
Freeze/cutover
 ↓
Switch application
 ↓
Monitor
```

AWS DMS can be considered for suitable migration scenarios.

---

### Q9. What happens if a Read Replica is behind?

Check:

```text id="f0k4q9"
Replication lag
Write rate
Replica resources
Workload
Long-running operations
```

Then determine whether to:

```text id="u7x2p8"
Scale replica
Optimize workload
Change read routing
```

---

### Q10. What happens if the RDS primary fails?

With appropriate Multi-AZ architecture:

```text id="n6c8y3"
Primary failure
      ↓
Failover
      ↓
Standby becomes primary
      ↓
Application reconnects
      ↓
Validate
```

---

# 60. Final RDS Master Cheat Sheet

```text id="m8p4v2"
                 AMAZON RDS
                     │
 ┌───────────────────┼────────────────────┐
 │                   │                    │
NETWORK           SECURITY             HA/DR
 │                   │                    │
VPC               IAM                  Multi-AZ
Subnets           TLS                  Backups
SG                KMS                  PITR
Endpoint          Secrets              DR
 │                   │                    │
 └───────────────────┼────────────────────┘
                     │
                PERFORMANCE
                     │
          ┌──────────┼──────────┐
          │          │          │
        CPU        Memory       I/O
          │          │          │
          └──────────┼──────────┘
                     │
               SQL / Queries
                     │
              Performance Insights
                     │
                 Optimization
                     │
                  SCALING
                     │
          ┌──────────┴──────────┐
          │                     │
      Vertical              Read Replicas
          │                     │
       Bigger DB              More reads
```

---

# 🧠 Lesson 10 — Must Remember

```text id="z8x4p1"
1. Production RDS is a combination of networking, security, HA, backup, monitoring, and operations.
2. Keep production databases private where appropriate.
3. Restrict Security Groups to required application sources.
4. Use encryption at rest and TLS in transit.
5. Use Secrets Manager or another appropriate secure credential mechanism.
6. Use Multi-AZ for high availability.
7. Use Read Replicas for appropriate read scaling.
8. Use backups and PITR for recovery.
9. Define and test RPO/RTO.
10. Use DMS or another suitable strategy for large/minimal-downtime migrations.
11. Test upgrades before production.
12. Monitor with CloudWatch and Performance Insights.
13. Troubleshoot using evidence rather than guessing.
14. Document operational runbooks.
15. Test your DR plan.
```

# 🔥 Final Golden Rule

> **A production RDS solution is not just a database instance. It is an architecture covering connectivity, security, availability, scaling, backup, recovery, monitoring, migration, and operational discipline.**

---

# 🎓 Amazon RDS — 10-Lesson Learning COMPLETED

| Lesson | Topic                                                                     |
| -----: | ------------------------------------------------------------------------- |
|  **1** | RDS Fundamentals & Architecture                                           |
|  **2** | RDS Instances, Storage & Database Configuration                           |
|  **3** | RDS Networking & Connectivity                                             |
|  **4** | Backups, Snapshots & Point-in-Time Recovery                               |
|  **5** | Multi-AZ, High Availability & Failover                                    |
|  **6** | Read Replicas & Read Scaling                                              |
|  **7** | Security: VPC, IAM, Encryption & Secrets                                  |
|  **8** | Monitoring, Performance & Troubleshooting                                 |
|  **9** | Scaling, Read Replicas, HA & Disaster Recovery                            |
| **10** | **Production Architecture, Migration, Operations & Real-World Scenarios** |

**Amazon RDS Learning: 10/10 ✅ COMPLETED**
