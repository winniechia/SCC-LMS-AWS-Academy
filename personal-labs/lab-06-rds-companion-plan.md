# Lab 06 — RDS Personal Companion Plan

[Study index](../README.md) | [Class lab record](../labs/lab-06-amazon-rds.md) | [Cheat sheet](../cheat-sheets/lab-06-amazon-rds-cheat-sheet.md)

**Decision: 🟢 Yes — High Learning Value**

**Status: planned independent rebuild only. No personal lab has been built or cloud resources provisioned by this documentation task.** This document defines scope and success criteria; it is not an executed runbook.

## Why rebuild independently?

Recreating the inventory system connects VPC networking, EC2, RDS MySQL, IAM, and Secrets Manager. AWS Academy supplied a controlled environment; a personal account requires deliberate choices about prerequisites, permissions, software, cost, and cleanup.

> **Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

The key learning test is to show two separate connections: the application retrieves credentials from Secrets Manager, then connects directly to RDS.

## Planned scope and prerequisites

| Resource / decision | Why it exists |
| --- | --- |
| Region and budget | Bound the exercise and estimate costs before building |
| VPC, subnets, routes | Provide application and database connectivity |
| DB subnet group | Give RDS eligible subnets across at least two AZs for a standard regional deployment |
| Application and database security groups | Permit only the intended traffic, including application-to-MySQL access |
| EC2 and administration method | Run and maintain the inventory application |
| Application source and dependencies | Recreate functionality without assuming AWS Academy files are available or reusable |
| RDS MySQL and database | Store inventory data; distinguish instance settings from the database schema |
| IAM role and instance profile | Give the application scoped AWS access without embedded long-lived access keys |
| Secrets Manager secret | Store the connection information/credentials the application needs |
| Monitoring and inventory | Observe failures and track resources through cleanup |

Choose a permitted application source or create a small independent inventory application. Record every prerequisite and explain its purpose. The class settings (`db.t3.micro`, 20 GiB `gp2`, Single-AZ) are a reference, not an instruction to provision them without checking current availability and pricing.

## Security checkpoints

- Keep RDS private where practical; restrict database inbound access to the application security group instead of the public internet.
- Scope secret-read permissions to the intended secret; separate AWS authorization from database user privileges.
- Choose a database user with only the application privileges needed; avoid using the master user for routine application work.
- Decide how to provide HTTPS for the web application and encrypted database connections. Do not adopt the classroom HTTP workaround as a production recommendation.
- Plan application access to Secrets Manager through an appropriate service-network path; compare any endpoint/NAT cost before choosing.
- Keep credentials and identifiers out of published notes, logs, screenshots, and source control. Use placeholders in learning artifacts.

## Success criteria — future verification, not completed results

- [ ] All prerequisites are created or explicitly accounted for without AWS Academy resources.
- [ ] An EC2 application retrieves its secret through the intended role and instance profile.
- [ ] The application connects directly to RDS MySQL and uses the intended inventory database.
- [ ] A controlled inventory write and subsequent read demonstrate persistence; test records contain no private information.
- [ ] Explain the separate AWS-permission, network, and database-login checks using the key-box analogy.
- [ ] Distinguish browser HTTP/HTTPS behavior from MySQL connectivity without weakening permissions to hide a fault.
- [ ] Record configuration choices, validation evidence, and differences from classroom shortcuts.
- [ ] Complete the cleanup checklist and document retained resources and costs.

## Cost checkpoints

- [ ] Before provisioning, check current regional pricing and eligibility; do not assume free usage.
- [ ] Estimate RDS instance/storage/backup costs, EC2/EBS, Secrets Manager, and any public IPv4, NAT, endpoints, data transfer, or monitoring selected.
- [ ] Set a spending limit for the exercise, expected runtime, and a cleanup time; configure budget alerts where appropriate. Alerts are not a hard spending cap.
- [ ] Review cost implications before enabling additional availability, backups, or networking resources.
- [ ] Recheck the resource inventory and billing after cleanup; account for delayed billing data.

## Cleanup checklist and future runbook requirements

The future runbook must provide concrete deletion steps, dependencies, and verification for each resource it creates. This plan alone is not a completed cleanup procedure.

- [ ] Stop test traffic and decide whether any data must be retained before deletion.
- [ ] Delete the personal RDS instance using an explicit final-snapshot decision; review manual/final snapshots and retained automated backups separately.
- [ ] Terminate the test EC2 instance and inspect remaining EBS volumes, snapshots, and any created AMIs.
- [ ] Delete the test secret with an intentional recovery-window decision; verify its deletion state and any replicas if created.
- [ ] Remove unused lab-specific endpoints, NAT gateways, public IP allocations, monitoring resources, and other supporting services if created.
- [ ] Remove unused DB subnet groups, security groups, IAM instance profiles/roles/policies, and VPC components after their dependencies are gone.
- [ ] Check every Region used; record removed resources, retained resources, and ongoing cost implications.

**停止主機不等於完成清理。計畫完成不等於實作完成。** A future build is complete only after its functional checks and cleanup review are documented.
