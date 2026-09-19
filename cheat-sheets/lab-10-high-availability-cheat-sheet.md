# Lab 10 — High Availability Cheat Sheet

[Full notes](../labs/lab-10-creating-a-highly-available-environment.md) | [Study index](../README.md)

## Architecture memory

```text
Internet
   |
ALB across 2 public subnets / 2 AZs
   |
Target Group + health checks
   |
ASG: EC2 in 2 private subnets / 2 AZs
   |
RDS Multi-AZ

Private Subnet 1 -> NAT 1 in AZ 1
Private Subnet 2 -> NAT 2 in AZ 2
```

## Who does what?

| Service | Job | 3rd-grade memory |
| --- | --- | --- |
| ALB | distributes requests to healthy targets | 總接待員 |
| Target Group | destinations + health state | 員工名單 |
| Listener | accepts traffic on protocol/port | 接待員的耳朵 |
| AMI | reusable machine image | 房子藍圖 |
| Launch Template | launch specification | 施工規格書 |
| ASG | maintains/scales EC2 capacity | 管家維持房子數量 |
| SG | stateful traffic permission | 警衛 |
| RDS Multi-AZ | DB HA/failover | 備援老闆 |
| NAT Gateway | private outbound internet path | 跑腿管家 |
| Route Table | decides next hop | 路標 |

## SAA distinctions

**ALB vs ASG**

- ALB: where should this request go?
- ASG: how many instances should exist?

**EC2 status vs Target health**

- EC2 checks: instance/platform health.
- Target Group check: can the application target serve the configured health request?

**Multi-AZ vs Read Replica**

- Multi-AZ = availability/failover.
- Read Replica = read scaling.

**IGW vs NAT**

- IGW = VPC connection for internet-routed public resources.
- NAT = lets private resources initiate outbound internet access.

**Route vs Security Group**

- Route = where traffic goes.
- SG = whether traffic is allowed.

## Three-tier SG pattern

```text
Internet -> ALB SG -> App SG -> DB SG
                         HTTP      MySQL 3306
```

Prefer tier-to-tier SG references where appropriate instead of unnecessarily broad CIDRs.

## Failure sequence to remember

```text
EC2 fails
 -> ALB health check marks target unhealthy
 -> ALB stops routing to it
 -> ASG detects lost/unhealthy capacity
 -> ASG launches replacement from Launch Template
 -> replacement becomes healthy
 -> ALB resumes distribution
```

## Troubleshooting lesson from the lab

**Symptom:** Target Group timed out and ASG kept replacing instances.

**Root cause:** ALB had the `default` SG attached, while App SG trusted only the intended `Inventory-LB` SG.

**Fix:** attach the intended ALB SG to the ALB.

Memory:

> Correct inbound rule + wrong source SG attachment = traffic still fails.
>
> 目的端規則寫對，但來源資源掛錯 SG，一樣不通。

## Optional HA upgrades completed

- RDS: Multi-AZ = Yes; `db.t3.small`; 20 GiB; final status Available.
- NAT: second zonal NAT in second public subnet.
- Private Route Table 2: `0.0.0.0/0 -> NAT 2`.
- Private Subnet 2 associated with Private Route Table 2.

## Exam prompts

If a question says **survive an AZ failure**, inspect every tier for single-AZ dependencies.

If it says **database HA/failover**, think RDS Multi-AZ.

If it says **scale reads**, think Read Replica.

If it says **automatically replace failed EC2**, think ASG.

If it says **distribute requests across healthy instances**, think ALB + Target Group health checks.

If private instances need **outbound internet without being publicly reachable**, think NAT + correct private route.

## Class result

**75/75 — complete.**
