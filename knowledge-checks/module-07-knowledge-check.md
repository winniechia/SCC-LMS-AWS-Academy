# Module 7 Knowledge Check — VPC Networking

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/module-07-knowledge-check-cheat-sheet.md)

## Study-session record

This completed study session reviewed VPC fundamentals, subnet design, Internet access, Elastic IPs, Bastion access, Network ACLs, VPC Flow Logs, and VPC endpoints. These notes summarize the tested concepts and the corrections learned during review rather than reproducing the quiz verbatim.

The strongest connections came from the completed Challenge Lab 07, where the architecture was built and tested hands-on.

## 1. What a VPC is

A VPC is a **logically isolated virtual network that you define in the AWS Cloud**.

### Mansion analogy / 大宅院比喻

```text
AWS Cloud = a huge area of land
VPC       = your logically isolated mansion estate
Subnet    = a smaller yard/section inside the estate
```

> **VPC = my logically isolated network inside AWS. / VPC = 我在 AWS 裡自己圈出來的大宅院。**

A VPC is not itself a VPN, an on-premises extension, or a service that extends AWS hardware into customer premises.

## 2. EC2 can live in either a public or private subnet

An EC2 instance is not inherently public or private. Its placement, addressing, routing, and security configuration determine connectivity.

```text
VPC
├── Public Subnet
│   └── EC2 instance  ✅ possible
└── Private Subnet
    └── EC2 instance  ✅ possible
```

A public-subnet EC2 that communicates directly over IPv4 Internet typically needs:

```text
Public IPv4/EIP + route to IGW + security permission
```

A private-subnet EC2 can still reach the Internet outbound through NAT without having a public IP.

## 3. Public vs private Internet access

For a public workload:

```text
Public EC2 -> public route table -> IGW -> Internet
```

For a private workload that must download updates but must not be directly reachable from the Internet:

```text
Private EC2 (no public IP)
  -> private route table
  -> NAT Gateway
  -> IGW
  -> Internet
```

> **Bastion = people go IN. NAT = Private EC2 goes OUT. / Bastion = 人進去；NAT = Private EC2 出去。**

An IGW is the VPC's Internet gateway, but it does not itself assign public IP addresses.

## 4. CIDR sizing and AWS reserved IPv4 addresses

AWS reserves **5 IPv4 addresses in every subnet**. Therefore:

```text
usable IPv4 addresses = total subnet addresses - 5
```

Quick values:

| CIDR | Total | Usable in AWS subnet |
| --- | ---: | ---: |
| /25 | 128 | 123 |
| /24 | 256 | 251 |
| /23 | 512 | 507 |
| /22 | 1024 | 1019 |

A subnet that must grow to 254 usable addresses cannot use `/24`, because `/24` provides only 251 usable addresses in AWS. `/23` provides sufficient capacity.

> **Do not confuse total addresses with usable addresses. / 不要把總地址數當成可用地址數。**

## 5. Elastic IP — stable public identity

An Elastic IP provides a stable public IPv4 address that can remain associated with an EC2 instance across stop/start cycles when configured appropriately.

### Personal memory analogy

> **EIP = Employee ID / 固定員工編號**
>
> **Auto-assigned public IP = visitor badge / 臨時訪客證**

For networking precision, EIP is best understood as a **stable public street address**; the Employee ID analogy helps remember persistence.

## 6. Bastion security-group path

To SSH from the Internet through a Bastion to a Private EC2 instance, the intended SG chain is:

```text
Your source IP
   -> Bastion SG: allow SSH TCP 22
   -> Bastion Host
   -> Private EC2 SG: allow SSH TCP 22 from Bastion SG
   -> Private EC2
```

Security Groups are **stateful**, so separate rules are not required merely to allow return traffic for an already allowed connection.

> **SG = stateful bodyguard / 有記憶的貼身警衛。**

## 7. Custom NACL behavior and the implicit deny correction

A custom Network ACL is useful when subnet-level traffic must be limited to specific addresses.

Important behavior:

- NACLs are **stateless**.
- NACLs support **ALLOW and DENY** rules.
- Lower numbered rules are evaluated first.
- First match wins.
- A custom NACL already has an **implicit deny** (`* DENY`) for traffic that does not match an earlier allow rule.

### Learning correction

During the review, the first answer incorrectly assumed that an extra explicit "deny all other traffic" rule should be created. The course feedback clarified that this is unnecessary because the custom NACL already has the implicit deny.

Correct mental model:

```text
100  allowed addresses   ALLOW
*    everything else     DENY   <- already present
```

An **explicit DENY** is useful when a more specific flow must be blocked before a broader ALLOW rule, as in the Challenge Lab 07 `/32` ICMP experiment:

```text
90   one-host/32 ICMP   DENY
100  all traffic        ALLOW
*    all traffic        DENY
```

> **Implicit deny catches unmatched traffic. Explicit deny is for intentionally blocking something before a broader allow.**

## 8. VPC/subnet address-planning best practices

The reviewed Knowledge Check emphasized three design practices:

1. Create separate subnets where groups of hosts have unique routing requirements, typically per Availability Zone as needed.
2. Distribute address space sensibly across the Availability Zones used by the design.
3. Reserve address space for future growth.

Subnet CIDR blocks in the same VPC **cannot overlap**.

Also avoid reusing the same CIDR as an on-premises network if the environments may later connect through VPN or Direct Connect, because overlapping address space creates routing ambiguity.

### Learning correction

The first review answer chose "match the VPC CIDR block to the current number of required hosts." The course feedback instead emphasized planning enough address space for **significant growth**, not sizing the VPC only to today's host count.

> **Do not buy a mansion lot sized only for today's residents. Leave room for future wings. / 不要只按今天的人數買剛剛好的土地，要留擴充空間。**

## 9. VPC Flow Logs destinations

The Knowledge Check reinforced three direct delivery destinations for VPC Flow Logs:

```text
VPC Flow Logs
├── CloudWatch Logs
├── Amazon S3
└── Kinesis Data Firehose
```

Athena can query flow-log data stored in S3, but Athena is not itself a direct VPC Flow Logs delivery destination. The AWS Management Console is an administrative interface, not a log destination.

### Mansion analogy

- CloudWatch Logs = guard-room monitoring records
- S3 = long-term records warehouse
- Firehose = delivery conveyor/stream
- Athena = analyst who queries records in the warehouse

## 10. Gateway VPC Endpoint for S3

For an EC2 instance that must access Amazon S3 privately with **no additional endpoint charge and no throughput/packet limit imposed by the endpoint**, the key concept is a **Gateway VPC Endpoint**.

For SAA recall:

```text
Gateway VPC Endpoint -> Amazon S3 + DynamoDB
```

A Gateway Endpoint adds routes so VPC resources can reach supported services without sending that traffic through a NAT Gateway or Internet Gateway.

> **S3/DynamoDB -> think Gateway Endpoint first. / S3、DynamoDB -> 先想到 Gateway Endpoint。**

## Weak Spots Repaired / 已修正觀念

| Earlier confusion | Corrected distinction |
| --- | --- |
| EC2 is always private | EC2 can live in public or private subnets |
| `/24` means 256 usable addresses | AWS reserves 5; `/24` gives 251 usable |
| IGW alone makes an EC2 Internet-reachable | Public address + route + security are separate requirements |
| Custom NACL needs explicit deny-all | It already has implicit `* DENY` |
| VPC CIDR should match today's host count | Plan address space for meaningful future growth |
| Flow Logs -> Console | Direct destinations are CloudWatch Logs, S3, and Firehose |
| S3 private access -> Interface Endpoint | Gateway Endpoint is the key S3/DynamoDB option in this module |

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| AWS concept | Mansion analogy |
| --- | --- |
| VPC | 大宅院 / mansion estate |
| Public Subnet | 前院 / front yard |
| Private Subnet | 內院 / inner yard |
| IGW | 對外大門 / outside gate |
| Route Table | 路牌 / road signs |
| Bastion | 前門管家 / front-door manager |
| NAT Gateway | 採購管家 / purchasing butler |
| Security Group | EC2 貼身警衛 / personal bodyguard |
| NACL | Subnet 門衛 / subnet gate guard |
| EIP | 固定員工編號 / stable public identity |
| VPC Flow Logs | 大宅院車流紀錄 / traffic logbook |
| Gateway Endpoint | AWS 內部專用道路 / private service road |

### Short memory story

```text
VPC = 大宅院
Public EC2 can live in 前院
Private EC2 can live in 內院
IGW = 大門
Bastion = 人進去
NAT = Private EC2 出去
SG = 有記憶的貼身警衛
NACL = 無記憶、按號碼讀規則的門衛
EIP = 固定公開身分
S3/DynamoDB Gateway Endpoint = AWS 內部專用道路
```

## SAA exam takeaways

- VPC = logically isolated virtual network in AWS.
- EC2 can be in a public or private subnet.
- Public/private behavior depends on addressing, routing, gateways, and security—not instance type.
- Private EC2 outbound Internet -> NAT Gateway; direct inbound Internet is not required.
- AWS reserves 5 IPv4 addresses per subnet.
- Stable public IPv4 requirement -> Elastic IP.
- Bastion access requires Your IP -> Bastion SG and Bastion SG -> Private EC2 SG on SSH 22.
- Security Groups are stateful; NACLs are stateless.
- Custom NACL already includes implicit deny.
- Subnet CIDRs cannot overlap; hybrid networks should avoid overlapping on-premises/AWS CIDRs.
- Plan VPC address space for growth.
- Flow Logs destinations: CloudWatch Logs, S3, Firehose.
- S3/DynamoDB private service access -> Gateway VPC Endpoint.

## Final memory map

```text
VPC       -> logically isolated network
EC2       -> can be public-subnet or private-subnet
IGW       -> Internet gate
NAT       -> Private EC2 outbound Internet
EIP       -> stable public IPv4
SG        -> stateful, allow only
NACL      -> stateless, allow + deny, ordered first match
Flow Logs -> CloudWatch Logs / S3 / Firehose
Gateway Endpoint -> S3 / DynamoDB

AWS subnet usable IPv4 = total - 5

Bastion = IN
NAT     = OUT
SG      = bodyguard
NACL    = subnet gate guard
Route   = road sign
IGW     = outside gate
```
