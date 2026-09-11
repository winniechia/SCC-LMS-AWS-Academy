# Challenge Lab 07 — Personal AWS VPC Networking Rebuild Plan

[Class lab notes](../labs/challenge-lab-07-vpc-networking.md) | [Cheat sheet](../cheat-sheets/challenge-lab-07-vpc-networking-cheat-sheet.md) | [Study index](../README.md)

## Decision

**🟢 Yes — Very High Priority / Very High Learning Value**

**Status: plan only. Do not deploy until explicitly starting the future SAA review session.**

## Purpose

This companion lab is deliberately **not** a replay of the AWS Academy click sequence. The future exercise must start in a personal AWS account without the Academy lab environment and reconstruct the architecture and prerequisites ourselves.

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。**

The objective is to understand why every component exists, how the traffic flows, how to prove each layer works, and how to clean everything up safely.

## Academy assumptions to remove

The classroom challenge began inside a controlled Academy environment with a starting Lab VPC/context and temporary lab credentials/access. The personal rebuild must assume none of the following are ready for us:

```text
VPC
Subnets
Internet Gateway
Route tables
NAT Gateway / Elastic IP
EC2 instances
Security groups
NACLs
SSH key pairs
IAM permissions
Temporary lab credentials
Test Instance
```

Before building, inspect the personal account and Region so existing unrelated resources are not mistaken for lab prerequisites.

## Target architecture from scratch

```text
Personal AWS account

Internet
   |
   v
Internet Gateway
   |
   +---------------- Public Subnet ----------------+
   |                                               |
Bastion Host                                  NAT Gateway
(public IPv4)                                 + public IPv4/EIP
   |                                               |
   |                                               |
   +------------------- VPC local -----------------+
                                                   |
Private Route Table: 0.0.0.0/0 -> NAT             |
                                                   v
                                           Private Subnet
                                                   |
                                              Private NACL
                                                   |
                                            Private Instance

Later:
Test Instance in Public Subnet
Private Instance -> Test private IP -> NACL /32 deny experiment
```

## Build sequence

### Phase 0 — cost, security, and identity checkpoint

Before creating anything:

- Confirm the intended AWS account and Region.
- Confirm current IAM identity/permissions.
- Estimate/understand charges for NAT Gateway, public/Elastic IPv4, EC2, and data processing/transfer where applicable.
- Choose a short build window so cost-bearing resources can be removed the same day.
- Create a cleanup checklist before deployment.
- Never place AWS credentials or SSH private keys in the repository.

### Phase 1 — create the network foundation

Build the VPC ourselves instead of using an Academy-provided Lab VPC.

Suggested learning CIDRs:

```text
VPC:            10.0.0.0/16
Public Subnet:  10.0.0.0/24
Private Subnet: 10.0.1.0/24
```

Create and attach an Internet Gateway. Create/identify the public route table and explicitly associate the Public Subnet. Add:

```text
0.0.0.0/0 -> IGW
```

Create a separate Private Route Table and explicitly associate the Private Subnet.

Checkpoint question: **What makes the Public Subnet public?** Answer in terms of routing, not its name.

### Phase 2 — build traditional Bastion access

Create a Bastion security group allowing SSH only from the learner's current public IP `/32`. Launch a small Linux Bastion in the Public Subnet with a public IPv4 address.

Verify:

```text
Local PC -> Bastion SSH
```

Do not open SSH to `0.0.0.0/0` merely for convenience.

### Phase 3 — build Private Instance access

Create a Private Instance SG whose SSH inbound source is the Bastion SG. Launch a small Linux Private Instance in the Private Subnet with **no public IPv4 address**.

Use separate credentials for Bastion and Private Instance so credential separation is visible in the exercise.

Configure local SSH agent forwarding. Keep the private key on the local machine rather than copying it to Bastion.

Verify:

```text
Local PC -> Bastion -> Private Instance
```

Troubleshooting rule:

```text
Timeout -> investigate road/guards/service
Permission denied (publickey) -> investigate authentication/key
```

Remember the Academy lesson: **same key-pair label does not guarantee the same cryptographic key material.**

### Phase 4 — create NAT outbound path

Create a NAT Gateway in the Public Subnet with the required public addressing. Update the Private Route Table:

```text
0.0.0.0/0 -> NAT Gateway
```

From the Private Instance, verify outbound Internet connectivity. Do not confuse NAT with Bastion:

> **Bastion = people go IN. NAT = Private EC2 goes OUT.**

### Phase 5 — custom Private NACL

Create a custom NACL. Before associating it with the Private Subnet, deliberately inspect its default restrictive behavior.

Add safe baseline rules in both directions, then associate it with the Private Subnet and re-test connectivity.

This phase must reinforce:

```text
SG = stateful + allow only
NACL = stateless + allow/deny + numbered first-match rules
```

### Phase 6 — reproduce the `/32` deny experiment

Launch a temporary Test Instance and permit ICMP appropriately for the experiment.

From Private Instance, continuously ping the Test Instance's **private IPv4**. Confirm replies first.

Then add a higher-priority Private NACL outbound rule that denies ICMP only to the Test Instance `/32`, ahead of the general allow rule.

Expected evidence:

```text
Before deny: ping replies
After lower-number /32 deny: replies stop
```

Explain why this traffic uses the VPC local route and does not require NAT/IGW.

### Phase 7 — modern access comparison

After successfully rebuilding the traditional Bastion architecture, compare it with **AWS Systems Manager Session Manager**.

Questions to answer:

- Which Bastion-related exposure and credential-handling concerns can Session Manager reduce?
- What IAM, SSM Agent, and network prerequisites does Session Manager introduce?
- When might a Bastion still be encountered in legacy architectures or exam scenarios?

Do not skip the Bastion build: the purpose is to understand the traditional architecture first, then compare the modern alternative.

## Evidence checklist

The rebuild is not complete merely because resources exist. Capture sanitized evidence for each claim:

```text
[ ] Public subnet has default route to IGW
[ ] Private subnet uses separate private route table
[ ] Bastion has intended public reachability
[ ] Private Instance has no public IPv4
[ ] Private SG references Bastion SG for SSH
[ ] Local -> Bastion SSH works
[ ] Bastion -> Private SSH works via agent forwarding
[ ] Private -> Internet works through NAT
[ ] Custom NACL associated without breaking baseline traffic
[ ] Private -> Test private-IP ping works
[ ] Higher-priority /32 NACL deny stops only intended ICMP flow
[ ] SSM comparison documented
[ ] Cleanup verified
```

Never record temporary IDs, exact public addresses, credentials, private keys, or key fingerprints in GitHub.

## 3rd-grade teaching checkpoint

Before declaring success, explain the entire architecture using the mansion model:

```text
VPC = 大宅院
Public Subnet = 前院
Private Subnet = 內院
IGW = 對外大門
Route Table = 路牌
Bastion = 前門管家（人進去）
NAT = 採購管家（Private EC2 出去）
SG = EC2 貼身警衛
NACL = Subnet 門衛
```

If Bastion vs NAT or SG vs NACL cannot be explained without memorized AWS wording, review before moving on.

## Cleanup plan — mandatory

Cleanup is part of the lab, not an optional afterthought.

Suggested reverse-order review:

```text
Test Instance
Private Instance
Bastion Host
NAT Gateway
Elastic/Public IPv4 resources
Custom NACL
Security groups
Route tables
Subnets
Internet Gateway detach/delete
VPC
```

Wait for dependencies to release where AWS requires it. Then inspect the Region again for retained lab resources.

**Cost warning:** NAT Gateway, public/Elastic IPv4 resources, EC2 instances, and related traffic can incur charges in a personal account. Pricing can change, so check current AWS pricing immediately before the rebuild rather than relying on this study plan for dollar amounts.

## Final success definition

The future personal rebuild is complete only when we can say:

> I created the network architecture myself without AWS Academy pre-lab setup, explained every traffic path, proved Bastion and NAT solve different problems, demonstrated SG vs NACL behavior, reproduced the `/32` deny experiment, compared Bastion with Session Manager, and verified cleanup.

**我不是把課堂按鈕再按一次；我是從空白 AWS 環境把整座大宅院重新蓋起來，而且知道每一扇門、每一條路、每一個警衛為什麼存在。**
