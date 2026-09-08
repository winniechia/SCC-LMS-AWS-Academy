# Lab 06 — Creating an Amazon RDS Database

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-06-amazon-rds-cheat-sheet.md) | [Personal companion plan](../personal-labs/lab-06-rds-companion-plan.md)

## Purpose and completed lab record

AWS Academy Guided Lab: **Creating an Amazon RDS Database**. We created an RDS MySQL database for an EC2-hosted inventory application and tested the application. The lab record includes two troubleshooting incidents: an unintended engine/deployment choice in the console and an HTTPS timeout at the application.

These notes preserve the supplied settings and outcomes. Engine version, exact subnet group name, secret name/format, SQL commands, and specific inventory records were not supplied. They are not reconstructed. No password, secret value, account ID, role ARN, temporary public address, EC2 instance ID, or exact RDS endpoint is included.

## 1. Recorded database configuration

| Setting | Completed lab selection |
| --- | --- |
| Console path | Full Configuration → MySQL |
| Engine | MySQL |
| Deployment | Single-AZ |
| DB instance identifier | `inventory-db` |
| DB instance class | `db.t3.micro` |
| Allocated storage | 20 GiB |
| Storage type | General Purpose SSD (`gp2`) |
| VPC | Lab VPC |
| Subnet selection | DB subnet group in the lab VPC; exact name omitted |
| Database security group | `DB-SG` |
| Database name | `inventory` |

**`inventory-db` identifies the RDS instance; `inventory` is the database inside MySQL.** 主機識別名稱與資料庫名稱不同。

RDS manages the database infrastructure, while we remain responsible for application access, database users, configuration choices, and data. `db.t3.micro` describes compute capacity; 20 GiB `gp2` describes storage. These are recorded lab settings, not a promise of free usage or a default recommendation for every workload.

**Single-AZ** means this deployment does not have a Multi-AZ standby for automatic Availability Zone failover. A DB subnet group lists subnets RDS may use; for a standard regional RDS deployment it spans at least two Availability Zones even when the instance is Single-AZ. A subnet group spanning AZs does not itself enable Multi-AZ. See [AWS RDS VPC guidance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.Scenarios.html).

## 2. Architecture: credentials and database traffic are separate

```text
Browser -- HTTP --> EC2 inventory application

EC2 application -- AWS API / HTTPS --> Secrets Manager
                 retrieves connection information / credentials

EC2 application -- MySQL connection --> RDS MySQL
                 direct database traffic (normally TCP 3306)
```

> **Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

The application retrieves connection information/credentials from Secrets Manager, then connects **directly to RDS**. Queries and inventory data do not pass through Secrets Manager. It is neither the network path nor a database proxy. The arrows describe separate interactions, not a chain that forwards database traffic. See [AWS's retrieve-then-connect example](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets_jdbc.html); the example illustrates the concept, not this lab's programming language.

Three separate checks matter:

| Layer | What must work |
| --- | --- |
| AWS authorization | The application needs permission to retrieve the intended secret, typically through an EC2 IAM role/instance profile |
| Network access | The client needs a network path to RDS and database-port access allowed by security groups |
| Database authentication | The supplied database credentials and database name must be valid |

**Security Group = network access. IAM Role = AWS permissions.** Secret-read permission does not open the database port, and an open port does not grant secret-read permission. Database login is another check; retrieving a password is not the same as using IAM database authentication.

`DB-SG` is the recorded database security group. MySQL normally uses **TCP 3306**; a scoped design allows that inbound traffic from the application security group, with corresponding client outbound access. The exact lab rule source was not supplied, so this is explanatory guidance rather than a claimed console setting. See [RDS security groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html).

## 3. Troubleshooting incident: 2026 console configuration choices

The first RDS console screen in this lab offered **Express Configuration** and **Full Configuration**. Choosing Express Configuration accidentally attempted **Aurora PostgreSQL Serverless**. AWS Academy denied the **`CreateDBCluster`** action.

| Observation | Interpretation / correction |
| --- | --- |
| Unexpected Aurora PostgreSQL Serverless attempt | The selected configuration did not match the intended RDS MySQL lab |
| `CreateDBCluster` denied | The classroom's controlled permissions blocked that attempted operation |
| Correct lab path | **Full Configuration → MySQL**, then the recorded settings above |

This records the UI and outcome observed in the 2026 lab. It does not claim Express Configuration always chooses that engine for all accounts. Console labels and defaults can vary. We corrected the lab path; broadening classroom IAM permissions was not the solution.

**先核對設定模式與資料庫引擎，再判斷是否需要權限。** An access-denied message can accompany an unintended resource choice; verify the intended operation before treating it as a request for more permissions.

## 4. Inventory application test and protocol troubleshooting

Opening the Inventory application using **HTTPS** produced **`ERR_CONNECTION_TIMED_OUT`**. Explicitly using **HTTP** worked, and the inventory application test was completed. The record does not specify individual create/update/delete actions or row counts, so none are asserted here.

Address pattern only; replace the placeholder within the lab environment:

```text
http://<INVENTORY_APP_HOST>
```

This was an **application/protocol troubleshooting incident, not an RDS failure**. HTTP and HTTPS normally use different ports (80 and 443) and HTTPS requires TLS support. The observations establish that HTTP worked while HTTPS timed out; they do not identify whether the HTTPS failure was caused by a listener, firewall, or another component.

Do not infer database health from a browser timeout alone. Check the URL scheme and application access first, then investigate secret retrieval, the direct database network path, and database authentication if application data still fails.

**HTTP 在這次課堂測試可用，不代表正式網站應放棄 HTTPS。** A future independent deployment should configure HTTPS where practical; the classroom workaround is not a general security recommendation.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| Concept | Simple analogy |
| --- | --- |
| EC2 inventory application | A shop worker using a computer |
| RDS MySQL | An organized inventory filing room |
| Database `inventory` | The inventory cabinet inside that room |
| Secrets Manager | The locked key box containing the room's address and key |
| IAM role | The worker's badge allowing access to the key box |
| Security group | The guard checking who can use a door |
| Network connection | The hallway the worker uses to reach the filing room |
| DB subnet group | A list of places where AWS may put the database room |
| Single-AZ | One location, without a ready standby in another AZ |

The worker gets the key from the box, then walks down the hallway to the filing room. The worker does not climb through the key box to reach the database.

> **Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

**拿得到鑰匙、走得到房間、鑰匙能開門，是三件不同的事。** AWS permission, network reachability, and database authentication must each work.

## SAA Review

- **RDS MySQL** is a managed relational database. The EC2 application and RDS database have separate responsibilities.
- **Single-AZ** does not provide the standby of a Multi-AZ DB instance deployment. Do not confuse a subnet group with a high-availability setting.
- **DB subnet group** controls eligible subnet placement; **security groups** control network traffic.
- **TCP 3306** is the normal MySQL port. Browser HTTP/HTTPS access is a different connection from application-to-database access.
- **Secrets Manager** stores/retrieves secrets; it is not a proxy. Applications need both AWS authorization and direct database connectivity.
- **EC2 instance profiles** associate IAM roles with instances; secret retrieval does not require embedding permanent AWS access keys in application code.
- **Instance identifier ≠ database name:** `inventory-db` vs `inventory`.
- **Troubleshoot by layer:** wrong engine/configuration, denied AWS action, browser protocol, secret retrieval, database connectivity, database login.

## Lab Completion Checkpoint / Lab 結束檢查點

### 1. Class Lab Complete

| Check | Record |
| --- | --- |
| Lab completed successfully | Yes — based on the supplied completed-lab record and inventory application test |
| Important troubleshooting captured | Yes — configuration/engine mismatch and HTTPS timeout |
| Full notes documented | Yes |
| Cheat sheet documented | Yes |
| GitHub documentation | This Lab 06 update awaits review; not committed or pushed |

### 2. SAA Takeaways

Explain RDS MySQL, Single-AZ, subnet groups, `DB-SG`, EC2 role permissions, secret retrieval, and the direct application-to-RDS connection. Recognize that the browser's protocol failure did not establish a database failure.

**記憶：鑰匙箱提供憑證；應用程式直接連資料庫。**

### 3. 🧒 3rd-Grade Understanding Check

Can I explain why the worker gets a key from Secrets Manager and then uses a separate hallway to reach RDS? Can I distinguish the badge, guard, and database key? Use the analogy table above for the check; a separate learner assessment result has not been supplied.

### 4. What AWS Academy Prepared for Me

The classroom provided controlled AWS access and a lab environment containing the Lab VPC and inventory application context. The supplied record does not identify exactly which subnet, IAM, application, or secret prerequisites were precreated versus configured during the lab. A personal rebuild must inventory and recreate what it needs instead of assuming classroom resources exist.

**These notes are a Class Lab Record, not a guaranteed from-scratch runbook for a personal AWS account.** 個人帳號必須自行準備網路、主機、應用程式與權限。

### 5. Personal AWS Rebuild Decision

**🟢 Yes — High Learning Value**

An independent rebuild would connect networking, EC2, RDS, IAM, and Secrets Manager while exposing prerequisites the classroom workflow prepared. It would test the difference between retrieving credentials and reaching a database. Evaluate cost, security, setup time, application availability, and cleanup before building.

### 6. Follow-up Companion Lab

The [Lab 06 RDS companion plan](../personal-labs/lab-06-rds-companion-plan.md) exists as a **plan only**. The personal lab has not been built. It must work without AWS Academy resources, explain each prerequisite, use safer defaults where practical, and include cost/security checkpoints and complete cleanup instructions.

### 7. Cleanup Check

No cloud cleanup result was supplied for the class lab. Do not infer cleanup from a successful application test. A future personal rebuild must verify removal or intentional retention of RDS instances, manual/final snapshots, retained automated backups, EC2, EBS, secrets, and any additional networking or monitoring resources across every Region used. Document ongoing cost for retained resources.

Class Lab -> Documentation -> SAA Review -> 3rd-Grade Check -> Personal Rebuild Decision -> Cleanup Review

上課 Lab -> 文件化 -> SAA 複習 -> 三年級理解檢查 -> Personal Rebuild 判斷 -> Cleanup 檢查
