# Module 05 — Adding a Compute Layer Using Amazon EC2
AWS SAA-C03 | 中英雙語考前筆記 | 2026-10-20

> 學習方式：先看小學三年級比喻，再看 AWS 技術重點，最後用考題關鍵字排除選項。適合列印複習。

## 0. Learning Results / 學習成果

| Checkpoint | Topic | Correct / Total |
|---|---|---|
| 1 | EC2 Fundamentals | 5/5 |
| 2 | EC2 Pricing & Purchasing | 4/6 |
| 3 | Launching & Configuring EC2 | 4/6 |
| 4 | EC2 Storage | 4/7 |
| 5 | EC2 Networking & Availability | 7/7 |
| 6 | Architecture & Exam Scenarios | 6/7 |
| **Total** | | **30/38 = 78.9%** |

重點：此為練習題分數，不等於正式考試預測通過率。最需要複習：EC2 Storage、Standard RI vs Compute Savings Plans。

## 1. EC2 Fundamentals / 基礎概念

**Third-grade analogy:** EC2 像在 AWS 租一台電腦。AMI 像電腦的安裝範本；Instance Type 像選擇 CPU、RAM 規格；Security Group 像門口警衛；User Data 像第一次開機時的工作清單。

- **EC2 (Elastic Compute Cloud):** AWS 提供的虛擬運算執行個體（Instances）。
- **AMI (Amazon Machine Image):** 啟動 EC2 的作業系統／軟體映像範本。
- **Instance Type:** 決定 vCPU、記憶體、網路等資源。General Purpose（T/M）、Compute Optimized（C）、Memory Optimized（R/X）、Storage Optimized（I/D）依工作負載選擇。
- **User Data:** 啟動時執行初始化指令；一般 Linux cloud-init 預設首次啟動時執行，非每次重啟都執行。
- **Instance Metadata / IMDS:** 從 Instance 內取得執行個體相關資訊；優先使用 IMDSv2。
- **Stop vs Terminate:** Stop 停止運算（EBS-backed root 通常保留，仍有 EBS 儲存費）；Terminate 結束執行個體，EBS 是否刪除取決於 DeleteOnTermination。
- **Reboot:** 重新啟動作業系統；通常保留 Instance identity。Stop/Start 可能移至其他底層主機，未使用 Elastic IP 的自動公有 IPv4 可能改變。
- **Shared Responsibility:** AWS 管底層雲端基礎設施；使用者管理 Guest OS 修補、IAM、資料、Security Groups 等。

**Exam cues:** workload requires more RAM → Memory Optimized；CPU-heavy → Compute Optimized；預先設定 OS/應用程式 → AMI；首次啟動腳本 → User Data。

## 2. EC2 Pricing & Purchasing / 購買模式

**Third-grade analogy:** On-Demand 像按次付費；Reserved Instances 像固定三年租約；Savings Plans 像承諾每小時消費的會員方案；Spot 像便宜的臨時空位，但可能被收回。

| Option | 適合情境 | 考試陷阱 |
|---|---|---|
| On-Demand | 短期、不可預測、無長期承諾 | 便利但長期通常較貴 |
| Standard Reserved Instances (RI) | 長期穩定 EC2、可承諾 1/3 年 | 相對 Compute Savings Plans 彈性較低 |
| Convertible RI | 需要在支援範圍內調整 RI 屬性 | 通常折扣低於 Standard RI |
| Compute Savings Plans | 承諾每小時運算支出，需跨 Instance Family、Region、EC2/Fargate/Lambda 彈性 | 彈性較高，折扣上限通常低於最固定的選項 |
| EC2 Instance Savings Plans | 特定 Region 與 Instance Family 的穩定使用 | 可有接近 Standard RI 的高折扣 |
| Spot Instances | 可中斷的 Batch、Stateless、Fault-tolerant 工作 | AWS 可回收容量；不適合不能中斷的核心服務 |
| Dedicated Instances | 使用專用硬體但不提供 Dedicated Host 同等實體主機可見性 | 不等於可追蹤特定實體 CPU sockets/cores |
| Dedicated Hosts | BYOL 需實體 socket/core 授權、特定主機追蹤與控制 | 需核對軟體授權條款 |

**重要細節：** 題目若給「3 年、穩定、無須彈性、最高折扣」，且選項只有 Standard RI 和 Compute Savings Plans，通常選 **3-year Standard RI / All Upfront**。不要誤記成「所有 RI 永遠比所有 Savings Plans 便宜」。Spot 可能更便宜，但不能保證不中斷。

## 3. Launching, IAM & Management / 啟動、權限、管理

**Third-grade analogy:** Security Group 是建築物門衛，決定網路封包可否進出；IAM Role 是員工工作證，決定能否讀 S3 文件；Instance Profile 是把工作證交給 EC2 的方式；Session Manager 是不用開外部大門的管理通道。

- **Security Group (SG):** Stateful，作用於 ENI，控制 Allow 規則；不提供 S3 API 的 IAM 授權。
- **Network ACL (NACL):** Stateless，Subnet 層級，可 Allow/Deny，需考慮雙向流量及 ephemeral ports。
- **IAM Role + Instance Profile:** EC2 取得可自動輪替的暫時性憑證；避免在 EC2 儲存 IAM User 長期 Access Keys。
- **Least Privilege:** 例如僅允許指定 Bucket 物件 ARN 的 `s3:GetObject`。Explicit Deny 優先於 Allow。
- **Systems Manager Session Manager:** 無須 inbound SSH/22、公有 IP、Bastion；需 SSM Agent、Instance IAM 權限與對 SSM 服務的網路連線（可用 VPC Interface Endpoints）。
- **SSH Key Pair:** 用於傳統 Linux SSH；SSH 需要網路路徑及適當 SG 規則。Windows 常用 RDP（3389），但也應限制曝露。
- **CloudWatch:** Metrics/Alarms；**CloudTrail:** AWS API 活動稽核；**Systems Manager:** 主機管理與自動化。

**Exam cues:** EC2 reads private S3 without stored keys → IAM Role；private EC2 administration without port 22 → Session Manager；network access → SG/NACL；audit API calls → CloudTrail。

## 4. EC2 Storage / 儲存 — 高優先複習

**Third-grade analogy:** EBS 是放在同一個園區（AZ）的可攜式硬碟；Snapshot 是硬碟的時間點備份；Instance Store 是機器內建的臨時抽屜；EFS 是多台機器都可使用的共享檔案櫃。

| Storage | 特性 | 適用 |
|---|---|---|
| EBS | Block storage；Volume 屬於單一 AZ；可獨立於 EC2 生命週期 | OS/root disk、Database block volumes |
| EBS Snapshot | Point-in-time 增量式備份；可由 Snapshot 在其他 AZ 建立新 Volume | Backup、Cross-AZ restore |
| EBS Multi-Attach | 特定 io1/io2 Volume 可供同 AZ 多台相容 EC2 掛載；須自行處理並行寫入／叢集檔案系統 | 特定共享 Block storage 需求 |
| Instance Store | 高速本機暫存；底層 Instance 停止、休眠或終止時資料可能遺失 | Cache、可重建暫存 |
| EFS | NFS-based managed shared file system，可供多台 Linux EC2 存取 | 共享檔案 |
| S3 | Object storage，不是 EC2 的原生 Block disk | 物件、備份、靜態內容 |

### Snapshot vs Multi-Attach
- **Different AZ + independent copy:** 原 AZ Volume → Snapshot → 在目標 AZ 建立新 EBS Volume → Attach。
- **Incremental backups:** EBS Snapshots；不是 Multi-Attach。
- **Multi-Attach:** 不是 Cross-AZ 分享機制；同 AZ、支援的 Volume 類型與執行個體限制。
- Snapshot 是時間點副本，**不會**讓兩個新舊 Volumes 自動同步。

### DeleteOnTermination vs Termination Protection
- **Termination Protection:** 防止 Instance 被終止（適用其支援的終止路徑）；不是保留已終止 Instance 的資料設定。
- **`DeleteOnTermination=false`:** 允許 Instance 終止後保留指定 EBS Volume。
- Root EBS 一般預設 `DeleteOnTermination=true`；實際以 Block Device Mapping 設定為準。
- 未掛載的 EBS Volume 仍可能收費；Snapshot 也有儲存費。

**Exam cues:** Cross-AZ copy → Snapshot and restore；EC2 terminate but keep root disk → DeleteOnTermination false；protect instance from accidental termination → Termination Protection。

## 5. Networking, Placement & Availability / 網路與可用性

**Third-grade analogy:** ALB 是餐廳接待員，分配客人給健康的廚師（EC2）；ASG 是經理，負責補足廚師；AZ 是不同地點的廚房，避免一個地點停電就全停。

- **VPC/Subnet:** EC2 位於某個 VPC 的某個 AZ Subnet；Public Subnet 具到 Internet Gateway 的適當路由；Private Subnet 不直接對 Internet Gateway 提供入站公開路徑。
- **Public IPv4 / Elastic IP:** Elastic IP 是可保留並重新關聯的靜態公有 IPv4；公有 IPv4 可能產生成本。
- **ALB (Layer 7):** HTTP/HTTPS，Listener → Target Group → 健康 Targets；Host/Path routing。
- **NLB (Layer 4):** TCP/UDP/TLS，適合高效能連線與需要固定 IP 的情境。
- **ASG:** Desired/Min/Max Capacity、Health Checks、Scaling Policies；能替換故障 Instances。
- **Multi-AZ HA:** ALB 跨 AZ，ASG 分散 EC2 並設定足夠容量，避免單一 AZ 故障造成服務全面停止。
- **Placement Groups:** Cluster（低延遲／高吞吐、彼此靠近）；Spread（少量重要 Instances 分散在不同底層硬體；一般每 AZ 每群組最多 7 台運行 Instances）；Partition（大型分散式工作負載，將分區隔離於不同底層硬體集合）。
- **Dedicated Hosts** 不是 Placement Group 策略。

### HA reference architecture
```text
Clients
   |
   v
Application Load Balancer (multi-AZ)
   |                         |
   v                         v
AZ-a: EC2 instance      AZ-b: EC2 instance
   \                         /
    Auto Scaling Group (healthy capacity)
```
單純加大一台 EC2、在同一 AZ 放兩台、或只設定 CloudWatch Alarm 都不足以滿足 Multi-AZ 自動復原要求。

## 6. Wrong-Answer Review / 錯題追蹤（8 題）

以下是本次練習的八次錯答，包含最後一輪複習結果；題目文字以概念摘要為主，不視為原題逐字紀錄。

| Initial checkpoint | Missed concept | Correct decision | Final retest |
|---|---|---|---|
| CP2 Q2 | Standard RI vs Compute Savings Plans | Stable 3-year EC2, highest discount/no flexibility → Standard RI (among given options) | **Still missed** |
| CP2 Q4 | Dedicated Instances vs Hosts | Physical cores/sockets, BYOL, host visibility → Dedicated Hosts | Correct |
| CP3 Q5 | Session Manager vs SSH | No inbound SSH/public IP/bastion → Session Manager | Correct |
| CP3 Q6 | IAM Role vs Security Group | EC2 to private S3 with temporary credentials → IAM Role / Instance Profile | Correct |
| CP4 Q5 | EBS Multi-Attach vs Cross-AZ | Cross-AZ independent EBS copy → Snapshot and new Volume | Correct |
| CP4 Q6 | Incremental EBS backup | EBS Snapshot, not Multi-Attach | Reviewed in CP6 Q5 |
| CP4 Q7 | DeleteOnTermination vs Termination Protection | Terminate EC2 but keep Volume → DeleteOnTermination=false | Correct |
| CP6 Q1 | Standard RI vs Compute Savings Plans | 3-year fixed usage, maximum discount, no flexibility → Standard RI | Review again |

**Caution:** CP4 Q5 and Q6 were distinct original errors around Snapshot/Multi-Attach; CP6 Q5 directly retested the Cross-AZ case and reinforced Snapshot backups. The 8 incorrect answers include a repeat mistake on RI vs Savings Plans.

## 7. Rapid Exam Decision Tree / 30 秒判題

1. **Compute purchase:** Interruptible? → Spot. Stable long-term fixed EC2? → RI / EC2 Instance Savings Plans. Changing family/Region or EC2/Fargate/Lambda? → Compute Savings Plans.
2. **Access:** Need API permission? → IAM. Need network port filtering? → SG. Need private shell without SSH? → Session Manager.
3. **Storage:** Block disk? → EBS. Shared Linux file system? → EFS. Object storage? → S3. New AZ EBS copy? → Snapshot restore.
4. **Resilience:** Instance failure? → ASG. AZ failure? → Multi-AZ. HTTP routing to healthy Instances? → ALB.
5. **Placement:** Low latency → Cluster. Separate critical Instances → Spread. Large distributed system → Partition.
6. **Preservation:** Keep Instance from terminating → Termination Protection. Keep disk after termination → DeleteOnTermination=false.

## 8. Personal AWS Rebuild — Completion Checkpoint

**Worth rebuilding?** Yes, but in a **small, cost-controlled** lab, not the entire architecture at once.

Suggested phases:
1. **Low-cost EC2 basics:** Launch a small eligible instance, attach IAM Role, inspect SG/User Data/metadata, use Session Manager, then terminate.
2. **EBS preservation:** Create small test EBS Volume, take Snapshot, restore into another AZ, verify files, and clean up.
3. **Optional HA:** ALB + ASG across two AZs for a brief scheduled lab only. This can incur meaningful hourly charges; delete promptly.

**LFM connection:** Practice serving a *non-production mock* Love Fills Memory landing page. Do not point real production traffic or modify existing production AWS resources.

**Cost checklist before launch:** Confirm Region, account, EC2 instance price, EBS size/type, Snapshot charges, public IPv4, ALB hourly/LCU charges, NAT Gateway hourly/data charges (avoid for a simple lab where possible), data transfer, and CloudWatch logs. AWS credits do not necessarily cover every charge.

**Cleanup checklist:** Delete ALB/Target Groups, ASG/Launch Templates, test Instances, unattached EBS Volumes, test Snapshots, Elastic IPs, NAT Gateways, and temporary endpoints if created; verify AWS Billing/Cost Explorer after cleanup. Beware ASG will relaunch Instances unless scaled down/deleted.

## 9. Final Checkpoint / 考前自問

- Can I explain why Compute Savings Plans is more flexible than Standard RI?
- Do I recognize socket/core BYOL as Dedicated Hosts?
- Can I distinguish IAM Role, Security Group, and Session Manager?
- Can I restore EBS into another AZ via Snapshot?
- Can I distinguish Multi-Attach from backup?
- Can I retain root EBS after termination?
- Can I design ALB + ASG + Multi-AZ without a single point of failure?

**Next review priority:** 5 targeted questions on EC2 Storage and RI vs Savings Plans; then proceed to Module 6.
