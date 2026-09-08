# SCC LMS AWS Academy — Study Notes

Hands-on AWS Academy / AWS Solutions Architect study notes, with English and Traditional Chinese reminders（繁體中文重點）and simple analogies for review.

## Study index

| Lab | Full notes | Quick review |
| --- | --- | --- |
| Lab 05 — Introducing Amazon Elastic File System (Amazon EFS) | [Lab 05 notes](labs/lab-05-amazon-efs.md) | [Lab 05 cheat sheet](cheat-sheets/lab-05-amazon-efs-cheat-sheet.md) |
| Challenge Lab 05 — Creating a Dynamic Website for the Café | [Café challenge notes](labs/challenge-lab-05-dynamic-cafe-website.md) | [Café challenge cheat sheet](cheat-sheets/challenge-lab-05-dynamic-cafe-website-cheat-sheet.md) |
| Lab 06 — Creating an Amazon RDS Database | [Lab 06 notes](labs/lab-06-amazon-rds.md) | [Lab 06 cheat sheet](cheat-sheets/lab-06-amazon-rds-cheat-sheet.md) |

The Amazon EFS lab and the café challenge are separate Lab 05 activities. The full notes preserve the supplied work and troubleshooting lessons; the compact cheat sheets support printing and exam review.

> **Before big writes -> df -hT -> verify nfs4 -> then run workload.**
>
> 大量寫入前，先確認 EFS 已掛載；資料夾存在不代表掛載成功。

> **Secrets Manager is the KEY BOX, not the HALLWAY. / Secrets Manager 是鑰匙箱，不是走廊。**

## Personal companion labs

These are plans for independent learning, separate from completed classroom labs.

| Companion | Status |
| --- | --- |
| [Lab 06 RDS companion plan](personal-labs/lab-06-rds-companion-plan.md) | 🟢 Yes — High Learning Value; planned only, not built |
