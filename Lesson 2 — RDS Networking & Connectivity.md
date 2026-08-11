# ☁️ Amazon RDS — Lesson 2/10

## RDS Networking & Connectivity

Lesson 2 is **strictly about how applications, administrators, and other AWS resources connect to Amazon RDS**.

The core idea is:

> **If you don't understand RDS networking, you cannot troubleshoot RDS connectivity, design secure production environments, or explain many real-world outages.**

---

# 1. RDS Networking — The Big Picture

A typical production architecture looks like this:

```text
                         AWS REGION
                             │
                           VPC
                             │
              ┌──────────────┴──────────────┐
              │                             │
        Application Tier               Database Tier
              │                             │
        Private Subnets              Private Subnets
              │                             │
              │                        ┌────▼────┐
              └───────────────────────►│   RDS   │
                                       └─────────┘
```

The application doesn't simply connect directly to "AWS."

It goes through several networking layers:

```text
Application
    ↓
DNS / RDS Endpoint
    ↓
VPC Routing
    ↓
Subnet
    ↓
Security Group
    ↓
RDS
    ↓
Database Port
```

If any layer is wrong, the connection can fail.

---

# 2. What is a VPC?

A **VPC (Virtual Private Cloud)** is your logically isolated network in AWS.

Think of it as:

```text
AWS
 │
 └── VPC
      │
      ├── Application resources
      │
      ├── Load balancers
      │
      ├── EC2
      │
      └── RDS
```

The VPC gives you control over:

* IP addressing
* Subnets
* Routing
* Security groups
* Network connectivity

RDS integrates with this VPC networking model.

---

# 3. What is a subnet?

A subnet is a range of IP addresses inside a VPC.

For example:

```text
VPC
10.0.0.0/16
    │
    ├── Subnet A
    │    10.0.1.0/24
    │
    ├── Subnet B
    │    10.0.2.0/24
    │
    └── Subnet C
         10.0.3.0/24
```

Subnets are associated with Availability Zones.

For example:

```text
AZ-A → 10.0.1.0/24
AZ-B → 10.0.2.0/24
AZ-C → 10.0.3.0/24
```

---

# 4. Public vs Private Subnets

This distinction is extremely important.

### Public subnet

A subnet with a route that provides a path to an Internet Gateway.

Conceptually:

```text
Subnet
  │
  ▼
Internet Gateway
  │
  ▼
Internet
```

### Private subnet

Doesn't have a direct route to the internet through an Internet Gateway.

For databases, private subnets are commonly preferred.

```text
Internet
    X
    │
    ▼
Private Subnet
    │
    ▼
RDS
```

---

# 5. Should RDS be placed in private subnets?

For a typical production application:

**Yes, preferably.**

A common architecture is:

```text
                 Internet
                    │
                    ▼
              Load Balancer
                    │
                    ▼
             Application Tier
                    │
             Private Network
                    │
                    ▼
                  RDS
```

The database doesn't need to be directly reachable from the public internet.

This reduces the attack surface.

---

# 6. What is a DB subnet group?

A **DB subnet group** tells RDS which subnets it can use.

Example:

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

For a Multi-AZ architecture, having subnets in multiple AZs is important.

---

# 7. Why do we need multiple subnets?

Suppose:

```text
RDS
 │
 └── Only AZ-A
```

You have less infrastructure isolation.

Instead:

```text
VPC
│
├── AZ-A → Private Subnet
│
├── AZ-B → Private Subnet
│
└── AZ-C → Private Subnet
```

This gives RDS more placement flexibility and is important for HA configurations.

---

# 8. What is an RDS endpoint?

Applications connect to RDS using a DNS endpoint.

Conceptually:

```text
Application
     │
     ▼
mydb.xxxxx.region.rds.amazonaws.com
     │
     ▼
RDS
```

The application doesn't need to know the underlying infrastructure IP address.

---

# 9. Why use a DNS endpoint instead of an IP address?

Because infrastructure can change.

For example:

```text
Before
Endpoint
   ↓
Instance A
```

After a relevant infrastructure event:

```text
Endpoint
   ↓
Instance B
```

The application continues using the endpoint.

This abstraction becomes especially important for **Multi-AZ failover**, which we'll study in Lesson 4.

---

# 10. What is DNS?

DNS translates a hostname into an address that the client can use.

Conceptually:

```text
mydb.example.rds.amazonaws.com
              │
              ▼
        DNS Resolution
              │
              ▼
        Database address
```

If DNS resolution fails, the application may not even reach the database.

---

# 11. What is a security group?

A security group acts as a virtual firewall associated with the RDS network interface.

Example:

```text
Application
   │
   │ TCP 5432
   ▼
RDS Security Group
   │
   ▼
RDS
```

The security group determines which network sources can reach the database.

---

# 12. Example security-group design

Suppose:

```text
Application Security Group
        = sg-app

RDS Security Group
        = sg-db
```

Instead of:

```text
sg-db
Allow TCP 5432
Source = 0.0.0.0/0   ❌
```

Prefer:

```text
sg-db
Allow TCP 5432
Source = sg-app      ✅
```

Meaning:

> Only resources associated with the application security group can access the database on that port.

This is much more precise.

---

# 13. What port does RDS use?

The port depends on the database engine.

Common defaults:

```text
PostgreSQL       → 5432
MySQL            → 3306
MariaDB          → 3306
SQL Server       → 1433
Oracle           → 1521
Db2              → engine-specific configuration
```

Don't memorize the numbers alone.

The important troubleshooting concept is:

```text
Application
    ↓
Correct endpoint
    ↓
Correct port
    ↓
Security group allows port
    ↓
RDS reachable
```

---

# 14. What happens if the security group doesn't allow the port?

Example:

```text
Application
    │
    │ TCP 5432
    ▼
RDS Security Group
    │
    X
    │
    ▼
RDS
```

The connection fails.

The database itself might be perfectly healthy.

This is why:

> **"I can't connect to RDS" does not automatically mean RDS is down.**

---

# 15. Connection troubleshooting flow

When an application cannot connect:

```text
1. Is RDS available?
        ↓
2. Does DNS resolve?
        ↓
3. Is routing correct?
        ↓
4. Is the subnet reachable?
        ↓
5. Does the security group allow access?
        ↓
6. Is the correct port being used?
        ↓
7. Is authentication correct?
        ↓
8. Is the database accepting connections?
```

This is one of the most important operational workflows in this lesson.

---

# 16. Network ACL vs Security Group

You should know the distinction.

### Security Group

```text
Instance/network interface level
Stateful
```

### Network ACL

```text
Subnet level
Stateless
```

Conceptually:

```text
VPC
 │
 ├── Network ACL
 │       ↓
 │    Subnet
 │       ↓
 │   Security Group
 │       ↓
 │      RDS
```

If troubleshooting connectivity, both can matter.

---

# 17. What does "stateful" mean for security groups?

A security group is stateful.

If an allowed connection is established, the return traffic is automatically handled according to the stateful behavior.

You don't generally need to create a separate return rule simply because traffic is returning.

---

# 18. What does "stateless" mean for Network ACLs?

Network ACL rules are evaluated independently for inbound and outbound traffic.

Therefore, both directions need appropriate rules.

This can make NACL troubleshooting more complicated than security-group troubleshooting.

---

# 19. RDS public accessibility

When configuring RDS, you'll encounter the concept of **public access**.

Conceptually:

```text
Publicly accessible RDS

Internet
   │
   ▼
Network path
   │
   ▼
RDS
```

Versus:

```text
Private RDS

Internet
   X
   │
   ▼
Private VPC
   │
   ▼
RDS
```

For production databases, private access is generally preferred unless there is a specific architecture requiring otherwise.

---

# 20. Does a public subnet automatically make RDS public?

No.

Subnet routing and RDS's public-access configuration are separate considerations.

You need to understand:

```text
Subnet
+
Routing
+
Public accessibility
+
Security groups
+
Network architecture
```

all together.

---

# 21. Can an EC2 instance connect to private RDS?

Yes.

A common architecture:

```text
VPC
│
├── Private App Subnet
│      │
│     EC2
│      │
│      └──────────┐
│                 │
└── Private DB Subnet
                  │
                 RDS
```

The security group on RDS must allow the EC2/application source on the appropriate database port.

---

# 22. Can Lambda connect to private RDS?

Yes, with appropriate VPC/network configuration.

Conceptually:

```text
Lambda
   │
   ▼
VPC Networking
   │
   ▼
Private Subnet
   │
   ▼
RDS
```

You still need:

* Appropriate networking
* Security groups
* Correct endpoint
* Correct port
* Appropriate database credentials

---

# 23. Can an application in another VPC connect to RDS?

Yes, but there must be network connectivity between the VPCs.

Possible architecture patterns include:

```text
VPC-A
Application
    │
    │
    ▼
Inter-VPC Connectivity
    │
    ▼
VPC-B
RDS
```

The exact solution depends on the AWS network architecture.

The important point:

> **Being in AWS doesn't automatically mean two resources can communicate.**

---

# 24. Can on-premises applications connect to RDS?

Yes.

For example:

```text
On-Premises
    │
    │ Secure AWS connectivity
    ▼
AWS VPC
    │
    ▼
Private RDS
```

The network architecture might involve dedicated or VPN-based connectivity.

You must also configure:

```text
Routing
Security groups
Network ACLs where applicable
DNS
Firewall rules
Database authentication
```

---

# 25. RDS connection layers

Think about connectivity as seven layers:

```text
              Application
                   │
             ┌─────▼─────┐
             │   DNS     │
             └─────┬─────┘
                   │
             ┌─────▼─────┐
             │  Routing  │
             └─────┬─────┘
                   │
             ┌─────▼─────┐
             │  Subnet   │
             └─────┬─────┘
                   │
             ┌─────▼─────┐
             │    NACL   │
             └─────┬─────┘
                   │
             ┌─────▼─────┐
             │    SG     │
             └─────┬─────┘
                   │
             ┌─────▼─────┐
             │    RDS    │
             └───────────┘
```

Plus:

```text
Correct port
Correct credentials
Correct database
```

---

# 26. Real-time Scenario #1 — "Connection timed out"

Suppose the application says:

```text
Connection timed out
```

Don't immediately investigate the username/password.

A timeout often points toward connectivity.

Check:

```text
DNS
 ↓
Route
 ↓
Subnet
 ↓
Security Group
 ↓
NACL
 ↓
Port
 ↓
RDS availability
```

Authentication failures usually behave differently from a pure network timeout.

---

# 27. Real-time Scenario #2 — "Connection refused"

Suppose:

```text
Connection refused
```

Potential areas to investigate include:

```text
Correct endpoint?
Correct port?
RDS status?
Database accepting connections?
Network path?
Security controls?
```

Again, don't jump straight to:

> "The database is down."

---

# 28. Real-time Scenario #3 — DNS doesn't resolve

Suppose:

```text
nslookup mydb.xxxxx.rds.amazonaws.com
```

fails.

Investigate:

```text
DNS configuration
VPC DNS settings
Resolver behavior
Network environment
Endpoint spelling
```

If DNS doesn't resolve, checking SQL permissions won't solve the problem.

---

# 29. Real-time Scenario #4 — Security group is wrong

You have:

```text
Application SG
    = sg-app

RDS SG
    = sg-db
```

But RDS has:

```text
TCP 5432
Source = sg-wrong
```

Result:

```text
Application
     │
     │ 5432
     ▼
   sg-db
     │
     X
```

Fix:

```text
sg-db
Allow TCP 5432
Source = sg-app
```

Then retest.

---

# 30. Real-time Scenario #5 — Application is in another VPC

You have:

```text
VPC-A
Application
```

and:

```text
VPC-B
RDS
```

The security group looks correct.

Still no connection.

Why?

Because security groups don't magically create network connectivity.

You need:

```text
VPC-A
   │
   ▼
Inter-VPC connectivity
   │
   ▼
VPC-B
   │
   ▼
RDS
```

Routing must work in both directions as required.

---

# 31. Real-time Scenario #6 — DBA can connect but application cannot

This is a classic production situation.

If:

```text
DBA → RDS ✅
Application → RDS ❌
```

then the database itself may be healthy.

Compare:

```text
DBA network path
        vs
Application network path
```

Check:

* Source security group
* Source subnet
* Routing
* DNS
* Port
* Credentials
* Database name

Don't assume because the DBA can connect that the application's network path is identical.

---

# 32. Real-time Scenario #7 — Application suddenly loses connectivity

Suppose it worked for six months and suddenly fails.

Investigate recent changes:

```text
Security-group change
Route change
NACL change
RDS maintenance
Application deployment
DNS/configuration change
Credential rotation
Network infrastructure change
```

A timeline is extremely valuable:

```text
Incident: 10:05
Change:   10:02
```

That correlation may immediately point toward the root cause.

---

# 33. Real-time Scenario #8 — RDS is healthy but application latency increased

This is not necessarily an RDS failure.

Check:

```text
Application
    ↓
Network latency
    ↓
Database connection establishment
    ↓
Query execution
    ↓
Database response
```

Measure each component.

The issue could be:

* Application CPU
* Network latency
* Connection pooling
* DNS
* SQL execution
* Database contention

---

# 34. Real-time Scenario #9 — Application connects from development but not production

Compare environments systematically:

```text
                 DEV          PROD
VPC                ?            ?
Subnet             ?            ?
Route               ?            ?
Security Group      ?            ?
NACL                ?            ?
DNS                 ?            ?
Endpoint            ?            ?
Port                ?            ?
Credentials         ?            ?
```

Don't randomly change production settings.

Find the difference.

---

# 35. Real-time Scenario #10 — Security team asks for zero internet exposure

A typical architecture:

```text
                    INTERNET
                       │
                       ▼
                 Load Balancer
                       │
                       ▼
                 Application
                       │
                  Private VPC
                       │
                       ▼
                     RDS
```

RDS remains private.

Access is controlled through:

```text
Security Groups
Routing
Authentication
Encryption
```

This is a common production pattern.

---

# 36. Connection troubleshooting command mindset

For Linux-based troubleshooting, you might conceptually test:

```text
DNS:
nslookup <RDS-endpoint>

Connectivity:
nc -vz <RDS-endpoint> <port>
```

For PostgreSQL:

```text
psql -h <RDS-endpoint> -p 5432 -U <user> -d <database>
```

For MySQL:

```text
mysql -h <RDS-endpoint> -P 3306 -u <user> -p
```

The idea is to test progressively:

```text
DNS
 ↓
TCP
 ↓
Database protocol
 ↓
Authentication
 ↓
SQL
```

---

# 37. The golden troubleshooting sequence

Memorize this:

```text
              RDS CONNECTION FAILURE
                       │
                       ▼
                1. RDS Status
                       │
                       ▼
                2. DNS Resolution
                       │
                       ▼
                3. Network Route
                       │
                       ▼
                4. Subnet
                       │
                       ▼
                5. NACL
                       │
                       ▼
                6. Security Group
                       │
                       ▼
                7. Port
                       │
                       ▼
                8. Authentication
                       │
                       ▼
                9. Database
                       │
                       ▼
                10. SQL/Application
```

This is a **production DBA troubleshooting pattern** worth memorizing.

---

# 38. Common networking mistakes

### Mistake #1

```text
Allow 0.0.0.0/0
```

Just because it makes connectivity work.

**Better:** restrict the source.

---

### Mistake #2

Assuming:

> "Same AWS account = automatically connected."

False.

Networking still matters.

---

### Mistake #3

Assuming:

> "Same VPC = automatically connected."

Not necessarily.

Security groups, routes, ports, and other controls still matter.

---

### Mistake #4

Troubleshooting SQL when the network is broken.

If TCP cannot connect:

```text
SQL tuning won't help.
```

---

### Mistake #5

Changing multiple network controls simultaneously.

That destroys your ability to identify the actual cause.

---

# 39. Interview Question — What is a DB subnet group?

A strong answer:

1. A DB subnet group is a collection of subnets designated for RDS.
2. It is associated with a VPC.
3. RDS uses these subnets when placing database resources.
4. Subnets should generally span multiple Availability Zones for appropriate HA designs.
5. Production databases are commonly placed in private subnets.
6. Network and security configuration must still permit required application connectivity.

---

# 40. Interview Question — How would you troubleshoot RDS connectivity?

Answer:

1. Check the RDS instance status and endpoint.
2. Verify DNS resolution from the application environment.
3. Check routing and subnet connectivity.
4. Verify NACL and security-group rules.
5. Confirm the correct database port.
6. Finally check authentication, database configuration, and application behavior.

---

# 41. Interview Question — Why should RDS usually be private?

1. Databases generally don't need direct internet exposure.
2. Private placement reduces attack surface.
3. Applications can access RDS through controlled private networking.
4. Security groups can restrict database access to application resources.
5. Administrative access can use controlled network paths.
6. This follows defense-in-depth principles.

---

# 42. Interview Question — Security Group vs NACL?

| Security Group                       | Network ACL                                   |
| ------------------------------------ | --------------------------------------------- |
| Resource/network-interface level     | Subnet level                                  |
| Stateful                             | Stateless                                     |
| Allows rules                         | Allows and denies rules                       |
| Commonly used for RDS access control | Additional subnet-level control               |
| Return traffic handled statefully    | Return traffic needs appropriate rules        |
| Very important for RDS connectivity  | Important when subnet-level filtering is used |

---

# 43. The complete RDS network architecture

Put everything together:

```text
                         AWS REGION
                             │
                             ▼
                            VPC
                             │
          ┌──────────────────┴──────────────────┐
          │                                     │
     App Subnets                           DB Subnets
          │                                     │
      Application                         DB Subnet Group
          │                                     │
          │                              ┌──────▼──────┐
          │                              │     RDS     │
          │                              └──────┬──────┘
          │                                     │
          └─────────────── Network ─────────────┘

                       Security Group
                             │
                       Network ACL
                             │
                          Routing
                             │
                            DNS
                             │
                         Endpoint
```

---

# 🧠 Lesson 2 — What You Must Remember

```text
1. RDS lives inside a VPC.
2. Subnets determine network placement.
3. DB subnet groups define eligible subnets.
4. Production RDS is commonly private.
5. Applications connect using an RDS endpoint.
6. Security groups control network access.
7. NACLs operate at subnet level.
8. The database port must be reachable.
9. DNS and routing are part of connectivity.
10. Authentication comes after network connectivity.
```

## 🔥 Golden Rule

> **When RDS connectivity fails, troubleshoot from the outside in: DNS → routing → subnet → NACL → security group → port → authentication → database → application.**

**Next: Lesson 3 — RDS Storage & Instance Sizing**
