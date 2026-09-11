# Challenge Lab 07 — VPC Networking Cheat Sheet

[Full notes](../labs/challenge-lab-07-vpc-networking.md) | [Study index](../README.md) | [Companion plan](../personal-labs/challenge-lab-07-vpc-networking-companion-plan.md)

## One-picture mental model

```text
Internet
   |
  IGW  <- outside gate / 對外大門
   |
Public Subnet (front yard / 前院)
   |-- Bastion Host  <- people go IN / 人進去
   |-- NAT Gateway   <- Private EC2 goes OUT / 私有 EC2 出去
   |
Private Route: 0.0.0.0/0 -> NAT
   |
Private Subnet (inner yard / 內院)
   |
Private Instance
```

> **Bastion = IN. NAT = OUT. / Bastion = 進；NAT = 出。**

## Fast SAA map

| Concept | Remember |
| --- | --- |
| VPC | 大宅院 / mansion estate |
| Public Subnet | Has route to IGW; name alone proves nothing |
| Private Subnet | No direct IGW default route; lab default route goes to NAT |
| IGW | Internet gate; does not assign public IPs |
| Route Table | Road signs / 路牌 |
| Bastion | Administrative jump host / 前門管家 |
| NAT Gateway | Private outbound Internet / 採購管家 |
| SG | Resource/ENI bodyguard; stateful; allow only |
| NACL | Subnet gate guard; stateless; allow + deny |
| `/32` | One IPv4 address |
| `0.0.0.0/0` | All IPv4 destinations |
| SSH | TCP 22 by default |
| ICMP | Ping/connectivity testing, not SSH |

## Roads vs permission

> **有路 ≠ 有權限；有權限 ≠ 有路。**
>
> **A route is not permission, and permission is not a route.**

For direct public IPv4 communication, think separately about:

```text
Public address + route to IGW + security permission
```

## Bastion vs NAT

**Bastion Host**

```text
Administrator -> Bastion -> Private Instance
```

Traditional inbound administration path.

**NAT Gateway**

```text
Private Instance -> NAT -> IGW -> Internet
```

Outbound path. NAT does not accept unsolicited inbound Internet SSH to the Private Instance.

## SG vs NACL

| Security Group | Network ACL |
| --- | --- |
| Resource/ENI level | Subnet level |
| Stateful | Stateless |
| Allow rules | Allow + Deny rules |
| Return traffic tracked | Both directions evaluated independently |
| No numbered priority | Lower rule number evaluated first |

**NACL: first matching rule wins.**

Example:

```text
90   ICMP   one-host/32   DENY
100  ALL    0.0.0.0/0     ALLOW
```

Traffic to that `/32` matches 90 first and is denied. Rule 100 never gets a chance.

## SG-to-SG SSH

Private Instance inbound:

```text
SSH TCP 22
Source: Bastion Host SG
```

This authorizes traffic based on the source security-group relationship rather than a temporary Bastion IP.

## SSH troubleshooting ladder

```text
Timeout
  -> investigate route, address, SG/NACL, host/service reachability

Permission denied (publickey)
  -> network reached SSH; investigate authentication/key
```

Useful diagnostics from the completed lab:

```bash
ssh-add -L
ssh -vvv ec2-user@<PRIVATE_IP>
```

The lab exposed an important temporary-environment trap: Pageant contained an old key with the same `vockey2` label. Loading the newly downloaded current-session key fixed SSH.

> **Same name != same cryptographic key. / 同名 key pair != 同一把密碼學鑰匙。**

Never commit `.pem`, `.ppk`, fingerprints, or private-key material.

## Agent forwarding

```text
Local Pageant holds key
       |
       v
Bastion requests signature through forwarded agent
       |
       v
Private Instance authenticates
```

Goal: do not copy the Private Instance private key onto the Bastion.

## Lab verification chain

```text
Bastion SSH -> Private Instance        PASS
Private Instance -> 8.8.8.8 via NAT   PASS
Private NACL allow both directions     PASS
Private -> Test private-IP ping        PASS
Add outbound NACL rule 90 /32 DENY     Ping stops
```

The Private-to-Test private-IP ping uses the VPC `local` route, not NAT/IGW.

## Question concepts worth remembering

- IGW: Internet communication for appropriately routed/addressed public resources.
- NAT Gateway: Private Instance outbound Internet.
- Separate Bastion/Private keys: reduces credential blast radius.
- Bastion cannot ping Private merely because SSH 22 is allowed; ICMP is separate.
- Private -> Test ping: Private SG outbound + Test SG inbound ICMP; SG statefulness permits return traffic.

## 3rd-grade memory line

> **IGW = 大門；Route = 路牌；Bastion = 進；NAT = 出；SG = 貼身警衛；NACL = 子網門衛。**

## Personal rebuild

**🟢 Yes — Very High Priority / Very High Learning Value**

Future SAA review must rebuild the architecture from scratch in a personal AWS account without assuming AWS Academy resources exist.

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。**

Personal-account warning: NAT Gateway, public/Elastic IPv4 resources, and EC2 can incur charges. Build in a tightly timed session and clean up immediately afterward.
