# Module 8 Knowledge Check — Quick Review

[Full bilingual notes](../knowledge-checks/module-08-knowledge-check.md) | [Study index](../README.md)

## Core decision map / 核心判斷圖

| Requirement | First thought | 大宅院比喻 |
| --- | --- | --- |
| Two VPCs | VPC Peering | 私人橋 |
| Many VPCs | Transit Gateway | 中央轉運站 |
| TGW across Regions | TGW Peering Attachment | 跨區鐵路 |
| Temporary on-prem -> AWS | Site-to-Site VPN | 加密臨時隧道 |
| Predictable/dedicated on-prem -> AWS | Direct Connect | 專用高速公路 |
| Optimize path into AWS edge | Global Accelerator | 最近 AWS 高速公路入口 |
| Private S3 connectivity | Gateway or Interface Endpoint | AWS 私人道路 / 私人門 |

## 10-question memory / 十題記憶

1. **100 VPCs -> Transit Gateway.** / 很多 VPC 不要蓋 Peering 蜘蛛網，使用中央轉運站。
2. **TGW <-> TGW across Regions -> TGW Peering Attachment.** / 兩座中央車站用跨區鐵路相連。
3. **Two non-overlapping VPCs -> VPC Peering.** / 兩座大宅院蓋私人橋。
4. **S3 private path -> Gateway Endpoint OR Interface Endpoint.** Course grading was inconsistent even when both were selected. / 課程第二輪判分與自己的 feedback 不一致。
5. **A-B + B-C != A-C.** VPC Peering is non-transitive. / Peering 不能借道。
6. **Temporary + ASAP + secure + Internet -> Site-to-Site VPN.** / 快速搭加密隧道。
7. **Nearby AWS edge/global network path -> Global Accelerator.** / 找最近 AWS 高速公路入口。
8. **Most consistent performance -> Direct Connect.** / 專用道路較可預測。
9. **Existing DX + cost-effective backup -> Site-to-Site VPN.** / 主幹專線 + 便宜備用道路。
10. **Predictable bandwidth + resiliency -> DX primary + VPN failover.** / Direct Connect 管穩，VPN 管斷線備援。

## Critical corrections / 重要修正

> **Private IP != private path. / Private IP 不等於 private path。**

A private IP alone does not create private S3 connectivity. Use a VPC endpoint path.

> **S3 != Gateway Endpoint only.**

Amazon S3 supports Gateway Endpoints and Interface Endpoints. Gateway Endpoint remains the strong answer when the question emphasizes no additional endpoint charge.

> **Peering is non-transitive. / Peering 不能借道。**

```text
A <-> B
B <-> C
A <-X-> C
```

## VPN vs Direct Connect / VPN 與 Direct Connect

```text
Temporary / ASAP / existing Internet
-> Site-to-Site VPN

Consistent / predictable / dedicated
-> Direct Connect

Direct Connect already exists + cost-effective backup
-> Site-to-Site VPN backup

Predictability + failover
-> Direct Connect primary + VPN secondary
```

## 3rd-grade analogy / 三年級比喻

```text
VPC Peering       = 私人橋
Transit Gateway   = 中央轉運站
TGW Peering       = 跨區鐵路
Site-to-Site VPN  = 公共道路上的加密隧道
Direct Connect    = 專用高速公路
Global Accelerator= 最近的 AWS 高速公路入口
Gateway Endpoint  = Route Table 上的 AWS 專用道路
Interface Endpoint= Subnet 裡的 PrivateLink 私人門
```

## Exam triggers / 考試關鍵字

```text
many VPCs                     -> Transit Gateway
two VPCs                      -> VPC Peering
non-transitive                -> VPC Peering limitation
between TGWs                  -> TGW Peering Attachment
temporary + ASAP              -> Site-to-Site VPN
predictable bandwidth         -> Direct Connect
cost-effective DX backup      -> Site-to-Site VPN
edge location                 -> Global Accelerator
S3 private                    -> Gateway or Interface Endpoint
S3 + no additional charge     -> Gateway Endpoint
```

## One-line memory / 一句記憶

> **Peering = bridge; Transit Gateway = station; VPN = encrypted tunnel; Direct Connect = dedicated highway; Endpoint = private service entrance.**
>
> **Peering = 橋；Transit Gateway = 車站；VPN = 加密隧道；Direct Connect = 專用高速公路；Endpoint = 私人服務入口。**
