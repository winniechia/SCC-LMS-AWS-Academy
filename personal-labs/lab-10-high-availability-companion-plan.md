# Lab 10 — Highly Available Environment Personal Rebuild Plan

[Class lab notes](../labs/lab-10-creating-a-highly-available-environment.md) | [Cheat sheet](../cheat-sheets/lab-10-high-availability-cheat-sheet.md) | [Study index](../README.md)

## Decision

**Yes — Very High Learning Value. Plan only; not yet built in the personal AWS account.**

The purpose is not to replay AWS Academy clicks. It is to rebuild the architecture that Academy had prepared and understand every dependency from zero.

## Learning rule

For **every single step**, stop and answer:

1. What am I creating?
2. Why does it exist?
3. What does it connect to?
4. What traffic/data depends on it?
5. What breaks if I omit or misconfigure it?
6. How would I verify it?
7. What SAA-C03 concept does it represent?

Use simple 3rd-grade analogies first, then map them back to AWS terminology.

## Target architecture

```text
Internet
   |
Internet Gateway
   |
+------------------------- VPC -------------------------+
|                                                       |
| AZ A                              AZ B                |
| Public Subnet A                   Public Subnet B     |
|   ALB node                          ALB node           |
|   NAT A                             NAT B              |
|      |                                |               |
| Private App Subnet A              Private App Subnet B|
|   EC2 / ASG                         EC2 / ASG          |
|          \                          /                 |
|           \------ Target Group ----/                 |
|                                                       |
|              RDS Multi-AZ                             |
+-------------------------------------------------------+
```

## Rebuild phases

### Phase 0 — Cost and safety guardrails

Before deployment:

- choose one Region intentionally;
- establish a naming/tagging convention;
- review expected ALB, NAT Gateway, RDS, EC2/EBS, public IPv4/EIP, CloudWatch, and snapshot costs;
- decide a strict cleanup time;
- do not reproduce Academy credentials or temporary identifiers;
- prefer current AWS Console behavior at rebuild time rather than blindly copying old screenshots.

### Phase 1 — Build the network foundation Academy had prepared

Create and understand:

- VPC and CIDR
- two Availability Zones
- two public subnets
- two private application subnets
- database subnet design / DB subnet group as needed
- Internet Gateway
- public route table(s)
- NAT Gateway per AZ
- private route table per AZ
- subnet associations
- Security Groups

Checkpoint: explain **public vs private subnet** without using the subnet's name as evidence.

### Phase 2 — Build the database tier

Create an RDS MySQL database using a safe lab-sized configuration, then deliberately configure Multi-AZ when the exercise reaches HA.

Understand:

- DB subnet placement
- DB Security Group
- application-to-database SG reference
- endpoint abstraction
- Multi-AZ primary/standby
- why Multi-AZ is not a Read Replica

Checkpoint: explain what happens to the endpoint during managed failover at the conceptual level.

### Phase 3 — Build one working application server

Before adding HA, prove one private EC2 application instance works.

Create:

- IAM role/instance profile with only required permissions
- application Security Group
- EC2 instance
- User Data/bootstrap
- application-to-RDS connectivity

Verify application and logs before creating an AMI.

Checkpoint: distinguish EC2 `2/2` checks from application health.

### Phase 4 — Create repeatable compute

Create an AMI or otherwise choose the current best repeatable-image/bootstrap approach for the exercise.

Then create a Launch Template containing the intentional machine configuration.

Checkpoint:

> AMI = what the house looks like.
>
> Launch Template = instructions for launching houses.

### Phase 5 — Add the load-balancing layer

Create:

- ALB Security Group
- internet-facing ALB across both public subnets
- Target Group
- health-check settings
- listener/default action

Verify the **actual SG attached to the ALB**, specifically to prevent repeating the classroom failure.

Checkpoint: explain Listener vs Target Group vs Health Check.

### Phase 6 — Add Auto Scaling

Create ASG across both private application subnets using the Launch Template.

Configure:

- desired/min/max capacity
- Target Group attachment
- ELB health checks
- health-check grace period
- monitoring appropriate for the exercise

Checkpoint: explain why ALB does not replace EC2 and why ASG does not distribute HTTP requests.

### Phase 7 — Failure testing

Perform controlled tests:

- terminate one application instance;
- observe Target Group health;
- verify application remains reachable;
- observe ASG replacement;
- verify replacement becomes healthy;
- inspect CloudWatch/ASG activity.

Do not claim HA until failure behavior is observed.

### Phase 8 — NAT/AZ dependency test

Trace outbound paths:

```text
Private App A -> route table A -> NAT A
Private App B -> route table B -> NAT B
```

Verify no private application subnet accidentally depends on the other AZ's NAT path.

### Phase 9 — SAA reconstruction test

Without notes, draw:

```text
User -> ALB -> Target Group -> EC2/ASG -> RDS
                     |
                 health checks

Private EC2 -> NAT -> IGW -> Internet
```

Then explain:

- ALB vs ASG
- Multi-AZ vs Read Replica
- SG vs route table
- IGW vs NAT
- AMI vs Launch Template
- EC2 status vs application health

## Troubleshooting drills to intentionally revisit

1. Attach the wrong SG in a controlled way, predict the symptom, then restore it.
2. Inspect Target Group reason codes before changing anything.
3. Trace traffic source SG -> destination SG.
4. Verify route-table associations, not just route-table contents.
5. Read User Data/cloud-init logs and decide whether a log anomaly is causal or incidental.

Do not perform destructive/misconfiguration drills against anything outside the isolated personal lab.

## Cleanup plan

Cleanup is part of the lab, not an afterthought. Inventory before deletion and verify after deletion.

Pay special attention to cost-bearing resources:

- NAT Gateways and EIPs/public IPv4 allocations
- ALB
- RDS
- EC2 instances and EBS volumes
- AMIs and backing snapshots
- CloudWatch resources/log retention if created

Then remove dependent networking resources in a safe order and confirm no intentionally temporary resource remains.

## Completion standard

The personal rebuild is complete only when I can:

- build the architecture without Academy pre-created infrastructure;
- explain every connection in plain language;
- predict failure behavior before testing;
- troubleshoot by layers instead of random changes;
- connect each component to SAA-C03;
- clean up the environment safely.
