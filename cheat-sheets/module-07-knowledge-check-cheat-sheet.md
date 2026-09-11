# Module 7 Knowledge Check — VPC Networking Cheat Sheet

[Study index](../README.md) | [Full study notes](../knowledge-checks/module-07-knowledge-check.md)

## Core VPC map

```text
VPC = logically isolated virtual network

Internet
   |
  IGW
   |
Public Subnet
   |-- Public EC2
   |-- Bastion
   |-- NAT Gateway
   |
Private Subnet
   |-- Private EC2
```

**EC2 is not inherently public or private. It can live in either subnet.**

## Public vs private Internet

```text
Public EC2 -> route -> IGW -> Internet

Private EC2 (no public IP)
 -> private route
 -> NAT Gateway
 -> IGW
 -> Internet
```

> **Bastion = IN. NAT = OUT. / Bastion = 人進去；NAT = Private EC2 出去。**

Direct public IPv4 communication requires thinking separately about **public addressing + route + security**.

## CIDR sizing

AWS reserves **5 IPv4 addresses per subnet**.

| CIDR | Total | AWS usable |
| --- | ---: | ---: |
| /25 | 128 | 123 |
| /24 | 256 | 251 |
| /23 | 512 | 507 |
| /22 | 1024 | 1019 |

```text
usable = total - 5
```

**254 usable required -> /24 is too small -> /23 works.**

Subnet CIDRs in one VPC cannot overlap. Avoid overlapping AWS/on-premises CIDRs when networks may connect through VPN or Direct Connect.

## Elastic IP

```text
Auto public IPv4 -> may change after stop/start
Elastic IP       -> stable public IPv4
```

Personal memory:

> **EIP = Employee ID. Auto Public IP = visitor badge.**

Networking precision: EIP is a stable public address.

## Bastion SSH security groups

```text
Your source IP
 -> Bastion SG: SSH TCP 22
 -> Bastion
 -> Private EC2 SG: SSH TCP 22 from Bastion SG
 -> Private EC2
```

No extra SG return rule is required merely for response traffic because **Security Groups are stateful**.

## SG vs NACL

| SG | NACL |
| --- | --- |
| Resource/ENI level | Subnet level |
| Stateful | Stateless |
| Allow only | Allow + Deny |
| Return traffic remembered | Both directions evaluated |
| No numbered first-match list | Lower rule number first; first match wins |

### Important correction

A **custom NACL already has implicit deny**:

```text
100  allowed traffic   ALLOW
*    everything else   DENY
```

Do **not** add another deny-all rule just to reproduce the implicit deny.

Use an explicit deny when it must beat a broader allow:

```text
90   specific /32   DENY
100  all traffic    ALLOW
*    all traffic    DENY
```

## VPC address planning

Remember the reviewed Academy guidance:

- Separate subnet groups for unique routing requirements.
- Distribute address capacity across the AZs used by the design.
- Reserve address space for future growth.
- Do not size the whole VPC only for today's host count.
- Do not overlap subnet CIDRs.
- Avoid overlapping on-premises/AWS CIDRs for future hybrid connectivity.

## VPC Flow Logs

Direct destinations:

```text
VPC Flow Logs
├── CloudWatch Logs
├── S3
└── Kinesis Data Firehose
```

Not direct destinations:

```text
Athena -> can query logs in S3
AWS Console -> management UI
```

## Gateway VPC Endpoint

For this module's SAA recall:

> **Gateway Endpoint = S3 + DynamoDB**

Private EC2 can access S3/DynamoDB through the Gateway Endpoint without sending that service traffic through NAT/IGW.

## Weak spots repaired / 已修正

```text
EC2 != always private
/24 != 256 usable in AWS
IGW != public IP assignment
Custom NACL already has implicit DENY
Plan VPC CIDR for future growth
Flow Logs -> CloudWatch / S3 / Firehose
S3 private endpoint -> Gateway Endpoint
```

## 🧒 Mansion memory / 大宅院記憶

```text
VPC       = 大宅院
Public    = 前院
Private   = 內院
IGW       = 對外大門
Route     = 路牌
Bastion   = 前門管家（人進去）
NAT       = 採購管家（Private EC2 出去）
SG        = 有記憶的貼身警衛
NACL      = 無記憶的 Subnet 門衛
EIP       = 固定公開身分 / Employee ID
Flow Logs = 車流紀錄
Gateway Endpoint = AWS 內部專用道路
```

## Final SAA recall

```text
VPC = logically isolated network
Public EC2 can exist
Private EC2 can exist

AWS subnet usable IPv4 = total - 5

Stable public IPv4 -> EIP
Private outbound Internet -> NAT
Direct Internet gate -> IGW

SG   -> stateful
NACL -> stateless + allow/deny + first match

Flow Logs -> CloudWatch Logs / S3 / Firehose
Gateway Endpoint -> S3 / DynamoDB
```
