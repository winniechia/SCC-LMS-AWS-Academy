# AWS SAA-C03 Exam Review — Module 4: Amazon S3

**Module:** Adding a Storage Layer with Amazon S3  
**Language:** English + 繁體中文  
**Study date:** 2026-10-18  
**Checkpoint result:** **36 / 37 (97.3%)** — 6 of 6 checkpoints completed  
**Missed concept:** S3 Lifecycle Rules vs. Cross-Region Replication (CRR)

> **Purpose / 用途**：可列印的雙語考前複習筆記。使用 AWS SAA-C03 題目關鍵字、簡單比喻、架構選擇與易錯題，協助快速辨認最佳 AWS 解決方案。

---

## 1. Amazon S3 Fundamentals / 基本概念

**Amazon Simple Storage Service (Amazon S3)** is **object storage**. It stores objects (data + metadata) inside buckets and identifies them using object keys.  
**Amazon S3 是物件儲存**：Bucket（儲存貯體）中放 Objects（物件），每個物件有 Key（名稱／路徑識別字）。

**三年級比喻 / Analogy：**
- **Bucket** = 圖書館 / Library
- **Object** = 一本書 / Book
- **Key** = 書的索引編號 / Catalog label
- **Region** = 圖書館所在的大城市 / City of the library

| Concept | English | 繁體中文與考試重點 |
|---|---|---|
| Storage model | Object storage | 存檔案物件，不是 EC2 磁碟 |
| Durability | Data is not lost | **耐久性**：S3 Standard 設計為 11 個 9（99.999999999%）|
| Availability | Can access when needed | **可用性**：S3 Standard 設計為 99.99% |
| Multi-AZ | S3 Standard stores redundantly across AZs | 同一 Region 內跨多個 AZ |
| Bucket naming | Globally unique within the applicable bucket namespace | 一般用途 Bucket 名稱需在對應命名範圍內唯一 |
| Object access | Private by default | 預設不公開，避免開放整個 Bucket |
| Storage comparison | S3 vs. EBS vs. EFS | S3 = object；EBS = block；EFS = shared file storage |

**Exam trap / 易錯：** Durability ≠ Availability. **耐久性高**不代表服務永遠不會短暫無法存取。

---

## 2. S3 Storage Classes / 儲存類別與成本

| Storage class | Suitable workload / 適用場景 | Access time / 取回速度 | Exam keyword / 考試線索 |
|---|---|---|---|
| **S3 Standard** | Frequently accessed / 經常讀取 | Milliseconds / 毫秒 | Frequent access |
| **S3 Standard-IA** | Infrequent, but immediate access / 少讀取但需立即存取 | Milliseconds | Rarely accessed + immediate |
| **S3 One Zone-IA** | Infrequent, re-creatable data / 可重建、能容忍單 AZ 風險 | Milliseconds | One AZ acceptable |
| **S3 Intelligent-Tiering** | Unknown/changing access patterns / 存取頻率難預測 | Frequent/IA tiers immediate; archive tiers vary | Unpredictable access |
| **S3 Glacier Instant Retrieval** | Rarely accessed archive with instant retrieval / 少用封存、立即取回 | Milliseconds | Archive + immediate |
| **S3 Glacier Flexible Retrieval** | Archive with flexible retrieval / 可等待取回 | Minutes to hours | Minutes/hours acceptable |
| **S3 Glacier Deep Archive** | Long-term archival / 長期低價封存 | Typically hours, often 12–48 hours | Very low cost + long wait |

**三年級比喻：** Standard 是放書桌上；Standard-IA 是放櫃子裡但隨拿隨用；Deep Archive 是存進遠方倉庫，要提前安排取回。

**Cost considerations / 成本提醒：** 比較 storage price、retrieval fees、minimum storage duration、monitoring/automation fees（若適用）與資料取回時間。不一定「每 GB 最便宜」就是總成本最低。

### Lifecycle Rules / 生命週期規則

**S3 Lifecycle** can automatically transition objects to supported cheaper classes and expire objects or noncurrent versions.  
**Lifecycle 是自動調整儲存狀態與成本**，例如在符合服務轉換限制的前提下，30 天後轉 Standard-IA、長期不用時再封存。

**Exam phrase:** “Automatically move objects to a lower-cost storage class after N days” → **S3 Lifecycle Rules**.

---

## 3. Security & Access Control / 安全與存取

### IAM vs. Bucket Policy / 身分政策與資源政策

- **IAM identity policies** define what users/roles can do. / IAM 決定身分可以做什麼。
- **S3 bucket policies** are resource-based policies. / Bucket Policy 決定誰能對此資源做什麼。
- **Explicit Deny overrides Allow.** / 明確拒絕（Deny）優先於允許（Allow）。
- **Least privilege** means grant only required permissions. / 最小權限，別給不需要的操作能力。
- On **EC2**, use an **IAM role** and temporary credentials, **not hard-coded access keys**. / EC2 程式使用 Role，別把金鑰寫進程式。

| Permission | Purpose / 中文 |
|---|---|
| `s3:GetObject` | Read an object / 讀取 |
| `s3:PutObject` | Upload an object / 上傳 |
| `s3:DeleteObject` | Delete an object / 刪除 |
| `s3:ListBucket` | List keys in bucket / 列出物件 |

### Encryption / 加密

| Choice | Main benefit / 適用情境 |
|---|---|
| **SSE-S3** | S3-managed encryption; default for new S3 uploads / 預設靜態加密 |
| **SSE-KMS** | AWS KMS key policies, key-use control, and CloudTrail records of KMS API use / **金鑰權限控制 + 稽核** |
| **SSE-C** | Customer supplies encryption key with requests / 客戶提供金鑰 |
| **HTTPS/TLS** | Encryption in transit / 傳輸中加密 |

**Exam phrase:** “Control encryption key access + audit usage in CloudTrail” → **SSE-KMS**. Remember KMS permissions may be needed in addition to S3 permissions.

### Keep S3 private / 保持私人

- **S3 Block Public Access** helps prevent accidental public exposure. / 防止誤設公開。
- **Pre-signed URL** offers temporary, scoped access to private objects; protect the URL because holders may use it until expiration. / 私人檔案限時通行證。
- **CloudFront + Origin Access Control (OAC)** caches global website images and securely reads from a private S3 origin. / 全球快取圖片，但 Bucket 不公開。

**三年級比喻：** Bucket 是有鎖的圖書館；IAM 是員工識別證；Bucket Policy 是圖書館規則；pre-signed URL 是限時借閱證；OAC 是讓 CloudFront 代表訪客安全取書的專用通行證。

---

## 4. Versioning, Lifecycle, Replication & Object Lock

| Business need / 需求 | Feature / 正確服務 |
|---|---|
| Recover overwritten content / 防意外覆寫 | **Versioning** |
| Recover from ordinary delete in versioned bucket / 恢復一般刪除 | **Versioning + delete marker behavior** |
| Automatically transition objects to cheaper storage / 自動省成本 | **Lifecycle Rules** |
| Expire old noncurrent versions / 清理舊版本 | **Lifecycle noncurrent version expiration** |
| Automatically copy objects to another Region / 跨 Region 災難備援 | **Cross-Region Replication (CRR)** |
| Replicate within same Region / 同 Region 複寫 | **Same-Region Replication (SRR)** |
| Keep versions immutable for mandated retention / 保存期內不能刪 | **S3 Object Lock** |

### Versioning / 版本控制

Enabling Versioning allows multiple versions of the same object key. An ordinary DELETE on a versioned bucket normally inserts a **delete marker**, instead of erasing all older versions.  
開啟 Versioning 後，相同名稱可以保留新舊版本；一般刪除通常新增 Delete Marker（刪除標記），舊版本仍可能存在。

**Exam keyword:** “Accidental overwrite / Restore previous version” → Versioning.

### Replication / 複寫

- **CRR** copies eligible objects across AWS Regions for DR, compliance, or locality. / 跨區域複製。
- **SRR** copies within one Region. / 同區域複製。
- Source and destination buckets must have **Versioning enabled**. / 來源與目的地需要開啟 Versioning。
- Replication is generally **asynchronous**. Existing data may require **S3 Batch Replication**. / 通常非同步；既有物件可用批次複寫。
- **S3 Replication Time Control (RTC)** provides an SLA for 99.99% of new objects being replicated within 15 minutes; this is **not** an individual-object guarantee. / 15 分鐘是 SLA 指標，不是每一個物件的絕對保證。

**Exam keyword:** “Copy to a different AWS Region” → CRR.

### Object Lock / 物件鎖定

- **Governance mode:** privileged users with appropriate bypass permission can override restrictions. / 特殊授權可繞過。
- **Compliance mode:** protected versions cannot be overwritten or deleted before retention expires, even by root. / 保存期限內連 Root 也不能刪除。
- Object Lock complements Versioning; it is not the same as simply making a bucket private. / Object Lock 用於不可變保留，不等於一般存取權限設定。

**三年級比喻：** Versioning = 保留每次修改的作業；Lifecycle = 把舊書搬去便宜書庫；CRR = 在另一個城市備份一份；Object Lock = 封存後在到期前不能銷毀。

---

## 5. Migration & Performance / 搬遷與效能

| Scenario / 題目情境 | AWS service / 解法 |
|---|---|
| On-prem file share → S3 over network, repeat nightly / 內部檔案夜間定期同步 | **AWS DataSync** |
| Massive offline data transfer, network inadequate / 網路不足、大量離線搬遷 | **AWS Snowball Edge** (traditional exam scenario; verify current service availability for production) |
| Customers globally upload to distant S3 bucket / 全球遠距離上傳 | **S3 Transfer Acceleration** |
| Upload very large objects, retry failed portions / 大檔分段重試 | **Multipart Upload** |
| Global website content delivery and caching / 全球內容快取 | **Amazon CloudFront** |
| Dedicated network connection to AWS / 專線 | **AWS Direct Connect** |

### Multipart Upload / 分段上傳

Splits a large object into parts; unsuccessful parts can be retried independently. AWS recommends considering it for objects around **100 MB or larger**. Unfinished multipart uploads can incur storage costs, so use cleanup/abort rules.  
把大檔拆成數段；失敗的只重傳那一段，並記得清理未完成上傳。

### Transfer Acceleration vs. CloudFront

**Transfer Acceleration** improves transfers between far-away clients and S3 using the AWS edge network.  
**CloudFront** caches content at edge locations for low-latency delivery.  
**前者重點：遠距離傳檔；後者重點：快取與全球配送。**

### DataSync vs. Snowball

**DataSync = high-bandwidth network + recurring/incremental synchronization.**  
**Snowball = offline physical data migration when network transfer is impractical** (classic exam interpretation; verify current product availability for real architecture).

---

## 6. Architecture Exam Scenarios / 綜合架構題

**Scenario 1 — Global storefront images, S3 must remain private / 全球網站產品圖片、S3 不公開**  
**Best:** CloudFront + S3 origin + OAC.  
**Why:** Edge caching reduces latency and repeated origin fetches; OAC authorizes CloudFront to reach the private origin.  
**Trap:** Transfer Acceleration does not replace CDN caching; making buckets public violates requirement.

**Scenario 2 — Financial documents, audited key use / 金融文件，金鑰需審計**  
**Best:** SSE-KMS + appropriate KMS permissions and policies + Block Public Access.  
**Why:** Key-level control and CloudTrail KMS API audit, with access kept private.

**Scenario 3 — 200 TB on premises, fast network, nightly synchronization / 200 TB 高速網路每日同步**  
**Best:** AWS DataSync.  
**Trap:** Snowball is an offline migration tool, not the natural choice for ongoing nightly network synchronization.

**Scenario 4 — Protect overwrites + private + regional DR + transition after 30 days / 防覆寫、私人、異地備援、30 天後省成本**  
**Best:** Versioning + Block Public Access + CRR + Lifecycle → Standard-IA where suitable.  
**Caution:** Evaluate transition eligibility, minimum durations, and replication destination policies.

**Scenario 5 — Must not be deleted before regulatory retention period / 法規要求保存期內不能刪除**  
**Best:** Object Lock Compliance Mode, configured appropriately for protected object versions.

---

## 7. Personal Mistake Review / 本次唯一錯題

**Checkpoint 4 — Lifecycle Rules vs. Cross-Region Replication**

**Question pattern / 題型：** Old objects need to **automatically move into a lower-cost storage class** after a retention period; moving a copy to another Region is **not** required.  
**需求：** 舊資料過一段時間後自動轉到便宜儲存類別，沒有要求另一 Region 的副本。

- **Previous answer / 當時選擇：** CRR ❌
- **Correct / 正確答案：** **S3 Lifecycle Rules** ✅
- **Why / 為什麼：** CRR handles **where copies live**; Lifecycle handles **when objects transition or expire**. / CRR 處理副本位置；Lifecycle 處理自動轉儲存類別及清理。

**Memory cue / 口訣：**  
**Lifecycle = move to save money（轉儲存類別、省錢）**  
**Replication = copy to another place（複製到別處、備援）**

---

## 8. Quick Flashcards / 考前速記

| If the question says... / 關鍵字 | Think... / 答案 |
|---|---|
| Global delivery + caching / 全球內容快取 | CloudFront |
| Private S3 origin through CloudFront / 私人來源 | OAC |
| Far-away upload to S3 / 遠距離上傳 | Transfer Acceleration |
| Large file; retry parts / 大檔分段 | Multipart Upload |
| Frequent on-prem synchronization / 內部資料定期同步 | DataSync |
| Poor network + offline migration / 離線大量搬遷 | Snowball |
| Key policies + CloudTrail KMS audit / 金鑰控制與稽核 | SSE-KMS |
| Prevent public bucket access / 防止公開 | Block Public Access |
| Temporary access to private object / 私人限時分享 | Pre-signed URL |
| Restore overwritten file / 還原被覆寫 | Versioning |
| Automatically lower storage cost / 自動調整成本 | Lifecycle Rules |
| Copy to different Region / 跨區域 | CRR |
| Copy in same Region / 同區域 | SRR |
| Mandatory unchangeable records / 法規不可刪除 | Object Lock Compliance |

---

## 9. Final Checkpoint Record / 六關成績

| Checkpoint | Topic | Correct / Total |
|---|---|---:|
| 1 | S3 Fundamentals | 6 / 6 |
| 2 | Storage Classes & Cost Optimization | 7 / 7 |
| 3 | Security & Access Control | 6 / 6 |
| 4 | Versioning, Lifecycle & Replication | 7 / 8 |
| 5 | Moving Data & Performance | 5 / 5 |
| 6 | Architecture Scenarios & Exam Review | 5 / 5 |
| **TOTAL** | **Module 4 complete / 完成** | **36 / 37 — 97.3%** |

**Next step / 下一步：** Begin Module 5 after reviewing the **Lifecycle vs. CRR** distinction once more.

**Study reminder / 學習提醒：** Always identify the business requirement first: **cost, security, durability, regional DR, retention, or performance**; then select the AWS service that directly solves that requirement. / 先看題目要解決的「成本、安全、耐久、災難備援、法規保留、效能」，再選服務。
