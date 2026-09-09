# Lab 6 Challenge Lab — Migrating a Database to Amazon RDS

**Personal AWS Companion Plan — café MariaDB migration.** Separate from [Lab 06 — Creating an Amazon RDS Database](../labs/lab-06-amazon-rds.md), the MySQL inventory lab.

[Study index](../README.md) | [Class migration notes](../labs/challenge-lab-06-migrating-database-to-amazon-rds.md) | [Cheat sheet](../cheat-sheets/challenge-lab-06-migrating-database-to-amazon-rds-cheat-sheet.md)

**Decision: 🟢 Yes — High Priority / High Learning Value**

**Status: plan only. No AWS deployment has been performed by this documentation task.** This is a scope and verification plan; a future runbook must provide concrete creation, rollback, and cleanup steps.

## Learning objective

Build an independent application/database environment, move its local MariaDB data to RDS for MariaDB, and prove the application uses the destination. Recreate prerequisites without relying on AWS Academy permissions, files, resources, or temporary credentials.

During future **SAA review**, rebuild the Challenge Lab 06 architecture **from scratch with ChatGPT guidance and without relying on any AWS Academy pre-lab setup**. Assume **none of the classroom resources exist** in the personal account.

**Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**

**「不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。」**

Before building, identify what AWS Academy had pre-created from the available lab instructions and evidence. Record each prerequisite's purpose, whether classroom preparation is confirmed or still unknown, and how we will create its independent equivalent. Do not invent the classroom inventory or silently reuse it.

**Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

## Prerequisites to design and explain

| Component | Purpose |
| --- | --- |
| Region, budget, runtime | Bound cost and the exercise |
| VPC/subnets/routes and DB subnet group | Provide application/service/database connectivity and eligible RDS placement |
| Application/database security groups | Scope TCP 3306 to the application clients |
| EC2, local MariaDB, application | Source database and a client that can prove migration behavior |
| RDS for MariaDB | Managed destination with compatible database configuration |
| IAM role/instance profile and Secrets Manager | Authorize secret retrieval and supply destination settings |
| Synthetic order data and SQL dump | Safe historical data for migration tests |
| Monitoring and resource inventory | Observe failures and verify cleanup |

Choose reusable application assets with appropriate permission or build a small independent café-style application. Decide engine compatibility, schema creation, privileges, dump options, and handling of routines/triggers if present; do not assume classroom setup exists.

The source host may be **EC2 CafeServer or an equivalent source/application host**. Explicitly build its source MariaDB and create sample café data ourselves. Include Secrets Manager configuration, the IAM role/instance profile, and routing as needed; creating RDS alone is not a from-scratch rebuild.

## Future SAA review workflow

```text
Build prerequisites → Build source system → Create sample data → Build RDS → Configure SG-to-SG connectivity → Test network → mysqldump → Import → Verify data → Update Secrets Manager → Stop old database → Verify application READ + WRITE → Cleanup
```

Use ChatGPT guidance to explain and review each stage. Establish rollback considerations and cost/security checkpoints before execution, then collect data and application verification evidence before complete cleanup. **For now this remains a plan only; do not deploy AWS resources.**

## Migration and rollback plan

- [ ] Establish a baseline: source tables, row counts, and selected non-sensitive field values.
- [ ] Plan a maintenance window or write freeze so a dump and cutover do not lose orders written afterward. Do not claim zero downtime.
- [ ] Verify TCP 3306 from the authorized application client, then test database authentication.
- [ ] Export a consistent logical dump, import into the destination, and inspect errors and destination contents.
- [ ] Update only the intended connection configuration; define secret refresh behavior explicitly.
- [ ] Stop the source database service for the cutover test, then write and read a new destination order.
- [ ] Define rollback criteria before cutover. If destination writes have occurred, reconcile data before redirecting the application to an older source copy.

## Success criteria — not yet completed

- [ ] All prerequisites exist independently of AWS Academy.
- [ ] Explain **“First prove the road is open, then test the key / 先證明路是通的，再測鑰匙.”**
- [ ] Distinguish Nmap options from database-client options and interpret password `NO` versus `YES` correctly.
- [ ] Historical data is verified in RDS using counts and selected data checks.
- [ ] A new order is created and read after stopping the source service, with evidence that RDS is the destination.
- [ ] Explain **“Successful import command does not equal verified migration / Import command 成功不等於 migration 已驗證.”**
- [ ] Record security choices, cost checkpoints, rollback limitations, and cleanup evidence.

The class had 26 existing orders and then Order #27. A personal run may use different synthetic counts; correctness depends on its own baseline and verification, not reproducing those numbers.

## Cost and security checkpoints

- [ ] Check current pricing before provisioning; estimate RDS, EC2, EBS, backups/snapshots, secrets, and any selected IPv4, NAT, endpoints, or monitoring costs.
- [ ] Set budget alerts, runtime limits, and a cleanup time; do not assume alerts stop spending.
- [ ] Keep the destination private where practical and restrict database access to the application security group.
- [ ] Use scoped IAM secret-read permissions and an appropriately privileged database user; do not embed credentials in commands or source control.
- [ ] Plan encrypted database connections and safe application administration; distinguish classroom shortcuts from real-world recommendations.
- [ ] Protect dump files and use synthetic data. Publish only sanitized evidence.

## Complete cleanup requirements for the future runbook

- [ ] Decide retention needs before deleting databases or dumps.
- [ ] Delete the personal RDS instance with an explicit final-snapshot decision; separately review manual snapshots and retained automated backups.
- [ ] Terminate EC2 and inspect leftover EBS volumes, snapshots, and any created AMIs.
- [ ] Remove test secrets with an intentional recovery-window decision and verify their deletion state.
- [ ] Remove dump files and any copies/backups according to the retention decision.
- [ ] Remove lab-specific endpoints, NAT gateways, IP allocations, monitoring, and other supporting resources if created.
- [ ] Remove unused IAM, security groups, subnet groups, and VPC components after dependent resources are gone.
- [ ] Audit every Region used, record intentionally retained resources, and check ongoing costs.

**停止舊資料庫只是在驗證切換，不是完成清理。** The future build is complete only after data/application validation and the resource cleanup review.
