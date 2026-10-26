# Module 6 — Adding a Database Layer / 新增資料庫層
AWS SAA-C03 | 中英雙語考前筆記 | 2026-10-21

> 學習方式：三年級比喻 → 技術說明 → Exam keywords → 排除錯誤選項 → 一次一題。適合列印複習。
> Source: AWS Academy Cloud Architecting v3, Module 6 Student Guide (2024), reviewed through the uploaded screen recording. Some scrolling text was not fully readable. Notes combine the readable guide, this tutoring session, and linked AWS documentation.
> Practice completed; this session did not execute or verify either hands-on lab. 練習完成不代表本次已執行實作 Lab。

## 0. Learning Results / 學習成果

| Checkpoint | Topic / 主題 | Correct / Total |
|---|---|---|
| 1 | Database selection, scaling, responsibilities | 4/4 |
| 2 | RDS, Aurora compatibility, Serverless, security | 4/5 |
| 3 | Multi-AZ, read replicas, Proxy, backups | 7/7 |
| 4 | DynamoDB keys, indexes, consistency, capacity | 9/10 |
| 5 | Purpose-built databases and caching | 6/7 |
| 6 | DMS and schema conversion | 2/2 |
| 7 | Integrated scenarios and targeted retests, including Q41 | 5/6 |
| **Total** | **All scored answers, including retests** | **37/41 = 90.2%** |

- Q1–Q40: 36/40 = 90%.
- Additional targeted Q41: correct; final cumulative score 37/41.
- The invalid answer “V” was not scored. 「V」無效輸入未計分。
- Four incorrect answers: Q6, Q23, Q31, Q39.
- This is guided practice with explanations and repeated concepts, not an independent mock-exam score or pass-probability prediction.
- Original checkpoint time estimates were planning estimates, not measured time.

## 1. Selecting a Database / 資料庫選擇

**Third-grade analogy:** Relational database 像固定欄位的學生名冊，成績表用學號連回學生；NoSQL 像可有不同欄位的個人資料卡。先問「要怎麼找資料」，再挑工具。

| Requirement / 需求 | Direction / 方向 |
|---|---|
| Related tables, complex SQL JOINs, operational transactions | RDS / compatible relational engine |
| Key-based lookups, flexible attributes, large-scale low-latency access | DynamoDB |
| Large historical datasets, aggregations, BI reports | Redshift |
| Multi-hop relationship exploration | Neptune |
| Existing MongoDB application with verified feature compatibility | DocumentDB |
| Existing Cassandra / CQL workload | Keyspaces |
| Durable Redis-compatible in-memory primary database | MemoryDB |
| Rebuildable cache in front of an authoritative database | ElastiCache |
| Time-window analysis of measurements | Time-series data model; see Timestream lifecycle note |

**Access patterns / 存取模式:** Identify lookup keys, range queries, joins, read/write volume, latency, consistency, availability, growth, and cost before selecting a service.

**Scaling analogy:** 換大爐子 = vertical scaling；增加爐子分工 = horizontal scaling。
- Vertical / scale up: more CPU or RAM in one instance.
- Horizontal / scale out: add resources and distribute work; the supported read/write scaling behavior depends on architecture.
- High memory, adequate CPU/storage → evaluate a DB instance class with more RAM.
- Storage autoscaling increases storage capacity; it is not compute autoscaling.

**Elimination:** SQL alone does not mean RDS; Redshift also uses SQL. Transactions alone do not exclude every NoSQL service; DynamoDB supports transactions. Look at the complete requirements.

## 2. RDS and Aurora / 引擎、責任與安全

### Managed responsibilities / 責任分工

**Third-grade analogy:** 租有管理員的廚房，管理員修設備，你仍決定菜單、保管鑰匙、安排做菜流程。

AWS handles underlying infrastructure and routine managed maintenance. You still own:
- Data modeling, indexes, SQL tuning, and application correctness.
- Network access, database permissions, and suitable authentication.
- Capacity choices, backup retention, availability configuration, and recovery planning.

### Engine compatibility / 引擎相容性

**Third-grade analogy:** 都是遊戲片，不代表每台遊戲機都能讀。搬到 managed service 前，先確認原有功能能否使用。

- Aurora offers **MySQL-compatible** and **PostgreSQL-compatible** engines.
- SQL Server-specific features + minimal changes → evaluate RDS for SQL Server.
- Oracle PL/SQL dependencies + minimal changes → evaluate RDS for Oracle.
- Always validate supported engine versions, extensions, editions, licensing, and features.
- Compatibility is not a guarantee of identical behavior.

**Exam cues:** retain engine-specific features, minimal application changes → preserve a compatible engine before comparing other benefits.

### Aurora and Aurora Serverless / 雲端關聯式資料庫

**Third-grade analogy:** 廚房人力可以隨客流增減，而不是每次都自己搬去更大的廚房。

- Aurora is a cloud-designed relational database with a shared, distributed cluster storage architecture.
- A writer modifies data; Aurora replicas can offload reads and provide failover targets.
- Aurora Serverless v2 automatically adjusts compute within a configured capacity range.
- Variable or unpredictable relational workload + less capacity management → consider Serverless.
- Storage growth and compute scaling are separate concepts.
- Do not memorize the old guide's Aurora storage ceiling as a current universal limit. Limits depend on engine/version and change over time.

### Private access and encryption / 私有連線與加密

**Third-grade analogy:** Application 像餐廳服務人員，database 像後方保管訂單的保險櫃；客人不需直接接觸保險櫃。

- Typical design: RDS in private subnets, public access disabled.
- Database SG permits only required application SG / port, e.g. MySQL TCP 3306.
- Private placement and security-group restrictions serve different purposes; configure both.
- IAM controls AWS management actions; database authorization controls database operations. Supported IAM database authentication has engine-specific requirements.
- **At rest:** KMS-backed encryption protects stored database data and associated encrypted logs/backups/snapshots.
- **In transit:** TLS protects application-to-database connections.
- Security groups control network access; they do not encrypt traffic.
- Encrypt an existing unencrypted RDS database through snapshot → encrypted snapshot copy → restore an encrypted DB instance, then validate and plan application cutover.

**Analogy:** 鎖住抽屜 = at rest；用上鎖盒子運送日記 = in transit。

## 3. Availability, Read Scaling, Connections, Recovery

### Multi-AZ vs. read replicas / 高可用與讀取擴展

**Third-grade analogy:** Standby 是準備接手的備用廚房；read replica 是增加可提供閱讀服務的副本。

| Architecture / 功能 | Main purpose | Important distinction |
|---|---|---|
| Traditional RDS Multi-AZ **DB instance** deployment | HA and automatic failover | Primary + synchronous standby in another AZ; standby does not serve application reads |
| RDS Multi-AZ **DB cluster** deployment | HA plus readable instances | One writer + two readers in three AZs; semisynchronous replication |
| Standard RDS read replica | Offload read queries | Asynchronous replication; lag possible; route queries to its endpoint |
| Aurora replica | Read scaling and failover target | Aurora cluster architecture differs from standard RDS replicas |
| RDS Proxy | Pool/reuse connections | Not a query-result cache |
| ElastiCache | Reuse cached data/results | Application cache design and invalidation are needed |
| Backups / snapshots | Recover historical data | Not a running read endpoint |

- Adding a read replica does not automatically redirect existing primary connections.
- Traditional Multi-AZ standby is not the same as a readable Multi-AZ cluster instance.
- A replica may reflect an accidental deletion too; replication is not a historical backup.
- Immediate confirmation after a successful write → read from the primary when an asynchronous replica might lag.
- Do not assume every read replica is automatically promoted like a Multi-AZ standby.

### RDS Proxy / 連線池

**Third-grade analogy:** 共乘車減少抵達廚房的車輛；增加讀取工作站才是分擔查詢本身。

Proxy establishes a pool and reuses database connections, reducing repeated connection setup overhead. It can help applications reconnect during failover and supports authentication options subject to engine/version support.

**Exam cues:**
- Too many connections, Lambda concurrency, frequent open/close → RDS Proxy.
- High load from SELECT queries with acceptable lag → read replicas.
- Repeated reusable results and authoritative data elsewhere → ElastiCache.

**Critical trap:** Proxy does not automatically cache SELECT results. It does not add database query-processing instances by itself.

### Automated backups, PITR, manual snapshots

**Third-grade analogy:** 每天拍白板照片，再記錄每次修改，就能重建擦錯前的狀態。手動挑出的照片則一直保留到你主動刪除。

| Option | Use / 用途 |
|---|---|
| Automated backups + transaction logs | Point-in-time recovery within the available recovery window |
| Manual snapshot | Preserve a particular database state until explicitly deleted |
| Cross-Region backup / snapshot copy | Support regional recovery planning; restoration and application cutover are still required |

- PITR restores a **new DB instance**; it does not rewind the existing one in place.
- Accidental deletion at 14:15, recoverable 14:14 → restore to 14:14, validate, then reconnect as appropriate.
- Pre-upgrade state retained for months → manual snapshot.
- A post-deletion snapshot preserves the post-deletion state.
- Snapshot restoration creates a database to query; a snapshot itself is not a database endpoint.
- Manual/final snapshots are independent of automated-backup deletion behavior.

## 4. DynamoDB / 主鍵、索引、一致性與容量

### Primary keys / 主鍵

**Third-grade analogy:** Partition key 像學生編號，找到抽屜；sort key 像作業日期，區分同一學生的作業。

- Simple primary key: partition-key value uniquely identifies an item.
- Composite primary key: **partition key + sort key together** uniquely identify an item.
- Multiple items may share a partition key when their sort keys differ.
- Sensor readings, one reading per sensor/timestamp → DeviceID + Timestamp.
- Real partition keys determine hashing/distribution; the “drawer” analogy describes logical grouping, not a literal dedicated physical drawer.

### Query vs. Scan

**Third-grade analogy:** Query 直接去已知學生的抽屜；Scan 把所有抽屜逐一檢查。

- Query requires equality on the target table/index partition key; optionally constrain its sort key.
- DeviceID = sensor-42 + Timestamp BETWEEN endpoints → Query key conditions.
- Scan examines the table/index rather than directly targeting one partition-key value.
- FilterExpression is applied after reading and does not reduce the capacity already consumed by those reads.

### GSI vs. LSI

**Third-grade analogy:** GSI 像另外建立作者目錄；LSI 像同一學生抽屜內，改用科目整理的目錄。

| Feature | GSI | LSI |
|---|---|---|
| Partition key | Can differ from base table | Same as base table |
| Sort key | Can differ; optional depending on index key design | Alternate sort key |
| Creation | Can be added after table creation | Must be created with table |
| Reads | Eventually consistent | Eventual or strongly consistent |
| Example | OrderID table → query orders by CustomerID | CustomerID + OrderDate → CustomerID + OrderAmount |

**Exam traps:** A base-table Query cannot omit its required partition-key value and substitute a filter. An existing table cannot receive a new LSI after creation.

### Read consistency / 讀取一致性

**Third-grade analogy:** Eventual 像可能還沒更新的影本；strong 像要求查看已包含成功更新的正式紀錄。

- Default eventually consistent reads may briefly return older data.
- Base tables and LSIs support strong reads.
- GSIs do not support strong reads.
- GetItem with **ConsistentRead=true** on the base table reflects prior successful writes.
- A fixed one-second delay cannot guarantee propagation.
- For the same nontransactional read size, eventual reads consume half the read capacity of strong reads.
- Strong reads do not promise data will never be modified by another writer immediately afterward.

### Global Tables / 跨 Region 複寫

**Third-grade analogy:** 美國與歐洲都有辦公室，家長可就近更新聯絡簿，各辦公室自動互傳變更。

- DynamoDB provides multi-AZ resilience within a Region.
- Global Tables provide managed multi-Region, multi-active replication.
- Applications can read/write through regional replica tables.
- GSI changes how you find data; Global Tables change where replicated data is available.
- Current Global Tables support **MREC** (multi-Region eventual consistency) and **MRSC** (multi-Region strong consistency), with mode-specific restrictions. Do not assume every global-table deployment uses the same consistency.
- Cross-Region replication does not replace historical backups.

### Streams / 事件驅動

**Third-grade analogy:** 圖書館變更簿記錄新增、修改、移除；小幫手讀紀錄後執行後續工作。

DynamoDB Streams records item-level changes. Lambda can process records to perform follow-up actions, e.g. prepare a shipping request after a new order. Filter/handle event types appropriately and design processing for retries/idempotency.

**Exam cues:** react to insert/update/delete, CDC, event-driven processing → Streams + Lambda.

### Capacity modes and hot keys / 容量與分布

**Third-grade analogy:** On-demand 像按實際使用服務付費；Provisioned 像預先安排窗口，可搭配 Auto Scaling。若所有人都排同一窗口，即使其他窗口空著仍會塞車。

- On-demand: no preconfigured RCUs/WCUs; useful for new or unpredictable workloads.
- Provisioned: specify read/write capacity; Auto Scaling can adjust within configured bounds.
- Neither mode means unlimited throughput or immunity to hot keys.
- Choose keys using access patterns, cardinality, and traffic distribution.
- 100,000 similarly active sensors → SensorID helps spread writes.
- Country when all sensors share one country, Status when most values are OK, or CurrentDate for all current writes → concentrated key traffic.
- Many distinct keys alone do not solve a workload dominated by one hot key.

### DynamoDB security and recovery / 補充重點

- IAM roles/policies govern access; use least privilege and fine-grained conditions where appropriate.
- Encryption at rest uses AWS KMS integration; protect data in transit too.
- VPC endpoints can provide private access paths without using public internet routing.
- PITR and backups help recover from logical errors; restored data still requires validation and application planning.
- CloudTrail records API activity; AWS Config evaluates tracked resource configurations. Their roles differ from database query performance monitoring.

## 5. Purpose-Built Databases / 特殊用途資料庫

| Service | Third-grade analogy / 三年級比喻 | Workload and exam cues |
|---|---|---|
| Redshift | 分析多年訂單的分析員 | Data warehouse, OLAP, BI, columnar storage, parallel processing |
| Neptune | 沿朋友關係圖找朋友的朋友 | Nodes/edges, multi-hop relationships, fraud graphs |
| DocumentDB | 每人欄位不同、可有小資料夾的個人檔案 | JSON-like documents, MongoDB compatibility; validate APIs |
| Keyspaces | 保留既有積木格式、有人維護的新教室 | Cassandra compatibility, CQL, wide-column |
| MemoryDB | 桌面快速資料 + 可靠變更紀錄簿 | Durable in-memory primary database, Redis-compatible workload |
| ElastiCache | 正式課本旁可重建的便條紙 | Cache reusable copies/results; cache-aside, reduce DB reads |
| Timestream | 每小時記錄花園溫度，研究變化 | Time-series measurements, time-window averages, trends |
| QLDB — historical guide topic | 不能隨便改寫且可驗證的記帳簿 | Immutable, cryptographically verifiable ledger; lifecycle note below |

### MemoryDB vs. ElastiCache

MemoryDB can hold authoritative primary data using in-memory access plus a durable transaction log. ElastiCache commonly accelerates access to data stored elsewhere. In Q31, **Memcached** also failed the required **Redis compatibility**.

Cache-aside:
1. Application checks cache.
2. On a miss, it reads the primary database.
3. It populates cache for later requests.
Plan expiration/invalidation so stale cache entries do not violate application requirements.

### Course-era content and current availability / 舊教材與現況

- **QLDB:** The 2024 guide covers ledger concepts. AWS documentation announced end of support on **2025-07-31**. Keep it as historical course vocabulary, not a new deployment recommendation.
- **Timestream for LiveAnalytics:** AWS closed new-customer access effective **2025-06-20**; existing eligible customers can continue. The time-series concept in Q33 remains valid, but do not treat that answer as confirmation a new personal account can enable LiveAnalytics.
- **Timestream for InfluxDB** is a separately documented managed time-series offering; check the precise engine/service before planning a lab.
- These updates do not establish whether any topic appears on the user's particular exam. Do not infer exam inclusion/exclusion solely from lifecycle status.

## 6. AWS DMS and Schema Conversion / 資料庫遷移

**Third-grade analogy:** 圖書館先搬現有書，再持續記錄搬家期間的借還與新增。換成另一種語言的圖書館，還要先翻譯目錄與規則。

- **Full load:** copy existing data.
- **CDC — Change Data Capture:** replicate ongoing source changes for supported configurations.
- Source can remain operational through much of migration; cutover still needs planning and validation.
- **Homogeneous:** same engine, e.g. MySQL → MySQL.
- **Heterogeneous:** different engines, e.g. Oracle → PostgreSQL.
- **AWS SCT / DMS Schema Conversion:** assess and convert supported schema and code objects.
- Resolve objects needing manual changes; validate stored procedures, data types, application behavior.
- **DMS data migration tasks:** move data and replicate changes; do not assume every task automatically converts all source code.
- Schema conversion and data movement can be capabilities within the broader DMS toolset, but they are distinct stages.
- One-time export does not capture later changes. RDS Proxy does not migrate tables.

**Exam cues:** minimal downtime + active source → full load + CDC; engine change + stored procedures → schema assessment/conversion + remediation + migration.

## 7. Well-Architected Decisions / 架構整合

**Third-grade analogy:** 蓋早餐店先知道客人怎麼點餐，再選設備、鎖好門、安排備援，最後確認花費合理。

| Pillar / 方向 | Database decision |
|---|---|
| Performance efficiency | Match data model/access patterns; identify CPU, RAM, I/O, connections, or read-query bottleneck before scaling |
| Reliability | Multi-AZ failover, backups, restore validation, application retries/cutover |
| Security | Least privilege, private access, KMS at rest, TLS in transit |
| Cost optimization | Right-size, choose capacity mode, avoid unnecessary replicas and retained snapshots |
| Operational excellence | Monitor metrics/logs, document migration and recovery steps |
| Sustainability | Avoid idle overprovisioning while meeting workload needs |

**30-second decision sequence:**
1. Engine-specific application? Confirm compatibility.
2. Operational transactions or historical analytics? RDS/Aurora vs. Redshift.
3. Too many connections? Proxy.
4. Too many read queries? Read replicas; reusable results may also fit cache.
5. AZ failure? Appropriate Multi-AZ architecture.
6. Accidental deletion? PITR/backup, not failover alone.
7. DynamoDB latest-value requirement? Check consistency and target table/index.
8. Cross-Region local reads/writes? Global Tables and its chosen consistency mode.
9. Data changes trigger code? Streams + Lambda.
10. Migration while source changes? Full load + CDC; convert schema if engines differ.

## 8. Wrong-Answer Review / 四次錯答

| Question | Answer given → correct | Reason / 關鍵修正 | Later evidence |
|---|---|---|---|
| Q6 | D → C | SQL Server-specific features require a supported compatible engine; Aurora MySQL does not supply SQL Server compatibility | Q7 and Oracle scenario Q36 correct |
| Q23 | A → C | Immediate reflection of a successful write requires a strong read from an eligible target | Eventual-read scenario Q24 and strong-read scenario Q37 correct |
| Q31 | A → D | Memcached cache is not a Redis-compatible durable primary database; MemoryDB fits | Cache scenario Q32 and MemoryDB retest Q38 correct |
| Q39 | B → C | Proxy pools connections; read replica offloads reporting SELECTs | Proxy scenario Q40 and read-load retest Q41 correct |

Retests demonstrate improvement within this session, not permanent mastery. Schedule no reminder automatically; revisit these during later study.

## 9. Complete Answer Log / 完整作答紀錄

Question descriptions are concept summaries, not verbatim copies.

| Q | Topic | Given | Correct | Result |
|---|---|---|---|---|
| 1 | Related tables and JOINs | B | B | Correct |
| 2 | Player-ID lookup | C | C | Correct |
| 3 | Memory bottleneck | B | B | Correct |
| 4 | Managed responsibilities | C | C | Correct |
| 5 | Private RDS access | A | A | Correct |
| 6 | SQL Server compatibility | D | C | Missed |
| 7 | Aurora compatible engines | B | B | Correct |
| 8 | Aurora Serverless compute | C | C | Correct |
| 9 | TLS in transit | B | B | Correct |
| 10 | Multi-AZ automatic failover | C | C | Correct |
| 11 | Offload catalog reads | B | B | Correct |
| 12 | Read after write on primary | C | C | Correct |
| 13 | Lambda connection pressure | B | B | Correct |
| 14 | RDS PITR | C | C | Correct |
| 15 | Manual snapshot retention | A | A | Correct |
| 16 | Multi-AZ DB cluster | C | C | Correct |
| 17 | DeviceID + Timestamp | D | D | Correct |
| 18 | Query key conditions | B | B | Correct |
| 19 | CustomerID GSI | C | C | Correct |
| 20 | LSI and strong reads | B | B | Correct |
| 21 | Global Tables | C | C | Correct |
| 22 | Streams + Lambda | B | B | Correct |
| 23 | Immediate strong read | A | C | Missed |
| 24 | Eventual reads acceptable | B | B | Correct |
| 25 | On-demand capacity | C | C | Correct |
| 26 | Distribute sensor writes | B | B | Correct |
| 27 | Redshift analytics | C | C | Correct |
| 28 | Neptune graph | A | A | Correct |
| 29 | DocumentDB | B | B | Correct |
| 30 | Keyspaces | C | C | Correct |
| 31 | MemoryDB primary database | A | D | Missed |
| 32 | ElastiCache cache-aside | A | A | Correct |
| 33 | Time-series model | C | C | Correct |
| 34 | DMS full load + CDC | B | B | Correct |
| 35 | Schema conversion + DMS | C | C | Correct |
| 36 | Oracle compatibility retest | B | B | Correct |
| 37 | Strong-read retest | D | D | Correct |
| 38 | MemoryDB retest | B | B | Correct |
| 39 | HA plus report reads | B | C | Missed |
| 40 | Proxy connection pooling | B | B | Correct |
| 41 | Read-load retest | C | C | Correct |

## 10. Personal AWS Rebuild — Completion Checkpoint

**Worth rebuilding?** Yes, selectively, after study; no AWS resources were created in this session.

**Third-grade analogy:** 先用小模型學會門鎖、備份與搬家，不必一開始就蓋兩間完整餐廳。

Suggested future practice:
1. DynamoDB mock LFM memory metadata using fictitious data: OwnerID + EntryTimestamp; Query vs. Scan; optional GSI; observe Streams with a small Lambda.
2. Optional small RDS lab: private access, TLS, manual snapshot, PITR restore; validate with sample data.
3. Optional short read-replica or DMS exercise only when needed to answer a specific learning objective.

**LFM connection:** Use non-production mock memory entries. Do not upload real caregiving stories or change production infrastructure for a practice exercise.

Before starting, check current regional pricing and budget. RDS compute/storage, replicas, restored DBs, snapshots, Proxy, DMS, network endpoints/NAT, logs, and transfer can incur charges. On-demand/serverless does not mean free.

Cleanup: delete test DBs/clusters/replicas and restored instances, decide deliberately which snapshots to retain, remove DMS tasks/resources and Proxy if created, remove DynamoDB test tables/replicas, Lambda triggers/functions and unneeded logs, and inspect for remaining billable network resources.

## 11. Official References / 官方查證

The 2024 Academy recording is the course source. Links below support important distinctions and lifecycle corrections; feature availability remains engine/version/Region-dependent.

- [RDS Multi-AZ DB clusters](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts.html)
- [Amazon RDS Proxy](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.html)
- [RDS backup introduction](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
- [Aurora Serverless](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)
- [DynamoDB read consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
- [DynamoDB Global Tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
- [AWS DMS overview](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
- [Timestream for LiveAnalytics availability change](https://docs.aws.amazon.com/timestream/latest/developerguide/AmazonTimestreamForLiveAnalytics-availability-change.html)
- [Timestream documentation: LiveAnalytics and InfluxDB](https://docs.aws.amazon.com/timestream/)
- [QLDB legacy documentation with end-of-support notice](https://docs.aws.amazon.com/qldb/latest/developerguide/getting-started-step-7.html)

**Next module:** Read the Module 7 Student Guide before choosing its checkpoints. Do not infer its precise syllabus solely from its number.
