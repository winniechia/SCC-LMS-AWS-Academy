# Lab 06 — Amazon RDS Cheat Sheet

[Study index](../README.md) | [Full notes](../labs/lab-06-amazon-rds.md) | [Companion plan](../personal-labs/lab-06-rds-companion-plan.md)

## Recorded configuration

| Item | Lab value |
| --- | --- |
| Engine / deployment | MySQL / Single-AZ |
| Instance / database | `inventory-db` / `inventory` |
| Compute / storage | `db.t3.micro` / 20 GiB `gp2` |
| Network | Lab VPC, DB subnet group, `DB-SG` |
| Correct console path | Full Configuration → MySQL |

## Credentials ≠ database traffic

```text
Browser -- HTTP --> EC2 inventory application
EC2 app -- secret retrieval --> Secrets Manager
EC2 app -- direct MySQL connection --> RDS
```

> **Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

Secrets Manager provides connection information/credentials. It is not a database proxy or network path. Check **IAM authorization**, **network access**, and **database authentication** separately.

## Two actual troubleshooting lessons

1. **2026 console:** Express Configuration accidentally attempted Aurora PostgreSQL Serverless; AWS Academy denied `CreateDBCluster`. Correct path: **Full Configuration → MySQL**. Check the intended engine/operation before seeking broader permissions.
2. **Application URL:** HTTPS returned `ERR_CONNECTION_TIMED_OUT`; explicit **HTTP worked** and the inventory application test completed. This was an application/protocol issue, **not an RDS failure**. Exact HTTPS root cause was not established. Configure HTTPS appropriately for real deployments.

## SAA quick review

- RDS = managed relational database; MySQL is the engine.
- `inventory-db` = instance identifier; `inventory` = database name.
- Single-AZ has no Multi-AZ standby. A DB subnet group spanning AZs does not itself create a standby.
- Security group = network rules; IAM role = AWS permissions. An EC2 instance profile associates the role with the instance.
- MySQL normally uses **TCP 3306**; HTTP **80** and HTTPS **443** are browser/web-server connections.
- Secret retrieval does not establish database connectivity or replace database login.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

**EC2 app = shop worker; RDS = inventory filing room; Secrets Manager = key box; IAM role = badge; security group = guard; network = hallway.**

The worker takes the key from the box, then walks directly to the filing room. **拿鑰匙不是走廊；有權限不等於網路一定通。**

**Personal rebuild: 🟢 Yes — High Learning Value.** Plan only; nothing provisioned. Before completion, audit database/compute resources, snapshots/backups, secrets, and supporting resources for cleanup and retained cost.
