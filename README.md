# SCC LMS AWS Academy — Study Notes

Hands-on AWS Academy / AWS Solutions Architect study notes, with English and Traditional Chinese reminders（繁體中文重點）and simple analogies for review.

## Study index

| Lab | Full notes | Quick review |
| --- | --- | --- |
| Lab 05 — Introducing Amazon Elastic File System (Amazon EFS) | [Lab 05 notes](labs/lab-05-amazon-efs.md) | [Lab 05 cheat sheet](cheat-sheets/lab-05-amazon-efs-cheat-sheet.md) |
| Challenge Lab 05 — Creating a Dynamic Website for the Café | [Café challenge notes](labs/challenge-lab-05-dynamic-cafe-website.md) | [Café challenge cheat sheet](cheat-sheets/challenge-lab-05-dynamic-cafe-website-cheat-sheet.md) |
| Lab 06 — Creating an Amazon RDS Database | [Lab 06 notes](labs/lab-06-amazon-rds.md) | [Lab 06 cheat sheet](cheat-sheets/lab-06-amazon-rds-cheat-sheet.md) |
| Lab 6 Challenge Lab — Migrating a Database to Amazon RDS | [Migration notes](labs/challenge-lab-06-migrating-database-to-amazon-rds.md) | [Migration cheat sheet](cheat-sheets/challenge-lab-06-migrating-database-to-amazon-rds-cheat-sheet.md) |
| Lab 07 — Creating a VPC | [Lab 07 notes](labs/lab-07-creating-a-vpc.md) | [Lab 07 cheat sheet](cheat-sheets/lab-07-creating-a-vpc-cheat-sheet.md) |
| Challenge Lab 07 — VPC Networking, Bastion, NAT, and NACL | [Challenge Lab 07 notes](labs/challenge-lab-07-vpc-networking.md) | [Challenge Lab 07 cheat sheet](cheat-sheets/challenge-lab-07-vpc-networking-cheat-sheet.md) |
| Lab 08 — Creating a VPC Peering Connection | [Lab 08 notes](labs/lab-08-vpc-peering.md) | [Lab 08 cheat sheet](cheat-sheets/lab-08-vpc-peering-cheat-sheet.md) |

The Amazon EFS lab and the café challenge are separate Lab 05 activities. The full notes preserve the supplied work and troubleshooting lessons; the compact cheat sheets support printing and exam review.

> **Before big writes -> df -hT -> verify nfs4 -> then run workload.**
>
> 大量寫入前，先確認 EFS 已掛載；資料夾存在不代表掛載成功。

> **Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

> **Public IP = address. Route = road. IGW = outside gate. / Public IP 是門牌；Route 是道路；IGW 是對外大門。**

> **Bastion = people go IN. NAT = Private EC2 goes OUT. / Bastion = 人進去；NAT = Private EC2 出去。**

> **Peering = bridge. Route = road sign. Flow Logs = traffic evidence. / Peering = 橋；Route = 路牌；Flow Logs = 車流證據。**

## Personal companion labs

These are plans for independent learning, separate from completed classroom labs.

| Companion | Status |
| --- | --- |
| [Lab 06 RDS companion plan](personal-labs/lab-06-rds-companion-plan.md) | 🟢 Yes — High Learning Value; planned only, not built |
| [Challenge Lab 06 café migration — future SAA rebuild plan](personal-labs/challenge-lab-06-rds-migration-companion-plan.md) | 🟢 Yes — High Priority / High Learning Value; from scratch with ChatGPT guidance, no AWS Academy pre-lab setup; plan only |
| [Lab 07 VPC companion plan](personal-labs/lab-07-vpc-companion-plan.md) | 🟢 Yes — Very High Priority / Very High Learning Value; from scratch, plan only |
| [Challenge Lab 07 VPC networking companion plan](personal-labs/challenge-lab-07-vpc-networking-companion-plan.md) | 🟢 Yes — Very High Priority / Very High Learning Value; rebuild from scratch in personal AWS without Academy pre-lab setup; plan only |
| [Lab 08 VPC peering companion plan](personal-labs/lab-08-vpc-peering-companion-plan.md) | 🟢 Yes — Very High Priority / Very High Learning Value; recreate Academy prerequisites from scratch and use the newest AWS Console GUI available at rebuild time; plan only |

## Knowledge Checks

Concept summaries and learning corrections from completed study sessions; these do not reproduce the quizzes verbatim.

| Module | Study notes | Quick review |
| --- | --- | --- |
| Module 5 — EC2 and storage concepts | [Module 5 Knowledge Check](knowledge-checks/module-05-knowledge-check.md) | [Module 5 cheat sheet](cheat-sheets/module-05-knowledge-check-cheat-sheet.md) |
| Module 6 — Database concepts | [Module 6 Knowledge Check](knowledge-checks/module-06-knowledge-check.md) | [Module 6 cheat sheet](cheat-sheets/module-06-knowledge-check-cheat-sheet.md) |
| Module 7 — VPC networking concepts | [Module 7 Knowledge Check](knowledge-checks/module-07-knowledge-check.md) | [Module 7 cheat sheet](cheat-sheets/module-07-knowledge-check-cheat-sheet.md) |
| Module 8 — Hybrid and multi-VPC networking | [Module 8 Knowledge Check](knowledge-checks/module-08-knowledge-check.md) | [Module 8 cheat sheet](cheat-sheets/module-08-knowledge-check-cheat-sheet.md) |
