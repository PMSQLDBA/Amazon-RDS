# 🔥 Amazon RDS — Top 50 Real-Time Scenario-Based Questions & Answers

---

## 1. Production RDS CPU suddenly reaches 95–100%. What do you do?

1. Check whether CPU usage is sustained or caused by a temporary traffic spike.
2. Use Performance Insights to identify top SQL statements and database waits.
3. Check recent deployments, traffic changes, batch jobs, and scheduled workloads.
4. Review query execution plans and indexes for expensive queries.
5. Scale the DB instance only if the workload genuinely requires more compute.
6. Monitor after remediation and document the root cause.

---

## 2. RDS CPU is only 30%, but the application is extremely slow. Why?

1. CPU is only one database performance metric.
2. Check I/O latency, storage throughput, locks, waits, and connection count.
3. Investigate slow SQL and execution plans.
4. Check whether the application connection pool is exhausted.
5. Verify network latency between application and RDS.
6. Determine whether the bottleneck is actually outside the database.

---

## 3. RDS storage is 95% full. What is your immediate action?

1. Check the free-storage metric and rate of storage consumption.
2. Increase storage capacity or use supported storage autoscaling.
3. Investigate which tables, indexes, logs, or temporary objects are consuming space.
4. Determine whether the growth is expected or abnormal.
5. Add/adjust CloudWatch alarms before the database reaches exhaustion.
6. Validate that the database has sufficient headroom after remediation.

---

## 4. Application cannot connect to RDS after deployment. What do you check?

1. Verify the RDS instance status and endpoint.
2. Check Security Group rules and the configured database port.
3. Verify subnet routing, NACLs, DNS, and application-to-database connectivity.
4. Check database credentials and secret configuration.
5. Validate TLS/certificate requirements if encryption in transit is enabled.
6. Test connectivity from the actual application environment, not just your laptop.

---

## 5. Developers ask you to make RDS publicly accessible. Would you?

1. Normally, no.
2. Prefer private subnets and private application-to-database connectivity.
3. Use controlled Security Group rules to permit only required sources.
4. If external access is genuinely required, use a secure architecture rather than broad internet exposure.
5. Apply TLS and strong authentication.
6. Document and review the exception from a security perspective.

---

## 6. A developer requests `0.0.0.0/0` access to PostgreSQL port 5432. What do you do?

1. Reject the rule for normal production use.
2. Identify the actual source that needs database access.
3. Allow only the required Security Group or IP range.
4. Prefer application Security Group references where appropriate.
5. Verify connectivity after tightening the rule.
6. Record the security rationale and remove unnecessary access.

---

## 7. Production database fails over during peak traffic. What happens next?

1. RDS performs the managed failover in a Multi-AZ configuration.
2. The application may temporarily lose existing database connections.
3. The application should reconnect through the RDS endpoint.
4. Connection pools and retry logic must handle transient failures.
5. Monitor application errors and database health during recovery.
6. Investigate why failover occurred afterward.

---

## 8. RDS has failed over, but the application doesn't recover automatically. Why?

1. Existing database connections may have become invalid.
2. The application connection pool may not automatically reconnect.
3. DNS caching may delay endpoint resolution changes.
4. Retry logic may be missing or incorrectly configured.
5. Check application logs and database connection errors.
6. Perform controlled failover testing to verify the complete recovery path.

---

## 9. Your Read Replica is 30 minutes behind. What do you investigate?

1. Confirm the replication-lag metric and whether the lag is increasing.
2. Check the primary's write volume.
3. Check replica CPU, memory, storage, and I/O pressure.
4. Investigate long-running operations and workload characteristics.
5. Determine whether the replica needs scaling or workload separation.
6. Ensure applications don't depend on the replica for strongly consistent reads.

---

## 10. Users don't see data immediately after creating an order. What is likely happening?

1. The write may have gone to the primary.
2. The subsequent read may have gone to a Read Replica.
3. Replication lag can cause stale reads.
4. Route read-after-write requests to the primary when consistency is required.
5. Monitor replica lag.
6. Review the application's read/write routing design.

---

## 11. Reporting queries are slowing down the production database. What do you do?

1. Identify the expensive reporting queries.
2. Optimize SQL and indexes where possible.
3. Consider moving reporting workloads to a Read Replica.
4. Route reporting traffic separately from transactional traffic.
5. Monitor production workload after moving the reports.
6. For very large analytics workloads, consider a dedicated analytics architecture.

---

## 12. Your application has 10,000 database connections. What do you investigate?

1. Check the configured database connection limit.
2. Examine application connection-pool settings.
3. Look for connection leaks.
4. Check for excessive idle or long-lived sessions.
5. Use connection pooling appropriately.
6. Don't simply increase the database instance size without understanding connection behavior.

---

## 13. RDS connection count suddenly doubles. What could cause it?

1. Increased application traffic.
2. A deployment that changed connection-pool settings.
3. Connection leaks.
4. New application instances or containers.
5. A failed connection-reuse strategy.
6. Investigate the source of connections before changing RDS capacity.

---

## 14. A single SQL query consumes most of the database workload. What do you do?

1. Identify the query through database performance monitoring.
2. Examine its execution plan.
3. Check indexes, joins, filters, and returned rows.
4. Determine whether application behavior is generating excessive executions.
5. Optimize or redesign the query.
6. Validate performance after the change.

---

## 15. A query became slow after a database upgrade. What do you check?

1. Compare execution plans before and after the upgrade.
2. Check optimizer behavior and database statistics.
3. Verify indexes and query compatibility.
4. Check parameter changes and engine-specific behavior.
5. Test the query against the previous and upgraded environments.
6. Apply an appropriate optimization or rollback strategy based on evidence.

---

## 16. An application deployment suddenly increases RDS load. What do you do?

1. Correlate database metrics with deployment time.
2. Identify new SQL statements.
3. Compare query volume and execution time.
4. Look for N+1 queries, missing indexes, or excessive transactions.
5. Roll back the deployment if it is causing an active production incident.
6. Fix and retest before redeploying.

---

## 17. A production table was accidentally deleted. How do you recover?

1. Stop or isolate further destructive activity.
2. Determine exactly when the deletion occurred.
3. Identify the appropriate recovery point.
4. Use PITR or another suitable backup recovery mechanism.
5. Restore into a separate environment where practical and validate the required data.
6. Implement safeguards against recurrence.

---

## 18. Someone accidentally deleted important rows. Would you restore the entire production database?

1. Not automatically.
2. First determine the exact affected data and recovery requirement.
3. Consider PITR to recover to a separate RDS instance.
4. Extract and validate the required data.
5. Carefully reintroduce the recovered data into production.
6. Avoid overwriting valid post-incident transactions unnecessarily.

---

## 19. A production RDS instance is accidentally deleted. What is your recovery plan?

1. Check whether automated backups/snapshots are available.
2. Identify the latest usable recovery point.
3. Restore the database into a new RDS instance.
4. Recreate required networking, security, parameter, and application configuration.
5. Validate data and application connectivity.
6. Redirect application traffic after recovery validation.

---

## 20. Your company requires an RPO of 5 minutes and RTO of 30 minutes. What do you design?

1. Start by documenting those requirements as business targets.
2. Configure a backup/replication strategy capable of supporting the required RPO.
3. Use HA architecture appropriate to the availability requirement.
4. Prepare the application and infrastructure recovery process for the 30-minute RTO.
5. Automate repetitive recovery steps where practical.
6. Perform regular recovery tests to prove the targets are achievable.

---

## 21. Your company wants protection from an AWS Region failure. Is Multi-AZ enough?

1. No.
2. Multi-AZ primarily protects against instance/AZ-level failures within the Region.
3. Regional failure requires a cross-Region DR strategy.
4. Consider supported cross-Region replication or backup/recovery approaches.
5. Prepare application, networking, secrets, and DNS recovery.
6. Test the complete regional failover process.

---

## 22. Your DR database exists in another Region, but the application cannot connect to it. Why?

1. Database recovery alone isn't sufficient.
2. Check VPC/network connectivity in the DR Region.
3. Verify Security Groups and routing.
4. Check DNS/application configuration.
5. Ensure credentials and secrets are available in the DR environment.
6. Include all dependencies in DR testing.

---

## 23. Your RDS database is running out of storage every month. What is the long-term solution?

1. Analyze the storage-growth pattern.
2. Identify large tables, indexes, logs, or application-generated data.
3. Determine whether growth is expected.
4. Configure appropriate storage capacity/autoscaling where supported.
5. Implement lifecycle/archive strategies where appropriate.
6. Establish alerts before storage becomes critical.

---

## 24. RDS is using very high IOPS. Is that automatically a problem?

1. No.
2. High IOPS may be normal for a high-throughput workload.
3. Check I/O latency and database response time.
4. Identify the queries generating the I/O.
5. Determine whether the workload is expected.
6. Optimize or scale only if performance or capacity requirements justify it.

---

## 25. RDS has high disk latency. What do you investigate?

1. Check storage type and provisioned performance.
2. Review workload I/O patterns.
3. Identify high-I/O SQL queries.
4. Check CPU, memory, and connection behavior.
5. Consider whether storage scaling or a different storage configuration is appropriate.
6. Validate improvements with actual latency measurements.

---

## 26. A Read Replica is overloaded by reporting queries. What do you do?

1. Identify which queries consume the replica resources.
2. Optimize the reporting queries.
3. Separate different reporting workloads if necessary.
4. Consider additional replicas for independent workloads.
5. Scale the replica when appropriate.
6. Ensure the reporting workload doesn't compromise replication health.

---

## 27. The primary database is write-heavy. Will Read Replicas solve the problem?

1. Not directly.
2. Read Replicas primarily reduce read workload.
3. Writes still go to the primary.
4. Identify whether the bottleneck is CPU, I/O, locking, or transaction volume.
5. Optimize write queries and transaction design.
6. Consider architectural changes if write scalability is the fundamental limitation.

---

## 28. Your application is receiving "too many connections." What is your approach?

1. Confirm the database connection limit and current usage.
2. Identify which application hosts are opening connections.
3. Inspect connection-pool configuration.
4. Find connection leaks or excessive idle connections.
5. Reduce unnecessary connections and use pooling correctly.
6. Scale the database only if capacity analysis supports it.

---

## 29. RDS is healthy but application latency increased after moving to another AZ. What do you check?

1. Check network path and latency.
2. Verify application and database subnet architecture.
3. Examine cross-AZ traffic.
4. Check application connection behavior.
5. Compare latency from different application instances.
6. Optimize placement and networking where the architecture permits.

---

## 30. A developer wants to modify production RDS parameters immediately. What is your approach?

1. Identify exactly which parameter needs changing.
2. Determine whether the parameter is dynamic or requires a restart.
3. Test the configuration in a lower environment.
4. Evaluate performance and compatibility impact.
5. Schedule the change appropriately if downtime/restart is required.
6. Keep the change documented and reversible where possible.

---

## 31. A minor RDS engine upgrade is available. Do you apply it immediately?

1. First determine the reason and urgency of the upgrade.
2. Review release notes and compatibility implications.
3. Test the upgrade in a non-production environment.
4. Verify application, drivers, extensions, and database parameters.
5. Schedule production maintenance appropriately.
6. Monitor closely after the upgrade.

---

## 32. A major database upgrade is required with minimal downtime. How do you approach it?

1. Build and test the target database version.
2. Identify application compatibility issues.
3. Use an appropriate migration/replication strategy.
4. Perform extensive validation before cutover.
5. Define a short production cutover and rollback plan.
6. Monitor application and database behavior after switching traffic.

---

## 33. You need to migrate a 5-TB production database with almost zero downtime. What approach?

1. Avoid relying solely on a traditional dump/restore.
2. Use an appropriate online migration/replication mechanism.
3. Perform the initial bulk load.
4. Continuously replicate changes where supported.
5. Validate source and target while replication is active.
6. Perform a controlled cutover once the target is synchronized.

---

## 34. Migration completed, but application performance is worse on RDS. Why?

1. Compare database instance sizing with the original environment.
2. Check storage configuration and I/O performance.
3. Compare query execution plans.
4. Review connection latency and application configuration.
5. Check database parameters and engine-version differences.
6. Optimize based on measurements rather than assuming RDS is inherently slower.

---

## 35. Your RDS database has no backups configured. What do you do?

1. Treat this as a production risk.
2. Determine the required RPO and retention period.
3. Configure appropriate automated backups.
4. Create appropriate snapshots where required.
5. Test restoration into a separate environment.
6. Document the backup and recovery procedure.

---

## 36. Your backups exist, but nobody has ever tested restoring them. Is that enough?

1. No.
2. A backup that hasn't been successfully restored is an unverified recovery assumption.
3. Perform periodic restore tests.
4. Validate application/database consistency after restoration.
5. Measure recovery time against the RTO.
6. Document the test results and improve the process.

---

## 37. Your RDS snapshot costs are growing rapidly. What do you investigate?

1. Review snapshot retention requirements.
2. Identify obsolete manual snapshots.
3. Check whether snapshots are being created unnecessarily.
4. Establish a lifecycle/retention policy.
5. Preserve snapshots required for compliance or DR.
6. Monitor storage-related costs after cleanup.

---

## 38. Your application stores database credentials in environment variables. Is that automatically secure?

1. Not necessarily.
2. Environment variables can still be exposed through logs, diagnostics, or operational access.
3. Prefer managed secret storage for sensitive production credentials.
4. Control who can retrieve the secret.
5. Rotate credentials appropriately.
6. Ensure applications don't log secrets accidentally.

---

## 39. A database password was leaked in a public GitHub repository. What do you do first?

1. Treat the credential as compromised immediately.
2. Rotate or revoke the database credential.
3. Determine whether the credential was used maliciously.
4. Review relevant audit and database activity.
5. Remove the secret from source-management workflows.
6. Implement secret scanning and secure secret management.

---

## 40. Security audit finds that developers can connect directly to production RDS. What do you change?

1. Review why direct access exists.
2. Remove unnecessary network access.
3. Restrict Security Groups to approved sources.
4. Separate application access from administrative access.
5. Implement controlled administrative access procedures.
6. Audit and monitor privileged database activity.

---

## 41. An application needs read-only access to production RDS. How do you implement it?

1. Create a dedicated database user/role.
2. Grant only required read permissions.
3. Do not give the application administrative privileges.
4. Store credentials securely.
5. Restrict network access through Security Groups.
6. Monitor usage and periodically review permissions.

---

## 42. A DBA needs temporary production access. What is the safest approach?

1. Grant only the required privileges.
2. Use controlled administrative access rather than broad permanent access.
3. Set an appropriate time limit where supported by your access system.
4. Audit the activity.
5. Remove access after the task is complete.
6. Avoid sharing permanent administrator credentials.

---

## 43. Your RDS database has high lock contention. What do you investigate?

1. Identify blocking and blocked sessions.
2. Find the transactions holding locks.
3. Check transaction duration.
4. Review application transaction boundaries.
5. Optimize queries/indexes and transaction behavior.
6. Confirm that the fix reduces contention without introducing data-integrity problems.

---

## 44. A transaction remains open for hours. Why is this dangerous?

1. Long transactions can hold locks.
2. They can block other operations.
3. They may increase resource consumption.
4. They can interfere with cleanup processes depending on the engine.
5. Application connection/transaction handling should be reviewed.
6. Establish monitoring for abnormal transaction duration.

---

## 45. Production RDS has intermittent timeouts but no obvious CPU spike. What do you investigate?

1. Check database waits and locks.
2. Review storage I/O latency.
3. Check connection saturation.
4. Investigate network latency and packet-related issues.
5. Examine application connection-pool behavior.
6. Correlate the timeout timestamps across application, database, and infrastructure logs.

---

## 46. You need to make an RDS database highly available. What architecture do you propose?

1. Use Multi-AZ where supported and appropriate.
2. Deploy RDS into a properly designed subnet group spanning Availability Zones.
3. Keep the database private.
4. Configure appropriate automated backups and PITR.
5. Add Read Replicas if read scaling is also required.
6. Test failover and application reconnection behavior.

---

## 47. Your company says "we need zero downtime." What questions do you ask?

1. Clarify what "zero downtime" actually means operationally.
2. Define the acceptable RTO and RPO.
3. Identify which operations must be online: failover, upgrades, migrations, etc.
4. Determine application retry and connection behavior.
5. Design HA/DR and deployment mechanisms accordingly.
6. Validate the claimed availability through realistic testing.

---

## 48. You are designing RDS for an e-commerce production system. What would you include?

1. Private RDS networking with restricted Security Groups.
2. Multi-AZ for high availability.
3. Automated backups/PITR and tested restore procedures.
4. Read Replicas if read traffic requires scaling.
5. Secrets management, encryption, monitoring, and alerting.
6. Documented DR, migration, failover, and incident-response procedures.

---

## 49. Your RDS architecture is becoming expensive. What do you optimize first?

1. Review DB instance utilization.
2. Identify oversized instances.
3. Review unused Read Replicas and databases.
4. Analyze storage and backup/snapshot usage.
5. Optimize expensive SQL workloads.
6. Make cost changes without compromising required availability and recovery objectives.

---

## 50. You are asked in an interview: "Design a production-ready Amazon RDS environment." What is your answer?

1. **Networking:** Deploy RDS in private subnets with tightly controlled Security Groups.
2. **Security:** Enable encryption, TLS where required, least-privilege DB users, and secure secrets management.
3. **HA:** Use Multi-AZ according to the availability requirement.
4. **Scaling:** Use vertical scaling and Read Replicas based on measured workload needs.
5. **Recovery:** Configure automated backups/PITR and a DR strategy aligned to RPO/RTO.
6. **Operations:** Implement CloudWatch/Performance Insights monitoring, alerting, upgrades, migration procedures, runbooks, and regular recovery/failover testing.

---

# 🧠 The 10 Scenarios You MUST Master

If you're preparing for **real-time production support or interviews**, make sure you can answer these without memorizing:

| #  | Scenario                | Core Skill                   |
| -- | ----------------------- | ---------------------------- |
| 1  | CPU 100%                | Performance troubleshooting  |
| 2  | Storage 95%             | Capacity management          |
| 3  | Connection failures     | Networking + Security Groups |
| 4  | Replica lag             | Replication                  |
| 5  | Stale reads             | Consistency                  |
| 6  | Multi-AZ failover       | HA                           |
| 7  | Accidental DELETE       | PITR/Recovery                |
| 8  | Regional outage         | DR                           |
| 9  | 5-TB migration          | DMS/online migration         |
| 10 | Production architecture | End-to-end RDS design        |

### 🔥 One-line mental model

**RDS Production = Secure Networking + HA + Backups/PITR + Scaling + Monitoring + Migration + DR + Tested Operations.**
