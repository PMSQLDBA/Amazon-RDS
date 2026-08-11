# ☁️ Amazon RDS — Lesson 1/10

## RDS Fundamentals & Architecture

Welcome to the **Amazon RDS 10-Lesson course**.

This course will stay **strictly focused on Amazon RDS** and will build from fundamentals → architecture → HA → scaling → backup/DR → security → monitoring → production operations.

---

# 1. What is Amazon RDS?

**Amazon Relational Database Service (RDS)** is an AWS managed service for running relational database engines.

Instead of managing the database server yourself:

```text
Traditional Self-Managed DB
───────────────────────────

You
 │
 ├── Server
 ├── OS
 ├── Storage
 ├── Database software
 ├── Patching
 ├── Backups
 ├── Monitoring
 └── Recovery
```

With RDS:

```text
Amazon RDS
───────────────────────────

AWS
 │
 ├── Infrastructure
 ├── OS management
 ├── Database service integration
 ├── Backups
 ├── Monitoring integration
 └── HA capabilities
        │
        ▼
      You
        │
        ├── Database
        ├── Schema
        ├── Users
        ├── Queries
        ├── Indexes
        └── Application design
```

The key idea:

> **RDS removes much of the infrastructure administration required to run a relational database.**

---

# 2. What does "managed database" mean?

Managed doesn't mean AWS manages everything.

Think about responsibilities:

```text
AWS manages
     │
     ├── Underlying infrastructure
     ├── Host hardware
     ├── Many maintenance tasks
     ├── Managed backups
     ├── Monitoring integration
     └── Availability capabilities

You manage
     │
     ├── Schema
     ├── SQL
     ├── Users/roles
     ├── Indexes
     ├── Application connections
     ├── Query performance
     └── Database configuration
```

This distinction is extremely important in interviews.

---

# 3. Which database engines does Amazon RDS support?

Amazon RDS supports several relational database engines, including:

```text
PostgreSQL
MySQL
MariaDB
Oracle
Microsoft SQL Server
Db2
```

Each engine has its own:

* SQL behavior
* Configuration
* Features
* Version lifecycle
* Licensing considerations
* Monitoring characteristics

Therefore:

> **RDS is the managed service; PostgreSQL/MySQL/etc. is the database engine running within that service.**

---

# 4. RDS vs database engine

This distinction causes confusion for beginners.

Suppose you create:

```text
Amazon RDS
    │
    └── PostgreSQL
```

Then:

**RDS** provides the managed AWS environment.

**PostgreSQL** is the actual relational database engine.

Similarly:

```text
Amazon RDS
    │
    ├── MySQL
    ├── PostgreSQL
    ├── Oracle
    └── SQL Server
```

The RDS management model is common, but engine-specific behavior differs.

---

# 5. What is an RDS DB instance?

A **DB instance** is the compute environment where your RDS database engine runs.

Conceptually:

```text
              RDS DB Instance
             ┌───────────────┐
             │ Database      │
             │ Engine        │
             │               │
             │ PostgreSQL    │
             │ / MySQL       │
             │ / etc.        │
             └───────┬───────┘
                     │
              ┌──────▼──────┐
              │   Storage   │
              └─────────────┘
```

The instance provides resources such as:

```text
CPU
Memory
Network
Storage connectivity
```

---

# 6. DB instance class

When creating an RDS instance, you select an instance class.

Think:

```text
Instance Class
      │
      ├── CPU
      ├── Memory
      ├── Network capability
      └── Other hardware characteristics
```

For example:

```text
Small workload
      ↓
Smaller instance

Large workload
      ↓
Larger instance
```

Choosing the right instance class is part of capacity planning.

---

# 7. Why does memory matter?

Database workloads often depend heavily on memory.

Memory can be used for:

```text
Buffer/cache
Connections
Sort operations
Query execution
Database processes
Operating-system-related activity
```

If the workload needs more memory than the instance can comfortably provide, performance can suffer.

Therefore:

> **Don't size an RDS instance based only on CPU.**

---

# 8. What is RDS storage?

RDS separates database compute from its configured storage volume.

Conceptually:

```text
              RDS
               │
        ┌──────▼──────┐
        │ DB Instance │
        └──────┬──────┘
               │
        ┌──────▼──────┐
        │   Storage   │
        └─────────────┘
```

Storage capacity and performance are important independently of instance CPU/memory.

---

# 9. Storage capacity vs storage performance

These are different concepts.

### Capacity

How much data can be stored:

```text
100 GB
500 GB
1 TB
...
```

### Performance

How quickly storage operations can be handled:

```text
IOPS
Throughput
Latency
```

You can therefore have:

```text
Enough storage capacity
+
Insufficient storage performance
```

or:

```text
Excellent storage performance
+
Insufficient storage capacity
```

Both can cause production problems.

---

# 10. What is a Region?

An AWS Region is a geographic area containing multiple Availability Zones.

Conceptually:

```text
AWS Region
│
├── Availability Zone A
├── Availability Zone B
└── Availability Zone C
```

Your RDS database is created in a specific AWS Region.

For example:

```text
US Region
     ↓
RDS
     ↓
DB Instance
```

The Region is important for:

* Latency
* Compliance
* Availability
* Disaster recovery
* Data residency
* Cost

---

# 11. What is an Availability Zone?

An Availability Zone is an isolated location within an AWS Region.

For example:

```text
Region
│
├── AZ-1
├── AZ-2
└── AZ-3
```

Multiple AZs provide infrastructure isolation.

This becomes especially important when we discuss **RDS Multi-AZ** in Lesson 4.

---

# 12. What is an RDS DB subnet group?

An RDS DB subnet group tells RDS which subnets it can use for database deployment.

Conceptually:

```text
VPC
 │
 ├── Private Subnet AZ-A
 │
 ├── Private Subnet AZ-B
 │
 └── Private Subnet AZ-C
          │
          ▼
     DB Subnet Group
          │
          ▼
          RDS
```

For production systems, database subnets are commonly private.

---

# 13. RDS and VPC

RDS databases are deployed into an AWS VPC.

A simplified architecture:

```text
                    VPC
                     │
       ┌─────────────┴─────────────┐
       │                           │
 Application                    Database
       │                           │
       ▼                           ▼
 App Subnet                   RDS Subnet
       │                           │
       └──────────────┬────────────┘
                      │
                  Network Path
```

This allows you to control:

* Network routing
* Security groups
* Subnet placement
* Connectivity

---

# 14. What is an RDS endpoint?

Applications don't normally connect to an RDS database by discovering its underlying server hardware.

They connect using an endpoint such as:

```text
mydb.xxxxx.region.rds.amazonaws.com
```

Conceptually:

```text
Application
     │
     ▼
RDS Endpoint
     │
     ▼
Database
```

The endpoint provides the hostname needed by the client.

---

# 15. What does the application need to connect?

At a high level:

```text
Application
     │
     ├── Endpoint
     ├── Port
     ├── Database name
     ├── Username
     └── Authentication
          │
          ▼
        RDS
```

But network connectivity must also work.

For example:

```text
Application
     │
     │ TCP
     ▼
Security Group
     │
     ▼
RDS
```

---

# 16. What is a security group?

A security group controls network access to the RDS instance.

For example:

```text
Application SG
      │
      │ TCP 5432
      ▼
RDS Security Group
      │
      ▼
PostgreSQL
```

For MySQL:

```text
TCP 3306
```

For PostgreSQL:

```text
TCP 5432
```

The exact port depends on the database engine/configuration.

---

# 17. Should RDS be publicly accessible?

For most production application architectures:

```text
Internet
   │
   X
   │
   ▼
RDS
```

is undesirable.

A more typical design is:

```text
Internet
    │
    ▼
Application
    │
    ▼
Private Network
    │
    ▼
RDS
```

This reduces the database's direct exposure to the internet.

---

# 18. What is an RDS database name?

A database engine can contain databases depending on the engine.

For example, in PostgreSQL:

```text
RDS DB Instance
      │
      ├── Database A
      ├── Database B
      └── Database C
```

Applications normally connect to a specific database.

The exact database model differs between engines.

---

# 19. What does RDS manage automatically?

Depending on configuration and engine, RDS can handle many infrastructure-oriented tasks such as:

```text
Infrastructure provisioning
Backups
Monitoring integration
Certain maintenance operations
Software patching
High availability options
Storage management
```

But the exact behavior depends on the engine and configuration.

Always distinguish:

> **Managed infrastructure ≠ fully managed database administration.**

---

# 20. What does RDS NOT magically solve?

RDS doesn't automatically make bad SQL good.

For example:

```sql
SELECT *
FROM huge_table
WHERE function(column) = 'X';
```

RDS won't automatically redesign the query.

You still need to manage:

```text
Indexes
SQL
Schema
Transactions
Connection behavior
Permissions
Application design
```

---

# 21. RDS lifecycle

A simplified lifecycle looks like:

```text
Create
  ↓
Configure
  ↓
Run
  ↓
Monitor
  ↓
Maintain
  ↓
Scale
  ↓
Backup
  ↓
Upgrade
  ↓
Retire
```

A production DBA needs to understand the entire lifecycle, not just creation.

---

# 22. RDS creation decisions

When creating an RDS database, think about:

```text
1. Engine
2. Engine version
3. Instance class
4. Storage
5. Network/VPC
6. Subnet group
7. Security groups
8. Authentication
9. Backup
10. Availability
```

Don't treat the creation wizard as merely:

> "Click Create."

It is an architecture decision.

---

# 23. Example production architecture

A simple production setup:

```text
                     Users
                       │
                       ▼
                 Application
                       │
                Private Network
                       │
                       ▼
              ┌────────────────┐
              │      RDS       │
              │                │
              │ DB Instance    │
              │                │
              └───────┬────────┘
                      │
                RDS Storage
```

Then we add HA:

```text
                ┌──────────────┐
                │   RDS HA     │
                └──────┬───────┘
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          AZ-A                AZ-B
        Primary              Standby
```

We'll study that architecture deeply in **Lesson 4**.

---

# 24. RDS vs EC2 self-managed database

This is a common interview question.

### Self-managed on EC2

```text
EC2
 │
 └── Database
      │
      ├── OS management
      ├── Patching
      ├── Backups
      ├── HA
      └── DB administration
```

You control almost everything.

### RDS

```text
RDS
 │
 └── Managed database
      │
      ├── AWS-managed infrastructure
      ├── Managed backups
      ├── Maintenance capabilities
      ├── HA options
      └── Monitoring integration
```

You give up some infrastructure-level control in exchange for managed operations.

---

# 25. RDS vs Aurora — important distinction

Since you previously studied Aurora, remember this:

```text
Amazon RDS
     │
     ├── PostgreSQL
     ├── MySQL
     ├── MariaDB
     ├── Oracle
     ├── SQL Server
     └── Db2

Aurora
     │
     ├── Aurora PostgreSQL
     └── Aurora MySQL
```

Both are AWS managed database services, but their architectures differ.

For this course:

> **We are focusing strictly on Amazon RDS.**

---

# 26. Common RDS misconception

### ❌ Incorrect

> "RDS is just a PostgreSQL server hosted by AWS."

### Better understanding

```text
RDS
 │
 ├── Managed database service
 ├── Infrastructure abstraction
 ├── Backup integration
 ├── Monitoring
 ├── HA capabilities
 ├── Maintenance
 └── AWS operational integration
```

The database engine is only one part of the service.

---

# 27. Real-world scenario

### Scenario

Your company currently runs PostgreSQL manually on an EC2 instance.

The DBA team spends time handling:

```text
OS patching
Database backups
Monitoring
Storage expansion
Maintenance
Failover
```

Management wants less infrastructure administration.

### What could RDS provide?

```text
Self-managed PostgreSQL
        ↓
       RDS
        ↓
Managed infrastructure
        +
Backup capabilities
        +
Monitoring integration
        +
Multi-AZ options
        +
AWS operational integration
```

But you still retain responsibility for:

```text
Schema
SQL
Indexes
Database users
Application behavior
Performance tuning
```

---

# 28. Real-world scenario: RDS is slow

Suppose users complain:

> "The application is slow."

Don't immediately say:

> "Increase the RDS instance size."

Use:

```text
Application
    ↓
CloudWatch
    ↓
CPU / Memory / Connections / I/O
    ↓
Performance Insights
    ↓
Top SQL
    ↓
Execution plan
    ↓
Root cause
```

This approach will become central in **Lesson 8**.

---

# 29. Real-world scenario: RDS is unavailable

You discover the database isn't accepting connections.

Possible causes include:

```text
Database instance issue
Network problem
Security group
DNS/connectivity
Maintenance
Failover
Application configuration
Database engine issue
```

Your investigation should therefore be:

```text
Application
   ↓
Network
   ↓
RDS endpoint
   ↓
Security group
   ↓
RDS status/events
   ↓
Database
```

Never assume every connection failure is a database failure.

---

# 30. The RDS mental model

This is the most important diagram from Lesson 1:

```text
                         AWS REGION
                             │
                    ┌────────┴────────┐
                    │       VPC       │
                    │                 │
                    │  ┌───────────┐  │
                    │  │  RDS      │  │
Application ────────┼─►│ DB Instance│ │
                    │  └─────┬─────┘  │
                    │        │        │
                    │    ┌───▼────┐   │
                    │    │Storage │   │
                    │    └────────┘   │
                    │                 │
                    └─────────────────┘
```

And around it:

```text
                 ┌───────────────┐
                 │    SECURITY   │
                 └───────┬───────┘
                         │
                 ┌───────▼───────┐
                 │   MONITORING  │
                 └───────┬───────┘
                         │
                 ┌───────▼───────┐
                 │    BACKUP     │
                 └───────┬───────┘
                         │
                 ┌───────▼───────┐
                 │      HA       │
                 └───────────────┘
```

---

# 🧠 Interview Questions

### Q1. What is Amazon RDS?

**Answer:**

1. Amazon RDS is an AWS managed relational database service.
2. It supports multiple relational database engines.
3. AWS manages much of the underlying infrastructure.
4. It provides capabilities such as backups, monitoring, and HA options.
5. Customers still manage database-level objects, SQL, users, and application behavior.
6. It reduces operational overhead compared with self-managed databases.

---

### Q2. What is an RDS DB instance?

1. It is the compute environment running the selected database engine.
2. It has CPU, memory, networking, and attached storage.
3. Its capacity depends on the selected instance class.
4. It is associated with network configuration such as subnets and security groups.
5. It can be monitored through AWS monitoring services.
6. Availability characteristics depend on the RDS configuration.

---

### Q3. What is the difference between RDS and EC2-hosted PostgreSQL?

1. RDS is a managed database service.
2. EC2-hosted PostgreSQL is generally self-managed.
3. With EC2, you manage the OS and more infrastructure responsibilities.
4. RDS provides managed backup, maintenance, monitoring integration, and HA capabilities.
5. EC2 provides greater infrastructure-level control.
6. The right choice depends on operational requirements and required control.

---

### Q4. What is a DB subnet group?

1. It identifies subnets that RDS can use.
2. It is associated with a VPC.
3. Production databases are commonly placed in private subnets.
4. Multi-AZ designs use subnets across multiple AZs.
5. It is part of RDS network architecture.
6. Incorrect subnet configuration can prevent proper deployment/connectivity.

---

### Q5. What are the main things you configure when creating RDS?

1. Database engine and version.
2. DB instance class.
3. Storage configuration.
4. VPC, subnet group, and security groups.
5. Authentication and backup settings.
6. Availability and maintenance options.

---

# 🎯 Lesson 1 Summary

Remember these **10 fundamentals**:

```text
1. RDS = Managed relational database service
2. PostgreSQL/MySQL/etc. = Database engine
3. DB instance = Compute environment
4. Instance class = CPU + memory + network capacity
5. Storage = Separate capacity/performance consideration
6. RDS runs inside a VPC
7. DB subnet groups control subnet placement
8. Security groups control network access
9. RDS provides managed capabilities but doesn't manage everything
10. Production design = Network + Security + HA + Backup + Monitoring
```

## 🏆 The one-line definition

> **Amazon RDS is an AWS managed service that simplifies running relational database engines by handling much of the underlying infrastructure and operational work while leaving database-level design, SQL, security, and application responsibilities with you.**

**Next: Lesson 2 — RDS Networking & Connectivity**
