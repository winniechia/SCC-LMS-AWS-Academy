# Containerized Microservices — Quick Review

[Full notes](../labs/guided-lab-containerized-microservices.md) | [Personal rebuild plan](../personal-labs/containerized-microservices-companion-plan.md) | [Study index](../README.md)

## Core progression

```text
Node.js monolith
   ↓
Dockerized monolith
   ↓
ECR
   ↓
ECS on EC2
   ↓
Users / Posts / Threads microservices
   ↓
ALB path-based routing
```

## Fast mental model

- **Dockerfile** = recipe / 食譜
- **Docker image** = packaged box / 做好的包裹
- **Container** = running box / 跑起來的實例
- **ECR** = image warehouse / Image 倉庫
- **ECS cluster** = factory / 工廠
- **EC2 container instance** = workstation / 工作站
- **Task Definition** = job description / 工作說明書
- **Task** = running worker / 正在工作的 worker
- **ECS Service** = manager maintaining worker count / 經理維持 worker 數量
- **ALB** = receptionist / 接待員
- **Target Group** = qualified worker list / 可接工作的 worker 名單

## Final microservices routing

```text
Priority 1  /api/users*    -> mb-users-target
Priority 2  /api/posts*    -> mb-posts-target
Priority 3  /api/threads*  -> mb-threads-target
Default                    -> mb-users-target
```

Each microservice container listened on **port 3000**.

## Build / tag / push

```text
docker build -t NAME .
docker tag NAME:latest <ECR-URI>/NAME:latest
docker push <ECR-URI>/NAME:latest
```

- Build = make image.
- Tag = attach destination name.
- Push = upload to ECR.

## Important troubleshooting

### ECR push says local tagged image does not exist

Build succeeded, but ECR-qualified tag is missing.

Fix: re-run `docker tag`, verify with `docker images`, then push.

### Endpoint returns Not Found even though target is Healthy

Check ALB listener rule path.

Actual lab issue:

```text
Wrong:   /
Correct: /api/threads*
```

A healthy target proves the service is alive; it does not prove path routing is correct.

### Task definition port was wrong

First Users revision used `80:80`. New revision corrected it to `3000:3000`.

Multiple Active task-definition revisions are normal.

## Classroom stop checkpoint

Working:

- `mb-users-service` — healthy
- `mb-posts-service` — healthy
- `mb-threads-service` — healthy
- Users, Posts, Threads endpoints — working after Threads listener rule correction

Not fully cleared before time expired:

- Grader Task 4D — original monolith ECS Service configuration.
- Original service used AWS-generated name instead of requested `mb-ecs-service`.
- Lab was not resubmitted after final Threads fix.

## SAA-C03 triggers

| Clue | Answer |
| --- | --- |
| Store Docker images | ECR |
| Manage/orchestrate containers | ECS |
| Serverless containers | Fargate |
| ECS on owned/managed EC2 capacity | EC2 launch type |
| Maintain desired running tasks | ECS Service |
| Define image, CPU, memory, ports | Task Definition |
| Route `/users`, `/posts`, `/threads` differently | ALB path-based routing |
| Independent deploy/scale by business function | Microservices |

> **Containerization is not automatically microservices. Microservices require independent responsibilities and deployment boundaries.**

> **用了 Container 不等於 Microservices；Microservices 的重點是功能責任與部署邊界能獨立。**
