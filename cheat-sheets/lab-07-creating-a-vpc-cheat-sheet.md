# Lab 07 — Creating a VPC Cheat Sheet

[Study index](../README.md) | [Full notes](../labs/lab-07-creating-a-vpc.md) | [Companion plan](../personal-labs/lab-07-vpc-companion-plan.md)

## Core architecture

```text
Internet
  |
Lab IGW
  |
Lab VPC 10.0.0.0/16
  |
  +-- Public Subnet 10.0.0.0/24
  |     Public Route Table
  |       10.0.0.0/16 -> local
  |       0.0.0.0/0   -> IGW
  |     App Server + App-SG HTTP/80
  |
  +-- Private Subnet 10.0.2.0/23
        Private/Main Route Table
          10.0.0.0/16 -> local
```

**No NAT Gateway or Bastion is part of this Guided Lab.**

## CIDR quick review

| CIDR | Total IPv4 addresses | Memory |
| --- | ---: | --- |
| `/16` | 65,536 | Whole estate / 整座大宅院 |
| `/23` | 512 | Two `/24` blocks |
| `/24` | 256 | Smaller subnet |
| `/32` | 1 | One exact address |

`10.0.2.0/23` spans `10.0.2.x` and `10.0.3.x`.

## What makes a subnet public?

Not its name. Not public-IP auto-assignment alone.

```text
Public Subnet
   + associated Public Route Table
   + 0.0.0.0/0 -> Internet Gateway
```

For direct IPv4 internet reachability, the instance also needs a public IPv4/EIP and security controls that allow the intended traffic.

> **Public IP = address. Route = road. IGW = outside gate.**
>
> **Public IP 是門牌；Route 是道路；IGW 是對外大門。**

## Route Table vs Security Group

- Route Table: **Where should traffic go? / 路怎麼走？**
- Security Group: **Is this traffic allowed at the resource? / 警衛放不放行？**
- `10.0.0.0/16 -> local`: route inside the VPC.
- `0.0.0.0/0 -> IGW`: IPv4 default route to the internet.

**有路 ≠ 有權限；有權限 ≠ 有路。**

## App Server

- Amazon Linux 2023
- `t2.micro`
- `vockey`
- Lab VPC
- Public Subnet
- public IPv4 enabled
- `App-SG`
- HTTP TCP 80 from `0.0.0.0/0`
- `Inventory-App-Role` — Academy-prepared
- User Data installs and starts the inventory web application

## Actual troubleshooting lessons

1. **Missing public route:** the Public Route Table initially had only the local route. The web app timed out until `0.0.0.0/0 -> Lab IGW` was added.
2. **Check by layers:** IGW, route table, subnet association, SG, NACL, EC2 status, bootstrap/application, then protocol.
3. **DNS name vs protocol:** explicit `http://<PUBLIC_DNS_NAME>` reached the app. The lab allowed HTTP/80; a copied hostname alone did not specify the intended scheme.
4. **2/2 status checks ≠ application health:** system logs/cloud-init help verify bootstrap work.
5. **Console timing matters:** refresh/verify before creating replacement VPC resources when a newly created dependency appears missing.

## 🧒 Mansion analogy / 大宅院

| AWS | Analogy |
| --- | --- |
| VPC | Entire mansion estate / 整座大宅院 |
| Public Subnet | Front yard / 前院 |
| Private Subnet | Inner/back yard / 內院、後院 |
| IGW | Outside gate / 對外大門 |
| Route Table | Road signs / 路標 |
| Security Group | Server's guard / 主機旁警衛 |
| NACL | Subnet gate guard / 院子大門警衛 |
| Public IPv4 | Public address / 對外門牌 |
| User Data | First-day work list / 開工清單 |

> The house may have a public address, but visitors still need a road to the outside gate, and the guard must allow the intended traffic.

## SAA traps

- “Public Subnet” is a routing property, not a naming property.
- IGW attaches to the **VPC**, not to a subnet.
- Auto-assign public IPv4 does not create an IGW route.
- SG and route table solve different problems.
- SG is stateful and allow-only; NACL is subnet-level and stateless with allow/deny rules.
- `/16` has more addresses than `/23`; `/23` has more than `/24`.
- HTTP normally uses TCP 80; HTTPS normally uses TCP 443.
- This Guided Lab's private subnet has no NAT Gateway.

**Personal rebuild: 🟢 Yes — Very High Priority / Very High Learning Value.** Plan only; no personal deployment is claimed.
