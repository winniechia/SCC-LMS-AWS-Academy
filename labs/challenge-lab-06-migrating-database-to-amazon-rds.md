# Lab 6 Challenge Lab — Migrating a Database to Amazon RDS

[Study index](../README.md) | [Cheat sheet](../cheat-sheets/challenge-lab-06-migrating-database-to-amazon-rds-cheat-sheet.md) | [Companion plan](../personal-labs/challenge-lab-06-rds-migration-companion-plan.md)

## Completed work and scope

We migrated the café MariaDB database from **CafeServer** to **Amazon RDS for MariaDB**, verified the existing orders, changed the application's connection configuration, stopped the old EC2 MariaDB service, and successfully created and viewed **Order #27** afterward.

This café MariaDB migration is separate from [Lab 06 — Creating an Amazon RDS Database](lab-06-amazon-rds.md), which documents the MySQL inventory application. No engine version, instance class, storage allocation, exact database/table name, or unprovided command transcript is inferred. The examples below are sanitized command patterns, not exact historical commands. Passwords, secret values, resource IDs, ARNs, and temporary addresses are omitted.

## 1. Migration and connection paths

```text
Data migration:
CafeServer MariaDB -> mysqldump -> CafeDbDump.sql -> RDS import

Application configuration:
CafeServer application -> Secrets Manager (retrieve settings)

Application database traffic after cutover:
CafeServer application -> RDS for MariaDB (TCP 3306)
```

> **Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

Secrets Manager supplies connection information and credentials. The application then connects directly to RDS; database queries do not pass through Secrets Manager. IAM authorization to retrieve secrets, network access to the database, and database authentication are separate requirements.

## 2. First prove the road is open

The recorded security-group access was **CafeSG → dbSG on TCP 3306**. On the database side, `dbSG` permits inbound database traffic with **CafeSG as the source security group**. This references the client's group rather than a temporary client IP. The client also needs suitable outbound access and network routing. Security groups are controls on traffic, not intermediate network hops. See [RDS security groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html).

We used **`nmap -Pn`** verification and observed **3306 open**. Safe illustrative pattern:

```bash
nmap -Pn -p 3306 <RDS_ENDPOINT>
```

`-Pn` skips host discovery and attempts the requested scan even if discovery probes would not receive a response. An open port supports reachability to a listening service from the scanning client; it does not validate the username, password, database contents, or application behavior. See [Nmap host discovery](https://nmap.org/book/host-discovery-controls.html).

> **First prove the road is open, then test the key / 先證明路是通的，再測鑰匙**

## 3. Command and authentication troubleshooting

### Accidental `Pn` in the MySQL command

`Pn` was accidentally included in the MySQL command. **`-Pn` belongs to Nmap**, not to the database login command. Remove the stray token and check which program each option belongs to. The exact mistaken command and resulting error text were not supplied, so they are not reconstructed.

Illustrative interactive database login:

```bash
mysql -h <RDS_ENDPOINT> -P 3306 -u <DB_USER> -p
```

Here, uppercase `-P` specifies the port; lowercase `-p` requests a password prompt. Type the password privately at the prompt, not into published commands or notes. A client named `mysql` can connect to MariaDB; that client name does not change the RDS engine into MySQL.

### Interpret the authentication message precisely

| Message fragment | Meaning | What to check |
| --- | --- | --- |
| `using password: NO` | The attempt did not use a password | Check the login invocation and password prompt/input |
| `using password: YES` | The attempt used a password | It does **not** mean the password was accepted; check credentials, intended server, and account/host permissions |

An access-denied response is different evidence from a network timeout: a database server responded to the authentication attempt. Do not assume `YES` proves a correct key or that every authentication failure has the same cause. See [MySQL access-denied troubleshooting](https://dev.mysql.com/doc/refman/8.4/en/problems-connecting.html).

**結果：成功登入 RDS。** Successful RDS login was achieved after troubleshooting. No literal password or exact intermediate credential mistake is recorded here.

## 4. Export, import, and verify the existing data

The completed transfer used **`mysqldump` → `CafeDbDump.sql` → RDS import**. This is a logical export/import workflow, not an AWS DMS task or an AMI migration.

Illustrative patterns only; database selection must match how the dump was generated:

```bash
mysqldump -u <SOURCE_DB_USER> -p <CAFE_DATABASE> > CafeDbDump.sql
mysql -h <RDS_ENDPOINT> -P 3306 -u <DB_USER> -p <CAFE_DATABASE> < CafeDbDump.sql
```

These patterns do not claim the original dump's exact flags, database-creation behavior, or consistency settings. A future independent runbook must address those explicitly. SQL dumps may contain private application data and should not be committed. See [RDS MariaDB data import guidance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MariaDB.Procedural.Importing.html).

We verified **26 existing order rows in RDS**. This confirms the recorded pre-cutover order count; a count alone does not prove every field or every table is identical.

> **Successful import command does not equal verified migration / Import command 成功不等於 migration 已驗證.**

An import must be followed by checking the destination data and exercising the application against the new database.

## 5. Change the application's connection settings

We updated the following Secrets Manager entries:

| Entry | Recorded action |
| --- | --- |
| `/cafe/dbUrl` | Changed to the RDS connection destination |
| `/cafe/dbUser` | Changed to the destination database user |
| `/cafe/dbPassword` | Changed to the destination database password; value omitted |
| `/cafe/dbName` | Left unchanged |
| `/cafe/currency` | Left unchanged |
| `/cafe/timeZone` | Left unchanged |
| `/cafe/showServerInfo` | Left unchanged |

These are configuration names, not secret values. Changing the key-box contents changes where the application connects and which credentials it uses. It does not create a network route, open TCP 3306, or make Secrets Manager a proxy. No secret-refresh or application-restart mechanism is inferred from the record.

## 6. Prove the application uses RDS

| Evidence | What it establishes |
| --- | --- |
| RDS login succeeded | A database connection and authentication worked |
| RDS contained 26 existing order rows | The recorded order data was present at the destination |
| Old EC2 MariaDB was stopped; status showed `inactive (dead)` | The old local service was no longer serving the application at that check |
| Created and viewed Order #27 afterward | The application successfully wrote and read an order after the cutover |

Together with the updated destination configuration, these observations demonstrate that the application was reading/writing **RDS**, rather than relying on the old local MariaDB service. The order number alone would not prove this; the stopped source service and destination verification make the evidence meaningful.

**停掉舊資料庫後，新訂單仍可新增與查看，才真正驗證應用程式已切換。** `inactive (dead)` is a service status, not evidence that EC2, its disks, or the database files were deleted. No zero-downtime or ongoing replication claim is made.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| Concept | Café analogy |
| --- | --- |
| CafeServer application | The café worker taking orders |
| Local MariaDB → RDS | Moving the order ledger from an old cabinet to a managed records room |
| `mysqldump` / `CafeDbDump.sql` / import | Copying the ledger into a moving box, then unpacking it |
| CafeSG → dbSG | Guards allowing the café worker through the records-room door |
| TCP 3306 | The database door number |
| Nmap | Checking whether the road reaches an open door |
| Database login | Trying the key in that door |
| Secrets Manager | The key box holding the room's address and key |
| 26 rows, then Order #27 | Check the old pages arrived, then write and read a new page |

**先看路，再試鑰匙；搬完舊帳本，還要試寫新訂單。** A box arriving is not the same as a verified move.

## SAA takeaways

- Logical database migration requires export/import, destination verification, and application cutover.
- SG-to-SG rules can authorize application clients without tying database access to a temporary public IP.
- **Network reachability ≠ database authentication ≠ AWS secret-read authorization.**
- `using password: YES` means a password was used, not accepted.
- Secrets Manager is the key box; the application connects directly to RDS.
- Verify both historical data and new reads/writes; a successful command is insufficient.
- RDS manages database infrastructure; the customer still owns migration correctness, access configuration, and application validation.

## Lab Completion Checkpoint / Lab 結束檢查點

### 1. Class Lab Complete

| Check | Record |
| --- | --- |
| Lab completed successfully | Yes — 26 existing orders verified in RDS; Order #27 created/viewed after stopping local MariaDB |
| Important troubleshooting captured | Yes — network test, stray `Pn`, and password-message interpretation |
| Full notes / cheat sheet documented | Yes |
| GitHub documentation | This migration update awaits approval; not committed or pushed |

### 2. SAA Takeaways

Explain the separate road, key, and AWS badge checks; SG-to-SG access; logical export/import; configuration cutover; and evidence-based migration verification.

### 3. 3rd-Grade Understanding Check

Can I explain why the key box is not the hallway, why an open door does not prove the key fits, and why checking old pages plus writing a new page is stronger than merely unpacking a box? The session's recorded learning is summarized above; no additional assessment score is invented.

### 4. What AWS Academy Prepared for Me

The classroom supplied controlled access and a café application environment with existing database data and supporting setup. The record does not identify exactly which underlying network, IAM, or software prerequisites were precreated versus configured during the exercise. A personal rebuild must establish each independently.

**These notes are a Class Lab Record, not a guaranteed from-scratch runbook for a personal AWS account.** 個人帳號不能假設課堂資源已存在。

### 5. Personal AWS Rebuild Decision

**🟢 Yes — High Priority / High Learning Value**

This rebuild combines real data movement, networking, authentication, Secrets Manager configuration, cutover, and validation. Its especially valuable lesson is proving the new database serves the application after the old service stops. Evaluate cost, security, prerequisites, time, and rollback/cleanup before building.

### 6. Follow-up Companion Lab

The [migration companion plan](../personal-labs/challenge-lab-06-rds-migration-companion-plan.md) is a **plan only**. No personal deployment has been built. It must create prerequisites without AWS Academy, explain their purpose, use safer defaults, and include cost/security checkpoints and complete cleanup instructions.

During future **SAA review**, rebuild this Challenge Lab 06 architecture **from scratch with ChatGPT guidance, without relying on any AWS Academy pre-lab setup**. Assume none of the classroom resources exist. First identify what AWS Academy had pre-created, distinguishing confirmed classroom setup from details that still need investigation; then plan and build the required equivalents ourselves.

**Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**

**「不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。」**

The independent scope includes networking/VPC and subnets, routing as needed, EC2 CafeServer or an equivalent source/application host, source MariaDB, sample café data, IAM role/instance profile, security groups, Secrets Manager configuration, an RDS DB subnet group, RDS MariaDB, database migration, application cutover, verification, rollback considerations, cost/security checkpoints, and complete cleanup. These are future requirements, not claims about resources already built.

Future workflow:

```text
Build prerequisites → Build source system → Create sample data → Build RDS → Configure SG-to-SG connectivity → Test network → mysqldump → Import → Verify data → Update Secrets Manager → Stop old database → Verify application READ + WRITE → Cleanup
```

Plan rollback and cost/security checks before executing this workflow. **Current authorization is documentation only; do not deploy AWS resources.**

### 7. Cleanup Check

Stopping local MariaDB is a cutover test, not cloud cleanup. No completed AWS resource cleanup audit was supplied. A future rebuild must inspect RDS, snapshots/backups, EC2/EBS, secrets, dump files, and all supporting resources across each Region used, documenting retained resources and cost implications.

Class Lab -> Documentation -> SAA Review -> 3rd-Grade Check -> Personal Rebuild Decision -> Cleanup Review

上課 Lab -> 文件化 -> SAA 複習 -> 三年級理解檢查 -> Personal Rebuild 判斷 -> Cleanup 檢查
