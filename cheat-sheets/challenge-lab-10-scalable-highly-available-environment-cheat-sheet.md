# Challenge Lab 10 — Scalability + High Availability Cheat Sheet

[Full notes](../labs/challenge-lab-10-scalable-highly-available-environment.md) | [Study index](../README.md)

## Architecture memory

```text
Internet
   |
CafeALB across 2 public subnets / 2 AZs
   |
CafeTG + health checks
   |
Cafe-ASG across 2 private subnets / 2 AZs
Min 2 / Max 6 / CPU target 25%
   |
webserver EC2 instances

Private Subnet 2 -> zonal NAT 2 -> outbound Internet
```

## Restaurant analogy

| AWS | Job | 三年級記法 |
| --- | --- | --- |
| ALB | distributes requests | 總接待員 |
| Target Group | healthy destinations | 健康員工名單 |
| EC2 | runs the app | 真正工作的員工 |
| ASG | changes/maintains fleet size | 餐廳經理 |
| Launch Template | repeatable launch config | 施工規格書 |
| IAM instance profile | gives EC2 AWS permissions | 員工證 |
| NAT Gateway | private outbound internet | 跑腿管家 |
| Route Table | next-hop decision | 路標 |

## HA vs Scaling

**High Availability:** survive failures / avoid one-AZ dependency.

> 不要把所有員工都放在同一間店。

**Scalability:** change capacity as demand changes.

> 客人變多就加員工；客人變少就減員工。

## Observed scaling sequence

```text
stress --cpu 1 --timeout 600
 -> CPU rises
 -> Target Tracking alarm
 -> Desired 2 -> 4 -> 6

stress ends
 -> CPU falls
 -> Scale in
 -> 6 -> 5 -> 4 -> 3 -> 2
```

Policy used:

- Average CPU target: **25%**
- Instance warmup: **60 sec**
- Min: **2**
- Max: **6**

**Max = ceiling, not target.** If 6 is insufficient, ASG cannot launch #7 until the maximum is raised; also check quotas, cost, and downstream bottlenecks.

## Scale-in lesson

During termination the lab showed **ELB connection draining**.

> Stop assigning new customers to a worker, let current work finish, then let the worker leave.
>
> 不再分新客人，先讓手上的工作完成，再下班。

## ALB vs TG vs ASG

- ALB = Where should this request go?
- Target Group = Which application targets are healthy/available?
- ASG = How many EC2 instances should exist?
- EC2 = Runs the actual application.

## NAT reminder

```text
Private EC2 -> private route -> NAT -> Internet
```

NAT supports **outbound** internet access. It does not make a private EC2 directly internet-reachable.

## SSM reminder

Session Manager allowed management of a private EC2 without exposing public SSH port 22.

IAM instance profile = EC2 employee badge / EC2 員工證.

## HTTP lab vs real production

Classroom lab: HTTP :80.

Production consideration:

```text
User -> HTTPS :443 -> ALB + TLS certificate (commonly ACM) -> targets
```

- HTTPS/TLS = encryption in transit.
- EBS/RDS/S3 encryption = encryption at rest.

## SAA triggers

- **traffic fluctuates / sudden demand** -> Auto Scaling.
- **distribute requests to healthy targets** -> ALB + Target Group.
- **survive AZ failure** -> Multi-AZ design; inspect every tier.
- **private instance outbound internet** -> NAT + route.
- **private administration without public SSH** -> Systems Manager Session Manager.
- **ASG reached maximum capacity** -> it cannot scale beyond max; investigate max, quotas, capacity, cost, and downstream bottlenecks.

## Challenge vs Guided Lab 10

- Guided: **店壞掉 -> 補回來** (HA replacement).
- Challenge: **客人變多 -> 開更多服務能力** (dynamic scaling + HA).

## Completed verification

- Tasks 2–6 hands-on grader: **25/25**
- ALB application path `/cafe`: working
- Scale-out: **2 -> 4 -> 6**
- Scale-in: **6 -> 5 -> 4 -> 3 -> 2**
- Targets/instances: Healthy / InService
- Sensitive temporary identifiers intentionally omitted
