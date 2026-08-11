# ☁️ Amazon RDS — Top 100 FAQs

## 1. What is Amazon RDS?

1. Amazon RDS is a managed relational database service from AWS.
2. It supports engines such as PostgreSQL, MySQL, MariaDB, Oracle, and SQL Server.
3. AWS manages much of the underlying infrastructure and common operational tasks.
4. You still manage database-level configuration, schemas, users, queries, and workload behavior.
5. RDS supports features such as backups, Multi-AZ, monitoring, and read replicas.
6. It is designed to reduce the operational burden of running relational databases.

---

## 2. Why use RDS instead of installing a database on EC2?

1. RDS removes much of the infrastructure-management overhead.
2. Automated backups and point-in-time recovery are available.
3. Managed Multi-AZ capabilities simplify high availability.
4. Monitoring and maintenance integrations are built into the service.
5. You don't have to manage the underlying database host operating system.
6. EC2 may still be preferable when you need deeper OS/database-level control.

---

## 3. Which database engines does Amazon RDS support?

1. RDS supports several major relational database engines.
2. Common choices include PostgreSQL and MySQL.
3. It also supports MariaDB, Oracle Database, and Microsoft SQL Server.
4. Feature availability varies by engine.
5. Version availability also varies over time.
6. Always verify the current AWS documentation before designing around an engine-specific feature.

---

## 4. What is an RDS DB instance?

1. A DB instance provides the compute resources for an RDS database.
2. It includes CPU and memory capacity.
3. It works with the configured database engine.
4. Storage is attached/configured separately according to the selected storage architecture.
5. Instance size directly affects available compute capacity.
6. You can change the DB instance class when more or less capacity is required.

---

## 5. What is an RDS endpoint?

1. An endpoint is the network address applications use to connect to RDS.
2. Applications should normally connect through the appropriate RDS endpoint rather than an underlying host IP.
3. Endpoints simplify managed failover scenarios.
4. Different RDS configurations can expose different endpoint types.
5. Application connection strings should be designed around the appropriate endpoint.
6. Avoid hardcoding database infrastructure IP addresses.

---

## 6. What is a DB subnet group?

1. A DB subnet group identifies subnets that RDS can use for database placement.
2. It normally contains subnets across multiple Availability Zones.
3. Production deployments commonly use private subnets.
4. Multi-AZ architectures benefit from subnet placement across AZs.
5. Networking and route-table design must allow application connectivity.
6. A correctly designed subnet group is a foundational part of RDS networking.

---

## 7. Should an RDS database be publicly accessible?

1. Production databases should generally not be directly exposed to the internet.
2. Place the database in private subnets where appropriate.
3. Applications can access it through controlled private networking.
4. Security Groups should restrict inbound access.
5. Administrative access should use secure, controlled mechanisms.
6. Public accessibility should only be enabled when there is a specific justified requirement.

---

## 8. What is a Security Group for RDS?

1. A Security Group controls network access to the RDS instance.
2. It defines allowed inbound and outbound traffic.
3. A common design allows the application Security Group to access the RDS Security Group.
4. Only the required database port should normally be exposed.
5. Avoid broad rules such as allowing database access from the entire internet.
6. Security Groups are a major part of the RDS network-security boundary.

---

## 9. How does an application connect to RDS?

1. The application resolves the RDS endpoint.
2. It connects using the database engine's appropriate port.
3. The RDS Security Group must allow the application source.
4. The database credentials must be valid.
5. Network routing, DNS, and subnet configuration must be correct.
6. TLS should be considered for protecting traffic in transit.

---

## 10. What are common RDS ports?

1. PostgreSQL commonly uses TCP 5432.
2. MySQL commonly uses TCP 3306.
3. MariaDB commonly uses TCP 3306.
4. Microsoft SQL Server commonly uses TCP 1433.
5. Oracle commonly uses TCP 1521.
6. The actual configured port should always be verified rather than assumed.

---

# 🔐 Security

## 11. How do you secure an RDS database?

1. Place it in private networking where appropriate.
2. Restrict Security Group access to trusted application sources.
3. Enable encryption at rest.
4. Use TLS for connections where required.
5. Store credentials securely using Secrets Manager or an equivalent mechanism.
6. Apply least privilege to both AWS and database-level permissions.

---

## 12. What is encryption at rest in RDS?

1. Encryption at rest protects stored database data.
2. RDS encryption uses AWS KMS keys.
3. Depending on the configuration, encryption can cover storage and associated database backup data.
4. It protects data if underlying storage is accessed improperly.
5. Encryption at rest does not automatically mean network traffic is encrypted.
6. You should separately consider TLS for data in transit.

---

## 13. What is encryption in transit?

1. Encryption in transit protects traffic between the application and database.
2. TLS is commonly used for this purpose.
3. It prevents network traffic from being transmitted as readable plaintext.
4. The database engine must support/configure the appropriate TLS behavior.
5. Clients and drivers must be configured correctly.
6. Production applications should verify certificate and TLS requirements.

---

## 14. What is AWS KMS used for with RDS?

1. AWS KMS manages cryptographic keys.
2. RDS can use KMS keys for encryption at rest.
3. Organizations can use customer-managed keys when their governance requires them.
4. KMS permissions must be correctly configured.
5. Key lifecycle and access should be managed carefully.
6. Losing access to required encryption keys can affect recovery operations.

---

## 15. Should the application use the RDS master/admin account?

1. The application should generally not use the master/admin account.
2. Create an application-specific database user or role.
3. Grant only the permissions required by the application.
4. Separate administrative access from application access.
5. Rotate application credentials regularly.
6. This reduces the impact of credential compromise.

---

## 16. What is least privilege?

1. Least privilege means giving users only the permissions they actually require.
2. Application users should not automatically receive administrative privileges.
3. Separate read-only, read-write, and administrative roles when appropriate.
4. AWS IAM permissions should also follow least privilege.
5. Review privileges periodically.
6. Least privilege limits the blast radius of mistakes and compromised credentials.

---

## 17. How should RDS credentials be stored?

1. Don't hardcode credentials in source code.
2. Don't store production passwords in Git repositories.
3. Use AWS Secrets Manager or an equivalent secure solution.
4. Applications should retrieve credentials securely.
5. Rotate credentials according to operational requirements.
6. Audit who and what can access the secret.

---

## 18. What happens if an RDS password is exposed?

1. Treat it as a security incident.
2. Rotate the affected credential immediately when appropriate.
3. Investigate where the credential was exposed.
4. Review database and AWS audit activity.
5. Remove unnecessary access.
6. Implement controls to prevent the credential from being exposed again.

---

# 💾 Storage & Backups

## 19. What storage options does RDS provide?

1. RDS supports different storage types depending on engine and configuration.
2. General-purpose SSD storage is commonly used for many workloads.
3. Provisioned IOPS storage is designed for workloads requiring higher predictable I/O performance.
4. Magnetic storage has more limited applicability today.
5. Storage performance and capacity should be selected based on workload requirements.
6. Always check current engine-specific storage availability.

---

## 20. What is storage autoscaling?

1. Storage autoscaling can automatically increase allocated storage when configured thresholds are reached.
2. It helps reduce the risk of running out of database storage.
3. It primarily solves storage-capacity growth.
4. It does not fix poor SQL queries.
5. It does not automatically solve CPU or memory bottlenecks.
6. Storage growth should still be monitored and investigated.

---

## 21. What happens when RDS runs out of storage?

1. Database operations can become severely impacted.
2. Monitoring should detect decreasing free storage before exhaustion.
3. Storage should be increased where appropriate.
4. Storage autoscaling can reduce this operational risk when supported/configured.
5. Investigate why storage consumption increased.
6. Never wait until the database reaches zero free storage before reacting.

---

## 22. What is an RDS automated backup?

1. Automated backups provide managed backup capability for an RDS database.
2. They support recovery within the configured backup retention period.
3. They are important for operational recovery.
4. They can support point-in-time recovery.
5. Retention should be aligned with business requirements.
6. Backup configuration should be tested rather than assumed to work.

---

## 23. What is Point-in-Time Recovery?

1. PITR allows recovery to a selected point within the available recovery window.
2. It is useful for accidental data changes or operational mistakes.
3. Example: recovering to just before an accidental DELETE.
4. It differs from restoring a single static snapshot.
5. Recovery time depends on database size and other factors.
6. RPO requirements should influence the backup strategy.

---

## 24. What is an RDS snapshot?

1. A snapshot captures a recoverable state of the database.
2. It can be retained for longer-term recovery requirements.
3. Snapshots can be used to create new RDS databases.
4. They are useful for migration, testing, and disaster-recovery workflows.
5. Snapshots differ from continuous point-in-time recovery.
6. Snapshot retention should be managed to avoid unnecessary cost.

---

## 25. Snapshot vs automated backup?

1. Automated backups support point-in-time recovery.
2. Snapshots represent specific recovery points.
3. Automated backups are useful for operational recovery.
4. Snapshots are useful for longer-term or explicitly retained copies.
5. Their retention and operational behavior differ.
6. A mature architecture may use both.

---

## 26. Are Read Replicas backups?

1. No.
2. Read Replicas are primarily designed for read scaling.
3. Replicated changes can include unwanted logical changes.
4. An accidental DELETE can propagate to a replica.
5. Backups and PITR provide historical recovery capabilities.
6. Never replace your backup strategy with Read Replicas.

---

# 🚀 Scaling

## 27. How do you scale an RDS database vertically?

1. Identify that compute capacity is the bottleneck.
2. Select a larger supported DB instance class.
3. Test the new size in a non-production environment where practical.
4. Schedule the change appropriately.
5. Monitor performance after resizing.
6. Verify that scaling actually addresses the bottleneck.

---

## 28. What is vertical scaling?

1. Vertical scaling means increasing the resources of a database instance.
2. You might increase CPU and memory capacity.
3. It is useful when the primary database is resource-constrained.
4. It does not inherently distribute read traffic.
5. It can be simpler than redesigning the application.
6. It should follow workload analysis rather than guesswork.

---

## 29. What is horizontal read scaling?

1. Horizontal read scaling distributes read workloads across multiple database instances.
2. RDS Read Replicas can support this pattern for supported engines/configurations.
3. The primary continues handling the write workload.
4. Eligible reads can be directed to replicas.
5. Replica lag must be considered.
6. Application routing logic is often required.

---

## 30. What is a Read Replica?

1. A Read Replica is a separate database instance receiving replicated data.
2. It can serve appropriate read queries.
3. It reduces read pressure on the source database.
4. It can be useful for reporting and read-heavy applications.
5. Replication is not necessarily instantaneous.
6. Therefore applications must understand consistency requirements.

---

## 31. When should you use Read Replicas?

1. When read traffic is a significant database bottleneck.
2. When workloads can tolerate replica consistency characteristics.
3. For reporting workloads that shouldn't compete with production transactions.
4. When additional read capacity is required.
5. When application architecture can route reads appropriately.
6. They should not be added without measuring actual workload demand.

---

## 32. Do Read Replicas handle writes?

1. Normal application writes go to the primary/source database.
2. Read Replicas are intended primarily for read workloads.
3. Adding replicas does not automatically distribute write traffic.
4. Write-heavy bottlenecks require different optimization strategies.
5. Query optimization may be more appropriate than adding replicas.
6. Extremely write-heavy systems may require architectural redesign.

---

## 33. What is replication lag?

1. Replication lag means the replica is behind the source.
2. A recent write on the primary may not immediately appear on the replica.
3. Heavy write workloads can contribute to lag.
4. Replica resource limitations can also contribute.
5. Long-running or resource-intensive operations can affect replication.
6. Monitor lag and design applications accordingly.

---

## 34. What is the read-after-write problem?

1. A user writes data to the primary.
2. The next request reads from a replica.
3. The replica may not yet contain the latest change.
4. The user sees stale data.
5. This can create confusing application behavior.
6. Consistency-sensitive reads may need to go to the primary.

---

## 35. Can you have multiple Read Replicas?

1. Yes, supported RDS engines/configurations can have multiple replicas.
2. Multiple replicas can distribute read workloads.
3. Different replicas can serve different application or reporting workloads.
4. Each replica introduces additional cost and operational considerations.
5. Replica lag should be monitored.
6. The exact replica limits depend on the engine and current AWS capabilities.

---

# 🟢 High Availability

## 36. What is Multi-AZ?

1. Multi-AZ is an RDS high-availability capability.
2. RDS maintains a standby in another Availability Zone in supported configurations.
3. The standby is primarily intended for failover.
4. It protects against certain infrastructure/AZ failures.
5. It is not the same thing as a Read Replica.
6. Multi-AZ should be considered for production workloads with HA requirements.

---

## 37. Multi-AZ vs Read Replica?

1. Multi-AZ primarily provides high availability and failover.
2. Read Replicas primarily provide read scaling.
3. Multi-AZ standby is not your normal read-scaling destination.
4. Read Replicas can have replication lag.
5. Both solve different architectural problems.
6. Many production systems may use both.

---

## 38. Does Multi-AZ improve read performance?

1. Traditional Multi-AZ standby is not designed for normal read scaling.
2. Application reads generally continue going to the primary.
3. The standby exists for availability/failover.
4. Read Replicas are the normal RDS mechanism for read scaling.
5. Don't treat Multi-AZ as two active application databases.
6. Always distinguish HA from read scalability.

---

## 39. What happens during RDS failover?

1. RDS detects an eligible failure condition.
2. It performs the managed failover process.
3. The standby becomes the primary in applicable Multi-AZ configurations.
4. The RDS endpoint abstracts the underlying infrastructure change.
5. Existing application connections may need to reconnect.
6. Applications should have appropriate retry/reconnection behavior.

---

## 40. Does RDS failover change the endpoint?

1. Applications should use the appropriate RDS endpoint.
2. The endpoint is designed to abstract the underlying database instance.
3. During failover, the endpoint can point to the new primary.
4. Existing connections may still break.
5. Application connection pools must recover appropriately.
6. Never build production architecture around a fixed underlying RDS IP.

---

## 41. How long does RDS failover take?

1. Failover duration varies by configuration and circumstances.
2. It is not safe to promise one universal number.
3. Application reconnect behavior affects perceived downtime.
4. DNS/client caching can influence recovery behavior.
5. Test actual failover behavior for your workload.
6. Use measured results when defining RTO.

---

## 42. Is Multi-AZ a disaster recovery solution?

1. Multi-AZ is primarily a high-availability mechanism.
2. It protects against certain instance/AZ failures.
3. It does not automatically solve every regional disaster.
4. Regional disasters require an appropriate DR strategy.
5. Backups and PITR remain important.
6. RPO/RTO requirements determine the appropriate DR architecture.

---

# 🌎 Disaster Recovery

## 43. What is RPO?

1. RPO means Recovery Point Objective.
2. It defines how much data loss the business can tolerate.
3. An RPO of five minutes means approximately five minutes of data loss may be acceptable.
4. Backup/replication architecture must support the required RPO.
5. RPO is a business requirement, not merely a database setting.
6. Always validate the actual achievable RPO through testing.

---

## 44. What is RTO?

1. RTO means Recovery Time Objective.
2. It defines how quickly the service must be restored.
3. An RTO of 30 minutes means the recovery process should complete within that target.
4. Database recovery alone isn't the entire RTO.
5. Application, networking, DNS, secrets, and validation are also involved.
6. Test the complete recovery process.

---

## 45. How would you design RDS for disaster recovery?

1. Define RPO and RTO first.
2. Enable appropriate automated backups and PITR.
3. Maintain appropriate snapshots/backup retention.
4. Consider cross-Region replication or recovery architecture when required.
5. Document the failover procedure.
6. Regularly test the DR process.

---

## 46. What is cross-Region RDS replication?

1. Supported RDS engines can use cross-Region replication mechanisms in appropriate configurations.
2. A replica can exist in another AWS Region.
3. It can help with regional DR.
4. It can also support regional read access in certain designs.
5. Cross-Region replication introduces latency and additional cost.
6. The architecture must be designed around RPO/RTO requirements.

---

## 47. Why isn't a cross-Region replica enough for DR?

1. You need more than a database copy.
2. Application infrastructure must also be available or recoverable.
3. Networking and Security Groups must be prepared.
4. Secrets and permissions must work in the DR environment.
5. DNS/application traffic must be redirected.
6. The entire recovery process must be tested.

---

# 📊 Monitoring & Troubleshooting

## 48. How do you monitor RDS?

1. Use Amazon CloudWatch for infrastructure/database metrics.
2. Monitor CPU utilization.
3. Monitor free storage.
4. Monitor database connections.
5. Monitor I/O and latency-related metrics.
6. Use database performance tooling such as Performance Insights where appropriate.

---

## 49. What is Performance Insights?

1. Performance Insights helps analyze database workload.
2. It can identify database load and top contributing SQL.
3. It helps investigate wait behavior.
4. It is useful when a database appears slow.
5. It complements infrastructure metrics.
6. It can help distinguish database workload problems from infrastructure capacity issues.

---

## 50. RDS CPU is 95%. What do you do?

1. Confirm whether the spike is sustained or temporary.
2. Identify top SQL and database waits.
3. Check for recent application/deployment changes.
4. Review execution plans and indexing.
5. Determine whether scaling is actually necessary.
6. Monitor after implementing the fix.

---

## 51. RDS CPU is low but application is slow. Why?

1. Database CPU is only one performance dimension.
2. The problem could be I/O or connection contention.
3. Queries may be waiting on locks.
4. Network latency may be involved.
5. The application could have a connection-pool bottleneck.
6. Investigate the complete request path rather than assuming RDS is the problem.

---

## 52. How do you troubleshoot a slow query?

1. Identify the query from database monitoring.
2. Examine its execution plan.
3. Check indexes and table statistics.
4. Look for full scans, inefficient joins, or excessive result sets.
5. Check locks and waits.
6. Optimize and measure again.

---

## 53. Why shouldn't you immediately increase the RDS instance size?

1. The root cause might be inefficient SQL.
2. A missing index can create excessive workload.
3. Lock contention won't necessarily disappear with more CPU.
4. Application connection problems can masquerade as database performance issues.
5. Scaling increases cost.
6. Measure the bottleneck before changing infrastructure.

---

## 54. What is connection exhaustion?

1. It occurs when the database approaches its connection capacity.
2. Applications may begin receiving connection failures.
3. Excessive connection pools are a common cause.
4. Connection leaks can make the situation worse.
5. Long-lived idle sessions can consume resources.
6. Monitor connections and tune application pooling appropriately.

---

## 55. How do you troubleshoot connection failures?

1. Check whether the RDS instance is available.
2. Verify DNS resolution and network routing.
3. Check Security Group rules.
4. Validate the database port.
5. Verify credentials and TLS configuration.
6. Check application connection-pool behavior.

---

## 56. How do you troubleshoot high storage usage?

1. Check the free-storage metric.
2. Identify the rate of storage growth.
3. Investigate large tables and indexes where applicable.
4. Check logs and temporary data.
5. Increase capacity if required.
6. Determine the root cause so the problem doesn't repeat.

---

# 🛠️ Operations

## 57. What is an RDS maintenance window?

1. It is a configured period for certain maintenance activities.
2. It helps organizations control when disruptive maintenance may occur.
3. Choose a window with minimal business impact.
4. Monitor pending maintenance events.
5. Test important upgrades before production.
6. Communicate planned changes to stakeholders.

---

## 58. What is a DB parameter group?

1. A parameter group controls database engine configuration parameters.
2. Parameters can influence database behavior.
3. Different workloads may require different configurations.
4. Parameter changes should be tested before production.
5. Some changes may require a restart depending on the parameter.
6. Never change parameters without understanding their engine-specific behavior.

---

## 59. What is an option group?

1. Option groups are used by supported RDS engines for certain additional features.
2. They can enable engine-specific functionality.
3. Options can affect database behavior and operations.
4. Some options may have deployment implications.
5. Review compatibility before enabling an option.
6. Option groups are distinct from parameter groups.

---

## 60. How should you handle an RDS engine upgrade?

1. Identify whether it is a minor or major upgrade.
2. Test the upgrade in non-production.
3. Verify application and driver compatibility.
4. Test extensions, parameters, and SQL behavior.
5. Confirm backups/recovery procedures.
6. Monitor closely after production upgrade.

---

## 61. Why are major database upgrades risky?

1. Database behavior can change.
2. SQL compatibility can change.
3. Extensions may have compatibility requirements.
4. Execution plans can change.
5. Application drivers may need updates.
6. Thorough testing is therefore essential.

---

## 62. Is RDS completely maintenance-free?

1. No.
2. AWS manages the underlying service infrastructure.
3. You still manage database configuration and workload behavior.
4. You must manage users, schemas, SQL, and application integration.
5. You must monitor performance and storage.
6. You must plan backups, upgrades, and operational procedures.

---

# 🔄 Migration

## 63. How do you migrate an existing database to RDS?

1. Assess source and target compatibility.
2. Prepare the target RDS environment.
3. Choose dump/restore or an online migration strategy.
4. Transfer and validate the data.
5. Perform application cutover.
6. Monitor and maintain a rollback strategy.

---

## 64. When is dump-and-restore appropriate?

1. It can work well for smaller databases.
2. It is conceptually straightforward.
3. It may require application downtime.
4. The database is exported and restored into RDS.
5. Data and schema validation must follow the restore.
6. It may not be suitable for very large databases with strict uptime requirements.

---

## 65. What is AWS DMS?

1. AWS Database Migration Service helps migrate databases.
2. It can support initial data loading.
3. It can support ongoing change replication in suitable migration scenarios.
4. This can reduce application downtime during cutover.
5. Source/target compatibility must be evaluated.
6. DMS does not eliminate the need for migration testing and validation.

---

## 66. How would you migrate a large production database with minimal downtime?

1. Prepare the target RDS environment.
2. Perform an initial full load.
3. Continuously replicate changes where supported.
4. Validate source and target.
5. Schedule a short cutover window.
6. Stop/freeze writes, synchronize, switch traffic, and monitor.

---

## 67. What should you check before an RDS migration?

1. Engine and version compatibility.
2. Schema and extension compatibility.
3. Users and permissions.
4. Network connectivity.
5. Database size and performance requirements.
6. Downtime, RPO/RTO, validation, and rollback requirements.

---

## 68. What should you validate after migration?

1. Table and row counts.
2. Indexes and constraints.
3. Sequences and database objects.
4. Users and permissions.
5. Application connectivity.
6. Real application workflows and query performance.

---

## 69. Why is rollback important during migration?

1. Migration can fail after application cutover.
2. The source may still be required for recovery.
3. New writes on the target complicate rollback.
4. A rollback procedure should be defined beforehand.
5. The team should know who makes the rollback decision.
6. The rollback plan should be tested where practical.

---

# 🏗️ Production Architecture

## 70. What does a production RDS architecture look like?

1. Applications run in controlled private networking.
2. RDS is placed in appropriate private subnets.
3. Security Groups restrict access.
4. Multi-AZ provides HA when required.
5. Backups/PITR provide recovery.
6. Monitoring, secrets management, and DR complete the architecture.

---

## 71. Should application and database be in the same VPC?

1. They are commonly deployed in the same VPC for simple private connectivity.
2. This allows controlled internal routing.
3. Security Groups can restrict access.
4. Separate application and database subnets improve segmentation.
5. More complex organizations may use multiple VPCs.
6. Connectivity can then be established through appropriate AWS networking mechanisms.

---

## 72. How should RDS Security Groups be designed?

1. Allow access only from required application sources.
2. Prefer Security Group references over broad CIDR ranges where appropriate.
3. Expose only the required database port.
4. Avoid public access.
5. Separate application and database security groups.
6. Review rules periodically.

---

## 73. What is a good production RDS architecture?

1. Private RDS networking.
2. Restricted Security Groups.
3. Encryption at rest and TLS.
4. Multi-AZ for required HA.
5. Automated backups/PITR and appropriate DR.
6. Monitoring, alerting, secrets management, and operational runbooks.

---

## 74. What should an RDS production runbook contain?

1. High CPU troubleshooting.
2. High storage troubleshooting.
3. Connection failure procedures.
4. Replica lag investigation.
5. Failover procedures.
6. Backup restore, migration, upgrade, and DR procedures.

---

# 💰 Cost Optimization

## 75. How can you reduce RDS costs?

1. Right-size DB instances.
2. Remove unused databases and replicas.
3. Select appropriate storage.
4. Review backup and snapshot retention.
5. Optimize queries to reduce unnecessary resource consumption.
6. Continuously monitor cost and utilization.

---

## 76. Can Read Replicas increase cost?

1. Yes.
2. Each replica consumes additional database resources.
3. Storage and other associated costs can also apply.
4. Cross-Region replicas introduce additional considerations.
5. Replicas should be justified by workload requirements.
6. Remove unused replicas after validating they are no longer needed.

---

## 77. Should you remove Multi-AZ to save money?

1. Not without understanding the business requirement.
2. Multi-AZ provides important availability benefits.
3. Removing it can increase outage risk.
4. Compare cost against required RTO/availability.
5. Production databases often justify HA costs.
6. Cost optimization should preserve required resilience.

---

# 🧩 Real-World Scenarios

## 78. Production RDS suddenly becomes slow. What do you check?

1. CloudWatch metrics.
2. Performance Insights.
3. Top SQL queries.
4. Database waits and locks.
5. Recent deployments or workload changes.
6. Only then decide whether optimization or scaling is required.

---

## 79. RDS CPU reaches 100% during peak traffic. What do you do?

1. Identify the SQL causing the load.
2. Determine whether the workload is read or write heavy.
3. Check indexes and execution plans.
4. Consider scaling if compute is genuinely insufficient.
5. Consider Read Replicas if reads are the bottleneck.
6. Monitor after remediation.

---

## 80. RDS storage is almost full. What do you do?

1. Confirm the storage trend.
2. Increase storage where necessary.
3. Use storage autoscaling where appropriate.
4. Identify what is consuming the storage.
5. Investigate abnormal growth.
6. Add monitoring/alerts to prevent recurrence.

---

## 81. Read Replica is significantly behind. What do you do?

1. Confirm replication lag.
2. Check source write volume.
3. Check replica resources.
4. Analyze workload and long-running operations.
5. Determine whether scaling or workload separation is needed.
6. Prevent applications from relying on stale data where consistency matters.

---

## 82. Users report that newly created records are missing.

1. Determine which database endpoint served the read.
2. Check whether the read went to a replica.
3. Check replication lag.
4. Confirm the write succeeded on the primary.
5. Route consistency-sensitive reads appropriately.
6. Review the application's read/write routing strategy.

---

## 83. RDS failover occurred but application is still down. Why?

1. Existing connections may have become invalid.
2. The application connection pool may not reconnect.
3. DNS caching may be involved.
4. Retry logic may be inadequate.
5. Network/security rules should be verified.
6. Application failover behavior must be tested independently of RDS failover.

---

## 84. An application deployment suddenly makes RDS slow. What do you investigate?

1. Compare database metrics before and after deployment.
2. Identify new SQL patterns.
3. Check query execution plans.
4. Look for N+1 queries or excessive connections.
5. Check transaction/lock behavior.
6. Roll back or optimize the deployment if it is the confirmed cause.

---

## 85. A large reporting query impacts production. What can you do?

1. Identify the expensive query.
2. Optimize the query if possible.
3. Consider a Read Replica for reporting.
4. Route reporting traffic separately.
5. Consider an analytics-specific architecture for very large workloads.
6. Monitor production workload after separation.

---

## 86. The database is healthy but the application is timing out. What do you check?

1. Application connection pools.
2. Network latency.
3. Security Groups.
4. DNS.
5. Application thread/worker saturation.
6. External dependencies and API latency.

---

## 87. A production database was accidentally deleted from application data. What do you do?

1. Stop further destructive activity.
2. Determine the exact incident time.
3. Identify the appropriate recovery point.
4. Use PITR or another backup recovery mechanism.
5. Validate recovered data.
6. Investigate the root cause and prevent recurrence.

---

## 88. An RDS credential is accidentally committed to Git. What do you do?

1. Treat the credential as compromised.
2. Rotate/revoke it.
3. Check where the credential may have been used.
4. Review relevant audit activity.
5. Remove the secret from future source history where appropriate.
6. Move credentials to secure secret management.

---

## 89. Your RDS database needs regional DR. What do you do?

1. Define RPO and RTO.
2. Choose an appropriate cross-Region recovery mechanism.
3. Prepare infrastructure and networking in the DR Region.
4. Ensure secrets and permissions are available.
5. Document application traffic redirection.
6. Test the complete regional recovery process.

---

## 90. A database migration completed, but application queries fail. What do you check?

1. Database schema compatibility.
2. Users and permissions.
3. Extensions.
4. Engine/version differences.
5. Connection and TLS configuration.
6. SQL behavior and application driver compatibility.

---

# 🧠 Advanced Interview FAQs

## 91. What is the difference between HA, scaling, and DR?

1. HA focuses on minimizing service interruption.
2. Scaling focuses on handling increasing workload.
3. DR focuses on recovering from major failures.
4. Multi-AZ primarily addresses HA.
5. Read Replicas primarily address read scaling.
6. Backups/PITR and cross-Region strategies address recovery requirements.

---

## 92. Why is "Multi-AZ + Read Replica + Backup" a powerful combination?

1. Multi-AZ provides high availability.
2. Read Replicas provide read scaling.
3. Backups provide historical recovery.
4. Each solves a different problem.
5. Together they provide layered resilience.
6. The exact combination should still be based on business requirements.

---

## 93. What is the biggest RDS troubleshooting mistake?

1. Immediately increasing instance size.
2. Assuming CPU is always the problem.
3. Ignoring SQL execution plans.
4. Ignoring locks and waits.
5. Ignoring application behavior.
6. Failing to establish a measurable root cause.

---

## 94. What is the biggest RDS DR mistake?

1. Assuming Multi-AZ equals complete DR.
2. Having backups but never testing restores.
3. Not defining RPO/RTO.
4. Ignoring application recovery.
5. Ignoring secrets/networking/DNS.
6. Never performing a realistic DR exercise.

---

## 95. What is the biggest Read Replica mistake?

1. Assuming replicas are always current.
2. Treating replicas as backups.
3. Sending consistency-sensitive reads to replicas.
4. Ignoring replication lag.
5. Adding replicas without measuring workload.
6. Expecting replicas to solve write bottlenecks.

---

## 96. What is the biggest RDS security mistake?

1. Publicly exposing the database unnecessarily.
2. Allowing broad Security Group access.
3. Using administrator credentials in applications.
4. Storing passwords in source code.
5. Disabling encryption/TLS without justification.
6. Granting excessive database permissions.

---

## 97. What is the best RDS troubleshooting methodology?

1. Confirm the problem and scope.
2. Check infrastructure/database metrics.
3. Identify the workload causing the problem.
4. Investigate SQL, waits, locks, and connections.
5. Apply the smallest appropriate corrective action.
6. Validate the result and document the root cause.

---

## 98. How would you explain your RDS production design in an interview?

1. Start with private networking and Security Groups.
2. Explain encryption and secrets management.
3. Explain Multi-AZ for HA.
4. Explain Read Replicas for read scaling.
5. Explain backups/PITR and cross-Region DR according to RPO/RTO.
6. Finish with monitoring, migration, upgrades, and operational runbooks.

---

## 99. What should an RDS senior engineer know beyond the AWS console?

1. Database fundamentals.
2. SQL performance and execution plans.
3. Networking and DNS.
4. Security and IAM.
5. Backup/restore and DR.
6. Application connection pooling, migrations, monitoring, and incident response.

---

## 100. What is the ultimate Amazon RDS architecture principle?

1. **Private and secure by default.**
2. **Multi-AZ for required high availability.**
3. **Read Replicas for appropriate read scaling.**
4. **Backups/PITR for recovery.**
5. **RPO/RTO-driven DR architecture.**
6. **Monitor, test, document, and continuously optimize.**

---

# 🔥 Amazon RDS — 100 FAQ Master Map

```text
                    AMAZON RDS
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    FOUNDATION      SECURITY       NETWORKING
        │              │              │
     Engines          IAM            VPC
     Instance         KMS            Subnets
     Endpoint         TLS            SG
        │           Secrets          DNS
        │              │              │
        └──────────────┼──────────────┘
                       │
              ┌────────┴────────┐
              │                 │
             HA                SCALE
              │                 │
          Multi-AZ        Vertical Scale
          Failover        Read Replicas
              │                 │
              └────────┬────────┘
                       │
                  RECOVERY
                       │
            ┌──────────┼──────────┐
            │          │          │
         Backup       PITR       DR
            │          │          │
            └──────────┼──────────┘
                       │
                  OPERATIONS
                       │
       ┌───────────────┼────────────────┐
       │               │                │
   Monitoring      Migration        Troubleshooting
       │               │                │
 CloudWatch         DMS             Performance
 Perf Insights      Cutover          Connections
 Alarms             Rollback         Replica Lag
       │               │                │
       └───────────────┼────────────────┘
                       │
                  PRODUCTION
                       │
                 RPO + RTO
                       │
                Tested DR/HA
                       │
              🚀 Production RDS
```

## 🎯 The 15 things to memorize for interviews

**If you remember only 15 things from all 100 FAQs, remember these:**

1. **RDS = managed relational database service.**
2. **Private subnet + restricted Security Group** is the normal production pattern.
3. **Encryption at rest ≠ encryption in transit** — consider both.
4. **Secrets Manager ≠ database authorization** — you need both secure credential storage and proper DB permissions.
5. **Multi-AZ = HA/failover.**
6. **Read Replica = read scaling.**
7. **Read Replica ≠ backup.**
8. **Replica lag = possible stale reads.**
9. **Backups + PITR = historical recovery.**
10. **RPO = acceptable data loss.**
11. **RTO = acceptable recovery time.**
12. **Cross-Region DR = regional disaster strategy, not merely "another copy."**
13. **Performance troubleshooting starts with measurement, not resizing.**
14. **Large migrations require planning, validation, cutover, and rollback.**
15. **Production RDS = security + HA + backup + scaling + monitoring + operations + tested recovery.**
