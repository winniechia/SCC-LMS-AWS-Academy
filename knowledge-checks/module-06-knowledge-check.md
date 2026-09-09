# Module 6 Knowledge Check — Databases

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/module-06-knowledge-check-cheat-sheet.md)

## Study-session record

This completed study session reviewed AWS database selection, security, availability, and migration concepts from the Module 6 Knowledge Check. These notes summarize the tested concepts and the learning corrections from the review session rather than reproducing the quiz verbatim.

The main improvement areas were:

- RDS vs DynamoDB vs Redshift vs Neptune
- RDS security vs DynamoDB security
- AWS SCT vs AWS DMS
- Multi-AZ vs Read Replica vs cross-Region replica
- EC2 database responsibility vs managed database responsibility
- Aurora compatibility and one internally inconsistent course question

## 1. Database on EC2 vs managed database service

If a database runs directly on an EC2 instance, the customer remains responsible for the EC2 guest operating system and its patching. Running database software on EC2 does not make the operating system AWS-managed.

A managed database service such as Amazon RDS shifts more database infrastructure maintenance to AWS. In the Knowledge Check, the intended distinction was that AWS manages database patching for the managed database service, while AWS does not patch the guest OS of a customer-managed EC2 instance.

> **EC2 gives control, and control brings responsibility. / EC2 給你控制權，也把更多管理責任交給你。**

## 2. RDS vs DynamoDB

Choose **Amazon RDS** when the workload is strongly relational and structured:

- multiple related tables
- SQL and JOINs
- foreign keys
- structured relational data
- transactional relationships across related data

Choose **Amazon DynamoDB** when the workload fits a serverless NoSQL model:

- flexible or unpredictable attributes
- very large request volume
- low-latency key-value/document access
- no requirement for relational JOINs

Encryption, backups, and high availability do not by themselves distinguish RDS from DynamoDB because both services can provide those capabilities.

### Memory rule

```text
Structured + related tables + SQL/JOINs -> RDS
Flexible attributes + massive scale + low latency -> DynamoDB
```

## 3. OLTP vs OLAP: RDS/DynamoDB vs Redshift

**OLTP** is about running the business now: orders, shopping carts, inventory updates, and other live transactions.

**OLAP** is about analyzing large amounts of historical data: trends, reports, aggregations, and business intelligence.

Amazon Redshift is the analytics/OLAP choice in this module. It is not the right fit for a live transactional cash-register-style workload.

> **RDS/DynamoDB = run the store today. Redshift = study years of store history.**
>
> **RDS/DynamoDB = 今天做生意；Redshift = 分析多年歷史資料。**

## 4. Neptune: relationships are the data

Amazon Neptune is the graph database choice when the important question is how entities connect to one another.

Typical clue:

```text
Customer -> Credit Card -> Device -> IP Address -> Other Accounts
```

Fraud detection and connected-data analysis are strong graph-workload signals.

**RDS** can store related tables, but **Neptune** is selected when traversing and analyzing relationships themselves is central to the workload.

## 5. Aurora compatibility — course question inconsistency

The Module 6 Knowledge Check includes a question whose wording refers to **Microsoft SQL Server**, while the displayed correct answer is **Amazon Aurora**. The feedback for that same question describes Aurora in terms of **MySQL and PostgreSQL compatibility**.

Those pieces are internally inconsistent. Do **not** memorize the quiz pairing as a general AWS rule.

### Safe SAA study memory

```text
Aurora -> MySQL-compatible / PostgreSQL-compatible
Microsoft SQL Server -> Amazon RDS for SQL Server
```

> ⚠️ **Course Question Inconsistency / 課程題目不一致**
>
> Preserve the course result as part of the study record, but do not use “SQL Server -> Aurora” as an SAA memory rule.

## 6. RDS security

The Module 6 security model for RDS emphasizes:

- **Encryption** at rest and in transit
- **VPC** network placement/isolation
- **Security Groups** for network access

For MySQL/MariaDB, a common application-to-database pattern is:

```text
EC2 application SG -- TCP 3306 --> RDS DB security group
```

This connects directly to Challenge Lab 06, where the application security group was allowed to reach the RDS security group on TCP 3306.

IAM is important for AWS authorization, but relational table/row/column permissions are database-level permissions rather than IAM policies attached to individual relational rows or columns.

### Memory rule

> **RDS security = VPC + Security Group + Encryption**

## 7. DynamoDB security and private access

DynamoDB is serverless. There are no DynamoDB instances inside your VPC to which you attach Security Groups.

The Module 6 model emphasizes:

- **IAM policies** for authorization
- **Encryption** for data protection
- **VPC Gateway Endpoint** for private VPC access to DynamoDB without requiring an Internet Gateway or NAT Gateway for that traffic

### Memory rule

```text
RDS      -> Security Group
DynamoDB -> IAM + Encryption + Gateway Endpoint
```

This was the main repeated weak spot during the study session. After scenario practice, the distinction was answered correctly without hints.

## 8. AWS SCT vs AWS DMS

For a heterogeneous migration, the two tools solve different problems:

- **AWS Schema Conversion Tool (SCT)** converts database schema/code objects for a different target engine.
- **AWS Database Migration Service (DMS)** moves the actual data and can support ongoing replication/change migration scenarios.

Example:

```text
Oracle -> Aurora PostgreSQL

1. SCT  -> convert schema
2. DMS  -> move data
```

For a homogeneous migration such as MySQL on EC2 -> RDS for MySQL, schema conversion is not the main problem, so DMS is the primary migration service.

> **SCT = translator. DMS = moving truck. / SCT = 翻譯架構；DMS = 搬資料。**

This was initially confused, then repeatedly answered correctly during review.

## 9. Multi-AZ vs Read Replica vs cross-Region replica

These solve different problems:

| Requirement | Best concept |
| --- | --- |
| Automatic failover if an AZ fails | RDS Multi-AZ |
| Reduce heavy read traffic | Read Replica |
| Copy in another Region for regional DR/read use | Cross-Region Read Replica |
| Need AZ HA and cross-Region copy | Multi-AZ + cross-Region replica |
| Need AZ HA and read scaling | Multi-AZ + Read Replica |

### Memory rule

> **Failure problem -> Multi-AZ. Read problem -> Read Replica. Region problem -> cross-Region strategy.**
>
> **故障問題 -> Multi-AZ；讀取壓力 -> Read Replica；跨 Region -> Cross-Region strategy。**

## Weak Spots Repaired / 已修正觀念

| Earlier confusion | Corrected distinction |
| --- | --- |
| SCT vs DMS | SCT converts schema; DMS moves data |
| DynamoDB security | IAM + encryption + VPC Gateway Endpoint; no SG attached to DynamoDB tables |
| RDS vs Redshift | RDS runs relational transactions; Redshift analyzes historical data |
| Multi-AZ vs Read Replica | Multi-AZ = HA/failover; Read Replica = read scaling |
| “Both requirements” scenarios | Map each requirement separately, then combine services when both are required |
| Aurora / SQL Server course question | Treat as an internal quiz inconsistency; Aurora study rule is MySQL/PostgreSQL compatibility |

These are learning corrections from the study session, not a newly invented official quiz score.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| AWS concept | Simple analogy |
| --- | --- |
| RDS | Organized filing cabinets with tables that relate to each other |
| DynamoDB | Fast boxes where each box may have different compartments |
| Redshift | A teacher studying years of report cards to find trends |
| Neptune | A detective wall with strings showing who is connected to whom |
| Multi-AZ | A backup classroom ready if the main classroom closes |
| Read Replica | Another teacher helping answer lots of reading questions |
| Cross-Region replica | A copy stored at another school far away |
| SCT | Translator who changes the forms into the new language |
| DMS | Moving truck that carries the actual boxes/data |
| Security Group | Door guard for an RDS house |
| DynamoDB Gateway Endpoint | Private door from the VPC to the DynamoDB service |

### Three short memory stories

**Database choice:** relational tables and JOINs -> RDS; flexible fast NoSQL -> DynamoDB; historical analytics -> Redshift; connection networks -> Neptune.

**Migration:** different database language -> SCT translates first; DMS then moves the data.

**Availability/performance:** main database might fail -> Multi-AZ; too many readers -> Read Replica.

## SAA exam takeaways

- Do not confuse a database running on EC2 with a managed database service.
- Pick the service based on the data model and access pattern, not generic features such as backups.
- **RDS:** relational, structured, SQL/JOINs.
- **DynamoDB:** serverless NoSQL, flexible attributes, very high scale/low latency.
- **Redshift:** OLAP/analytics, not the live transaction engine.
- **Neptune:** graph relationships/traversals.
- **Aurora:** MySQL/PostgreSQL-compatible; do not memorize the course's SQL Server mismatch.
- **RDS security:** VPC + SG + encryption.
- **DynamoDB security/access:** IAM + encryption + Gateway Endpoint.
- **SCT:** schema conversion. **DMS:** data migration.
- **Multi-AZ:** HA/failover. **Read Replica:** read scaling. Cross-Region replicas address cross-Region needs.
- When a question has two requirements, solve each requirement independently before choosing the combined architecture.

## Final memory map

```text
RDS       -> relational / structured / SQL / JOIN
DynamoDB  -> NoSQL / flexible / serverless / low latency
Redshift  -> analytics / OLAP / historical trends
Neptune   -> graph / connected relationships
Aurora    -> MySQL + PostgreSQL compatible

RDS security      -> VPC + SG + Encryption
DynamoDB security -> IAM + Encryption + Gateway Endpoint

SCT -> convert schema
DMS -> move data

Multi-AZ         -> HA / automatic failover
Read Replica     -> read scaling
Cross-Region RR  -> another Region / DR-read strategy
```
