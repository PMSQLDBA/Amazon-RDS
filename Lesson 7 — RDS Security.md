# ☁️ Amazon RDS — Lesson 7/10

## RDS Security: Network, Encryption, IAM & Secrets

Lesson 7 is strictly about **Amazon RDS security**.

The most important mental model:

> **RDS security is layered: network access + authentication + authorization + encryption + secrets management + auditing.**

---

# 1. The RDS Security Model

Think about security as multiple layers:

```text
                 Application
                      │
                      ▼
              ┌───────────────┐
              │ Authentication│
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ Authorization │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ Network/VPC   │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ RDS Database  │
              └───────────────┘
```

And around the database:

```text
Encryption at rest
Encryption in transit
Secrets management
Auditing
Monitoring
```

No single control provides complete security.

---

# 2. Where Does RDS Run?

RDS DB instances run inside an **Amazon VPC**.

Conceptually:

```text
AWS Region
   │
   ▼
   VPC
   │
   ├── Subnet
   │
   ├── Security Group
   │
   └── RDS
```

This means your database participates in your VPC networking architecture.

---

# 3. Public vs Private Database

For most production databases, the preferred architecture is generally:

```text
Internet
   │
   X
   │
   ▼
Private RDS
   ▲
   │
Application
```

Rather than:

```text
Internet
   │
   ▼
Public RDS
```

A database usually should not be directly exposed to the public internet unless there is a very specific, justified requirement.

---

# 4. Typical Production Architecture

A common pattern:

```text
                 Internet
                    │
                    ▼
             Load Balancer
                    │
                    ▼
              Application
               Subnets
                    │
                    ▼
              Private RDS
               Subnets
                    │
                    ▼
             RDS Database
```

The application can reach the database.

The internet cannot directly reach the database.

---

# 5. What Is a DB Subnet Group?

A DB subnet group tells RDS which subnets it can use for database deployment.

Conceptually:

```text
VPC
 │
 ├── Private Subnet AZ-A ──┐
 │                         │
 └── Private Subnet AZ-B ──┤
                           ▼
                    DB Subnet Group
                           │
                           ▼
                          RDS
```

For Multi-AZ architectures, subnets in multiple Availability Zones are important.

---

# 6. Why Multiple Subnets?

Suppose:

```text
AZ-A
Private Subnet A
     │
     ▼
Primary
```

and:

```text
AZ-B
Private Subnet B
     │
     ▼
Standby
```

This supports the Multi-AZ architecture.

Therefore:

> **A proper DB subnet group should be designed with the required Availability Zones in mind.**

---

# 7. What Is a Security Group?

A Security Group controls network traffic to and from the RDS network interface.

Conceptually:

```text
Application
    │
    │ TCP 5432
    ▼
Security Group
    │
    ▼
RDS
```

For PostgreSQL:

```text
TCP 5432
```

is the default PostgreSQL port.

For MySQL:

```text
TCP 3306
```

is the default MySQL port.

---

# 8. Security Group Best Practice

Bad:

```text
Inbound
0.0.0.0/0
TCP 5432
```

This potentially allows database access from anywhere on IPv4.

Better:

```text
Inbound
Source = Application Security Group
Port = 5432
```

Conceptually:

```text
App-SG
  │
  │ 5432
  ▼
RDS-SG
```

This is much more precise.

---

# 9. Security Group Referencing

Instead of allowing:

```text
10.0.0.0/16
```

you can often allow traffic from the application's Security Group.

Example:

```text
RDS-SG inbound:

Source:
Application-SG

Port:
5432
```

This means:

> Only resources associated with the specified application security group can reach the database on the allowed port.

This is much cleaner than maintaining large IP allowlists.

---

# 10. Security Group Is Not Database Authentication

This is extremely important.

A Security Group answers:

> **Can network traffic reach the database?**

Database authentication answers:

> **Who are you?**

Example:

```text
Security Group
     │
     ▼
Network access allowed
     │
     ▼
PostgreSQL
     │
     ▼
Username + password
     │
     ▼
Authentication
```

Passing the Security Group doesn't automatically grant database access.

---

# 11. Three Different Security Questions

Always separate:

### Network

```text
Can I reach the database?
```

### Authentication

```text
Who am I?
```

### Authorization

```text
What am I allowed to do?
```

Example:

```text
Network
   ↓
Security Group

Authentication
   ↓
Database credentials / supported IAM authentication

Authorization
   ↓
Database roles/privileges
```

---

# 12. PostgreSQL Example

Suppose your application connects:

```text id="8z7g3x"
Host: RDS endpoint
Port: 5432
Database: orders
Username: app_user
Password: ********
```

There are multiple security checks:

```text
VPC routing
    ↓
Security Group
    ↓
TLS
    ↓
PostgreSQL authentication
    ↓
PostgreSQL authorization
```

Every layer matters.

---

# 13. Encryption at Rest

Encryption at rest protects stored database data.

Conceptually:

```text
Application
     │
     ▼
Encrypted RDS Storage
     │
     ▼
Encrypted Backups
```

AWS KMS is used for key management in encrypted RDS configurations.

---

# 14. Why Encrypt at Rest?

Without encryption, someone who gains inappropriate access to underlying storage could potentially access stored data.

Encryption provides an additional protection layer.

It is particularly important for:

```text
Customer data
Financial data
Credentials
Personally identifiable information
Regulated workloads
Business-sensitive information
```

---

# 15. KMS

**AWS Key Management Service (KMS)** manages cryptographic keys used by AWS services.

Conceptually:

```text
             AWS KMS
                │
                ▼
             KMS Key
                │
                ▼
          RDS Encryption
```

You need to understand:

```text
Encryption
      ≠
Authorization
```

Encryption protects data cryptographically.

IAM controls who can perform AWS API actions involving resources.

---

# 16. Encryption at Rest vs In Transit

Two different problems:

### At rest

```text
Data stored
    ↓
Encryption
```

### In transit

```text
Application
    │
    │ encrypted connection
    ▼
RDS
```

You generally want both.

---

# 17. TLS / SSL for RDS

For database connections, TLS can encrypt traffic between:

```text
Application
     │
     │ TLS
     ▼
RDS
```

This protects credentials and database traffic from being exposed in transit.

---

# 18. Why TLS Matters

Without encryption in transit, network traffic may be exposed to inappropriate observers depending on the network path and threat model.

With TLS:

```text
Username
Password
SQL
Results
```

are protected during transmission.

---

# 19. Certificate Validation

TLS isn't only about encryption.

Proper clients should also validate the server certificate.

Conceptually:

```text
Client
  │
  ├── Establish TLS
  │
  ├── Validate certificate
  │
  └── Secure DB session
```

Skipping certificate verification can weaken the security guarantees of TLS.

---

# 20. Secrets Management

Where should you store:

```text
DB username
DB password
Connection credentials
```

Avoid:

```text
Source code
Git repository
Docker image
Plain-text config file
Hardcoded environment values where avoidable
```

A managed secrets solution such as **AWS Secrets Manager** is commonly used.

---

# 21. Secrets Manager Architecture

Conceptually:

```text
Application
     │
     ▼
Secrets Manager
     │
     ▼
DB Credentials
     │
     ▼
RDS
```

The application retrieves the secret rather than embedding the password directly in code.

---

# 22. Why Secrets Manager?

Benefits include:

```text
Centralized secret storage
Controlled access
Encryption
Rotation capabilities
Auditing
Reduced hardcoded credentials
```

This is especially useful in production.

---

# 23. Secrets Rotation

Instead of:

```text
Password
   │
   └── Never changes
```

you can design:

```text
Secret
  │
  ▼
Rotation
  │
  ▼
New credential
  │
  ▼
Application retrieves updated secret
```

But rotation must be tested carefully.

A poorly implemented rotation process can cause authentication failures.

---

# 24. IAM vs Database Users

Another common confusion.

### IAM

Controls access to AWS resources/APIs.

Example:

```text
Can this AWS identity modify an RDS instance?
```

### Database user

Controls access inside the database.

Example:

```text
Can app_user SELECT from orders?
Can app_user DELETE from orders?
```

Think:

```text
AWS layer
    ↓
IAM

Database layer
    ↓
DB roles/privileges
```

---

# 25. IAM Authentication for RDS

For supported RDS engines/configurations, IAM database authentication can allow clients to authenticate to the database using AWS IAM credentials rather than a long-lived database password.

Conceptually:

```text
Application
    │
    ▼
IAM
    │
    ▼
Authentication token
    │
    ▼
RDS
```

This can reduce reliance on long-lived passwords.

---

# 26. IAM Authentication Is Not IAM Authorization Inside PostgreSQL

Be precise.

IAM can help authenticate the connection.

But PostgreSQL still has database-level privileges and roles.

Conceptually:

```text
IAM authentication
       ↓
Connection established
       ↓
PostgreSQL role
       ↓
Database privileges
```

Don't say:

> "IAM automatically controls every SQL permission."

It doesn't.

---

# 27. Least Privilege

This is one of the most important security principles.

Don't give an application:

```text
SUPERUSER
```

unless absolutely necessary.

Instead:

```text
app_user
   │
   ├── SELECT
   ├── INSERT
   ├── UPDATE
   └── limited EXECUTE
```

only as required.

---

# 28. Example of Bad Permissions

Suppose the application only needs:

```sql id="9kh5xq"
SELECT
INSERT
UPDATE
```

but you give it:

```text
ALL PRIVILEGES
```

That's excessive.

If the application is compromised, the attacker inherits those excessive permissions.

---

# 29. Database Roles

For PostgreSQL, use roles to separate responsibilities.

Conceptually:

```text
readonly_role
      │
      └── SELECT

app_role
      │
      ├── SELECT
      ├── INSERT
      └── UPDATE

admin_role
      │
      └── Administrative privileges
```

This is cleaner than giving every application the same powerful account.

---

# 30. Separate Application and Admin Credentials

Don't use one account for everything.

Bad:

```text
Application
    │
    ▼
admin_user
```

Better:

```text
Application
    │
    ▼
app_user

DBA/Admin
    │
    ▼
admin_role
```

If the application is compromised, the blast radius is smaller.

---

# 31. Network Architecture Example

A strong production architecture:

```text
                     Internet
                        │
                        ▼
                 Load Balancer
                        │
                        ▼
                Application Tier
                 /            \
                /              \
           App-SG              App-SG
                \              /
                 \            /
                  ▼          ▼
                    RDS-SG
                      │
                      ▼
                 Private RDS
```

Security Group rule:

```text
RDS-SG
Inbound:
TCP 5432
Source:
App-SG
```

Not:

```text
0.0.0.0/0
```

---

# 32. Real-Time Scenario #1 — Application Can't Connect

Application reports:

```text
Connection timeout
```

Don't immediately change the database password.

Troubleshoot layer by layer:

```text
1. DNS
2. Route
3. Network ACL if relevant
4. Security Group
5. RDS availability
6. Port
7. TLS
8. Authentication
9. Database permissions
```

This prevents random troubleshooting.

---

# 33. Real-Time Scenario #2 — "Connection Refused"

Suppose:

```text
Application
    │
    ▼
RDS:5432
    │
    ❌
```

Check:

```text
RDS status
Security Group
Network path
Correct endpoint
Correct port
Subnet/routing
Database listener behavior
```

A connection problem is not automatically a credential problem.

---

# 34. Real-Time Scenario #3 — Authentication Failed

Suppose network connectivity works:

```text
Application
   │
   ▼
RDS
   │
   ▼
Authentication failed
```

Now investigate:

```text
Username
Password/token
Authentication mechanism
Secret value
Database user
Authentication configuration
```

The Security Group may already be working perfectly.

---

# 35. Real-Time Scenario #4 — Application Has Too Much Access

Security review discovers:

```text
app_user = excessive privileges
```

Fix:

```text
Create appropriate role
     ↓
Grant minimum permissions
     ↓
Remove unnecessary privileges
     ↓
Test application
```

This is least privilege.

---

# 36. Real-Time Scenario #5 — Database Is Publicly Accessible

Security audit:

```text
RDS Publicly Accessible = Yes
```

Don't automatically flip it without understanding dependencies.

First determine:

```text
Who connects?
From where?
Why is public access required?
Can the application use private networking?
Are there external integrations?
```

Then redesign where possible:

```text
Public
  ↓
Private RDS
  ↓
Application/VPN/Private connectivity
```

---

# 37. Real-Time Scenario #6 — Password Found in Git

Developer commits:

```text
DB_PASSWORD=MySecret123
```

Immediate concern:

```text
Credential exposure
```

Do not simply delete the line and assume the password is safe.

A proper response may include:

```text
1. Rotate credential
2. Remove secret from active configuration
3. Move secret to managed storage
4. Investigate repository exposure
5. Review access/audit logs
6. Prevent recurrence with secret scanning
```

---

# 38. Real-Time Scenario #7 — Need Encrypted Database

Requirement:

> "All customer data must be encrypted at rest."

Architecture:

```text
RDS
 │
 ▼
Encryption enabled
 │
 ▼
KMS key
```

Then verify:

```text
Backups
Snapshots
Access policies
Key permissions
```

Encryption must be part of the complete lifecycle.

---

# 39. Real-Time Scenario #8 — Need Secure DB Traffic

Requirement:

> "Database traffic must be encrypted."

Architecture:

```text
Application
     │
     │ TLS
     ▼
RDS
```

Configure the database/client appropriately and validate certificate handling.

---

# 40. Real-Time Scenario #9 — Developer Wants Admin Password

Don't automatically provide production admin credentials.

Instead ask:

```text
What task?
What permission?
How long?
Can a temporary role solve it?
Can a restricted database role solve it?
```

This follows least privilege.

---

# 41. Real-Time Scenario #10 — Security Group Is Open to the World

Found:

```text
TCP 5432
Source = 0.0.0.0/0
```

Preferred remediation:

```text
Remove public database access
        ↓
Restrict source
        ↓
Use Application Security Group
        ↓
Allow only TCP 5432
```

The exact remediation should account for legitimate external clients.

---

# 42. Security Layers — Memorize This

```text
                 RDS SECURITY
                      │
       ┌──────────────┼──────────────┐
       │              │              │
     Network       Identity       Data
       │              │              │
       ▼              ▼              ▼
 Security Group     IAM          Encryption
 VPC/Subnets      DB Roles       TLS
 Routing          Secrets        KMS
```

Then:

```text
Auditing
Monitoring
Logging
```

around the entire system.

---

# 43. Parameter Groups and Security

Parameter groups control database engine configuration.

Some parameters affect security-related behavior, such as:

```text
Authentication settings
Logging behavior
Connection/security configuration
```

Don't modify production parameters blindly.

Test changes first.

---

# 44. Option Groups

For engines/configurations that use option groups, options can add database functionality.

Security implications depend on the option being enabled.

Always evaluate:

```text
What does this option do?
What network access does it require?
What permissions does it require?
Does it expose additional functionality?
```

---

# 45. Auditing

Security isn't complete without knowing:

> **Who did what and when?**

Consider:

```text
AWS CloudTrail
RDS events
Database audit/logging capabilities
IAM activity
Secret access
Security Group changes
```

This helps with:

```text
Incident response
Compliance
Troubleshooting
Forensics
Accountability
```

---

# 46. Security Monitoring

Monitor for things such as:

```text
Unexpected security-group changes
Public accessibility
Authentication failures
Unusual database activity
Secret access anomalies
Unexpected RDS modifications
Configuration drift
```

A secure architecture must also be observable.

---

# 47. RDS Security Checklist

Before production:

```text
☐ RDS is private where appropriate
☐ DB subnet group uses appropriate private subnets
☐ Security Group restricted
☐ No unnecessary 0.0.0.0/0 access
☐ Application SG used where appropriate
☐ TLS enabled/required as appropriate
☐ Encryption at rest enabled
☐ KMS permissions reviewed
☐ Credentials stored securely
☐ Secrets rotation strategy considered
☐ IAM permissions follow least privilege
☐ DB roles follow least privilege
☐ Admin and application accounts separated
☐ Logging/auditing configured
☐ Security changes monitored
```

---

# 48. Interview Question — How do you secure RDS?

Strong answer:

1. Place RDS in private subnets where appropriate.
2. Restrict network access using Security Groups.
3. Allow database ports only from required application sources.
4. Encrypt data at rest using RDS encryption/KMS.
5. Encrypt connections in transit using TLS.
6. Store credentials securely using a managed secret solution.
7. Apply least privilege at both IAM and database levels.
8. Monitor and audit administrative and database activity.

---

# 49. Interview Question — Security Group vs IAM?

### Security Group

Controls:

```text
Network traffic
```

### IAM

Controls:

```text
AWS API/resource access
```

Example:

```text
Security Group
→ Can this server reach RDS?

IAM
→ Can this identity modify the RDS resource?
```

They solve different problems.

---

# 50. Interview Question — Encryption at Rest vs In Transit?

Answer:

```text
At rest
→ Protects stored database data.

In transit
→ Protects data moving between client and RDS.
```

For a strong production posture, consider both.

---

# 51. Interview Question — Why shouldn't you store passwords in code?

Because:

```text
Git history
CI/CD logs
Build artifacts
Container images
Developer machines
Code backups
```

can expose credentials.

Use a managed secrets solution and rotate compromised credentials.

---

# 52. Interview Question — How would you secure PostgreSQL application access?

A strong answer:

```text
Private RDS
   ↓
RDS Security Group
   ↓
Only application SG → TCP 5432
   ↓
TLS
   ↓
Application-specific DB role
   ↓
Least-privilege permissions
   ↓
Credentials in Secrets Manager
   ↓
Monitoring/auditing
```

That's a production-quality answer.

---

# 53. Complete Secure RDS Architecture

Memorize this:

```text
                         INTERNET
                            │
                            X
                            │
                    ┌───────▼────────┐
                    │ Load Balancer  │
                    └───────┬────────┘
                            │
                            ▼
                    Application Tier
                            │
                         App-SG
                            │
                       TCP 5432
                            │
                         RDS-SG
                            │
                            ▼
                    Private RDS
                     /          \
                    /            \
                 AZ-A            AZ-B
               Primary          Standby

        ┌─────────────────────────────────────┐
        │ Encryption at Rest → KMS             │
        │ TLS → Encryption in Transit          │
        │ Secrets → Secrets Manager             │
        │ IAM → AWS authorization               │
        │ DB Roles → Database authorization    │
        │ CloudTrail/Logs → Auditing            │
        └─────────────────────────────────────┘
```

---

# 🧠 Lesson 7 — Must Remember

```text
1. Put production RDS in private networking where appropriate.
2. Security Groups control network access.
3. IAM controls AWS resource/API permissions.
4. Database roles control database permissions.
5. Encryption at rest protects stored data.
6. TLS protects data in transit.
7. KMS manages encryption keys.
8. Store database secrets securely.
9. Use least privilege everywhere.
10. Monitor and audit security activity.
```

# 🔥 Golden Rule

> **Security Group answers "Can you reach RDS?", authentication answers "Who are you?", and database authorization answers "What can you do?"**

---

## 🎯 Lesson 7 Complete

### Next: **Lesson 8 — RDS Monitoring, Performance & Troubleshooting**

We'll cover **CloudWatch metrics, Enhanced Monitoring, Performance Insights, CPU/memory/I/O, connections, locks, slow queries, storage pressure, latency, alarms, and real-time production troubleshooting scenarios.**
