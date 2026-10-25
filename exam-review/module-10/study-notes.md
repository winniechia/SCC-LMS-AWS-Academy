# Module 10 — Implementing Monitoring, Elasticity, and High Availability
AWS SAA-C03 | 中英雙語考前筆記 | Completed 2026-10-25

> Learning method / 學習方式：三年級比喻 → 原理與用途 → exam keywords 與取捨 → 排除選項 → 一次一題。
> Source / 教材：Module 10 handoff and uploaded screen recording, `Student guide - Google Chrome 2026-10-08 17-55-47.mp4` (344.1104 seconds, about 5:44). Both attachments opened. The handoff was read completely. Sampled video frames confirmed the title, objectives, monitoring, scaling, database scaling, ELB, Route 53, and Well-Architected section. Fast scrolling, blank frames, and partially visible pages prevented a complete reading of all 90 pages. These notes record actual tutoring, not a complete guide transcription.
> No AWS resources were created. No hands-on labs were executed or verified in this session. 教學結束不等於實作 Lab 完成。
> Destination: `winniechia/SCC-LMS-AWS-Academy/exam-review/module-10/study-notes.md`. Module 9 notes were read for format; Module 10 returned 404 before creation. No PDF requested or created.

## 0. Results and Scoring Audit / 成績與計分核對

| Category / 類別 | Correct / Total | Result |
|---|---:|---:|
| Original scored questions Q1–Q19 / 原始題 | 15/19 | 78.9% |
| Valid scored final-review items Q22–Q23 / 有效最後複習題 | 2/2 | 100% |
| Scored targeted retests / 計分重測 | 0/0 | Not administered |
| **Cumulative guided practice / 累計引導練習** | **17/21** | **81.0%** |
| Unscored responses / 不計分作答 | 4/6 correct | Excluded |

- This was guided practice, with immediately preceding explanations and scaffolded checks, not an independent mock exam. The score does not prove independent mastery or predict certification results.
- Four final-review slots were promised and completed: Q20–Q23. Q20 and Q21 repeated disclosed concepts too closely to qualify as fresh scored retests. Both attempted scored classifications were voided; the responses remain documented as unscored review. Q22–Q23 count once in the cumulative total.
- Q20 and Q21 were answered incorrectly, but excluded because of assistant question-design problems. They remain useful learning evidence, not counted learner errors.
- Four additional unnumbered checks were all correct after explanations. Correct scaffolded responses are not independent retests.
- Q5 input `+b` was interpreted as B and counted once. No other invalid or unanswered input was counted.
- No questions were added after the agreed four final slots.
- Initial estimate: 95–125 minutes for tutoring; checkpoint estimates were planning estimates, not measured duration.

### Coverage / 教學範圍

| Checkpoint | Actual tutoring |
|---|---|
| Monitoring | Metrics, logs, events, CloudWatch alarms/SNS, EventBridge, CloudWatch agent vs CloudTrail |
| EC2 scaling | Elasticity, vertical/horizontal scaling, min/desired/max, launch templates, health integration, scheduled/target tracking/predictive distinctions |
| Databases | Vertical scaling, read replicas, traditional Multi-AZ DB instance standby, combined read scaling and HA |
| Load balancing | ALB path rules, NLB TCP/static IP, GWLB appliance use, capacity after AZ failure |
| Availability | Route 53 failover/cache, stateless compute and shared sessions |
| Final review | Four slots, two excluded repeats and two valid integrated questions |

## 1. Metrics, Logs, and Events / 指標、日誌與事件

**Third-grade analogy / 三年級比喻：**火車站每分鐘數乘客、記錄售票機錯誤、通知某台機器停止運作，是三種不同資訊。

| Type | Example / 例子 | Meaning |
|---|---|---|
| Metric / 指標 | CPU 82%; 300 waiting passengers | Numerical measurement over time / 隨時間變化的數值 |
| Log / 日誌 | 09:05 payment failed, order 123 | Detailed application/system record / 詳細紀錄 |
| Event / 事件 | EC2 changed to stopped | Something happened / 發生狀態改變等事情 |

### CloudWatch alarm and SNS / 警報與通知

CloudWatch alarm evaluates a metric using configured statistics, periods, evaluation rules, and a threshold. SNS = Simple Notification Service / 簡單通知服務, used here for email notifications.

**Analogy:** 測量候車人數 → 超過規則門檻 → 通知站長。Measurement → condition evaluation → configured action.

Q1 used average CPU >80% for each of three consecutive five-minute periods. An alarm can be configured for that condition and notify SNS. It does not invent a scaling policy automatically.

**Keywords:** CPU, threshold, consecutive periods, notification.
**Elimination:** Match a numerical monitoring requirement to metrics and alarms; API auditing and configuration compliance answer different questions.

### EventBridge / 事件路由

EventBridge rules match events and route them to configured targets, such as Lambda. Q2 matched EC2 state-change events for stopped, avoiding repeated polling.

**Analogy:** 接線員收到售票機故障通知，依通知內容把工作交給維修員。
**English:** EventBridge routes events; it is not a substitute for collecting guest operating-system measurements.

## 2. CloudWatch Agent vs CloudTrail / 測量與操作稽核

**Classroom analogy / 教室比喻：**
- CloudWatch agent = 教室內測量員、送件員。
- CloudWatch alarm = 根據數值判斷門檻的警報器。
- EventBridge = 收到事件後轉交工作的接線員。
- CloudTrail = 記錄誰對 AWS 做了什麼操作的紀錄簿。

| Requirement / 需求 | Appropriate approach / 做法 |
|---|---|
| EC2 CPU and network metrics | Standard EC2 metrics in CloudWatch |
| Guest memory utilization or filesystem free space | Additional collection, commonly CloudWatch agent |
| Search application logs after EC2 termination | Send local log records to CloudWatch Logs, e.g. using agent |
| Who terminated an EC2 instance? | CloudTrail AWS activity records |
| Invoke Lambda on EC2 stopped event | EventBridge rule |
| Check resource configuration compliance | AWS Config |

**Important / 重要：**
- EC2 detailed monitoring changes the frequency of standard metrics; it does not automatically add guest memory utilization or collect application log files.
- The agent must be installed/configured with appropriate authorization; naming it alone does not establish a working deployment.
- Already ingested CloudWatch Logs records can outlive the EC2 instance, subject to retention and deletion settings. Unsent local records are not guaranteed preserved.
- CloudTrail does not read the contents of an application's local log file.
- There is no default EC2 memory-pressure event that satisfies Q3's memory-percentage monitoring requirement.
- EventBridge can route an alarm state-change event in a designed workflow, but cannot replace the required collection/evaluation.

**Memory cue / 口訣：**CloudTrail 查操作；CloudWatch agent 量記憶體、送日誌。
**English:** Audit actions with CloudTrail; collect guest measurements and local logs with an agent.

**Learning evidence:** Q3 D and Q4 B were incorrect. Simple checks afterward were correct, but Q20 B again selected CloudTrail for memory monitoring. Keep this distinction as a priority.

## 3. Elasticity and Capacity / 彈性與容量

**Ticket-window analogy / 售票窗口比喻：**
- Scale out / 水平擴展：加開窗口，加 EC2。
- Scale in / 水平縮減：關閉多餘窗口，減 EC2。
- Scale up / 垂直擴展：把一個窗口換成能力更強的窗口，升級 instance 規格。

Elasticity adjusts capacity as demand changes. EC2 Auto Scaling adjusts instance count, subject to configured policies and capacity bounds.

| Setting | Example | Meaning / 意義 |
|---|---:|---|
| Minimum | 2 | Capacity floor / 下限 |
| Desired | 3 | Current target / 目前希望維持的容量 |
| Maximum | 6 | Capacity ceiling / 上限 |

Q6 explicitly used one capacity unit per instance, without weighted-instance complications. A policy requesting eight does not authorize exceeding maximum six. Actual capacity may temporarily differ from desired during launches/replacement. Do not treat desired as an instantaneous guarantee.

**Keywords:** add/remove instances, variable demand, minimum, desired, maximum.
**Elimination:** More powerful instances are vertical scaling; CloudTrail records actions; Route 53 chooses DNS answers.

## 4. Launch Templates and Health / 啟動範本與健康

### Launch template / 啟動範本

**Analogy:** 加開窗口的設備清單：電腦規格、軟體、安全規則與開機設定。

A launch template can define AMI (Amazon Machine Image), instance type, security groups, and user data scripts. The Auto Scaling group defines capacity and placement configuration such as selected subnets.

**Question to ask:** How should a new machine be built, or how many machines are needed?
**English:** Launch template = instance configuration; scaling policy = capacity adjustment.

### ALB versus Auto Scaling health / 分流與更換

**Analogy:** 引導員避開壞窗口；主管補上新窗口。
- ALB stops routing to unhealthy targets when healthy alternatives exist in the scenario.
- Auto Scaling maintains capacity and replaces instances determined unhealthy.
- EC2 running/status checks do not establish that the web application works.
- Enable Elastic Load Balancing health checks in the Auto Scaling group to use the load balancer's health assessment for replacement.
- Configure startup readiness and health-check grace periods appropriately; immediate replacement during normal startup can be harmful. No live grace-period setting was tested.

**Keywords:** EC2 healthy, ALB unhealthy, not replaced.
**Elimination:** Increasing maximum capacity or metric frequency does not enable the health integration.

## 5. Scaling Policies / 擴展策略

| Approach | Ticket-window analogy / 比喻 | Fit / 適用 |
|---|---|---|
| Target tracking | 根據忙碌程度增減窗口，維持目標 | Maintain a suitable metric near a target |
| Scheduled scaling | 已知每天九點有旅行團，提前開窗口 | Known timetable |
| Predictive scaling | 依歷史人潮預測，提前準備窗口 | Forecast recurring demand |

Target tracking with average CPU target 50% tries to move that metric toward 50%; 50% is not a hard ceiling or an exact continuous guarantee. Metric choice matters: in Q10 CPU reflected demand and work was spread evenly.

Scheduled scaling must allow enough launch/readiness time. Scheduling a change at a time does not guarantee all instances become ready instantaneously. Dynamic scaling can complement a schedule for unexpected demand.

**Keywords:** fixed time → scheduled; maintain target → target tracking; forecast historical pattern → predictive.
**Trade-off:** Preprovisioning costs more during spare periods; purely reactive scaling can require startup time.

## 6. RDS Scaling versus Availability / 讀取容量與備援

**Library analogy / 圖書館比喻：**
- Stronger librarian = larger DB instance / 垂直擴展。
- Extra query librarians with book-list copies = read replicas / 讀取分攤。
- Backup librarian in another building = standby for failover / 備援。

| Requirement | Design | Trade-off |
|---|---|---|
| More capacity on one database | Larger DB instance | Resizing and engine-specific limits/operational effects |
| Offload read-only reports | Read replica; route reads to replica endpoint | Replication lag possible |
| Automatic failover for DB/AZ failure | Multi-AZ DB instance deployment | Added cost; failover interruption/reconnect |
| Both failover and reporting offload | Multi-AZ DB instance primary/standby plus read replica | Separate mechanisms, added resources |

For the traditional **Multi-AZ DB instance deployment** taught here:
- RDS maintains a synchronously replicated standby in another AZ.
- The standby does not serve application reads.
- RDS can automatically fail over for supported failures.
- Applications continue using the DB endpoint but must handle interrupted connections and reconnect.

Do not generalize the no-read-standby rule to **Multi-AZ DB clusters**, whose architecture differs.

Read replicas for the RDS MySQL reporting cases use asynchronous replication. Reports in the questions tolerate lag. RDS Proxy pools/manages connections; keeping the same reports on the primary does not offload their query execution. Backup files are not a directly queryable SQL endpoint; restore is a separate process.

**Keywords:** SELECT/reporting + lag acceptable → replica; automatic failover → Multi-AZ; both → combine mechanisms.
**Elimination:** A larger Single-AZ instance does not survive loss of its AZ merely by being larger.

## 7. Load Balancer Selection / 負載平衡器選型

| Type | Analogy / 比喻 | Key fit / 適用 |
|---|---|---|
| ALB — Application Load Balancer | 服務台讀取申請內容再分窗口 | HTTP/HTTPS, URL path/hostname routing |
| NLB — Network Load Balancer | 指揮連線的交通員 | TCP/UDP workloads, static IP requirements |
| GWLB — Gateway Load Balancer | 將旅客分到不同安檢通道 | Third-party virtual firewalls/network appliances |

### ALB
Path rules can route `/orders/*` to one target group and `/catalog/*` to another under one hostname. DNS does not receive an HTTP URL path. Listener rules and target groups are not launch templates.

### NLB
An internet-facing NLB can use an Elastic IP for each enabled AZ. This matches Q14's non-HTTP TCP service and per-AZ public IP allowlist. ALB does not directly provide equivalent per-AZ Elastic IP assignment; other architectures involving additional services were not assessed.

### GWLB
Distributes traffic among virtual appliances; the appliances perform inspection. GWLB does not itself become the firewall policy engine.

**Elimination:** Identify routing information: URL content, transport connections/static IP, or network appliance inspection. “Network” alone is too broad to choose among them.

## 8. Availability Requires Surviving Capacity / 高可用需要剩餘容量

**Analogy:** 兩棟樓共有四個窗口，不代表一棟停電後還有四個。

ALB + multi-AZ + Auto Scaling help availability, but surviving resources must have enough capacity. Launching new instances takes time.

Q16 assumptions: peak needs four equal-sized instances; either of two AZs can fail; all peak traffic must be handled immediately without waiting.

| Placement | Worst surviving capacity | Meets requirement? |
|---|---:|---|
| 2 + 2 | 2 | No |
| 4 + 0 | 0 | No |
| 3 + 3 | 3 | No |
| 4 + 4 | 4 | Yes |

For equal placement across N AZs and tolerance of one AZ loss, a simplified planning calculation is `(N − 1) × capacity per AZ ≥ required capacity`. This assumes equal per-instance capability and the stated failure/load model.

This does not mean every multi-AZ system always needs double capacity. Acceptable recovery time, scaling delay, load shedding, and cost change the design.

**Keywords:** after failure, immediately, without waiting.
**Elimination:** Count what remains after failure, not only the normal total.

## 9. Route 53 Failover and DNS Cache / DNS 故障切換與快取

**Restaurant analogy / 餐廳比喻：**地址服務台平常提供主要分店地址；主要分店不能服務時，回答健康備援分店地址。

| Policy | Primary purpose / 用途 |
|---|---|
| Failover | Primary/standby, active-passive |
| Weighted | Configured distribution weights |
| Latency-based | Route toward lower-latency region |
| Geolocation | Answer based on geographic location |

Configure health evaluation appropriately, using endpoint health checks or supported alias target-health evaluation as applicable. The classroom scenario assumes unhealthy primary and healthy standby.

Route 53 changes DNS responses; it does not copy website data, provision backup capacity, or move established connections between Regions. A standby website/data strategy must be prepared separately.

Resolvers/clients may keep prior DNS answers until cache expiry; failover is not instantaneous for every client. Lower TTL may reduce some caching delay but does not remove health-detection and application-reconnect considerations.

**English:** ALB chooses targets for requests; Route 53 supplies DNS answers.
**Keywords:** primary + standby → failover; old endpoint briefly persists → cached DNS answer.
**Elimination:** A URL path is not part of DNS lookup.

## 10. Stateless Compute and Shared Sessions / 無狀態運算與共用資料

**Analogy:** 每位服務員自己的紙條 versus 共用訂單簿。

If carts/sessions live only in an EC2 process's memory, instance termination loses those values. Other instances cannot recover them merely because more capacity exists.

Shared session storage, such as DynamoDB in the questions, separates required session state from disposable compute. Applications still need correct identity/session lookup, expiry, and permissions. A DynamoDB table in one Region is not automatically a cross-Region session replication design.

| Change | What it solves | What it does not solve |
|---|---|---|
| Sticky sessions | Affinity to a target | Preserve failed instance's memory |
| Higher ASG maximum | Permit more capacity | Copy session data |
| Shared session store | Other instances can retrieve state | Automatically implement all application/session logic |
| Different load balancer | Different routing capabilities | Recover lost local state |

**English:** More servers add capacity; shared storage preserves required state independently of individual servers.
**Elimination:** Identify whether the failure is lack of compute capacity or loss of data. A running replacement can still lack the original session.

## 11. Wrong-Answer Review / 錯答複習

| Item | Given → Correct | Correction / 修正 | Later evidence |
|---|---|---|---|
| Q3 memory metric | D → B | Collect with agent; EventBridge does not measure memory | Agent check A correct; Q20 B later missed |
| Q4 local application logs | B → D | CloudTrail records AWS actions; agent sends local logs | CloudTrail “who terminated EC2” check A correct |
| Q16 surviving capacity | A → D | 2+2 leaves two after AZ loss; need four surviving | Simplified three-per-AZ check yes correct |
| Q19 cart state | A → B | Stickiness does not preserve lost memory; use shared store | Simple check B correct; Q21 C later missed |
| Q20 excluded memory repeat | B → C | CloudTrail still not memory measurement | Unscored; priority remains |
| Q21 excluded session repeat | C → B | Maximum capacity does not preserve sessions | Unscored; priority remains |

Do not claim the four priorities are independently mastered. Future study should use fresh scenarios, rather than repeat these disclosed questions. No additional quiz is appended here.

## 12. Complete Answer Log / 完整作答紀錄

The table records the actual displayed option order, learner response, and classification. Topic descriptions identify the original scenarios; options below are concise summaries, not verbatim transcript quotations.

| Q | Scenario / 主題 | Given | Correct | Classification | Result |
|---|---|---|---|---|---|
| 1 | CPU >80%, three five-minute periods; notify email | B | B | Original | Correct |
| 2 | EC2 running→stopped, invoke Lambda without polling | C | C | Original | Correct |
| 3 | Guest memory >85%, additional metric collection | D | B | Original | Incorrect |
| 4 | Preserve/search local app logs after instance termination | B | D | Original | Incorrect |
| 5 | Variable traffic, add and remove EC2 | +b → B | B | Original | Correct |
| 6 | Min2/desired3/max6, requested8, highest allowed | C | C | Original | Correct |
| 7 | Approved AMI and startup script | A | A | Original | Correct |
| 8 | ALB unhealthy but EC2 healthy, no ASG replacement | D | D | Original | Correct |
| 9 | Fixed weekday class schedule, prepare capacity early | B | B | Original | Correct |
| 10 | Unpredictable demand, maintain average CPU near50% | C | C | Original | Correct |
| 11 | Offload MySQL reporting reads, lag acceptable | A | A | Original | Correct |
| 12 | PostgreSQL automatic DB/AZ failover | D | D | Original | Correct |
| 13 | Same hostname, orders/catalog URL-path target groups | B | B | Original | Correct |
| 14 | Non-HTTP TCP with fixed public IP per AZ | C | C | Original | Correct |
| 15 | Fleet of third-party virtual firewall appliances | B | B | Original | Correct |
| 16 | Four instances required after loss of either AZ | A | D | Original | Incorrect |
| 17 | Primary/standby regional websites | C | C | Original | Correct |
| 18 | Some clients still use old endpoint after DNS change | A | A | Original | Correct |
| 19 | In-memory carts lost during instance replacement | A | B | Original | Incorrect |
| 20 | Memory metric, threshold, email; disclosed repeat | B | C | Unscored final review; scored classification voided | Incorrect, excluded |
| 21 | In-memory login sessions, AZ/instance failure; repeat | C | B | Unscored final review; scored classification voided | Incorrect, excluded |
| 22 | Both RDS automatic failover and reporting offload | D | D | Original integrated final review | Correct |
| 23 | Per-Region URL paths and cross-Region active/passive | B | B | Original integrated final review | Correct |

### Option audit / 選項核對

| Q | A | B | C | D |
|---|---|---|---|---|
| 1 | CloudTrail+SNS: not CPU evaluation | CloudWatch alarm+SNS: meets need | Config: compliance | Route53: endpoint health |
| 2 | Periodic Config: not event response | CPU alarm: not stopped-state identification | EventBridge: state event routing | Route53: not EC2 state identification |
| 3 | Detailed monitoring: no added memory metric | Agent+metric+alarm: meets need | CloudTrail: no memory API measurements | No default matching EC2 memory-pressure event |
| 4 | Detailed monitoring: no local log collection | CloudTrail: no local file collection | EventBridge alone: no file retrieval | Agent→Logs: central collection |
| 5 | Larger machines: vertical | ASG target tracking: elastic count | CloudTrail: audit | Route53: DNS failover |
| 6 | 2: minimum | 3: prior desired | 6: maximum | 8: exceeds bound |
| 7 | Launch template: AMI/user data | Target tracking: capacity | Target group: routing/health | Alarm: condition evaluation |
| 8 | Higher max: not health integration | Detailed monitoring: frequency | Indefinite timeout: not fix | Enable ELB checks: use app health |
| 9 | Target tracking: reacts to metric | Scheduled: fixed timetable | Manual: late and manual | Predictive: forecast, not requested schedule |
| 10 | Morning schedule: not demand target | Always max: no scale-in savings | Target tracking50%: meets need | Replace busy: not capacity response |
| 11 | Read replica: offloads reads | Standby: no reporting reads | Backups: not direct SQL endpoint | Proxy on primary: no query offload |
| 12 | Manual promotion: not automatic | Larger primary: no AZ protection | Snapshot restore: recovery workflow | Multi-AZ standby: automatic failover |
| 13 | TCP listeners: no URL inspection | ALB path rules: meets need | DNS: no URL path | ASG policy: no request routing |
| 14 | ALB path: not custom TCP/static IP fit | GWLB: appliance use | NLB+EIPs: meets need | Changing IPs: not fixed |
| 15 | ALB: app routing | GWLB: appliance integration | NLB: transport service | Classic: no GWLB integration |
| 16 | 2+2 leaves2 | 4+0 can leave0 | 3+3 leaves3 | 4+4 leaves4 |
| 17 | Weighted: allocation | Latency: latency selection | Failover: primary/backup | Geolocation: location |
| 18 | Cached old answer: explains delay | No hostname change required | Route53 does not copy files | ALB does not migrate connections |
| 19 | Sticky: still loses memory | Shared DynamoDB: state survives instance | More max: no state copy | DNS latency: no state preservation |
| 20 | Frequency: no memory metric | CloudTrail: audit | Agent+alarm+SNS: complete chain | Stopped event: not memory threshold |
| 21 | Sticky: local-state risk remains | Shared table: independent state | More max: no state preservation | NLB: no state preservation |
| 22 | Standby cannot serve reports | Proxy: no DB redundancy/read offload | Size+backup: neither requirement fully | Multi-AZ+replica: both |
| 23 | NLB: no URL-path routing | ALB paths+DNS failover: both | No health evaluation: not health-based standby | GWLB/geolocation: wrong mechanisms |

### Unnumbered checks / 未編號不計分確認

| After | Question | Given | Result |
|---|---|---|---|
| Q3 | Who measures/reports memory: agent or EventBridge? | A: agent | Correct |
| Q4 | Who terminated EC2: CloudTrail or payment app logs? | A: CloudTrail | Correct |
| Q16 | Three required, three per AZ; sufficient after one fails? | Yes | Correct |
| Q19 | Recover cart: local+sticky or shared DynamoDB? | B: shared | Correct |

All four were scaffolded checks. With the two excluded numbered reviews, six unscored responses are logged.

## 13. Personal AWS Rebuild — Completion Checkpoint

**Worth rebuilding? / 值得重建嗎？** Yes, selectively. Start with a paper architecture and a small monitoring exercise, especially because agent/log/audit distinctions remained weak. A two-Region website or Multi-AZ database is unnecessary for the first rehearsal.

**Objectives / 目標：**
1. Distinguish CPU metrics, guest memory metrics, app logs, AWS API activity, and state-change events.
2. Trace ALB health→ASG replacement and min/desired/max reasoning.
3. Calculate surviving capacity after an AZ failure.
4. Explain why sessions must survive individual instance replacement.
5. Separate RDS read scaling from automatic failover.

**Fictitious non-production LFM example / 虛構 LFM 範例：**A disposable story demo with fake sessions and fake logs. Paper design: ALB→ASG in two AZs; shared DynamoDB session storage; agent→CloudWatch metrics/logs; alarm→SNS. Use mock data, no caregiving records, no production modifications.

| Stage | Proposed scope |
|---|---|
| Paper first | Mark metric/log/event/audit flows, failure domains, capacity and cleanup |
| Small monitoring lab | One disposable EC2, agent memory metric, fake app log, alarm/SNS; inspect an AWS activity event |
| Optional HA lab | Short-lived ALB+ASG across two AZs, ELB health integration, test replacement and shared fake sessions |
| Database/Region failover | Paper exercises initially; provision only if later explicitly requested |

**Potential costs / 可能費用：**EC2, EBS, public IPv4, ALB/NLB capacity and runtime, data transfer, custom metrics, alarms, log ingestion/storage, SNS usage, DynamoDB, Route53 hosted zones/queries/health checks, and optional RDS primary/standby/read replicas. NAT gateways, if introduced by a later design, can add charges. No current dollar quote was established.

**Cleanup / 清理：**
- Set ASG desired/minimum to zero or remove the test group before terminating instances, to avoid automatic replacement.
- Delete disposable load balancer/listeners/target groups as dependencies allow.
- Terminate test EC2; inspect retained EBS, snapshots, Elastic IPs/public IPv4.
- Remove test alarms, log groups, SNS subscriptions/topics, EventBridge rules/targets and Lambda if created; review retained logs/custom-metric usage.
- Delete disposable DynamoDB tables, Route53 test records/health checks/zones only when no longer needed; domain registration is separate.
- If database tests were explicitly created, review deletion protection, snapshots/backups and replica relationships before cleanup. Retained snapshots/storage can continue billing.
- Remove test IAM roles/instance profiles after resources no longer depend on them.
- Confirm resources and charges after cleanup; do not delete production LFM infrastructure.

**Authorization / 授權界線：**Planning only. No rebuild was performed. Provision only when the learner explicitly requests it.

## 14. Coverage Limits and Official Checks / 範圍限制與查核

- The video listed guided “Creating a Highly Available Environment” and challenge “Creating a Scalable and Highly Available Environment for the Café” labs. Neither was executed/verified here.
- The handoff contains earlier lab history; that is not fresh hands-on verification in this tutoring session.
- Well-Architected principles appeared in the guide and were illustrated through fault isolation, spare capacity, health-based recovery, and externalized state. There was no dedicated assessment of the full framework/pillars.
- Predictive scaling was introduced as a distinction, not configured or independently assessed.
- No live DNS TTL/failover timing, memory-agent installation, scaling behavior, DB failover, or application session recovery was measured.
- No exhaustive database-engine scaling survey, Aurora implementation, GWLB encapsulation, health-check edge cases, or full Route53 policy lab was taught.
- No diagram was requested or delivered; analogy/comparison tables preserve the explanatory aids actually used.
- Course-era screenshots were not treated as a current console implementation guide. Changing specifications require current AWS documentation.

Official documentation checked during final documentation on 2026-10-25:

- [CloudWatch agent: collect internal metrics and logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Auto Scaling health checks and enabling ELB health checks](https://docs.aws.amazon.com/autoscaling/ec2/userguide/health-checks-overview.html)
- [RDS Multi-AZ DB instance: synchronous standby, not read scaling](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html)
- [Route53 health checks and DNS failover](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-failover.html)

**Next / 下一步：**Module 10 tutoring has ended; read the actual Module 11 material before selecting its title/topics. Carry forward monitoring distinctions, surviving capacity, and shared-state reasoning without restarting earlier modules.

