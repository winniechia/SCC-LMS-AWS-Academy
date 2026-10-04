# Optional Guided Lab — Breaking a Monolithic Node.js Application into Microservices

[Study index](../README.md) | [Quick review](../cheat-sheets/guided-lab-containerized-microservices-cheat-sheet.md) | [Personal rebuild plan](../personal-labs/containerized-microservices-companion-plan.md)

## Purpose and lab checkpoint

AWS Academy optional guided lab: **Breaking a Monolithic Node.js Application into Microservices**.

The lab started with a Node.js monolith, containerized it, deployed it to Amazon ECS on EC2, and then refactored the application into three independently deployed services: **Users, Posts, and Threads**.

**Class lab status: partially completed before the Academy session ended.**  
The architecture was functioning, but the final grader was not fully cleared before time expired.

Known checkpoint at stop time:

- Users microservice: working and healthy.
- Posts microservice: working and healthy.
- Threads microservice: working and healthy after correcting the ALB path rule.
- Original monolith service: stopped with desired count 0.
- Final grader still showed a failure for **Task 4D — ECS Service configured** because the original monolith service had an AWS-generated name instead of the lab-requested `mb-ecs-service`.
- **Task 6G — /api/threads endpoint** initially scored 0/5 because its listener rule path was accidentally `/`; this was corrected to `/api/threads*` and the endpoint worked afterward, but the lab was not resubmitted before time expired.

Temporary account IDs, ARNs, DNS names, and classroom identifiers are intentionally omitted.

## 1. Architecture progression

### Stage 1 — Node.js monolith without containers

```text
Node.js application
├── Users
├── Threads
└── Posts
```

The application ran locally on port 3000 and exposed endpoints such as:

```text
/api/users
/api/users/4
/api/threads
/api/posts/in-thread/1
```

### Stage 2 — Containerized monolith

```text
Dockerfile
   ↓
Docker image
   ↓
Amazon ECR: mb-repo
   ↓
Amazon ECS service on EC2
   ↓
Application Load Balancer
```

### Stage 3 — Containerized microservices

```text
                         Application Load Balancer
                                  |
              ---------------------------------------------
              |                    |                      |
        /api/users*           /api/posts*          /api/threads*
              |                    |                      |
      mb-users-target        mb-posts-target       mb-threads-target
              |                    |                      |
      mb-users-service       mb-posts-service      mb-threads-service
              |                    |                      |
    mb-users-container     mb-posts-container    mb-threads-container
         :3000                  :3000                  :3000
```

## 2. What AWS Academy prepared for me

**[AWS Academy prepared this]**

The lab environment supplied:

- Cloud9/IDE environment.
- IDE VPC and public subnets.
- Security groups, including an ECS-specific security group containing `ECSSG` in the name.
- Lab application source code for:
  - `1-no-container`
  - `2-containerized-monolith`
  - `3-containerized-microservices`
- Node.js application data and starter Dockerfiles.
- Temporary AWS account resources and permissions.

This means the guided lab was **not a complete from-scratch build**.

## 3. What I built/configured

**[I built/configured this]**

### Monolith validation

- Downloaded and extracted the lab files.
- Ran the original Node.js monolith from `1-no-container`.
- Installed Koa dependencies.
- Started the app with `npm start`.
- Verified the Users, Threads, and Posts API endpoints with `curl`.

### Containerized monolith

- Reviewed the Dockerfile in `2-containerized-monolith`.
- Created private ECR repository `mb-repo`.
- Authenticated Docker to ECR.
- Built the Docker image.
- Tagged the image with the ECR URI.
- Pushed the image to ECR.
- Created ECS cluster `mb-ecs-cluster` with self-managed EC2 capacity.
- Used two `t2.medium` EC2 container instances.
- Used the IDE VPC, three public subnets, and `ECSSG`.
- Created task definition `mb-task`.
- Used container name `mb-container`, port 3000.
- Created an ECS service and an Application Load Balancer.
- Created `mb-load-balancer` and target group `mb-target`.
- Verified the monolith through the ALB DNS name.

### Microservices

Created three private ECR repositories:

```text
mb-users-repo
mb-posts-repo
mb-threads-repo
```

Built, tagged, and pushed three separate images.

Created task definitions:

```text
mb-users-task
mb-posts-task
mb-threads-task
```

Each microservice used:

- Amazon EC2 launch type.
- 0.5 vCPU.
- 1 GB memory.
- Container port 3000.

Created ECS services:

```text
mb-users-service
mb-posts-service
mb-threads-service
```

Created target groups and path-based ALB routing:

```text
Priority 1: /api/users*   -> mb-users-target
Priority 2: /api/posts*   -> mb-posts-target
Priority 3: /api/threads* -> mb-threads-target
Default:                  -> mb-users-target
```

Verified all three target groups reached **Healthy = 1** and the API endpoints returned expected data.

## 4. Dockerfile mental model

The monolith Dockerfile:

```dockerfile
FROM mhart/alpine-node:7.10.1
WORKDIR /srv
ADD . .
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]
```

Mental model:

- `FROM` = choose the base workbench.
- `WORKDIR` = choose the workspace inside the container.
- `ADD` = put the application files in the container.
- `RUN npm install` = install dependencies into the image.
- `EXPOSE 3000` = document the application port.
- `CMD` = define the command that runs when the container starts.

> **Dockerfile = recipe. Docker image = packaged result. Container = running instance.**  
> **Dockerfile = 食譜；Image = 做好的包裹；Container = 真正跑起來的實例。**

## 5. ECR mental model

```text
docker build
   ↓
local Docker image
   ↓ docker tag
ECR-qualified image name
   ↓ docker push
Amazon ECR
```

- **Build** = make the box.
- **Tag** = attach the destination label.
- **Push** = send the box to the ECR warehouse.

A useful troubleshooting lesson occurred with `mb-threads-repo`: the image build succeeded, but the ECR-qualified tag was missing, so `docker push` failed with an image-not-found message. Re-running the `docker tag` command fixed the issue.

## 6. ECS concepts

### ECS cluster

The cluster is the logical environment that contains compute capacity and services.

> **Cluster = factory. / Cluster = 工廠。**

### EC2 container instances

The lab used self-managed EC2 capacity. Two `t2.medium` instances registered to the cluster.

> **EC2 container instances = workstations inside the factory. / EC2 = 工廠裡的工作站。**

### Task definition

A task definition describes how a container should run:

- which image;
- CPU and memory;
- ports;
- roles;
- runtime settings.

> **Task definition = job description. / Task Definition = 工作說明書。**

### Task

A task is a running instantiation of the task definition.

> **Task = worker currently doing the job. / Task = 真正在工作的員工。**

### Service

An ECS service maintains the desired number of tasks and replaces failed tasks.

> **Service = manager who keeps enough workers on duty. / Service = 經理，確保需要的 worker 一直存在。**

## 7. Application Load Balancer path-based routing

The same ALB can route requests to different services according to the URL path.

```text
/api/users*   -> Users service
/api/posts*   -> Posts service
/api/threads* -> Threads service
```

This is the key microservices routing pattern demonstrated by the lab.

> **ALB = receptionist. Target group = the list of workers who can handle that request.**  
> **ALB = 接待員；Target Group = 可以接這類工作的員工名單。**

## 8. Troubleshooting lesson — Threads endpoint returned Not Found

The final grader initially showed:

```text
Task 6G — /api/threads endpoint: 0/5
```

The browser also returned **Not Found** for `/api/threads`.

Inspection of the ALB listener rules revealed:

```text
Priority 3
Path = /
Forward -> mb-threads-target
```

The correct rule was:

```text
Priority 3
Path = /api/threads*
Forward -> mb-threads-target
```

Because `/api/threads` did not match `/`, the request fell through to the default rule, which forwarded it to `mb-users-target`. The Users service did not recognize the Threads route, so it returned **Not Found**.

After changing the rule to `/api/threads*`, the endpoint worked.

### Lesson

> **Healthy target does not prove the listener rule is correct.**  
> **Target Healthy 只證明服務活著，不代表 ALB 的 path rule 一定正確。**

## 9. Troubleshooting lesson — wrong container port revision

The first revision of `mb-users-task` accidentally used:

```text
80:80
```

The Node.js service actually listens on port 3000.

A new revision was created:

```text
mb-users-task:2
Host port : Container port
3000 : 3000
```

The old revision remained Active, which is normal. Active means it is available for use; it does **not** mean both revisions are running.

### Lesson

> **Task definition revisions are versioned configuration, not duplicate running tasks.**

## 10. Current-console differences from the lab guide

Several AWS Console labels differed from the supplied guide.

Examples encountered:

- Older guide: **Amazon EC2 instances**  
  Current cluster UI: **Fargate and Self-managed instances**.
- Older guide: **Create new role**  
  Current UI: **Create default role** / instance profile options.
- Task and service pages used newer layouts for task definitions, revisions, listener rules, and target groups.
- The console auto-generated a monolith service name when the requested lab name was not explicitly preserved.

### Lesson

> **Follow the architecture requirement, not the screenshot. Verify every generated name, port, path rule, and role before creating the resource.**

## 11. Unresolved grader item at stop time

The grader showed:

```text
Task 4D — ECS Service configured: 0/5
```

The original monolith service was created with the AWS-generated name:

```text
mb-task-service-p91dc1i9
```

The lab expected:

```text
mb-ecs-service
```

The service functioned during the lab, but the grader likely expected the requested resource name/configuration. ECS services cannot be renamed directly.

Because the Academy session was ending, this was intentionally left unresolved rather than risking the working microservices architecture.

## 12. Final classroom checkpoint

At stop time:

```text
mb-users-service    Active, 1/1, healthy
mb-posts-service    Active, 1/1, healthy
mb-threads-service  Active, 1/1, healthy

Original monolith service:
mb-task-service-p91dc1i9
Desired/running: 0/0

ALB rules:
1 /api/users*    -> mb-users-target
2 /api/posts*    -> mb-posts-target
3 /api/threads*  -> mb-threads-target
default          -> mb-users-target
```

The three microservice endpoints worked after the Threads rule correction.

The lab was **not resubmitted after the final correction**, and Task 4D remained unresolved when the Academy session ended.

## 13. 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

### Monolith

One restaurant worker does everything:

- takes orders;
- cooks;
- handles payment;
- checks inventory.

If one job gets busy, the whole worker becomes overloaded.

### Microservices

Split the jobs:

- Users worker;
- Posts worker;
- Threads worker.

Each has its own container and ECS service.

The ALB receptionist reads the request:

```text
"Users request?"   -> Users worker
"Posts request?"   -> Posts worker
"Threads request?" -> Threads worker
```

Each worker can be deployed and scaled independently.

## 14. SAA-C03 takeaways

| Exam clue | Think |
| --- | --- |
| Package app + dependencies portably | Container |
| Store private Docker images in AWS | Amazon ECR |
| Orchestrate containers | Amazon ECS |
| Run ECS on customer-managed compute | ECS on EC2 |
| Avoid managing EC2 hosts | AWS Fargate |
| Keep desired number of tasks running | ECS Service |
| Describe image/CPU/memory/port | ECS Task Definition |
| Route different URL paths to different backends | ALB path-based routing |
| Split monolith into independently deployable functions | Microservices |
| Shared traffic entry point, separate services | ALB + target groups |

Fast recognition:

> **ECR stores images. ECS runs containers. Task Definition describes the worker. Service keeps workers running. ALB routes customers to the correct workers.**

> **ECR 存 Image；ECS 跑 Container；Task Definition 寫工作說明；Service 維持 worker；ALB 把客人送到正確的 worker。**

## 15. Completion checkpoint

### Was this lab worth doing?

**Yes — very high learning value.**

It connected Docker, ECR, ECS, EC2 launch type, task definitions, services, target groups, and ALB path routing in one architecture.

### Should I rebuild it in my personal AWS account?

**Yes — but from scratch.**

The personal rebuild should not copy the Academy pre-created networking and IAM scaffolding. Rebuilding the architecture from zero will provide much more SAA-C03 and real-world value.

See the companion plan:

[Personal containerized microservices rebuild plan](../personal-labs/containerized-microservices-companion-plan.md)
