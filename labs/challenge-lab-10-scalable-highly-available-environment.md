# Challenge Lab 10 — Creating a Scalable and Highly Available Environment

[Study index](../README.md) | [Guided Lab 10 notes](lab-10-creating-a-highly-available-environment.md)

## Purpose and completed lab record

AWS Academy Challenge Lab: **Creating a Scalable and Highly Available Environment**.

Scenario: the Café expects a major temporary traffic spike after appearing on a popular TV show. The architecture therefore needs both **high availability** and **automatic scaling** rather than a fixed number of web servers.

**Hands-on result: Tasks 2–6 = 25/25.** The six Task 1 questions were exposed separately after lab completion in the current Academy interface.

Temporary account IDs, ARNs, instance/VPC/subnet/security-group/route-table/NAT IDs, public IPs, DNS names, key material, and other classroom identifiers are intentionally omitted.

## 1. Architecture I verified and extended

```text
                         Internet
                            |
                    CafeALB (HTTP :80)
                 Public Subnet 1 + 2
                            |
                         CafeTG
                      health checks
                            |
                   Cafe-ASG (2 to 6)
                    /             \
                   /               \
           webserver EC2       webserver EC2
          Private Subnet 1    Private Subnet 2
                AZ A               AZ B
                   \               /
                    \-------------/

Private outbound paths:
Private Subnet 1 -> existing NAT path
Private Subnet 2 -> NatGateway2 in Public Subnet 2
```

The ALB is the public entry point. The Auto Scaling web servers stay in private subnets across two Availability Zones. The Target Group contains healthy application targets. The ASG changes fleet size according to load.

## 2. What AWS Academy prepared for me

**[AWS Academy prepared this]**

The challenge began with substantial infrastructure already present, including the Lab VPC, multiple public/private subnets, route tables, an existing NAT path, an existing Café web application server, the Café web-server AMI, classroom IAM resources, and application/security prerequisites.

This is important: completing the challenge did **not** mean I built the whole architecture from zero.

## 3. What I built/configured

**[I built/configured this]**

- Inspected the prepared VPC, subnets, route tables, NAT, Security Group, EC2 server, and Café web-server AMI.
- Created a second **zonal** NAT Gateway in Public Subnet 2.
- Added the Private Subnet 2 default route through the second NAT path.
- Created `CafeWebServerTemplate` from the prepared Café web-server AMI.
- Used `t2.micro`, `CafeSG`, the classroom `CafeRole`, a new lab key pair, and instance tag `Name=webserver`.
- Created `Cafe-ASG` across Private Subnet 1 and Private Subnet 2.
- Configured ASG capacity: desired 2, minimum 2, maximum 6.
- Configured Target Tracking using **Average CPU utilization = 25%** and **60-second instance warmup**.
- Created internet-facing `CafeALB` across both public subnets.
- Created a dedicated ALB Security Group allowing classroom HTTP traffic.
- Created `CafeTG` and let the ASG register targets automatically.
- Attached the Target Group to the ASG.
- Verified healthy targets in both Availability Zones.
- Verified the Café application through the ALB using the `/cafe` path.
- Used SSM Session Manager to connect to one private web server.
- Installed `stress` and generated CPU load for 600 seconds.
- Observed automatic scale-out **2 -> 4 -> 6**.
- After load ended, observed automatic scale-in **6 -> 5 -> 4 -> 3 -> 2**.
- Observed ELB connection draining while instances were removed.
- Verified the hands-on grader: Tasks 2–6 all received full credit.

## 4. High availability vs scalability

These are related but different.

```text
High Availability
= keep serving when a component/AZ has trouble
= spread capacity across multiple AZs

Scalability
= change capacity when demand changes
= ASG adds/removes EC2 instances
```

### 3rd-grade analogy / 三年級比喻

> **High Availability = 不要把所有員工都放在同一間店。**
>
> **Scalability = 客人變多就加員工，客人變少就減員工。**

The challenge demonstrated both at the same time: the fleet was distributed across two AZs and dynamically grew/shrank with CPU demand.

## 5. ALB, Target Group, and ASG — restaurant model

```text
Customers
   |
   v
CafeALB = greeting host / 總接待員
   |
   v
CafeTG = healthy worker list / 健康員工名單
   |
   v
EC2 webservers = workers / 真正工作的員工

Cafe-ASG = restaurant manager / 經理
           maintains enough workers
```

Important distinction:

- **ALB:** Where should this request go?
- **Target Group:** Which targets are available/healthy?
- **ASG:** How many instances should exist?
- **EC2:** Actually runs the application.

## 6. Auto Scaling experiment — the key learning

The target-tracking policy tried to maintain average CPU around **25%**.

The observed sequence was:

```text
stress --cpu 1 --timeout 600
        |
        v
CPU load rises
        |
        v
CloudWatch target-tracking alarm enters ALARM
        |
        v
ASG changes desired capacity
        |
        +--> 2 -> 4
        |
        +--> 4 -> 6
```

The ASG did **not** jump immediately to the maximum. It added capacity as the policy reacted to metrics.

When the stress workload ended:

```text
CPU load falls
      |
      v
Target Tracking detects excess capacity
      |
      v
6 -> 5 -> 4 -> 3 -> 2
      |
      v
normal baseline restored
```

### What if Max = 6 is not enough?

Maximum capacity is a hard ceiling for the ASG. If demand still requires more capacity after the group reaches 6, the ASG cannot launch instance 7 unless the configured maximum is increased.

But the architecture question is bigger than simply raising the number:

- Is the ASG maximum too low?
- Is there an EC2/service quota or capacity constraint?
- Can the database and downstream services handle more web servers?
- Is the bottleneck actually CPU, database, network, cache, or another dependency?
- What is the acceptable cost ceiling?

**SAA memory:** scaling the web tier does not automatically remove bottlenecks in downstream tiers.

## 7. Scale-in and connection draining

During scale-in, instances were not simply killed immediately. ASG activity showed **Waiting for ELB Connection Draining** before termination completed.

### 3rd-grade analogy

> The restaurant manager does not tell a worker to disappear while serving a customer. The host stops assigning new customers to that worker and allows current work to finish before the worker leaves.
>
> 經理不會讓正在服務客人的員工突然消失；先停止分配新客人，讓手上的工作完成，再下班。

This is an important availability behavior during capacity reduction.

## 8. SSM Session Manager lesson

The private web server was managed through **AWS Systems Manager Session Manager**. The SSM agent was online and the EC2 instance used the classroom IAM role.

This demonstrated that administration of a private instance does not require opening SSH port 22 to the public internet.

### 3rd-grade analogy

> IAM instance profile = EC2's employee badge / EC2 的員工證.

The instance uses its role to prove what AWS services/actions it is allowed to use.

## 9. NAT and private subnet lesson

Private Subnet 2 initially had only its local VPC route. The challenge added a second zonal NAT Gateway in Public Subnet 2 and a default route from the corresponding private route table.

```text
Private EC2
   |
private route table
   |
NAT Gateway
   |
IGW / Internet
```

**NAT provides outbound internet access for private resources; it does not make the private EC2 directly reachable from the internet.**

Also remember:

> Public IP = address. Route = road. IGW = outside gate.
>
> 有 Public IP 不等於一定可以從 Internet 直接進來；路由與安全規則仍然重要。

## 10. HTTP in the classroom vs HTTPS in production

The challenge intentionally used **HTTP :80** so the exercise could focus on ALB, Target Groups, Auto Scaling, and high availability.

For a real internet-facing production application, evaluate HTTPS/TLS:

```text
User
  |
HTTPS :443
  v
ALB + TLS certificate (commonly ACM)
  |
  v
Target Group
  |
  v
Private application targets
```

HTTP can commonly redirect to HTTPS. Whether traffic from ALB to the backend must also be encrypted depends on security and compliance requirements.

**SAA memory:**

- HTTPS/TLS = encryption **in transit**.
- EBS/RDS/S3 encryption = encryption **at rest**.

Do not blindly copy the classroom HTTP-only configuration into a production architecture.

## 11. Challenge Lab vs Guided Lab 10

| Guided Lab 10 | Challenge Lab 10 |
| --- | --- |
| Main emphasis: fixed HA capacity and replacement | Main emphasis: HA **plus dynamic scaling** |
| ASG maintained fixed application capacity | ASG target tracking changed capacity with CPU |
| Failure test: instance breaks -> replacement | Load test: demand rises -> more instances |
| Core memory: **店壞掉 -> 補回來** | Core memory: **客人變多 -> 開更多服務能力** |

Both labs use ALB + Target Group + Launch Template + ASG, but they demonstrate different operational behaviors.

## 12. Troubleshooting and grading observations

### Task 1 question visibility

In the current Academy interface, the six Task 1 questions were not visible in the place described by the older lab instructions while the hands-on environment was being used. The question interface became visible after lab completion.

**Lesson:** distinguish an Academy UI/workflow difference from an AWS architecture failure.

### Task 4 grader timing

During the first grading view, Task 4 temporarily displayed 0/5 even though the ASG had demonstrably scaled out and its configuration matched the exercise. After the workload ended, scale-in completed, and grading refreshed, Task 4 displayed **5/5**.

Do not treat a transient grader result as proof that a working AWS architecture is wrong. Verify the actual AWS state first.

## 13. SAA-C03 takeaways

- High availability and scalability solve different problems.
- Multi-AZ placement reduces dependence on a single Availability Zone.
- ALB distributes requests; ASG manages compute capacity.
- Target Groups track destinations and application health.
- Target Tracking adjusts desired capacity toward a metric target.
- Minimum capacity is the floor; maximum capacity is the ceiling; desired capacity can change dynamically.
- A maximum of 6 does **not** mean ASG should always run 6.
- Scale-out and scale-in are not necessarily instantaneous.
- Connection draining helps protect in-flight work during deregistration/termination.
- Private EC2 administration can use SSM instead of public SSH exposure.
- NAT enables private outbound internet access; it does not create direct inbound internet reachability.
- Scaling EC2 does not guarantee the database or other downstream dependencies can scale equally.
- For production internet-facing designs, evaluate HTTPS/TLS rather than copying a classroom HTTP-only listener.

## 14. Lab Completion Checkpoint / Lab 結束檢查點

- [x] Class hands-on Tasks 2–6 complete — **25/25**
- [x] Task 1 question interface located after lab completion
- [x] Second zonal NAT path configured
- [x] Launch Template verified
- [x] ASG across two private subnets / two AZs
- [x] Target Tracking: CPU 25%, warmup 60 seconds
- [x] ALB + Target Group verified
- [x] `/cafe` application verified through ALB
- [x] Scale-out observed: **2 -> 4 -> 6**
- [x] Scale-in observed: **6 -> 5 -> 4 -> 3 -> 2**
- [x] Connection draining observed
- [x] SAA takeaways captured
- [x] 3rd-grade analogies captured
- [x] Sensitive temporary AWS identifiers omitted

## 15. Personal AWS Rebuild Decision

**Yes — Very High Learning Value.**

The future rebuild should start from an empty personal lab environment rather than relying on Academy-created networking and application prerequisites. It should deliberately build:

1. VPC, two AZs, public/private subnets, IGW, routes, and NAT design.
2. Security Groups and IAM/SSM management path.
3. One working private application instance.
4. Repeatable image/bootstrap + Launch Template.
5. ALB + Target Group + health checks.
6. ASG with target tracking.
7. Controlled load test proving scale-out and scale-in.
8. HTTPS/ACM as a production-oriented extension.
9. Cost/quotas/downstream-bottleneck review.
10. Full cleanup verification.

The goal is not to memorize Café resource names. The goal is to be able to explain **why every component exists, what it connects to, and what fails when it is missing**.
