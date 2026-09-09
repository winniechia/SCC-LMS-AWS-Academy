# Module 6 Knowledge Check — Cheat Sheet

[Study index](../README.md) | [Full study notes](../knowledge-checks/module-06-knowledge-check.md)

## Pick the database

| Clue | Service |
| --- | --- |
| Structured relational data, related tables, SQL/JOINs | **Amazon RDS** |
| Flexible attributes, serverless NoSQL, very high scale/low latency | **Amazon DynamoDB** |
| Historical analytics, aggregations, OLAP | **Amazon Redshift** |
| Connected relationships, graph traversal, fraud networks | **Amazon Neptune** |
| MySQL/PostgreSQL-compatible cloud relational engine | **Amazon Aurora** |

**OLTP = run the business now. OLAP = analyze lots of historical data.**

## ⚠️ Aurora course-question warning

The reviewed Knowledge Check contains an internally inconsistent item that mentions **Microsoft SQL Server** but marks **Aurora** correct while its explanation describes Aurora as MySQL/PostgreSQL compatible.

Do not memorize that pairing for SAA:

```text
Aurora -> MySQL / PostgreSQL compatible
Microsoft SQL Server -> Amazon RDS for SQL Server
```

## Security

```text
RDS      -> VPC + Security Group + Encryption
DynamoDB -> IAM + Encryption + VPC Gateway Endpoint
```

- DynamoDB is serverless; do not attach a Security Group to a DynamoDB table or imaginary DynamoDB instance.
- A DynamoDB **Gateway Endpoint** lets VPC resources reach DynamoDB privately without requiring Internet/NAT connectivity for that traffic.
- Relational table/row/column authorization belongs to the database permission model; do not treat IAM as a generic replacement for SQL grants.

## Migration

```text
SCT -> convert schema
DMS -> move data
```

- **Heterogeneous migration** such as Oracle -> Aurora PostgreSQL/MySQL: **SCT first, then DMS**.
- **Homogeneous migration** such as MySQL -> RDS MySQL: schema conversion is not the main problem; **DMS** is the primary migration service.

**SCT = translator / 翻譯員. DMS = moving truck / 搬家卡車.**

## Availability, scaling, and Region

| Problem | Solution |
| --- | --- |
| Primary AZ fails; need automatic failover | **Multi-AZ** |
| Too much read traffic | **Read Replica** |
| Need a replica in another Region | **Cross-Region Read Replica** |
| Need AZ failover + read scaling | **Multi-AZ + Read Replica** |
| Need AZ HA + another Region | **Multi-AZ + cross-Region replica strategy** |

**Failure problem -> Multi-AZ. Read problem -> Read Replica. Region problem -> cross-Region strategy.**

**故障 -> Multi-AZ；讀取壓力 -> Read Replica；跨 Region -> Cross-Region strategy。**

## EC2 vs managed database

A database installed on EC2 still leaves the **guest operating system** under customer responsibility, including OS patching. A managed database service such as RDS shifts more database infrastructure maintenance to AWS.

**EC2 gives control, and control brings responsibility. / EC2 給控制權，也帶來管理責任。**

## Weak spots repaired / 已修正觀念

- SCT vs DMS -> now separate **schema conversion** from **data movement**.
- DynamoDB security -> **Gateway Endpoint, not Security Group** for private service access.
- RDS vs Redshift -> **transactions vs analytics**.
- Multi-AZ vs Read Replica -> **failure vs read load**.
- Two requirements -> solve each requirement separately, then combine the architecture when needed.

## 🧒 3rd-grade memory

| Concept | Analogy |
| --- | --- |
| RDS | Organized related filing cabinets |
| DynamoDB | Very fast flexible boxes |
| Redshift | Study years of report cards |
| Neptune | Detective connection/string board |
| SCT | Translator |
| DMS | Moving truck |
| Multi-AZ | Backup classroom |
| Read Replica | Extra teacher for lots of readers |
| RDS Security Group | Door guard |
| DynamoDB Gateway Endpoint | Private door from VPC to DynamoDB |

## Final memory map / SAA recall

```text
RDS       -> relational / structured / SQL / JOIN
DynamoDB  -> NoSQL / flexible / serverless / low latency
Redshift  -> analytics / OLAP
Neptune   -> graph / relationships
Aurora    -> MySQL + PostgreSQL compatible

RDS security      -> VPC + SG + Encryption
DynamoDB security -> IAM + Encryption + Gateway Endpoint

SCT -> convert schema
DMS -> move data

Multi-AZ     -> HA / automatic failover
Read Replica -> read scaling
Cross-Region -> another Region / DR-read strategy
```
