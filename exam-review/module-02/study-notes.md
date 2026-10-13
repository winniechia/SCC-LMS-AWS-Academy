# Module 02 — Introducing Cloud Architecting | AWS SAA-C03 Review

> Study-session review (2026-10-12). These **six instructor-style learning checkpoints and practice questions were created for this review**; they are not represented as verbatim AWS Academy Student Guide questions. Based on the module topics reviewed and linked to completed Academy labs.

## Learning goals / 學習目標
Design AWS solutions that balance availability, performance, security, cost, operational effort, and sustainability. 能根據需求選擇適合的 AWS 架構，而不是只記服務名稱。

## Six checkpoints / 六個學習關卡

### 1. Cloud architecting fundamentals — 雲端架構基礎
- Architecture starts with business requirements and constraints: availability, security, performance, budget, operations.
- **Scenario:** Application must survive an AZ failure → **Deploy across multiple Availability Zones** (with sufficient capacity and failover design).
- **3rd-grade analogy / 三年級比喻:** A restaurant with two kitchens in separate buildings can keep serving when one building loses power.
- **Exam trap:** A larger EC2 instance is not a substitute for AZ redundancy.

### 2. AWS global infrastructure — 全球基礎設施
- **Region:** geographic AWS area / 地理區域.
- **Availability Zone (AZ):** isolated infrastructure location within a Region / 區域內彼此隔離的設施.
- **Edge Location:** nearby point of presence used for content delivery / 靠近使用者的內容傳遞節點.
- **Scenario:** Website origin in Oregon; users in Asia need faster delivery without migrating the application → **Amazon CloudFront**, caching eligible content at edge locations.
- **3rd-grade analogy:** Region = city; AZ = separate libraries; CloudFront edge = neighborhood book pickup station.
- **Exam trap:** **Route 53** resolves/routes DNS; **CloudFront** is a CDN. CloudFront does not move the entire application to the edge. Multi-AZ and Multi-Region are different designs.

### 3. AWS Well-Architected Framework — 六大支柱
| Pillar | 中文 | Guiding question |
| --- | --- | --- |
| Operational Excellence | 卓越營運 | Can we operate, observe, and improve the workload? |
| Security | 安全性 | Are data and systems protected? |
| Reliability | 可靠性 | Can the workload recover from failures? |
| Performance Efficiency | 效能效率 | Are resources matched to demand? |
| Cost Optimization | 成本最佳化 | Are we avoiding unnecessary spend? |
| Sustainability | 永續性 | Are we reducing environmental impact? |

- **Scenario:** Automatically adjust EC2 capacity as traffic changes → **Performance Efficiency** is the *most directly emphasized* pillar.
- **3rd-grade analogy:** A school needs secure doors (Security), spare classrooms (Reliability), the right room size (Performance), and sensible spending (Cost).
- **Exam trap:** Auto Scaling can also support Reliability, Cost Optimization, and Sustainability. Choose the pillar that best matches the wording of the question.

### 4. Security, reliability, and performance — 安全、可靠與效能
- **Scenario:** Critical application must survive an entire AZ outage with minimal manual intervention → **Multi-AZ Auto Scaling Group + Application Load Balancer**, correctly configured with health checks and enough capacity.
- **ALB:** routes traffic to healthy targets / 接待員.
- **Target Group:** list of application targets / 可接待客人的名單.
- **ASG:** maintains/replaces instances and adjusts capacity / 人力調度經理.
- **Multi-AZ:** removes a single-AZ failure dependency / 多個地點的餐廳.
- **Exam trap:** High Availability ≠ Load Balancing ≠ Scaling. RDS Multi-AZ DB instance standby is for availability/failover, not general read scaling.
- **Lab connection:** Lab 10 high-availability architecture.

### 5. Managed services and cost — 託管服務與成本
- **EC2 instance:** virtual server rented in AWS; customer manages guest OS, installed software, and application / AWS 租給你的虛擬電腦.
- **Amazon RDS for MySQL:** managed relational database; AWS handles many infrastructure and maintenance tasks, while customer still manages data, access, configuration, and appropriate security.
- **Scenario:** MySQL on EC2; reduce OS maintenance, backup administration, and infrastructure work while keeping relational MySQL → **Amazon RDS for MySQL**.
- **3rd-grade analogy:** EC2 + MySQL = rent a kitchen and maintain its operation; RDS = managed kitchen with many maintenance chores handled for you.
- **Exam trap:** Managed ≠ always lowest direct AWS bill. Consider **Total Cost of Ownership (TCO)**, including staffing and maintenance.
- **EC2 billing trap:** Stopped EC2 usually stops instance compute charges, but EBS and other retained resources can still incur charges; termination does not necessarily delete every associated resource.
- **Lab connection:** Lab 06 Amazon RDS.

### 6. Infrastructure automation and disposable resources — 自動化與可替換資源
- **Infrastructure as Code (IaC):** infrastructure defined in repeatable, version-controlled templates.
- **Scenario:** Inconsistent environments and slow manual deployments; want repeatable, version-controlled provisioning → **AWS CloudFormation**.
- **CloudFormation Template:** blueprint / 施工圖.
- **CloudFormation Stack:** deployed collection of managed resources / 依施工圖建立的一組資源.
- **Disposable resources:** design compute instances to be replaced rather than manually repaired; keep durable data in appropriate persistent storage.
- **3rd-grade analogy:** A restaurant blueprint can recreate the same layout at different locations; a broken standardized work station can be replaced.
- **Exam trap:** CloudWatch = monitoring, CloudFront = CDN, Trusted Advisor = recommendations; CloudFormation = IaC.
- **Lab connections:** Lab 11 CloudFormation; Lab 10 ASG; Lab 13 SQS decoupling.

## Completed review answers / 本次練習作答
| Checkpoint | Scenario concept | Learner answer | Result |
| --- | --- | --- | --- |
| 1 | Survive AZ failure | B — Multi-AZ | Correct |
| 2 | Accelerate Asia users without origin migration | B — CloudFront / Edge Location | Correct |
| 3 | Auto Scaling and efficient capacity | C — Performance Efficiency | Correct |
| 4 | Multi-AZ resilience with minimal intervention | C — ASG across AZs + ALB | Correct |
| 5 | Reduce MySQL operational overhead | B — Amazon RDS for MySQL | Correct |
| 6 | Repeatable, version-controlled infrastructure | B — AWS CloudFormation / IaC | Correct |

**Review result: 6/6 correct.** This is a formative concept review, **not** an official AWS Academy knowledge-check score or a predictor of SAA-C03 readiness.

## SAA-C03 quick recall / 考前速記
- AZ failure → Multi-AZ + tested failover; not merely a bigger EC2.
- Global cached content → CloudFront Edge Locations.
- Traffic-driven capacity → EC2 Auto Scaling.
- Healthy traffic routing → ALB.
- Managed relational MySQL → RDS for MySQL.
- Reproducible infrastructure → CloudFormation.
- Reduce operational overhead ≠ automatically minimize service charges.
- Reliability, scaling, and performance overlap but are not synonyms.

## Personal AWS rebuild checkpoint / 個人帳號重建評估
**No deployment needed for Module 2 itself.** Apply these design principles when planning later personal labs. Before any deployment, obtain explicit approval, estimate costs, configure budget alerts, and plan resource cleanup. 本章只做觀念複習，不建立付費資源。

## Suggested next review
Compare Regions vs AZs vs Edge Locations; test scenario-based distinctions among the six pillars; review Shared Responsibility and Well-Architected trade-offs.


## Additional review: Local Zones, Outposts, Wavelength, and CloudFront / 延伸複習

> These are **six original SAA-style practice questions** from our study session, not official AWS exam questions. 本節為自編情境練習，非 AWS 官方考題。

### Key concepts / 核心概念

- **AWS Local Zones — 城市附近的運算:** An extension of a parent AWS Region that places supported compute and other services near users in a metropolitan area. Typical clue: **single-digit-millisecond latency + compute near a city**. Service and instance availability is limited compared with a Region. / 延伸母區域，讓支援的運算服務更靠近城市使用者；並非完整獨立 Region。
- **Amazon CloudFront and Edge Locations — 內容靠近使用者:** CDN that caches and delivers eligible content from points of presence worldwide. Typical clue: **global website images, videos, static content, caching**. / 全球內容傳遞與快取；不等於把整個應用程式搬到 Edge Location。
- **AWS Outposts — AWS 在企業現場:** AWS-managed infrastructure installed at a customer's on-premises location for supported workloads. Typical clue: **on-premises + AWS infrastructure + local processing**. Some local workloads can continue during a temporary Region connectivity disruption, but management and certain services depend on connectivity. / AWS 設備設於客戶場所；不是完全離線的獨立 Region。
- **AWS Wavelength — 電信網路附近的運算:** Supported AWS services integrated with telecommunications providers' networks for low-latency mobile applications, including 5G scenarios. Typical clue: **5G mobile network + low-latency application compute**. / 適合需靠近電信網路的低延遲行動應用。
- **Multi-AZ + ALB — 高可用性:** Run application targets across multiple Availability Zones and route requests to healthy targets. Configure health checks and sufficient capacity for failover. / 跨 AZ 部署與健康檢查，以承受單一 AZ 故障。

### 3rd-grade analogy / 三年級比喻
Imagine a restaurant: **Region** = central kitchen; **AZs** = independent kitchens in separate buildings; **Local Zone** = AWS-operated kitchen nearer customers; **Outposts** = AWS installs a kitchen inside your restaurant; **Wavelength** = a kitchen integrated into a mobile carrier's delivery network; **CloudFront** = nearby pickup counters stocked with popular ready-made items. / 中央廚房、不同廠房、附近分店、店內設備、電信網路服務站，以及附近的取貨點，各自解決不同問題。

### Practice questions / 練習題（先作答，再看答案）

**Q1.** A real-time game hosted in Oregon has players in Los Angeles. The game requires interactive server-side compute close to players. Which is best? **A** CloudFront image caching; **B** Local Zone near Los Angeles; **C** S3 Cross-Region Replication; **D** More AZs in Oregon.

**Q2.** A global e-commerce site needs faster image delivery to users in Asia and Europe without deploying EC2 everywhere. Which is best? **A** EC2 in Local Zones worldwide; **B** CloudFront; **C** Three AZs in one Region; **D** Direct Connect for all visitors.

**Q3.** Physicians interact with remote 3D medical-imaging workstations and need compute nearer their hospital to reduce input lag. Which is best? **A** CloudFront; **B** Local Zones; **C** S3 Transfer Acceleration; **D** Route 53.

**Q4.** A factory requires AWS infrastructure physically on-premises for low-latency robotic control, with sometimes-unreliable network connectivity. Which is best? **A** Local Zones; **B** CloudFront; **C** Outposts; **D** Global Accelerator.

**Q5.** An AR mobile application needs low-latency processing integrated with a telecommunications provider's 5G network. Which is best? **A** Local Zones; **B** Outposts; **C** Wavelength; **D** CloudFront.

**Q6.** A global website must survive one AZ failure and load images quickly worldwide. Which combination is best? **A** Single AZ + Local Zones; **B** Multi-AZ EC2 behind ALB + CloudFront; **C** Outposts in every country; **D** Wavelength + S3 Cross-Region Replication.

### Answers and explanations / 答案與解析

1. **B — Local Zones.** Compute must be near city users; caching images, S3 replication, or additional AZs in Oregon do not address interactive server latency. / 需要靠近玩家的運算，而非僅快取圖片。
2. **B — CloudFront.** Caching images at edge locations improves worldwide delivery without running EC2 in every geography. / 全球圖片傳遞首選 CDN。
3. **B — Local Zones.** Remote workstations require nearby interactive compute, not static-content caching, S3 transfer, or DNS alone. / 需要低延遲遠端運算。
4. **C — Outposts.** The explicit **on-premises** requirement distinguishes Outposts from AWS-operated Local Zones. Connectivity and supported-service limitations still apply. / 設備必須在工廠內。
5. **C — Wavelength.** The distinguishing clue is compute integrated with a **telecom 5G network**. / 關鍵是電信業者的 5G 網路。
6. **B — Multi-AZ EC2 + ALB + CloudFront.** Multi-AZ targets and health-aware routing support availability; CloudFront accelerates global image delivery. Consider ASG for automatic replacement and scaling. / 跨 AZ 容錯與全球內容加速要分別解決。

**Session result / 本輪成績: 6/6 correct.** This is practice performance, not an official exam readiness score. / 練習全對，但不等於正式考試保證通過。

### Exam traps / 考試陷阱

- **Low latency** alone is not enough: identify *what* needs to be close — compute, cached content, mobile-network processing, or on-premises infrastructure.
- **Local Zone ≠ AZ:** a Local Zone targets proximity; Multi-AZ architecture targets fault isolation and resilience.
- **CloudFront ≠ Global Accelerator:** CloudFront is a CDN with caching; Global Accelerator improves routing to application endpoints using the AWS global network and static anycast IP addresses.
- **Outposts ≠ fully disconnected AWS Region:** review connectivity and service constraints.
- **Wavelength ≠ all 5G traffic automatically needs Wavelength:** the workload must benefit from compute in a supported telecom-network location.

### Learning checkpoint / 學習檢查

- [x] Distinguish Local Zones from CloudFront / 分辨運算與內容快取。
- [x] Recognize Outposts from on-premises requirements / 辨識企業現場部署。
- [x] Recognize Wavelength from telecom/5G clues / 辨識電信網路情境。
- [x] Combine Multi-AZ + ALB + CloudFront / 結合可用性與全球加速。
- [ ] Next: Compare **CloudFront vs AWS Global Accelerator** with scenario questions / 下一步比較兩者。

**Print note / 列印提醒:** Markdown-first; no PDF required. Answers are separated from the question block so they can be covered while practicing. / 採 Markdown，題目與解析分段，方便從 GitHub 列印後遮住答案練習。


## Additional practice: CloudFront vs. AWS Global Accelerator / 全球加速服務比較

> Six original SAA-style practice scenarios from this study session, not official AWS exam questions. 本節為自編題目，非官方考題。

### Core distinction / 核心差異

- **Amazon CloudFront** is a content delivery network (CDN). It caches eligible HTTP/HTTPS content at edge locations and can also forward and accelerate dynamic HTTP/HTTPS requests without caching them. / CloudFront 是 CDN，可快取靜態內容，也能轉送不快取的動態 HTTP/HTTPS 請求。
- **AWS Global Accelerator** provides static anycast IP addresses and routes supported TCP/UDP application traffic over the AWS global network to healthy regional endpoints. It does **not** cache application content. / Global Accelerator 提供靜態 Anycast IP，改善 TCP/UDP 到健康端點的網路路徑，不快取內容。
- **Amazon Route 53** provides DNS resolution and routing policies, including health-check-based DNS failover. DNS caching/TTL can affect how quickly clients observe changes. / Route 53 是 DNS 服務，支援 DNS 故障轉移，但可能受到 DNS 快取影響。
- **Amazon S3 Transfer Acceleration** improves long-distance object transfers to/from S3 using AWS edge infrastructure; it is not a general-purpose application accelerator. / S3 Transfer Acceleration 專注 S3 物件傳輸。

### 3rd-grade analogy / 三年級比喻
CloudFront is a neighborhood pickup counter stocked with popular goods; Global Accelerator is an efficient highway entrance that guides customers to an operating store; Route 53 is the directory that tells customers where a store is. / CloudFront 是附近取貨站；Global Accelerator 是智慧高速公路入口；Route 53 是查詢地址的電話簿。

### Practice questions / 練習題

**Q1.** A worldwide multiplayer game uses UDP and requires static IP addresses and optimized network routes. Best service? A CloudFront; **B Global Accelerator**; C Route 53; D Local Zones.

**Q2.** An application has endpoints in two Regions, uses TCP, requires static IPs, health-based failover, and optimized global routing. Best service? A Route 53 failover; B CloudFront; **C Global Accelerator**; D Direct Connect.

**Q3.** Images and videos stored in S3 should load quickly worldwide while reducing repeat origin requests. Best service? A Global Accelerator; **B CloudFront**; C Route 53; D Local Zones.

**Q4.** A financial application uses TCP, requires static anycast IPs, does not need caching, and wants improved global routing. Best service? A CloudFront; B Route 53; **C Global Accelerator**; D S3 Transfer Acceleration.

**Q5.** A global HTTPS website wants to cache /images/* at edge locations but forward /api/* requests to the origin. Best single service? A Global Accelerator; **B CloudFront**; C Route 53; D Direct Connect.

**Q6.** A TCP application requires fixed IPs, automatic failover between healthy endpoints in different Regions, optimized global routes, and no content caching. Best solution? A CloudFront multiple origins; B Route 53 failover; **C Global Accelerator with multi-Region endpoints**; D S3 Transfer Acceleration.

### Answers and explanations / 答案與解析

1. **B — Global Accelerator:** UDP + static anycast IPs + global network optimization. CloudFront is not a general UDP gaming accelerator. / UDP 與固定 Anycast IP 是線索。
2. **C — Global Accelerator:** Static IPs and AWS-network-based endpoint failover distinguish it from DNS-based Route 53 failover. Existing TCP connections may still need reconnecting after endpoint failure. / 靜態 IP 與端點故障轉移，不等於既有連線不中斷。
3. **B — CloudFront:** Caches eligible S3 content at edge locations, improving delivery and reducing repeated origin fetches. / 快取圖片與影片，減少回源請求。
4. **C — Global Accelerator:** TCP + static anycast IPs + no caching. Note CloudFront can still accelerate dynamic HTTP/HTTPS, but lacks this static anycast IP access model. / 不要只因為動態內容就排除 CloudFront；此題的固定 IP 才是關鍵。
5. **B — CloudFront:** Cache behaviors can apply different policies and origins to /images/* and /api/*; dynamic requests can pass through without caching. / 可依路徑設定不同快取行為。
6. **C — Global Accelerator:** Regional endpoints, health-aware traffic routing, static IPs, and global network optimization match all requirements. Route 53 provides DNS failover but not the same static anycast IP entry point. / 同時滿足跨區健康端點、固定 IP 與網路加速。

**Learning result / 本輪成績: 6/6 correct.** This reflects these practice questions only, not a full SAA-C03 diagnostic. / 六題全對，並非正式模擬考成績。

### Exam memory cues / 考前速記

- **Cache images / reduce origin requests → CloudFront.**
- **HTTP/HTTPS dynamic requests with path-based cache behaviors → CloudFront.**
- **TCP or UDP + static anycast IP + global network optimization → Global Accelerator.**
- **DNS resolution or DNS-based failover → Route 53.**
- **Long-distance S3 uploads → S3 Transfer Acceleration.**

### Learning checkpoint / 學習檢查

- [x] Identify CloudFront for static caching / 靜態快取。
- [x] Recognize CloudFront can forward dynamic requests / 動態請求也能轉送。
- [x] Identify Global Accelerator from TCP/UDP + static anycast IP / 辨認網路加速。
- [x] Distinguish DNS failover from endpoint traffic routing / 分辨 DNS 與流量路由。
- [ ] Review more ambiguous real-exam-style mixed scenarios / 後續挑戰混合情境。


## Final review: Shared Responsibility and Well-Architected / 最終複習

> Original formative SAA-style questions discussed on 2026-10-12, not official AWS Academy or certification questions. / 自編練習題，非官方題庫。

### Shared Responsibility Model / 共同責任模型

- **AWS: security OF the cloud / 雲端本身的安全** — physical data centers, hardware, foundational networking and virtualization infrastructure.
- **Customer: security IN the cloud / 雲端內部的安全** — data, IAM permissions, application security, and relevant resource configurations. Specific responsibilities depend on service type.
- **EC2:** customer patches the guest OS; AWS maintains underlying physical infrastructure. / EC2 客戶修補 Guest OS。
- **RDS:** AWS manages the underlying OS and supported managed maintenance; customers still control data, users, permissions, and application/database design. / RDS 底層 OS 由 AWS 管理。
- **S3:** AWS operates storage infrastructure; customers configure IAM and bucket/object access, including public-access protections. / S3 存取政策由客戶管理。
- **3rd-grade analogy / 三年級比喻:** AWS is the building owner responsible for the building structure; the tenant must lock their own room and control the keys. / AWS 管大樓結構，房客管自己的門鎖與鑰匙。

**Shared Responsibility practice (4/4 correct):**
1. Who patches an EC2 guest OS? **B — Customer.** / EC2 Guest OS 由客戶修補。
2. After migrating MySQL from EC2 to RDS, who patches the underlying OS? **C — AWS.** / RDS 底層 OS 由 AWS 管理。
3. Who configures S3 bucket policies? **C — Customer.** / S3 Bucket Policy 由客戶設定。
4. Who maintains AWS data center physical security? **C — AWS.** / 資料中心實體安全由 AWS 負責。

### Well-Architected Framework / 六大支柱

| Pillar | 中文 | Scenario clue / 情境線索 |
| --- | --- | --- |
| Operational Excellence | 卓越營運 | CloudFormation, repeatable deployments, automation / 自動化與標準化 |
| Security | 安全性 | IAM, encryption, least privilege / 權限與加密 |
| Reliability | 可靠性 | Multi-AZ, fault tolerance, recovery / 跨 AZ 容錯 |
| Performance Efficiency | 效能效率 | Scale capacity with demand / 依需求調整資源 |
| Cost Optimization | 成本最佳化 | Right-size, eliminate waste / 降低不必要成本 |
| Sustainability | 永續性 | Reduce environmental footprint / 減少環境影響 |

**3rd-grade analogy / 三年級比喻:** Running a school: repeatable procedures (Operations), secure gates (Security), backup classrooms (Reliability), enough seats for students (Performance), avoid renting empty rooms (Cost), and save energy (Sustainability). / 學校的流程、門禁、備用教室、座位、預算與節能。

**Final knowledge check (5/5 correct):**
1. Scale EC2 capacity up/down with traffic → **C — Performance Efficiency**.
2. Survive an AZ outage with multi-AZ targets and ALB → **B — Reliability**.
3. Right-size chronically underused EC2 to lower bills → **C — Cost Optimization**.
4. Use CloudFormation to standardize and automate deployment → **A — Operational Excellence**.
5. Reduce energy use and environmental impact → **C — Sustainability**.

**Exam trap / 考試陷阱:** One design choice can support several pillars. Choose the **MOST directly emphasized** business requirement: scaling for speed (Performance), recovery (Reliability), spending (Cost), or environmental impact (Sustainability). / 同一做法可能對應多個支柱，依題目主要目的判斷。

### Module 02 completion checkpoint / 模組完成檢查

- [x] Six foundational instructor-style checkpoints: **6/6**.
- [x] Local Zones / Outposts / Wavelength / CloudFront scenarios: **6/6**.
- [x] CloudFront / Global Accelerator scenarios: **6/6**.
- [x] Shared Responsibility scenarios: **4/4**.
- [x] Well-Architected final check: **5/5**.
- **Total original formative practice: 27/27 correct.** Not an official exam result or readiness guarantee. / 共 27 題自編練習全對，不代表正式考試成績。
- [x] Module 02 review completed; next: Module 03. / Module 02 複習完成，下一步 Module 03。
- **Personal AWS rebuild:** Not started. Requires explicit user approval, cost estimate, budget alerts, and cleanup plan before deployment. / 尚未啟動個人 AWS 重建。
