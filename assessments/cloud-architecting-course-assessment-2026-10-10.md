# AWS Academy Cloud Architecting — Course Assessment Review

**Date:** 2026-10-10  
**Result:** PASS — **220/250 (88%)**; passing threshold **175/250 (70%)**.  
**Questions:** 25; 10 points each; **22 correct, 3 incorrect**.

## Module score evidence

| Module | Score |
|---|---:|
| Securing Access | 10/20 |
| Adding a Storage Layer with Amazon S3 | 20/20 |
| Adding a Compute Layer Using Amazon EC2 | 20/20 |
| Adding a Database Layer | 10/20 |
| Creating a Networking Environment | 20/20 |
| Connecting Networks | 20/20 |
| Securing User, Application, and Data Access | 30/30 |
| Implementing Monitoring, Elasticity, and High Availability | 20/20 |
| Automating Your Architecture | 10/10 |
| Caching Content | 10/10 |
| Building Decoupled Architectures | 20/20 |
| Building Serverless Architectures and Microservices | 10/20 |
| Data Engineering Patterns | 10/10 |
| Planning for Disaster | 10/10 |

**Evidence boundary:** These scores prove one missed question in each of Securing Access, Adding a Database Layer, and Building Serverless Architectures and Microservices. The screenshots of Q1, Q8, and Q22 show *selected choices*, not explicit correct/incorrect indicators. They are likely candidates, **not confirmed wrong-answer IDs**.

## Three review candidates

### Q1 — IAM identities / IAM 身分

**Scenario:** Employees from different departments need common access to manage a support portal.  
**Selected:** IAM Role. **Likely academy-intended:** IAM Group.

- **IAM user:** an individual IAM identity.
- **IAM group:** collect IAM users and attach common permissions.
- **IAM role:** assumable identity with temporary credentials; useful for federation, cross-account access, and delegated access.
- **IAM Identity Center:** generally preferable for modern workforce access using corporate identities.

**Grade-school analogy / 三年級比喻:** A school activity committee (group) shares the same permission rules; a temporary hall monitor badge (role) can be assumed for a task.

**Exam cue:** *Many IAM users, same permissions* → IAM group. *Temporary / cross-account assumed permissions* → IAM role.

### Q8 — Oracle to Aurora migration / 資料庫遷移

**Selected:** AWS DMS Schema Conversion + AWS DMS. **Status:** The academy's expected pair is not shown in the supplied review screenshot.

- **AWS DMS:** migrates database data; supports full load and change data capture.
- **AWS DMS Schema Conversion:** converts schemas/code for supported heterogeneous migrations.
- **AWS Schema Conversion Tool (AWS SCT):** another schema conversion option; older course material may favor AWS SCT + DMS.

**Exam cue:** Oracle → Aurora is a *heterogeneous migration*: plan both **schema conversion** and **data movement**. Verify tool support and the specific exam wording; do not conclude DMS Schema Conversion is technically invalid from an unseen answer key.

**Analogy / 比喻:** Convert the furniture to fit the new home (schema); hire movers to transport it (DMS).

### Q22 — ECS production application / 容器正式環境

**Selected:** A — create an ECS service definition and run the service on EC2 nodes. **Status:** No answer-key feedback visible. Option C (task definition + ALB) is another plausible course-intended choice, but not verified.

- **Task definition:** blueprint for containers, image, CPU, memory, ports.
- **Task:** a running instantiation of a task definition.
- **ECS service:** maintains desired task count and supports deployment/scaling.
- **Application Load Balancer:** distributes HTTP(S) traffic to healthy targets.
- **PodSpec:** Kubernetes concept, not native ECS configuration.

**Production reference architecture:** **Task Definition → ECS Service → ALB**, with an ECS cluster and appropriate EC2 capacity or Fargate.

**Analogy / 比喻:** Recipe (task definition), cook (task), restaurant manager (service), host directing guests (ALB).

**Exam cue:** Keep tasks running → ECS Service. Define containers → Task Definition. Route web requests → ALB. Do not claim A or C is definitively marked correct without the assessment's answer feedback.

## SAA-C03 next steps / 下一步

1. Review **IAM group vs role vs Identity Center**, **heterogeneous DB migration**, and **ECS task/service/ALB**.
2. Retain the distinction between **course grading** and **current AWS service capabilities**.
3. Start the **Capstone Project** in a fresh lab session tomorrow; the course assessment is already passed.
4. End-of-lab checkpoint: **Is this worth rebuilding in a personal AWS account? / 是否值得在個人 AWS 帳號重建？** Evaluate cost and cleanup first.

**Source:** User-provided AWS Academy assessment result, module-score screenshot, and question-review screenshots (2026-10-10). No answer-key feedback was provided.
