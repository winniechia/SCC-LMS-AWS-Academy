# Lab 10 — Creating a Highly Available Environment

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-10-high-availability-cheat-sheet.md) | [Personal rebuild plan](../personal-labs/lab-10-high-availability-companion-plan.md)

## Purpose and completed lab record

AWS Academy Guided Lab: **Creating a Highly Available Environment**. The lab began with a partially prepared environment and extended it into a highly available three-tier design using an Application Load Balancer (ALB), Target Group, AMI, Launch Template, Auto Scaling Group (ASG), tier-to-tier Security Group references, RDS Multi-AZ, and a NAT Gateway per Availability Zone.

**Class result: 75/75.**

Temporary account IDs, ARNs, endpoints, instance/VPC/subnet/security-group/route-table/NAT IDs, public IPs, DNS names, and credentials are intentionally omitted.

## 1. Final architecture

```text
                         Internet
                            |
                 Application Load Balancer
                  Public Subnet 1 + 2
                     /             \
                    /               \
             App EC2                 App EC2
          Private Subnet 1       Private Subnet 2
             AZ A                    AZ B
                \                    /
                 \---- Auto Scaling /
                       Group
                         |
                     RDS MySQL
                   Multi-AZ standby

Private egress:
Private Subnet 1 -> NAT Gateway 1 in Public Subnet 1
Private Subnet 2 -> NAT Gateway 2 in Public Subnet 2
```

The ALB is internet-facing. Application instances remain in private subnets. The ALB distributes requests only to healthy registered targets. The ASG maintains application capacity. RDS Multi-AZ protects database availability, while the second NAT path removes the single-AZ dependency for private-subnet internet egress.

## 2. What AWS Academy prepared for me

**[AWS Academy prepared this]**

The lab environment pre-created major prerequisites, including:

- Lab VPC
- Public and private subnets across two Availability Zones
- Internet Gateway and existing public routing
- One NAT Gateway and existing private routing
- RDS database
- Existing Web Server 1
- Application and database Security Groups
- Classroom IAM resources such as the application instance role and lab key pair
- Lab application assets and User Data instructions

This matters because completing the guided steps is **not the same as building the architecture from zero**.

## 3. What I built/configured

**[I built/configured this]**

- Created an AMI from Web Server 1.
- Created the `Inventory-LB` Application Load Balancer across both public subnets.
- Created the `Inventory-App` Target Group and configured health checks.
- Created `Inventory-LT` Launch Template from the AMI.
- Configured the Launch Template with the application Security Group, IAM instance profile, detailed monitoring, and supplied User Data.
- Created `Inventory-ASG` across both private subnets.
- Attached the Target Group and enabled ELB health checks.
- Set desired/minimum/maximum capacity to 2/2/2 for the class exercise.
- Enabled ASG group metrics.
- Restricted application HTTP traffic to the ALB Security Group.
- Restricted MySQL/Aurora traffic to the application Security Group instead of the whole VPC CIDR.
- Tested application access through the ALB.
- Terminated an application instance and verified that the application remained available while ASG replaced capacity.
- Optional HA: converted RDS to Multi-AZ, selected `db.t3.small`, and confirmed 20 GiB storage.
- Optional HA: created a second zonal NAT Gateway, a second private route table, default route to the second NAT, and associated Private Subnet 2.
- Verified the RDS returned to `Available`.
- Submitted the lab and received **75/75**.

## 4. ALB, Target Group, and health checks

The ALB is the public entry point. It spans public subnets in two AZs and forwards HTTP traffic to the Target Group.

The Target Group is the ALB's list of application destinations. Health checks determine which targets are safe to receive requests.

> **ALB = front-desk receptionist. Target Group = employee list. Health check = “Are you ready to serve customers?”**
>
> **ALB = 總接待員；Target Group = 員工名單；Health Check =「你現在可以接客人嗎？」**

A running EC2 instance is not automatically a healthy application target. EC2 infrastructure status, web-server status, network permissions, and Target Group health are different layers.

## 5. AMI, Launch Template, and Auto Scaling

The AMI captured the reusable server image. The Launch Template defined how replacement application servers should be launched. The ASG used that template to maintain the requested number of instances across private subnets in two AZs.

```text
AMI              = house blueprint / 房子藍圖
Launch Template  = construction specification / 施工規格書
ASG              = manager maintaining the required houses / 維持房子數量的管家
```

The failure test demonstrated the division of responsibility:

- ALB detected an unhealthy/missing target and stopped sending traffic to it.
- ASG detected capacity loss and launched a replacement.
- The application remained reachable through the healthy target.

**SAA memory:** ALB distributes and health-checks traffic; ASG maintains compute capacity.

## 6. Three-tier Security Group design

The intended traffic chain was:

```text
Internet
   |
   v
ALB SG
   | HTTP
   v
Application SG
   | MySQL/Aurora 3306
   v
Database SG
```

Instead of allowing the database from the entire VPC CIDR, the final rule allowed database traffic from the application Security Group. This expresses the application-tier dependency directly and narrows access.

The classroom ALB accepted HTTP/HTTPS from the internet, while the application tier accepted HTTP from the ALB tier. HTTPS termination/offload at the load balancer can allow ALB-to-target HTTP in this lab architecture.

## 7. Actual troubleshooting record

### Incident A — stale Target Group after lab restart

An older `Inventory-App` Target Group remained associated with a previous lab VPC. The name collision and VPC mismatch prevented the intended current Target Group from appearing correctly for the new ALB.

Correction: identify the stale lab resource, delete only the stale Target Group, and recreate `Inventory-App` in the current Lab VPC.

**Lesson:** resource names can look correct while the underlying VPC dependency is wrong.

### Incident B — database Security Group rule could not be converted in place

Attempting to edit an existing IPv4 CIDR MySQL rule directly into a Security Group reference produced an AWS validation error.

Correction: remove the old CIDR rule and create a new MySQL/Aurora rule whose source is the application Security Group.

**Lesson:** changing the source type can require replacing the rule rather than editing it in place.

### Incident C — Target Group health checks timed out and ASG recycled instances

Symptoms:

- targets reported unhealthy / request timed out;
- ASG activity showed ELB health-check failures and replacement cycling;
- application instances themselves passed EC2 status checks;
- system logs showed Apache starting successfully.

The root cause was network identity, not the application: the **ALB was attached to the default Security Group**, while the application Security Group allowed HTTP only from the intended `Inventory-LB` Security Group.

Correction: attach the intended `Inventory-LB` Security Group to the ALB. The Target Group then stabilized with two healthy targets across two AZs.

> **The receiving SG trusted Guard A, but the visitor arrived wearing Guard B's badge.**
>
> **App SG 只信任 Inventory-LB SG；ALB 卻帶著 default SG 身分，所以健康檢查被擋住。**

This is a strong SAA troubleshooting lesson: verify the **actual Security Group attached to the source resource**, not only the inbound rule on the destination.

### Incident D — bootstrap logs contained duplicate downloaded files

The AMI already contained some application files. During later User Data execution, downloads could receive suffixes and unzip encountered existing files. Apache still started, so this was evidence to inspect but not the root cause of the Target Group timeout.

**Lesson:** logs may contain real anomalies that are not the causal failure. Troubleshoot layer by layer and correlate symptoms.

## 8. RDS Multi-AZ optional HA

The database was changed to:

- Multi-AZ: Yes
- Instance class: `db.t3.small`
- Storage: 20 GiB
- Final status verified: `Available`

Multi-AZ is for **availability/failover**, not read scaling. The standby is not the same concept as a Read Replica.

> **Multi-AZ = “I need the database to survive a failure.”**
>
> **Read Replica = “I need more read capacity.”**

Changing the DB instance class increases compute capacity; changing storage changes storage capacity. Neither change by itself creates HA. Multi-AZ is the HA feature.

## 9. Highly available NAT optional task

The original environment had one NAT Gateway, creating an AZ dependency for private-subnet internet egress. The optional task added a second **zonal** NAT Gateway in the second public subnet and a dedicated route table:

```text
AZ A                           AZ B
Public Subnet 1                Public Subnet 2
     |                              |
 NAT Gateway 1                  NAT Gateway 2
     ^                              ^
     |                              |
Private Subnet 1               Private Subnet 2
```

Final verification for Private Route Table 2:

```text
VPC CIDR      -> local
0.0.0.0/0     -> NAT Gateway 2
association   -> Private Subnet 2
```

The current console exposed a newer Regional NAT option, but the Academy exercise specifically required the zonal/per-AZ architecture above.

## 10. 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

Imagine a busy restaurant inside a two-building estate:

| AWS concept | Simple analogy |
| --- | --- |
| ALB | 總接待員：決定客人交給哪位服務員 |
| Target Group | 可以工作的服務員名單 |
| Health Check | 接待員問「你現在可以工作嗎？」 |
| EC2 | 一間正在營業的店/房子 |
| AMI | 房子的藍圖 |
| Launch Template | 蓋新房子的施工規格 |
| ASG | 管家：永遠維持指定數量的房子 |
| Security Group | 每一層門口的警衛 |
| RDS Multi-AZ | 老闆有另一個地點的備援資料庫 |
| NAT Gateway | 私人區域出去辦事的跑腿管家 |
| Route Table | 路標 |

Failure story:

> One shop breaks. The receptionist stops sending customers there. The manager notices one shop is missing and builds a replacement from the standard blueprint/specification. Customers continue using the surviving shop.
>
> 一間店壞掉時，總接待員先停止送客人過去；管家發現少了一間，就按照藍圖與施工規格補一間。客人仍可以由另一間正常的店繼續服務。

## 11. SAA-C03 takeaways

- High availability usually requires removing single-AZ dependencies at **each relevant tier**, not merely launching two EC2 instances.
- ALB can span multiple AZs and route only to healthy targets.
- Target Groups and ALB listeners are separate concepts: listener receives traffic; Target Group identifies destinations.
- ASG desired/min/max capacity controls fleet size; health mechanisms help replace failed capacity.
- Launch Templates standardize repeatable instance configuration.
- EC2 status checks and ALB Target Group health checks measure different things.
- Security Group references are powerful for tier-to-tier access.
- RDS Multi-AZ is HA/failover; Read Replicas are primarily read scaling.
- A NAT Gateway is not the same as an Internet Gateway: private resources use NAT for outbound internet access; internet-facing resources use routing through an IGW.
- A per-AZ NAT design avoids making one AZ's private egress depend on a NAT in another AZ.
- Routing answers **where traffic goes**; Security Groups answer **whether traffic is allowed**; load balancing answers **which healthy application target receives the request**.

## 12. Lab Completion Checkpoint / Lab 結束檢查點

- [x] Class Lab Complete — **75/75**
- [x] Architecture verified
- [x] SAA Takeaways captured
- [x] 3rd-Grade analogy captured
- [x] AWS Academy pre-created resources identified
- [x] Main troubleshooting documented
- [x] Optional RDS Multi-AZ verified
- [x] Optional NAT HA verified
- [x] Sensitive temporary AWS identifiers omitted

### Personal AWS Rebuild Decision

**Yes — Very High Learning Value.**

The Academy prepared much of the foundation, so a personal rebuild should deliberately create the architecture from zero rather than merely repeat the guided clicks. The rebuild should pause at every dependency and answer:

1. What does this resource do?
2. Why does it exist here?
3. What does it connect to?
4. What breaks if it is removed or misconfigured?
5. Which SAA-C03 concept does it demonstrate?

See [Lab 10 personal rebuild plan](../personal-labs/lab-10-high-availability-companion-plan.md).

### Cleanup Check

The Academy environment is temporary. For the future personal rebuild, cleanup must explicitly inventory NAT Gateways/EIPs, ALB, Target Groups, ASG/instances, Launch Templates/AMIs/snapshots, RDS, subnets/routes/IGW, Security Groups, IAM resources created for the exercise, logs/metrics, and the VPC. Cost-bearing resources must not be assumed to disappear automatically.
