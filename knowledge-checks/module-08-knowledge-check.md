# Module 8 Knowledge Check — Hybrid and Multi-VPC Networking

[Study index](../README.md) | [Quick review](../cheat-sheets/module-08-knowledge-check-cheat-sheet.md) | [Lab 08 notes](../labs/lab-08-vpc-peering.md)

## Study-session record / 學習紀錄

This Knowledge Check followed Lab 08 and focused on choosing the right AWS networking pattern for different scales and requirements: VPC Peering, Transit Gateway, Site-to-Site VPN, Direct Connect, Global Accelerator, and VPC endpoints.

本次 Knowledge Check 接續 Lab 08，重點不是只背服務名稱，而是根據 **規模、連線兩端、成本、速度、頻寬穩定性、是否需要 Internet、以及是否要集中式 routing** 來選擇正確的 AWS networking solution。

The most important habit is:

> **First identify WHAT is being connected, then WHY, then how large the network is.**
>
> **先看「誰跟誰連」，再看「為什麼要連」，最後看「規模有多大」。**

---

## Question 1 — Connecting a very large number of VPCs

### Solution / 答案

**AWS Transit Gateway**

### Why / 為什麼

For a small number of VPCs, point-to-point VPC Peering can be simple. At very large scale, such as 100 VPCs, a large mesh of individual peering connections becomes difficult to manage.

當 VPC 數量很多時，例如 100 個 VPC，如果每兩個 VPC 都自己建立 Peering，會形成大量 point-to-point connections 與 route management。Transit Gateway 提供集中式 hub-and-spoke connectivity。

```text
VPC A ─┐
VPC B ─┤
VPC C ─┤
 ...   ├── Transit Gateway
VPC 100┘
```

### Analogy / 比喻

- VPC Peering = two houses connected by one private bridge / 兩棟房子的私人橋
- Transit Gateway = central train station for many neighborhoods / 多個社區共用的中央轉運站

> **Two VPCs -> Peering. Many VPCs -> Transit Gateway.**
>
> **兩個 VPC -> Peering；很多 VPC -> Transit Gateway。**

### Lesson learned / 學到什麼

Do not build a giant peering mesh merely because peering works technically. Architecture changes when scale changes.

不要因為 Peering 可以連 VPC，就把它放大到所有規模。**規模變大，架構也要改。**

---

## Question 2 — Connecting Transit Gateways across Regions/accounts

### Solution / 答案

**Transit Gateway peering attachment**

### Why / 為什麼

When two Regions each have their own Transit Gateway and traffic must flow between them, connect the Transit Gateways with a peering attachment and configure the required routes.

如果兩個 Region 各自都有 Transit Gateway，需要讓兩邊 network domains 互通，就使用 Transit Gateway peering attachment。

```text
Region A                         Region B
VPCs -> TGW A <---- peering ----> TGW B <- VPCs
```

### Analogy / 比喻

Each Transit Gateway is a major train station. A TGW peering attachment is the intercity rail line joining two stations.

每個 Transit Gateway 是一座中央車站；TGW Peering Attachment 就是兩座中央車站之間的跨區鐵路。

### Lesson learned / 學到什麼

The phrase **"between the transit gateways"** is a strong clue. Do not jump to Direct Connect or VPN when both endpoints are already AWS Transit Gateways.

看到題目明確寫 **between the transit gateways**，先想到 TGW Peering，而不是 Direct Connect 或 VPN。

---

## Question 3 — Connecting two non-overlapping VPCs

### Solution / 答案

**VPC Peering**

### Why / 為什麼

Two VPCs with non-overlapping CIDR blocks that need direct private routing are a classic VPC Peering use case.

兩個 CIDR 不重疊的 VPC，如果只是要彼此直接 private communication，VPC Peering 通常是最簡單的選擇。

```text
VPC A 10.1.0.0/16
       |
       | VPC Peering
       |
VPC B 10.2.0.0/16
```

This is the same core model built in Lab 08.

### Analogy / 比喻

> **Peering = a private bridge between two mansion estates. / Peering = 兩座大宅院之間的私人橋。**

But remember:

> **Bridge + routes on both sides = working connectivity. / 有橋還要兩邊都有路牌。**

### Lesson learned / 學到什麼

CIDR overlap matters. VPC Peering is not a workaround for overlapping network address plans.

CIDR 規劃很重要。Peering 不能拿來解決兩個 VPC CIDR overlap 的問題。

---

## Question 4 — Keeping S3 traffic off the normal Internet path

### Course feedback / 課程回饋

The course feedback states that **both VPC gateway endpoints and VPC interface endpoints** can keep Amazon S3 traffic on the Amazon network rather than using the normal Internet path.

AWS official documentation also confirms that Amazon S3 supports **both gateway endpoints and interface endpoints (AWS PrivateLink)**, and that traffic remains on the AWS network in both cases.

### Expected technical solutions / 技術上正確的解法

- **VPC Gateway Endpoint for Amazon S3**
- **VPC Interface Endpoint for Amazon S3**

### Important grading inconsistency / 重要判分不一致

During this Knowledge Check, a second attempt selected exactly those two endpoint choices, but the course still marked the answer incorrect even though its own feedback named both endpoint types.

第二輪作答時已經選擇：

```text
Use VPC interface endpoints
Create a VPC gateway endpoint for Amazon S3
```

但課程仍判定 Incorrect；同時 feedback 卻明確說明兩種 endpoint 都可以把 S3 traffic 留在 Amazon network。這屬於 **course grading / answer-key inconsistency**，不應該為了迎合判分而背錯 AWS 概念。

### Correct AWS mental model / 正確 AWS 心智模型

```text
Private workload
   |\
   | \-- Interface VPC Endpoint -> S3
   |
   +---- Gateway VPC Endpoint ---> S3
```

**Gateway Endpoint**

- Classic choice for S3 and DynamoDB
- Route-table based
- Uses an AWS-managed service prefix list
- No additional endpoint charge

**Interface Endpoint**

- Uses AWS PrivateLink
- Creates endpoint ENIs/private IPs in selected subnets
- S3 supports it
- Billed differently from gateway endpoints

### Correction / 修正

Earlier simplified memory:

```text
S3 -> Gateway Endpoint only   ❌ too simplified
```

Better memory:

```text
S3 private connectivity -> Gateway Endpoint OR Interface Endpoint
S3 + no additional endpoint charge -> Gateway Endpoint is the strong clue
```

### Analogy / 比喻

- Gateway Endpoint = a special AWS-only road marked on the mansion's route map / 寫在路牌上的 AWS 專用道路
- Interface Endpoint = a private doorway/entrance inside the subnet / Subnet 裡的一扇 PrivateLink 私人門
- Private IP alone = living inside the mansion; it does NOT automatically create a private road to S3 / 只有私人門牌不等於自動有 S3 私人道路

> **Private IP != private path. / Private IP 不等於 private network path。**

### Lesson learned / 學到什麼

When course grading conflicts with the course explanation and current AWS documentation, preserve both facts in the study record and learn the technically correct AWS behavior.

當課程判分、課程解釋與 AWS 官方技術事實互相矛盾時，要把 **「課程怎麼判」與「AWS 實際怎麼運作」分開記錄**。

---

## Question 5 — A-B and B-C Peering does not give A-C connectivity

### Solution / 答案

**Create a direct VPC peering connection between VPC A and VPC C and add the required routes.**

### Why / 為什麼

VPC Peering is **non-transitive**.

```text
A <--> B ✅
B <--> C ✅
A <--> C ❌ not automatically
```

Adding routes to B cannot turn B into a transit router for the two peering connections.

### Analogy / 比喻

```text
House A -- bridge -- House B -- bridge -- House C
```

A cannot simply tell B, "Let me cross your property to reach C." Peering bridges are one-to-one and do not provide automatic pass-through transit.

A 不能因為 B 跟 C 有橋，就把 B 當成中轉站借道去 C。

> **VPC Peering = one-to-one and non-transitive. / VPC Peering = 一對一，而且不能借道。**

### Lesson learned / 學到什麼

Whenever an exam diagram shows A-B-C peering, immediately ask whether the question is testing transitive routing.

看到 A-B-C 的 Peering 圖，立刻檢查是不是在考 **non-transitive routing**。

---

## Question 6 — Temporary on-premises site needs secure AWS access quickly

### Solution / 答案

**AWS Site-to-Site VPN**

### Why / 為什麼

The scenario describes:

- an on-premises/temporary data center
- existing Internet connectivity
- secure connection needed quickly
- only a short temporary period

A Site-to-Site VPN can establish encrypted IPsec connectivity over the existing Internet without waiting for a dedicated physical circuit.

題目是暫時機房、已有 Internet、需要安全而且 ASAP，只用短時間，因此最適合先使用 Site-to-Site VPN。

### Analogy / 比喻

> **VPN = quickly build an encrypted temporary tunnel over the existing public road.**
>
> **VPN = 在現有公共道路上快速搭一條加密隧道。**

Direct Connect is more like constructing a dedicated road: useful, but normally not the fastest answer for a two-week temporary facility.

### Lesson learned / 學到什麼

> **Temporary + ASAP + secure + Internet available -> Site-to-Site VPN.**

---

## Question 7 — Reach AWS network quickly through a nearby edge location

### Solution / 答案

**AWS Global Accelerator**

### Why / 為什麼

The strong clue is routing traffic toward an AWS **edge location close to the customer gateway device** to reduce exposure to a longer public-Internet path.

題目關鍵不是單純建立 VPN，而是希望流量盡快進入靠近 customer gateway 的 AWS edge location，再利用 AWS global network。

```text
On-premises
   |
shorter Internet path
   |
AWS Edge Location
   |
AWS global network
   |
AWS destination
```

### Analogy / 比喻

> **Global Accelerator = find the nearest entrance ramp onto the AWS global highway.**
>
> **Global Accelerator = 趕快找到最近的 AWS 高速公路交流道。**

### Lesson learned / 學到什麼

"Edge location" and "optimize path into AWS global network" are distinct clues. Transit Gateway solves centralized routing; Direct Connect solves dedicated connectivity; Global Accelerator optimizes how Internet-origin traffic enters the AWS network.

---

## Question 8 — Most consistent network performance for on-premises backup

### Solution / 答案

**AWS Direct Connect**

### Why / 為什麼

A backup system that continuously transfers data from on premises to AWS and needs the **most consistent performance** benefits from dedicated connectivity rather than dependence on Internet path conditions.

### Analogy / 比喻

- Site-to-Site VPN = secure lane running over public roads / 公共道路上的加密車道
- Direct Connect = dedicated road built for the company / 公司專用道路

> **VPN = faster to establish. Direct Connect = more predictable dedicated connectivity.**
>
> **VPN = 建得快；Direct Connect = 專線，效能較可預測。**

### Lesson learned / 學到什麼

Words such as **consistent**, **predictable bandwidth**, or **dedicated connectivity** strongly point toward Direct Connect.

---

## Question 9 — Cost-effective backup for an existing Direct Connect

### Solution / 答案

**AWS Site-to-Site VPN across the Internet as the backup path**

### Why / 為什麼

The company already has Direct Connect and wants **high availability** while emphasizing **cost-effective backup**. An Internet-based Site-to-Site VPN can provide a diverse secondary path without requiring another dedicated Direct Connect circuit.

```text
              Direct Connect (primary)
On-premises =========================== AWS
     |                                  |
     +------ Site-to-Site VPN ----------+
                  backup
```

### Analogy / 比喻

> **Direct Connect = main private highway. VPN = lower-cost emergency road.**
>
> **Direct Connect = 主幹專用道路；VPN = 成本較低的緊急備用道路。**

### Lesson learned / 學到什麼

Do not automatically choose a second Direct Connect whenever the word "backup" appears. Read the qualifier: **most cost-effective** changes the answer.

如果題目強調 highest resiliency regardless of cost，第二條不同 location 的 Direct Connect 可能更有吸引力；但 **cost-effective backup** 常指向 VPN。

---

## Question 10 — Resiliency plus predictable bandwidth for multiple data centers

### Solution / 答案

**Use Direct Connect as the primary connection and VPN as the secondary/failover path from each data center.**

### Why / 為什麼

This requirement combines two different goals:

```text
Predictable bandwidth -> Direct Connect
Resiliency/failover    -> VPN backup
```

Using both provides a stable primary path and an alternate path if the primary fails.

### Analogy / 比喻

```text
Data center
   |==== dedicated highway ==== AWS
   |
   +---- encrypted backup road ---- AWS
```

> **Direct Connect handles "steady"; VPN backup handles "still connected after failure."**
>
> **Direct Connect 管「穩」；VPN backup 管「斷了還有路」。**

### Lesson learned / 學到什麼

Solve multi-requirement questions one requirement at a time, then combine the services.

遇到題目同時要求兩件事，不要找一個服務硬包全部；先拆開：

```text
Requirement A -> best service A
Requirement B -> best service B
Then combine if architecture requires both.
```

---

# Comparison map / 服務比較地圖

| Need / 需求 | Best first thought / 第一反應 | Analogy / 比喻 |
| --- | --- | --- |
| Two VPCs | VPC Peering | 私人橋 |
| Many VPCs | Transit Gateway | 中央轉運站 |
| TGW to TGW across Regions | TGW Peering Attachment | 跨區鐵路 |
| Private access to S3 | Gateway or Interface Endpoint | AWS 私人道路/私人門 |
| Temporary on-prem -> AWS, quick and secure | Site-to-Site VPN | 加密臨時隧道 |
| Dedicated/predictable on-prem -> AWS | Direct Connect | 專用高速公路 |
| Optimize Internet path into AWS edge/global network | Global Accelerator | 最近 AWS 高速公路入口 |
| DX + cost-effective backup | Site-to-Site VPN backup | 備用道路 |

---

# Weak Spots Repaired / 已修正觀念

## 1. Peering does not scale like Transit Gateway

**Before:** If Peering connects VPCs, maybe just keep adding Peering.

**Now:** Peering is excellent for direct one-to-one connections; Transit Gateway is designed for centralized connectivity at larger scale.

**以前：** Peering 既然能接 VPC，就一直加橋。

**現在：** 少量 VPC 可用 Peering；大量網路要想到 Transit Gateway 中央轉運。

## 2. VPC Peering is non-transitive

```text
A-B + B-C != A-C
```

B cannot automatically be used as a transit router.

## 3. Private IP is not a private path

A private IP on a workload does not itself determine whether S3 traffic traverses the normal Internet-oriented path.

> **Private IP != VPC Endpoint. / Private IP 不等於 VPC Endpoint。**

## 4. S3 supports both endpoint types

Do not memorize "S3 = Gateway Endpoint only." Gateway Endpoint is the classic and no-additional-endpoint-charge choice, but Amazon S3 also supports Interface Endpoints through PrivateLink.

## 5. Course grading can be inconsistent

The S3 question produced an inconsistent result: the second-round selections matched the course's own explanation, yet the grading still returned Incorrect. Record the anomaly; do not distort the AWS concept to fit it.

## 6. VPN vs Direct Connect is requirement-driven

```text
ASAP / temporary / existing Internet -> VPN
Predictable / dedicated / consistent -> Direct Connect
DX cost-effective backup             -> VPN
```

## 7. Look at the endpoints before choosing the network service

```text
VPC <-> VPC                 -> Peering / TGW depending scale
On-prem <-> AWS             -> VPN or Direct Connect
TGW <-> TGW                 -> TGW Peering Attachment
Workload <-> AWS service    -> VPC Endpoint
Internet path -> AWS edge   -> Global Accelerator
```

---

# 🧒 3rd-Grade Analogy / 三年級大宅院比喻

Imagine AWS networking as cities, houses, roads, and stations.

把 AWS networking 想成城市、房子、道路與車站：

| AWS concept | Simple analogy | 繁體中文 |
| --- | --- | --- |
| VPC | Mansion estate | 大宅院 |
| VPC Peering | Private bridge between two estates | 兩座大宅院的私人橋 |
| Transit Gateway | Central train/bus station | 中央轉運站 |
| TGW Peering | Rail line between major stations | 中央車站之間的跨區鐵路 |
| Site-to-Site VPN | Encrypted tunnel over public roads | 公共道路上的加密隧道 |
| Direct Connect | Dedicated private highway | 專用高速公路 |
| Global Accelerator | Nearest ramp onto AWS global highway | 最近的 AWS 高速公路入口 |
| Gateway Endpoint | Special AWS service road on route map | Route Table 上的 AWS 專用道路 |
| Interface Endpoint | Private service doorway inside subnet | Subnet 裡的 PrivateLink 私人門 |

### Story / 小故事

Two neighboring mansion estates need to exchange packages:

> Build a **Peering bridge**.

A hundred estates need to exchange traffic:

> Don't build thousands of bridges. Build a **Transit Gateway station**.

A company office needs a temporary encrypted route to AWS:

> Build a **VPN tunnel** over the existing Internet road.

The company needs a stable long-term route with predictable performance:

> Build a **Direct Connect private highway**.

The EC2 server needs to reach S3 privately:

> Give it a **VPC Endpoint path**; simply giving the server a private house number is not enough.

---

# SAA Recognition Patterns / SAA 題目辨識模式

```text
"100 VPCs" / "many VPCs"
    -> Transit Gateway

"two VPCs" + non-overlapping CIDRs
    -> VPC Peering

A-B and B-C, why A cannot reach C?
    -> Peering is non-transitive

"between Transit Gateways" + different Regions
    -> TGW Peering Attachment

"temporary" + "as soon as possible" + on-prem + Internet
    -> Site-to-Site VPN

"most consistent performance" / "predictable bandwidth"
    -> Direct Connect

existing DX + "cost-effective backup"
    -> Site-to-Site VPN backup

"edge location close to customer gateway"
    -> Global Accelerator

S3 + private access
    -> Gateway Endpoint or Interface Endpoint

S3 + no additional endpoint charge
    -> Gateway Endpoint
```

---

# Final memory map / 最後記憶地圖

```text
2 VPCs              -> Peering bridge
Many VPCs           -> Transit Gateway station
TGW Region A <-> B   -> TGW Peering Attachment
A-B + B-C            -> NOT A-C (non-transitive)

Temporary hybrid    -> Site-to-Site VPN
Predictable hybrid  -> Direct Connect
DX backup cheaply   -> Site-to-Site VPN
Edge path optimize  -> Global Accelerator

S3 private path     -> Gateway OR Interface Endpoint
Private IP alone    -> NOT enough to create that path
```

> **Peering is a bridge. Transit Gateway is a station. VPN is an encrypted tunnel. Direct Connect is a dedicated highway. Endpoint is a private service entrance.**
>
> **Peering 是橋；Transit Gateway 是中央車站；VPN 是加密隧道；Direct Connect 是專用高速公路；Endpoint 是通往 AWS Service 的私人入口。**
