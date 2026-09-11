# Lab 08 Companion — VPC Peering From Scratch

[Class lab notes](../labs/lab-08-vpc-peering.md) | [Quick review](../cheat-sheets/lab-08-vpc-peering-cheat-sheet.md)

## Decision

**🟢 Yes — Very High Priority / Very High Learning Value**

This is a future personal AWS rebuild plan. It is **not deployed yet**.

## Core rule

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。**

A second rule is equally important:

> **At rebuild time, use the newest AWS Management Console GUI available that day. Verify current navigation and labels instead of following old screenshots mechanically.**
>
> **未來重建時，以當天最新 AWS Management Console GUI 為準；重新確認選單、按鈕與欄位名稱，不機械照抄舊截圖。**

The architecture and networking concepts are the target. Console button locations are temporary UI details.

## Why rebuild this lab personally?

AWS Academy supplied much of the starting architecture. A personal rebuild should prove that I can create the prerequisites myself and understand why every component exists.

The rebuild should answer:

```text
Can I design two non-overlapping VPCs myself?
Can I create the subnets and route tables myself?
Can I create and accept peering without Academy scaffolding?
Can I configure both forward and return routes correctly?
Can I create security rules for app-to-database traffic?
Can I enable Flow Logs and interpret TCP/3306 records?
Can I deliberately break one route and diagnose the failure?
```

## Target architecture

Use private RFC1918 CIDRs that do not overlap. The exact ranges can be chosen at rebuild time; the classroom ranges may be reused only if they still make sense.

```text
VPC A — Application VPC
+-------------------------------------+
| Public subnet                       |
|   small EC2 test/application host   |
|                                     |
| route to VPC B CIDR -> Peering      |
+-------------------------------------+
                   |
                   | VPC Peering
                   |
+-------------------------------------+
| VPC B — Data VPC                    |
|                                     |
| Private subnet                      |
|   database/test TCP service         |
|                                     |
| route to VPC A CIDR -> Peering      |
+-------------------------------------+
                   |
             VPC Flow Logs
                   |
             CloudWatch Logs
```

## Academy-prepared resources to recreate

The classroom environment gave us resources that a real personal environment will not magically provide. Recreate or deliberately replace each prerequisite:

| Academy provided/prepared | Personal rebuild responsibility |
| --- | --- |
| Lab VPC | Create VPC A from scratch |
| Shared VPC | Create VPC B from scratch |
| Public/private subnets | Design and create subnets |
| Existing route tables | Create/associate route tables deliberately |
| Application EC2 | Launch a minimal test/application EC2 or equivalent |
| Database | Create a low-cost database/test service appropriate for the exercise |
| Security groups | Create least-privilege rules from scratch |
| IAM role for Flow Logs | Create the current required IAM role/policy or use the current console-supported setup |
| CloudWatch log group | Create/configure as needed |
| Peering connection | Create and accept manually |

## Rebuild sequence

### Phase 1 — Current-GUI reconnaissance

Before creating anything:

1. Open the current AWS Console.
2. Confirm the current VPC console navigation and terminology.
3. Confirm current VPC Flow Logs creation workflow and IAM requirements.
4. Confirm current EC2 public IPv4 behavior and pricing implications.
5. Confirm current RDS/test-database options and pricing.
6. Write down only conceptual differences from the 2026 Academy UI; do not rewrite the architecture merely because the GUI changed.

### Phase 2 — Build both VPC foundations

Create VPC A and VPC B with **non-overlapping CIDRs**.

For each VPC:

- create required subnet(s)
- verify subnet CIDRs do not overlap
- create/associate route tables deliberately
- configure Internet access only where the chosen test architecture requires it
- create security groups with least privilege

Checkpoint:

```text
VPC A local routing works
VPC B local routing works
CIDRs do not overlap
No peering exists yet
```

### Phase 3 — Create test workloads

Create a small application/test host in VPC A and a database or simple TCP service in VPC B.

For a MySQL-compatible test, allow TCP `3306` only from the appropriate VPC A application security context/range according to the architecture chosen at rebuild time.

Do not expose the private database directly to the Internet.

### Phase 4 — Prove failure before peering

Before creating peering, attempt the private connection and observe failure.

This creates a useful baseline:

```text
Two VPCs exist
Workloads exist
No private inter-VPC road exists
Connection fails as expected
```

### Phase 5 — Create VPC Peering

Create a one-to-one peering connection:

```text
VPC A <-> VPC B
```

Accept it and verify **Active** state.

Do not add routes immediately. First reinforce:

> **Active peering means the bridge exists; it does not mean traffic has a road to the bridge.**

### Phase 6 — Configure routes one side at a time

Add VPC A's route to VPC B through the peering connection.

Test and reason about what is still missing.

Then add VPC B's return route to VPC A.

Retest connectivity.

This deliberately teaches bidirectional routing instead of entering both routes mechanically.

### Phase 7 — Enable VPC Flow Logs

Using the **current AWS GUI at rebuild time**, enable Flow Logs for the chosen VPC/interface scope and send logs to CloudWatch Logs.

Generate application/database traffic and locate the flow records.

Identify:

```text
source address
destination address
source port
destination port
protocol
action
log status
```

For MySQL-compatible traffic, confirm:

```text
3306 = MySQL
6    = TCP
```

### Phase 8 — Deliberate failure experiment

After proving the working architecture, intentionally break **one safe networking element**—preferably a peering route in a disposable environment—and observe the symptom.

Then diagnose from evidence:

```text
1. Is the application itself reachable?
2. Is peering Active?
3. Is the forward route correct?
4. Is the return route correct?
5. Are security rules correct?
6. What do Flow Logs show: ACCEPT, REJECT, or no expected flow?
```

Restore the correct configuration and verify recovery.

This recreates the strongest lesson from the classroom Gateway Timeout incident without relying on an accidental typo.

## Modern AWS comparison to include at rebuild time

Because AWS evolves, the personal rebuild should not assume VPC Peering is automatically the best architecture for every modern scenario. After completing the lab objective, briefly compare the current use cases and pricing/operational tradeoffs of:

```text
VPC Peering
Transit Gateway
AWS PrivateLink / VPC endpoints
Cloud WAN (if relevant)
```

The goal is not to replace the lab's peering exercise. The goal is to understand when one-to-one peering remains appropriate and when a different connectivity pattern scales better.

## Cost and cleanup guardrails

Before deployment, review **current AWS pricing on the rebuild date**. Pricing changes, so do not rely on old classroom assumptions.

Potential cost-bearing resources may include:

- EC2 runtime
- public IPv4 addresses
- RDS/database runtime and storage
- CloudWatch Logs ingestion/storage
- inter-AZ or data-transfer charges depending on architecture
- any optional networking service introduced during comparison

After the exercise, verify cleanup explicitly:

```text
[ ] terminate EC2 test instances
[ ] delete database/test service and unwanted snapshots
[ ] delete VPC Flow Logs if no longer needed
[ ] delete CloudWatch log groups if no longer needed
[ ] delete peering connection
[ ] delete temporary security groups
[ ] delete route tables/subnets as appropriate
[ ] delete both practice VPCs
[ ] verify no unexpected public IPv4/EIP or other billable resources remain
```

## Security/privacy rules for documentation

Never commit:

- AWS account IDs
- ARNs tied to temporary lab resources
- exact temporary public/private IPs unless deliberately sanitized examples
- VPC/subnet/route-table/peering/ENI/instance IDs
- RDS endpoints
- passwords or database secrets
- access keys, session tokens, `.pem`, or `.ppk` files

## Success criteria

The personal rebuild is complete only when I can explain and demonstrate:

```text
Why the VPC CIDRs must not overlap
Why Active peering alone is insufficient
Why both route tables need routes
How the application reaches the private database
How the return path works
How Flow Logs prove the traffic path
How to recognize TCP/3306
How to troubleshoot a broken route from symptoms and evidence
How the current AWS GUI differs from the old classroom UI without confusing UI with architecture
```

## Final memory

> **Peering = bridge. Route = road sign. Flow Logs = traffic evidence.**
>
> **Peering = 橋；Route = 路牌；Flow Logs = 車流證據。**

And for every future Academy rebuild:

> **Architecture first. Current GUI second. Old screenshots never become the source of truth.**
>
> **架構是核心；當天最新 GUI 是操作方式；舊截圖永遠不是唯一標準答案。**
