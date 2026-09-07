# SCC LMS AWS Academy — Study Notes

Hands-on AWS Academy / AWS Solutions Architect study notes, with English and Traditional Chinese reminders（繁體中文重點）and simple analogies for review.

## Study index

| Lab | Full notes | Quick review |
| --- | --- | --- |
| Lab 05 — Introducing Amazon Elastic File System (Amazon EFS) | [Lab 05 notes](labs/lab-05-amazon-efs.md) | [Lab 05 cheat sheet](cheat-sheets/lab-05-amazon-efs-cheat-sheet.md) |

Only Lab 05 is documented. The full notes preserve the supplied commands, observations, and troubleshooting lesson; the compact cheat sheet is intended for quick printing and exam review.

> **Before big writes -> df -hT -> verify nfs4 -> then run workload.**
>
> 大量寫入前，先確認 EFS 已掛載；資料夾存在不代表掛載成功。
