# Module 16 — Disaster Recovery & Storage Gateway Knowledge Check

**Date:** 2026-10-10  
**Result:** **100% (10 questions)** — reported after submission in AWS Academy.  
**Status:** Knowledge Check complete. **Lab 16 remains blocked** by an AWS Organizations SCP explicit deny for S3 bucket creation in Ohio; do not confuse quiz completion with lab completion.

## Answer key / 答案總覽

| # | Answer | Concept / 核心概念 |
|---|---|---|
| 1 | **C (3)** | RPO = maximum acceptable data loss **measured in time**; RTO = maximum acceptable time to restore service. |
| 2 | **D (4)** | CloudFormation templates redeploy infrastructure rapidly (IaC). |
| 3 | **D (4)** | S3 Cross-Region Replication (CRR) plus handling pre-existing objects. |
| 4 | **B (2)** | Keep essential data separate from replaceable EC2 compute; automate rebuilding. |
| 5 | **C (3)** | Route 53 health checks and failover routing between geographic endpoints. |
| 6 | **B (2)** | Backup and Restore: usually lowest cost, longest RTO. |
| 7 | **D + E (4 + 5)** | Pilot Light: minimal core running; Warm Standby: scaled-down functioning environment. |
| 8 | **D (4)** | Multi-site DR: automatic failover to a fully functional, operational second site. **Correction: C was marked incorrect by the course, with feedback “This strategy is not DR. It is fault isolation.”** |
| 9 | **A (1)** | Warm Standby meets minute-level RTO/RPO with less cost than full multi-site. |
| 10 | **A + D + F (1 + 4 + 6)** | File Gateway NFS/SMB; Tape Gateway VTL; Volume Gateway iSCSI block volumes. |

## Detailed bilingual notes / 中英雙語解析

### Q1 — RPO vs RTO
- **RPO:** How far back can recovered data be? / **最多容許損失多少時間的資料**。
- **RTO:** How long may the service remain unavailable? / **最多容許停機多久**。
- Example: hourly backups can yield up to approximately one hour of lost changes; a 30-minute RTO requires service restoration within 30 minutes.
- **Analogy / 比喻:** RPO 是作文最後一次存檔到故障之間可能遺失的內容；RTO 是多久可以重新開始寫。

### Q2 — Infrastructure as Code
- **CloudFormation** templates can recreate VPCs, subnets, EC2, security groups and related infrastructure, subject to resource/Region support.
- **Important:** IaC alone does not restore application data; pair it with backups or replication.
- **Analogy:** 餐廳的完整建造藍圖，能快速重建設備配置。

### Q3 — S3 Cross-Region Replication
- S3 CRR asynchronously replicates eligible objects across Regions; enable **versioning on both buckets**.
- The course option describes **copying existing objects onto themselves** after enabling CRR. For modern architectures, **S3 Batch Replication** is an AWS-native option for existing objects. These are distinct from normal live replication.
- **Analogy:** Ohio 圖書館的新書自動送往 Oregon；舊書還需要補送安排。

### Q4 — EC2 DR
- Separate persistent data (e.g., S3, RDS, EFS) from EC2 compute. Rebuild EC2 using AMIs, Launch Templates, Auto Scaling or CloudFormation.
- **Analogy:** 電腦壞了可以換；書稿應另存安全位置。

### Q5 — Route 53 Failover
- Route 53 health checks + failover routing can direct new DNS lookups from primary to secondary endpoint; DNS caching can delay observed switchover.
- **Contrast:** ALB distributes traffic to targets in its Region; Route 53 can direct clients to endpoints in different Regions.
- **Analogy:** 主要餐廳關門，指路員改指向另一城市的餐廳。

### Q6 — Backup and Restore
- Low ongoing cost because a full standby stack need not be running; restoring infrastructure and data takes time.
- **Caveat:** RPO depends on backup frequency; RTO depends on recovery processes.
- **Analogy:** 保留食譜與設備清單，但餐廳需要重新搭建。

### Q7 — Pilot Light vs Warm Standby
- **Pilot Light:** only essential core components run continuously; other resources start on disaster.
- **Warm Standby:** smaller but operational copy of the application stack, scaled up on disaster.
- **Analogy:** Pilot Light = 只保留小火苗；Warm Standby = 小餐廳已營業、隨時增加人手。

### Q8 — Course-specific correction: Multi-site DR vs fault isolation
- **Course-correct answer: D.** Automatic failover to another fully functional site.
- **Observed grading feedback:** Choice C (load distributed across geographically separate sites) was marked incorrect: **“This strategy is not DR. It is fault isolation.”**
- **Nuance for SAA-C03:** Active/active multi-Region designs *can* form part of DR. The course distinguishes distribution/fault isolation from a specifically defined **failover and recovery** capability. Remember the course answer without treating distributed active/active designs as universally unrelated to DR.
- **Analogy:** 兩家餐廳各自營業是隔離故障；明確設計主店故障時自動接手才是題目強調的 DR。

### Q9 — DR pattern choice with minute-level RPO/RTO and cost constraint
- **Warm Standby:** a running scaled-down environment can be scaled and receive traffic quickly; data replication must separately meet RPO.
- **Compare:** Backup/Restore = lowest cost/slowest; Pilot Light = minimal core; Warm Standby = small operational copy; Multi-site = greater readiness and generally greater cost.
- **Analogy:** 備用餐廳平常只開小規模，但已經能接客，災難時加開座位。

### Q10 — Three Storage Gateway capabilities
- **S3 File Gateway:** SMB/NFS file access to S3-backed storage.
- **Tape Gateway:** virtual tape library (VTL) workflows for backup applications.
- **Volume Gateway:** iSCSI block volumes presented to on-premises applications.
- **Not:** EFS fully managed NFS is a different service; direct S3 API use is not a Storage Gateway-specific capability.
- **Analogy:** File Gateway = 檔案櫃；Tape Gateway = 備份磁帶室；Volume Gateway = 遠端磁碟。

## SAA-C03 quick review / 考試速記

- **RPO = Data loss window; RTO = Downtime window.**
- **DR selection = required RPO + RTO + budget.** Values in practice depend on tested implementation, not just strategy labels.
- **CloudFormation = Infrastructure as Code**, not automatic data recovery.
- **Route 53 = DNS health checks and failover**, not data replication.
- **S3 CRR = asynchronous geographic object replication; versioning required.**
- **S3 Lifecycle = transitions/expiration**, not a replacement for replication.
- **Storage Gateway:** NFS/SMB → File; VTL → Tape; iSCSI → Volume.
- **Q8 lesson:** Check grading feedback; distinguish the course's DR failover framing from general fault isolation.

## Lab 16 checkpoint / 實驗狀態

The Knowledge Check earned **100%**, but the separate guided Lab 16 **was not completed**. The intended three-Region architecture is Virginia (gateway), Ohio (source S3), Oregon (replica S3). Ohio bucket creation was denied by an AWS Organizations SCP. See [Lab 16 notes](../labs/lab-16-s3-file-gateway-hybrid-storage.md) and [Personal rebuild plan](../personal-labs/lab-16-hybrid-storage-lifecycle-companion-plan.md).

**Personal AWS Rebuild?** Yes, later, with cost controls, all three Regions, CRR, and a separate S3 Lifecycle Policy exercise.
