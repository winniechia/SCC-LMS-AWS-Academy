# Lab 6 Challenge Lab — Migrating a Database to Amazon RDS

**Cheat sheet — café MariaDB migration.** Separate from [Lab 06 — Creating an Amazon RDS Database](../labs/lab-06-amazon-rds.md), the MySQL inventory lab.

[Study index](../README.md) | [Full notes](../labs/challenge-lab-06-migrating-database-to-amazon-rds.md) | [Companion plan](../personal-labs/challenge-lab-06-rds-migration-companion-plan.md)

## Recorded migration

```text
CafeServer MariaDB -> mysqldump -> CafeDbDump.sql -> RDS MariaDB
CafeServer app -> Secrets Manager (retrieve connection settings)
CafeServer app -> RDS directly (TCP 3306)
```

**Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

## Troubleshooting: road → key

- **CafeSG → dbSG, TCP 3306:** database inbound source is the application security group. Routing/client outbound access must also work.
- **`nmap -Pn`:** skip host discovery; the recorded scan showed **3306 open**. This proves neither credentials nor data correctness.
- **Stray `Pn` in MySQL:** remove it. `-Pn` is a Nmap option, not a database login option.
- **`using password: NO`:** attempt did not use a password.
- **`using password: YES`:** password was used, not necessarily accepted. Check credentials, server, and account permissions.
- Successful RDS login was achieved.

**First prove the road is open, then test the key / 先證明路是通的，再測鑰匙**

Illustrative login only; no actual endpoint or credential:

```bash
mysql -h <RDS_ENDPOINT> -P 3306 -u <DB_USER> -p
```

Uppercase `-P` = port; lowercase `-p` = password prompt.

## Cutover evidence

1. Export/import using `CafeDbDump.sql`; verify **26 existing order rows in RDS**.
2. Update `/cafe/dbUrl`, `/cafe/dbUser`, `/cafe/dbPassword`.
3. Leave `/cafe/dbName`, `/cafe/currency`, `/cafe/timeZone`, `/cafe/showServerInfo` unchanged.
4. Stop old EC2 MariaDB; verify **`inactive (dead)`**.
5. Create and view **Order #27** afterward: application reads/writes RDS with the old service stopped.

**Successful import command does not equal verified migration / Import command 成功不等於 migration 已驗證.** Row count alone is also insufficient; include application behavior and destination evidence.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

Old cabinet → moving box → managed records room. Guards = security groups; door 3306 = database port; road check = Nmap; key test = login; key box = Secrets Manager. Check the **26 old pages**, then write/read **page #27** with the old room closed.

## SAA memory and completion

- Network access, database login, and IAM secret-read permission are separate checks.
- Logical export/import is not continuous replication or an AMI migration.
- Verify old data + new reads/writes after configuration cutover.
- Stopped MariaDB ≠ deleted EC2/EBS/RDS resources.
- **🟢 Yes — High Priority / High Learning Value:** personal companion is a plan only. Audit costs, rollback, and cleanup before a future build.
