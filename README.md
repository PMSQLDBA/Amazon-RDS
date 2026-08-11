# ☁️ Amazon RDS — 10-Lesson Roadmap

| Lesson | Topic                                              | What You’ll Learn                                                                                 |
| ------ | -------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **1**  | RDS Fundamentals & Architecture                    | What RDS is, engines, managed-service model, DB instances, storage, Regions/AZs                   |
| **2**  | RDS Networking & Connectivity                      | VPC, subnets, subnet groups, security groups, endpoints, ports, private/public access             |
| **3**  | RDS Storage & Instance Sizing                      | Storage types, IOPS, throughput, autoscaling, instance classes, capacity planning                 |
| **4**  | High Availability & Multi-AZ                       | Multi-AZ DB instances, standby, failover, DNS, RTO, HA design                                     |
| **5**  | Read Replicas & Read Scaling                       | Read replicas, replication, read traffic, lag, promotion, scaling patterns                        |
| **6**  | Backup, Restore & Disaster Recovery                | Automated backups, snapshots, PITR, restore, RPO/RTO, DR strategies                               |
| **7**  | Security & Access Control                          | IAM, database authentication, encryption, KMS, TLS, Secrets Manager, least privilege              |
| **8**  | Monitoring & Performance Troubleshooting           | CloudWatch, Enhanced Monitoring, Performance Insights, logs, slow queries, bottlenecks            |
| **9**  | RDS Operations & Maintenance                       | Parameter/option groups, patching, upgrades, maintenance windows, events, monitoring              |
| **10** | Production RDS Architecture & Real-World Scenarios | Production design, migrations, failures, scaling, DR, troubleshooting, operational best practices |

## Lesson progression

### **Lesson 1 — RDS Fundamentals & Architecture**

You will understand:

* What Amazon RDS actually manages
* RDS vs self-managed databases
* Supported database engines
* DB instances and DB clusters
* Storage architecture
* Regions and Availability Zones
* RDS lifecycle
* What a DBA still needs to manage

### **Lesson 2 — Networking & Connectivity**

* VPC architecture
* DB subnet groups
* Private vs public access
* Security groups
* DNS/endpoints
* PostgreSQL/MySQL/SQL Server/Oracle connection concepts
* Application-to-RDS connectivity
* Common connection failures

### **Lesson 3 — Storage & Sizing**

* General Purpose SSD
* Provisioned IOPS
* Storage throughput
* Storage autoscaling
* Instance classes
* CPU vs memory bottlenecks
* IOPS planning
* Capacity planning

### **Lesson 4 — High Availability & Multi-AZ**

* What Multi-AZ actually means
* Primary and standby
* Synchronous replication concepts
* Automatic failover
* DNS behavior
* Failover testing
* RTO considerations
* Multi-AZ vs Read Replica

### **Lesson 5 — Read Replicas**

* Why read replicas exist
* Asynchronous replication
* Replica lag
* Read scaling
* Reporting workloads
* Promotion
* Replica limitations
* Read replica vs Multi-AZ

### **Lesson 6 — Backup & DR**

* Automated backups
* Manual snapshots
* Point-in-time recovery
* Restore workflows
* Backup retention
* RPO
* RTO
* Cross-Region DR

### **Lesson 7 — Security**

* IAM
* Database users/roles
* KMS encryption
* Encryption at rest
* TLS encryption in transit
* Secrets Manager
* Security groups
* Least privilege
* Credential rotation

### **Lesson 8 — Monitoring & Performance**

* CloudWatch
* Enhanced Monitoring
* Performance Insights
* CPU
* Memory
* Connections
* IOPS
* Latency
* Database logs
* Slow queries
* Lock contention

### **Lesson 9 — Operations**

* Parameter groups
* Option groups
* Maintenance windows
* Minor/major upgrades
* Patching
* RDS events
* Reboots
* Configuration changes
* Operational runbooks

### **Lesson 10 — Production Mastery**

This is where everything comes together:

```text
                    AMAZON RDS
                        │
       ┌────────────────┼────────────────┐
       │                │                │
   NETWORKING          HA/DR          SECURITY
       │                │                │
      VPC            Multi-AZ          IAM
    Subnets          Backups           KMS
 Security Groups       PITR            TLS
    Endpoint            DR          Secrets
       │                │                │
       └────────────────┼────────────────┘
                        │
                   PERFORMANCE
                        │
              CloudWatch / PI
                        │
                 Storage / IOPS
                        │
                    OPERATIONS
                        │
             Patching / Upgrades
                        │
                        ▼
              PRODUCTION RDS
```
