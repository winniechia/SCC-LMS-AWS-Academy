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
