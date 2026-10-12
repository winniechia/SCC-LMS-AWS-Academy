# AWS Academy Cloud Architecting — Capstone Project
**Hands-on record / 實作紀錄 — October 2026**  
**Status:** Core technical validation passed; instructor submission requirements not independently checked.

## 1. Goal / 目標
Build a highly available, scalable web application backed by a relational database.  
建立有高可用性、可擴展、使用關聯式資料庫的網站。

**Request path / 請求路徑:** Browser → Application Load Balancer (ALB) → EC2 (private application subnets) → Amazon RDS MySQL → response.

> **3rd-grade analogy / 三年級比喻:** ALB = restaurant host (接待員); EC2 = chefs (廚師); RDS = recipe library (食譜圖書館); Auto Scaling Group = hiring manager (人事經理); Launch Template = training manual (訓練手冊).

## 2. Architecture / 架構
- **Region:** us-east-1; **Project VPC:** `10.0.0.0/16`.
- **Public subnets:** `10.0.1.0/24`, `10.0.2.0/24` — public-facing ALB.
- **Private application subnets:** `10.0.3.0/24`, `10.0.4.0/24` — EC2 Auto Scaling instances.
- **Private database subnets:** `10.0.5.0/24`, `10.0.6.0/24` — database tier.
- **ALB:** `Capstone-ALB`; **target group:** `Capstone-TG`.
- **ASG:** `Capstone-ASG`; min 2, desired 2, max 4.
- **Launch Template:** `Project-LT`, Version 2 selected explicitly by ASG.
- **Compute:** Amazon Linux 2023, `t2.micro`, Apache/httpd, PHP 8.2, PHP MySQL extension; AWS Systems Manager Session Manager used for access.
- **Database:** Amazon RDS MySQL 8.4.9, database `countries`, table `countrydata_final`, **214 imported rows**.
- **Access:** application security group permits ALB traffic; database security group permits MySQL TCP 3306 from application security group. Do not expose database credentials or secret values in study notes.

### Why these layers? / 為什麼分層？
The ALB accepts incoming requests, the private EC2 tier runs application code, and the database tier stores records. Separating tiers limits unnecessary exposure and supports independent scaling.  
ALB 負責接客；私有 EC2 執行網站；RDS 保存資料。分層可以減少不必要的對外暴露，並讓各層各司其職。

## 3. Implementation checkpoints / 實作步驟
1. **Networking:** configure VPC, public/private application/private DB subnets across two Availability Zones; review route tables and security groups.
2. **Database:** provision RDS MySQL and connect using permitted application-side access.
3. **Data:** place `Countrydatadump.sql` in a private S3 bucket, download it securely on EC2, import it into `countries`, verify **214 rows**. S3 here was an import staging location, not the application database.
4. **Web app:** install Apache/PHP and deploy the AWS Academy `Example.zip` application via Launch Template User Data.
5. **Load balancing:** configure `Capstone-ALB` and `Capstone-TG`, verify healthy instances across two AZs.
6. **Application tests:** Population and GDP worked; Life Expectancy initially failed to render records.
7. **Troubleshoot and fix:** inspect the PHP files, correct the connection variables, test each EC2 directly, and retest through the ALB.
8. **Make fix persistent:** create `Project-LT` Version 2 and update `Capstone-ASG` to explicitly use Version 2.
9. **Scale out and in:** temporarily change desired capacity **2 → 3 → 2**; verify the new instance and final healthy state.

## 4. Troubleshooting case: Life Expectancy / 排錯紀錄
**Symptom / 症狀:** The Life Expectancy page showed only “Pick another query,” although Population and GDP worked and RDS held valid data.

**Evidence / 證據:**
- `query2.php` routes selection `Q3` to the Life Expectancy query.
- `get-parameters.php` supplies `$ep`, `$un`, `$pw`, `$db`.
- Working `gdp.php` uses those variables; broken `lifeexpectancy.php` used `$_SESSION` values. No matching `session_start()` was found during the investigation.
- A direct SQL query for `name, birthrate, lifeexpectancy` returned records; therefore the missing web output was not evidence of missing database rows.

**Before / 修復前:**
```php
$conn = new mysqli($_SESSION['ep'], $_SESSION['un'], $_SESSION['pw'], $_SESSION['db']);
```
**After / 修復後:**
```php
$conn = new mysqli($ep, $un, $pw, $db);
```

**Safe verification / 驗證方式:** Back up `lifeexpectancy.php` before editing; run `php -l`; test locally on each EC2 to bypass ALB routing:
```bash
php -l /var/www/html/lifeexpectancy.php
curl -sS -X POST -d "selection=Q3" http://127.0.0.1/query2.php | grep -o 'Afghanistan'
```
Both original instances returned `Afghanistan` after the fix; the browser query through ALB also worked.

**Lesson / 重點:** A passing syntax check is not the same as a passing runtime/database test. ALB health checks alone do not prove every application feature works.  
語法正確 ≠ 功能正確；Healthy ≠ 所有 SQL 查詢都成功。

**Security follow-up:** During inspection, a diagnostic `error_log` in `get-parameters.php` appeared to include the database password. Remove or redact secret-bearing logging in any persistent deployment; never commit secrets, passwords, full logs, or credentials. This hardening was **not verified as completed** during the Capstone session. Also, malformed HTML such as `<br` was noticed; it is separate from the confirmed DB connection bug.

## 5. Persistent fix and Auto Scaling test / 永久修復與擴縮測試
**Why Version 2? / 為什麼要 Version 2？** Editing an existing EC2 instance does not update the source used for replacement instances. Version 2 adds a `sed` repair step **after** `unzip Example.zip` and **before** starting httpd. The ASG was updated to explicitly select Version 2; changing the template default was not required.

**Observed verification / 實際結果:**
- ASG desired capacity increased from 2 to 3.
- New EC2 `i-05732b52b9e390705` was **InService / Healthy**, launched from **Project-LT Version 2**.
- A local `Q3` POST on that new EC2 returned `Afghanistan`: fresh-instance deployment, PHP, RDS connectivity and Life Expectancy query passed.
- Desired capacity restored to **2**; ASG status **At desired capacity**; two instances **InService / Healthy**, distributed across **us-east-1a** and **us-east-1b**.
- Final instances included one from Version 2 and one originally launched from Version 1 but manually repaired. This is expected: updating ASG template does **not** retroactively relaunch existing instances.

**Important limit / 驗證範圍:** This demonstrates a successful new-instance launch and one scale-in cycle, **not** every failure scenario, automated demand-based scaling policy, RDS Multi-AZ failover, or disaster recovery.

## 6. SAA-C03 review / 考試重點
- **High Availability vs Auto Scaling:** HA maintains service during component failure; Auto Scaling adjusts compute capacity. 高可用 ≠ 自動擴展。
- **ALB vs Target Group:** ALB distributes requests; target group tracks backends and health checks. ALB 分流；Target Group 管理後端與健康狀態。
- **Private subnet vs Security Group:** Routing controls reachability; SG rules control permitted traffic. 路由決定能否到達；SG 決定是否允許。
- **User Data vs manual SSH/SSM edits:** User Data bootstraps replacement instances; manual fixes do not survive replacement unless carried into the deployment artifact/template.
- **RDS vs S3:** RDS executes relational SQL queries; S3 stores objects such as an import dump.
- **Scale-out vs scale-in:** 2 → 3 adds capacity; 3 → 2 removes capacity. AWS may choose which instance to terminate.
- **Functional test vs health check:** Test real business queries as well as infrastructure status.

## 7. Additional study session: Love Fills Memory (LFM) as an analogy / 用 LFM 理解 Capstone
**Important distinction:** The following is a **conceptual mapping**, not a claim that LFM currently runs this Capstone EC2/RDS design. LFM has used a Next.js application and an ECS-based production service; its real deployment architecture should be checked in the LFM repository before making operational changes.

| Capstone step | Imagine the same concern in Love Fills Memory / LFM 對照 |
| --- | --- |
| 1. VPC + subnets | Decide which LFM components should be publicly reachable and which should stay private. 先規劃誰能被外界看到。 |
| 2. ALB | Imagine a front desk receiving readers' requests to open an LFM story. 接待讀者請求。 |
| 3. App EC2 | Equivalent *responsibility* to the LFM web application: render a story, fetch permitted content, handle requests. 職責類似 LFM 網站服務；**not the same compute platform**. |
| 4. RDS | Think of a structured catalog of stories, authors, statuses and timestamps. This is an analogy only; **do not assume LFM uses RDS**. |
| 5. S3 | Store objects such as media or export/import files. For LFM, consider privacy and appropriate bucket access for family memories. |
| 6. Security Groups | Only allow required service-to-service traffic; keep personal memories and credentials protected. |
| 7. Launch Template / User Data | Equivalent *principle* to versioned, reproducible application deployments: a new server/task must get the same fix automatically. LFM's ECS task definitions/deployment pipeline are different mechanisms. |
| 8. ALB health checks | A green status is insufficient if a reader cannot open a story or retrieve an image. Add user-journey checks. |
| 9. Scale-out test | Imagine LFM receiving many readers after a Facebook post; verify a new app replica can serve the same story, not merely start. |
| 10. Scale-in test | After traffic drops, verify capacity returns to normal without breaking story access. |

### Walk-through: “A reader opens Mom's story” / 一位讀者打開媽媽的故事
1. **Request / 請求:** Reader clicks a Love Fills Memory story link. 讀者點擊故事連結。
2. **Routing / 分流:** A front-end routing layer sends the request to an available app replica. 分配給可用的應用程式。
3. **Application / 應用程式:** The app checks the requested story and its publication/access rules. 確認故事與權限。
4. **Data and media / 資料與圖片:** The app retrieves authorized content from its actual configured sources; a database or object store may be involved depending on architecture. 從實際設定的資料來源取回授權內容。
5. **Response / 回應:** The reader sees the story. 讀者看到文章。
6. **Failure / 故障:** A replica is replaced. The replacement must have the right code, environment, IAM permissions and configuration; otherwise a “healthy” process may still fail on real story requests.
7. **Validation / 驗證:** Test an actual published story and relevant images after deployment, while avoiding disclosure of private memories.

### Practice questions / 自我測驗
1. If an LFM app instance looks healthy but stories fail to load, what would you test first beyond the health check?  
   **Answer:** A real story request, application logs without secrets, source connectivity, access permissions and deployment version.
2. Why didn't fixing two Capstone EC2 instances guarantee the next instance would work?  
   **Answer:** New instances are created from Launch Template User Data and the source app archive, not from the modified running servers.
3. If traffic increases after a Facebook story, what is the difference between ALB and Auto Scaling?  
   **Answer:** ALB distributes requests among available targets; Auto Scaling adjusts the number of targets.
4. What is special about caregiving content?  
   **Answer:** Privacy, dignity, permissions and careful handling of personal memories must remain first-class requirements.

## 8. Completion checkpoint / 完成檢查
- **Hands-on technical validation:** PASS for observed Capstone paths and scale-out/scale-in test.
- **Instructor grading/submission:** NOT VERIFIED; check rubric, required screenshots and lab submission.
- **Security hardening:** Follow-up needed for credential-bearing logs and production-quality deployment artifacts.
- **Personal AWS Rebuild? / 值得個人 AWS 重建嗎？** **YES — Very High Learning Value**, but plan first and estimate costs. Rebuild VPC → private EC2/ALB → RDS → versioned deployment → scale-out/in in a personal sandbox; use sanitized sample data, budget alerts, least privilege and cleanup. This is a **future plan, not a completed personal-account build**.
