# Personal Rebuild Plan — Containerized Microservices on Amazon ECS

[Guided lab notes](../labs/guided-lab-containerized-microservices.md) | [Quick review](../cheat-sheets/guided-lab-containerized-microservices-cheat-sheet.md) | [Study index](../README.md)

## Recommendation

**🟢 Yes — Very High Learning Value**

Rebuild this architecture from scratch in a personal AWS account using the newest AWS Console available at rebuild time.

Do **not** rely on AWS Academy pre-created networking, security groups, IAM roles, source-code scaffolding, or auto-generated names.

## Goal

Build a small containerized application that progresses through:

```text
Local monolith
   ↓
Dockerized monolith
   ↓
Amazon ECR
   ↓
Amazon ECS
   ↓
Application Load Balancer
   ↓
Three independently deployed microservices
```

Suggested services:

```text
Users
Posts
Threads
```

## Why rebuild from scratch?

The Academy lab supplied several prerequisites. A personal rebuild should prove that I can create and connect every dependency myself.

The rebuild should answer:

- How does an ECS cluster get compute capacity?
- What IAM roles are required and why?
- How do task definitions reference ECR images?
- How do ECS services register tasks into target groups?
- How does an ALB listener evaluate path rules?
- How do security groups permit ALB-to-task traffic?
- How do I troubleshoot image, port, listener, health-check, and routing failures?

## Milestone 1 — Build the network

Create from zero:

- VPC
- two or three Availability Zones
- public subnets for the internet-facing ALB
- routing to an Internet Gateway
- security group for the ALB
- security group for ECS tasks/instances

Preferred security pattern:

```text
Internet
   ↓ TCP 80/443
ALB Security Group
   ↓ TCP 3000
ECS Task/Instance Security Group
```

Do not allow unnecessary direct Internet access to the application port.

## Milestone 2 — Create a tiny monolith

Build a small Node.js application locally with endpoints:

```text
/
/api
/api/users
/api/posts
/api/threads
```

Confirm it works locally before containerizing it.

## Milestone 3 — Dockerize the monolith

Write the Dockerfile manually.

Understand every line.

Build and run locally:

```bash
docker build -t app-monolith .
docker run -p 3000:3000 app-monolith
```

Verify:

```text
http://localhost:3000/api/users
```

## Milestone 4 — Create ECR repositories

Create repositories deliberately rather than accepting generated names.

Push the monolith image.

Practice:

```text
Authenticate -> Build -> Tag -> Push -> Verify
```

## Milestone 5 — Create ECS capacity

Choose one rebuild path first:

### Option A — ECS on EC2

Best for understanding the Academy lab and SAA architecture relationships.

Build:

- ECS cluster
- EC2 capacity
- instance profile / ECS instance role
- capacity limits
- networking

### Option B — Fargate follow-up

After the EC2 rebuild succeeds, repeat the deployment with Fargate and compare operational responsibility.

## Milestone 6 — Deploy the containerized monolith

Create:

- task definition
- ECS service
- target group
- ALB
- HTTP listener
- health check

Verify:

```text
ALB -> target group -> ECS task -> container:3000
```

## Milestone 7 — Refactor into microservices

Create three independent source folders:

```text
users/
posts/
threads/
```

Each gets:

- its own Dockerfile;
- its own ECR repository;
- its own task definition;
- its own ECS service;
- its own target group.

## Milestone 8 — Configure path-based routing

Use one ALB listener:

```text
/api/users*    -> Users target group
/api/posts*    -> Posts target group
/api/threads*  -> Threads target group
```

Set deliberate unique priorities.

Test each endpoint after the monolith is stopped.

## Milestone 9 — Failure drills

Intentionally create and repair these failures:

1. Wrong container port: 80 instead of 3000.
2. Missing ECR tag before push.
3. Wrong listener path: `/` instead of `/api/threads*`.
4. Wrong target group.
5. Security group blocks ALB-to-task traffic.
6. Health check path is wrong.
7. Task execution role cannot pull from ECR.
8. Service desired count is 0.
9. Create a new task-definition revision and update the service to it.

For every failure, collect evidence from:

- ECS service events;
- task status;
- target health;
- ALB listener rules;
- CloudWatch logs;
- Docker/ECR output.

## Milestone 10 — Compare monolith vs microservices

Document:

| Topic | Monolith | Microservices |
| --- | --- | --- |
| Deployments | one | independent |
| Scaling | whole app | per service |
| Failure blast radius | larger | smaller when well isolated |
| Operational complexity | lower | higher |
| Routing | simple | often path/host/service routing |
| Best for small MVP | often yes | only when boundaries justify it |

## Milestone 11 — LFM architecture checkpoint

After the rebuild, evaluate Love Fills Memory using the same questions:

- Which capabilities are currently modules in one deployment?
- Which capabilities are already independent workers/services?
- Which capabilities truly need independent scaling?
- Which boundaries reduce failure blast radius?
- Which splits would add complexity without current value?

Do **not** split LFM into microservices simply because microservices were learned in class.

## Completion criteria

The personal rebuild is complete only when I can:

- draw the architecture from memory;
- explain every IAM role;
- explain why every security-group rule exists;
- push an image to ECR without copying the Academy workflow blindly;
- create ECS task definitions and services from scratch;
- explain task vs task definition vs service;
- configure and troubleshoot ALB path routing;
- deploy a new task-definition revision;
- stop the monolith and prove all microservice routes still work;
- explain when a modular monolith is better than microservices.

## SAA-C03 checkpoint

This rebuild is valuable for:

- container orchestration;
- ECR vs ECS;
- ECS on EC2 vs Fargate;
- task definitions and services;
- ALB target groups;
- path-based routing;
- high availability and service replacement;
- IAM execution roles;
- operational tradeoffs between monoliths and microservices.

> **Build it once with help. Rebuild it from zero until every arrow makes sense.**  
> **先在 Lab 做一次，再從零重建，直到每一條箭頭都能自己解釋。**
