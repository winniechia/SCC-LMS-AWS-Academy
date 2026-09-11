# Challenge Lab 07 — VPC Networking, Bastion, NAT, and NACL

[Study index](../README.md) | [Cheat sheet](../cheat-sheets/challenge-lab-07-vpc-networking-cheat-sheet.md) | [Companion plan](../personal-labs/challenge-lab-07-vpc-networking-companion-plan.md)

## Completed work and scope

This AWS Academy Challenge Lab was completed, submitted, and graded successfully. The exercise built and verified a public/private VPC architecture with a bastion host, NAT Gateway, security-group references, SSH agent forwarding, a custom network ACL, and a final ICMP deny experiment.

Temporary IP addresses, DNS names, account IDs, instance IDs, security-group IDs, subnet IDs, route-table IDs, NACL IDs, NAT IDs, Elastic IPs, ARNs, key fingerprints, and private-key material are intentionally omitted.

## Architecture built

```text
Internet
   |
Internet Gateway (IGW)
   |
Public Subnet
   |-- Bastion Host
   |-- NAT Gateway + Elastic IP
   |
   +-----------------------------+
                                 |
Private Route Table             |
0.0.0.0/0 -> NAT Gateway        |
        |                        |
Private Subnet                  |
   |                            |
Private NACL                    |
   |                            |
Private Instance                |
```

A Test Instance was later launched in the Public Subnet for the final ICMP/NACL experiment.

## 1. VPC, subnets, IGW, and routes

The lab architecture used:

- Lab VPC: `10.0.0.0/16`
- Public Subnet: `10.0.0.0/24`
- Private Subnet: `10.0.1.0/24`
- Public default route: `0.0.0.0/0 -> IGW`
- Private default route: `0.0.0.0/0 -> NAT Gateway`
- VPC-local traffic: `10.0.0.0/16 -> local`

A subnet is not public merely because it is named "Public Subnet." The routing relationship matters: a public subnet has a route to an Internet Gateway. For direct IPv4 Internet communication, an instance also needs an appropriate public IPv4/EIP and security controls.

> **Public IP = address. Route = road. IGW = outside gate. / Public IP 是門牌；Route 是道路；IGW 是對外大門。**

## 2. Bastion Host — people go IN

The Bastion Host was launched in the Public Subnet with a public IPv4 address. Its security group allowed SSH TCP 22 from the learner's current public IP.

The successful management path was:

```text
Windows PC -> Internet -> IGW -> Public route -> Bastion SG -> Bastion Host
```

PuTTY used the Amazon Linux username `ec2-user`. A key-file name does not determine the Linux username.

> **Bastion = people go IN. / Bastion = 人進去。**

The bastion is an administrative jump host. It is not a NAT Gateway and does not provide the Private Instance's outbound Internet path.

## 3. NAT Gateway — Private EC2 goes OUT

A public NAT Gateway with an Elastic IP was created in the Public Subnet. The Private Route Table sent `0.0.0.0/0` to the NAT Gateway and was explicitly associated with the Private Subnet.

The verified outbound path was:

```text
Private Instance -> Private Route Table -> NAT Gateway -> IGW -> Internet
```

From the Private Instance, `ping -c 4 8.8.8.8` returned 4/4 replies with 0% packet loss. This verified outbound connectivity after the NAT path was configured.

> **Bastion = people go IN. NAT = Private EC2 goes OUT. / Bastion = 人進去；NAT = Private EC2 出去。**

NAT does not make the Private Instance directly reachable from unsolicited Internet connections.

## 4. Private Instance and SG-to-SG SSH

The Private Instance was launched in the Private Subnet without a public IPv4 address. Its security group allowed SSH TCP 22 with the Bastion Host security group as the source.

This is a security-group reference, not a hard-coded Bastion IP. It expresses which protected source resources may initiate the allowed traffic.

> **有路 ≠ 有權限；有權限 ≠ 有路。 / A route is not permission, and permission is not a route.**

The Private Instance used a separate key pair from the Bastion Host. Separate credentials can reduce the impact of a compromised bastion.

## 5. Pageant and SSH agent forwarding

On Windows, Pageant held the SSH keys locally. PuTTY was configured with **Allow agent forwarding**, enabling the Bastion to request authentication through the forwarded agent without copying the Private Instance's private key onto the Bastion.

```text
Windows
  |
Pageant (keys stay local)
  |
SSH agent forwarding
  |
Bastion Host
  |
SSH
  |
Private Instance
```

Private keys must never be committed to GitHub. Files such as `.pem` and `.ppk` are sensitive credentials.

### Troubleshooting: `Permission denied (publickey)`

The first Bastion-to-Private SSH attempt reached the Private Instance but failed with public-key authentication. This was different from a timeout: the network path and SSH service were reachable, but authentication failed.

Diagnostic evidence:

- `ssh-add -L` showed two public keys available through the forwarded agent.
- `ssh -vvv` showed SSH offering both agent keys, including a key labeled `vockey2`.
- Both were initially rejected by the Private Instance.
- Pageant had retained an older key from a previous Academy session with the same `vockey2` label.
- Removing that old loaded key and adding the `vockey2.ppk` downloaded for the current lab session fixed authentication.
- SSH from Bastion to Private Instance then succeeded.

> **Same key-pair name does not mean the same cryptographic key. / 同名 key pair 不代表是同一把密碼學鑰匙。**

This is especially important in temporary/reset lab environments where a same-named key pair may be recreated with new key material.

## 6. Security Group vs Network ACL

A custom `Private NACL` was created for the Private Subnet.

Initial safe rules before association:

```text
Inbound:
100  All traffic  0.0.0.0/0  ALLOW
*    All traffic  0.0.0.0/0  DENY

Outbound:
100  All traffic  0.0.0.0/0  ALLOW
*    All traffic  0.0.0.0/0  DENY
```

Only after both directions were allowed was the custom NACL associated with the Private Subnet. A new custom NACL begins restrictive, so associating it too early can interrupt traffic.

| Control | Mansion analogy | Behavior |
| --- | --- | --- |
| Security Group | EC2's personal bodyguard / 貼身警衛 | Stateful, allow rules only |
| Network ACL | Subnet gate guard / 子網門衛 | Stateless, allow and deny rules |

Because NACLs are stateless, inbound and outbound directions are evaluated independently. Security groups are stateful and automatically permit response traffic for an allowed connection.

## 7. Exact final ICMP experiment — Steps 43–44

A Test Instance was launched in the Public Subnet. Its Test SG allowed All ICMP IPv4 so the security group would not obscure the NACL experiment.

### Step 43 — establish the working baseline

From the Private Instance, a continuous ping to the Test Instance's **private IPv4 address** succeeded.

Because both instances were in the same VPC, this traffic used the VPC `local` route. NAT and the Internet Gateway were not required for this private-IP-to-private-IP path.

### Step 44 — add a higher-priority deny

An outbound Private NACL rule was added:

```text
90   All ICMP IPv4   <TEST_INSTANCE_PRIVATE_IP>/32   DENY
100  All traffic     0.0.0.0/0                      ALLOW
*    All traffic     0.0.0.0/0                      DENY
```

The previously working continuous ping stopped receiving replies after rule 90 was saved.

This verified:

- NACLs can explicitly deny traffic.
- Lower rule numbers are evaluated first.
- First matching rule wins.
- `/32` identifies one IPv4 address.
- A general allow rule 100 does not override a matching deny rule 90.

## 8. Knowledge-check connections

The completed questions reinforced these concepts:

- An Internet Gateway enables Internet communication for appropriately routed public-subnet instances with public addressing; it does not itself assign public IP addresses.
- A NAT Gateway provides outbound Internet connectivity for the Private Instance.
- Separate key pairs can reduce the impact of a compromised bastion.
- Bastion-to-Private ping is not automatically allowed when the Private Instance SG permits only SSH.
- For Private Instance -> Test Instance ping, the initiating Private SG needs outbound permission and the Test SG needs inbound ICMP permission; SG statefulness permits the response traffic.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

Think of the VPC as a large mansion estate:

| AWS concept | Mansion analogy |
| --- | --- |
| VPC | 大宅院 / mansion estate |
| Public Subnet | 前院 / front yard |
| Private Subnet | 內院 / inner yard |
| Internet Gateway | 對外大門 / outside gate |
| Route Table | 路牌 / road signs |
| Bastion Host | 前門管家 / front-door manager |
| NAT Gateway | 採購管家 / outbound purchasing butler |
| Security Group | EC2 貼身警衛 / personal bodyguard |
| NACL | Subnet 門衛 / estate gate guard |
| Public IP | 對外門牌 / public street address |
| SSH key | 私人鑰匙 / private key |
| Pageant | 隨身安全鑰匙圈 / secure key ring |
| `/32` | 只指定一個 IPv4 地址 / exactly one IPv4 address |

The shortest memory:

> **Bastion = IN. NAT = OUT. SG = bodyguard. NACL = subnet gate guard. Route = road sign. IGW = outside gate.**
>
> **Bastion = 進；NAT = 出；SG = 貼身警衛；NACL = 子網門衛；Route = 路牌；IGW = 對外大門。**

## SAA takeaways

- Public/private subnet behavior is primarily about routing, not names.
- IGW and NAT Gateway solve different problems.
- A NAT Gateway belongs in a public subnet when serving private-subnet outbound Internet access.
- Bastion and NAT are not interchangeable.
- SG-to-SG references avoid coupling access to temporary instance IPs.
- Security Groups are stateful; NACLs are stateless.
- NACLs support explicit deny and process lower rule numbers first.
- `/32` targets one IPv4 address.
- A timeout and an authentication failure are different troubleshooting evidence.
- SSH agent forwarding can avoid storing a private key on a bastion.
- Same key-pair label does not prove the underlying key material is the same.

## Lab Completion Checkpoint / Lab 結束檢查點

### 1. Class Lab Complete

**✅ Yes — completed, submitted, and graded successfully.**

Exact Steps 43–44 were completed in this run. The continuous Private-to-Test ping worked before the `/32` NACL deny and stopped after the higher-priority deny rule was applied.

### 2. SAA Takeaways

The key exam model is to separate addressing, routing, gateways, and security controls. Know why Bastion handles administrative inbound access, NAT handles private outbound access, SGs are stateful, and NACLs are stateless with ordered allow/deny rules.

### 3. 3rd-Grade Understanding Check

Can I explain these without AWS jargon?

- Why does the front yard use the outside gate while the inner yard uses the purchasing butler?
- Why can a road exist but a guard still refuse entry?
- Why can the subnet gate guard say DENY while the EC2 bodyguard cannot?
- Why does rule 90 beat rule 100?
- Why is `/32` one specific visitor/address?

### 4. What AWS Academy Prepared for Me

The Academy environment supplied the controlled lab context and at least the starting Lab VPC/environment needed for the challenge. The classroom environment also supplied temporary credentials and lab-specific access constraints. A future personal-account rebuild must not assume these resources, permissions, keys, or defaults exist.

**These notes are a Class Lab Record, not a guaranteed from-scratch personal-account runbook.**

### 5. Personal AWS Rebuild Decision

**🟢 Yes — Very High Priority / Very High Learning Value**

This is one of the strongest personal rebuild candidates because it connects VPC design, subnetting, routing, IGW, NAT, Bastion/SSH, SG-to-SG rules, agent forwarding, NACL behavior, and evidence-based troubleshooting.

### 6. Follow-up Companion Lab

See [Challenge Lab 07 companion plan](../personal-labs/challenge-lab-07-vpc-networking-companion-plan.md).

During future SAA review, rebuild this architecture **from scratch in a personal AWS environment with ChatGPT guidance, without relying on AWS Academy pre-lab setup**.

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。**

### 7. Cleanup Check

The Academy lab was submitted and graded. A complete resource-by-resource cleanup audit was not recorded here. In a personal AWS account, cleanup must be explicit and immediate, especially for NAT Gateway, Elastic/Public IPv4 resources, and EC2 instances because they can incur charges.

Class Lab -> Documentation -> SAA Review -> 3rd-Grade Check -> Personal Rebuild Decision -> Cleanup Review

上課 Lab -> 文件化 -> SAA 複習 -> 三年級理解檢查 -> Personal Rebuild 判斷 -> Cleanup 檢查
