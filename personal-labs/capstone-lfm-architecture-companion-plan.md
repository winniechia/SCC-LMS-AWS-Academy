# Capstone Companion Lab — Love Fills Memory Scenario (Personal AWS Rebuild)
[Completed Academy Capstone notes](../labs/aws-academy-capstone-project.md) | [Study index](../README.md)

**Status: PLAN ONLY — not deployed.** **Decision: YES — Very High Learning Value.**
This is an **isolated, fictional LFM-inspired training application**, not a change to the real Love Fills Memory production website, ECS services, data, or GitHub deployment pipeline. Never copy personal memories, real family stories, credentials, or production secrets into this sandbox.

## Learning contract / 學習規則
At **every step**, explain (1) what, (2) why, (3) connects to what, (4) data/traffic path, (5) what breaks, (6) how to verify, and (7) SAA-C03 relevance. Use a third-grade analogy, then actual AWS terminology. Work in **small checkpoints, one console action at a time**; save evidence and record costs.

### Scenario / 故事情境
A reader clicks a fictional “Love Fills Memory” story link. The sandbox serves a **made-up public story** and a placeholder image. If a post becomes popular, app instances scale; if an instance disappears, the service recovers. Reader privacy and story access rules are part of the design.

**Conceptual architecture (EC2 version):**
```text
Browser -> Internet-facing ALB (public subnets, two AZs)
                      |
                 Target Group
                      |
        +-------------+-------------+
        |                           |
   Private App EC2 A           Private App EC2 B
        |        \                /  |
        |       Auto Scaling Group   |
        +-------------+-------------+
                      |
             RDS MySQL (private DB subnets)
                      |
            Sample story metadata
  S3: fictional public media / private test artifacts
  IAM + Secrets Manager: permissions and database secret
  CloudWatch: metrics/logs/alarms
```
**LFM reality check:** This EC2/RDS lab illustrates the *responsibilities* of components; it is **not** a diagram of the real LFM stack. Before a later LFM-specific production study, read the actual LFM repository and deployment docs; ECS task definitions and CI/CD are not Launch Templates.

## Phase 0 — Safety, scope and budget / 成本與隔離
**Create:** a dedicated sandbox plan, resource prefix `lfm-study-`, tags `Project=LFM-Capstone-Study`, `Environment=Sandbox`, an AWS Budget alert, and a cleanup checklist. Choose Region deliberately. Estimate costs for **RDS, NAT gateways, ALB, EC2/EBS, public IPv4, logs, snapshots** before clicking Create. Set a short test window. No production credentials.
**Why:** always-on RDS/NAT/ALB can incur charges even with no traffic.
**Checkpoint:** can identify every billable component and when to delete it.
**Analogy:** 先設定零用錢上限，才開始蓋模型屋。

## Phase 1 — Draw and build the network / 網路
**Plan:** example VPC `10.20.0.0/16`, two AZs, two public `/24` subnets for ALB, two private `/24` app subnets, two private `/24` DB subnets. CIDRs are **proposed for this new sandbox**, not the Academy VPC.
**Console workflow:** VPC → Your VPCs → Create VPC; Subnets → Create subnet; Internet Gateways → Create/attach; Route Tables → create/associate public and private routes. Decide outbound access for private EC2 **before** deploying: NAT gateways (costly), endpoints where supported, or prebuilt artifacts; choose deliberately. If testing AZ independence, account for NAT per AZ.
**Verify:** route-table associations and public ALB placement; no direct public route/IP for DB or private EC2.
**Checkpoint:** explain why an internet-facing ALB can be public while application servers remain private.
**Analogy:** 大門可以讓客人進來，但故事資料庫放在上鎖的內室。

## Phase 2 — IAM and security boundaries / 權限
**Create:** `lfm-study-alb-sg` (inbound HTTP 80 from intended testers for the lab), `lfm-study-app-sg` (HTTP only from ALB SG), `lfm-study-db-sg` (3306 only from app SG); least-privilege EC2 IAM instance role for SSM and required reads; Secrets Manager for DB credentials. Do not publish the DB endpoint as an open internet service.
**Verify:** SG references, Session Manager connectivity and no credentials in User Data, GitHub or logs. Consider HTTPS/ACM with a controlled domain for a more realistic extension; HTTP-only is a temporary sandbox simplification.
**Checkpoint:** distinguish IAM (what AWS actions are permitted) from Security Groups (which network packets are permitted).
**Analogy:** IAM 是員工工作證；SG 是每個房間的門禁。

## Phase 3 — Database and fictional story records / 資料庫
**Create:** RDS MySQL, private DB subnet group, storage/backup settings appropriate for cost; database `lfm_study`; a minimal schema such as `stories(id, slug, title, body, visibility, published_at)`. Use 3–5 **fictional** entries (e.g., `a-morning-garden`) and a sample image that contains no family/private information.
**Verify:** app-side connection using a secret (not a pasted password); SELECT returns the intended sample rows; DB SG is restricted.
**Checkpoint:** explain why RDS stores structured records while S3 stores image objects.
**Analogy:** RDS 是故事目錄卡；S3 是照片收納櫃。

## Phase 4 — First app server and reproducible bootstrap / 第一台網站
**Build:** a small, deliberately non-production demo app (simple PHP/MySQL or equivalent) with `/health` and `/stories/<slug>`; include a published-only query and a clear “not found” response. Make the code and dependencies reproducible from a **versioned source artifact**; no manual fix should be required on replacement EC2.
**Console workflow:** EC2 → Launch Instances (initial proof) or Launch Templates → Create launch template; configure private subnet, app SG, IAM instance profile and User Data. Use SSM Session Manager, not a public SSH port.
**Verify:** syntax/build test; local health endpoint; actual sample-story request; database connectivity; inspect cloud-init/httpd/app logs **without printing secrets**.
**Checkpoint:** a green `/health` does not prove `/stories/a-morning-garden` works.
**Analogy:** 廚師有上班不代表真的做得出餐點。

## Phase 5 — ALB and multi-AZ application / 負載平衡
**Create:** target group (app port and `/health` path); internet-facing ALB in public subnets in both AZs; HTTP listener forwarding to target group. Verify ALB's **actual attached SG** and app SG's source reference.
**Verify:** ALB DNS serves fictional story and target health is Healthy; check both AZs.
**Checkpoint:** listener = routing rule, target group = registered app backends, health check = readiness signal.
**Analogy:** 接待員知道該把客人交給哪一位廚師。

## Phase 6 — Launch Template + Auto Scaling / 自動擴展
**Create:** immutable/versioned bootstrap (AMI or pinned artifact + User Data), Launch Template v1, ASG in two private app subnets, min=2 desired=2 max=3 or 4, attach target group, choose ELB health checks/grace period. For demand-based scaling, optionally add a carefully scoped target-tracking policy and CloudWatch metric; **manual desired-capacity changes do not prove a scaling policy**.
**Verify:** two instances InService/Healthy, both serve real story content. Record launch template versions.
**Checkpoint:** distinguish ALB traffic distribution from ASG capacity management.
**Analogy:** 接待員分配客人；經理決定要聘幾位廚師。

## Phase 7 — Controlled bug and permanent fix / 排錯演練
**Exercise:** in the sandbox only, intentionally introduce a *non-secret* config error that breaks the fictional story query but not `/health`. Compare a working and failing endpoint, inspect sanitized logs, isolate code/config vs RDS vs SG vs routing. Restore through a **new versioned deployment artifact/Launch Template**, not only by editing a running instance.
**Verify:** existing targets pass real query; new replacement target independently passes it. Do not intentionally leak credentials or disable security protections.
**Checkpoint:** distinguish a local hotfix from a durable deployment fix.
**Analogy:** 修好兩位廚師還不夠，要改新版訓練手冊。

## Phase 8 — Scale-out, scale-in and failure drill / 擴縮與復原
**Test A:** temporarily desired 2 → 3; identify new instance and Launch Template version; verify InService/Healthy **and** `/stories/a-morning-garden` on the new instance.
**Test B:** desired 3 → 2; wait for `At desired capacity`, two healthy instances, and a successful ALB story request. ASG chooses the terminated instance.
**Optional Test C:** after documenting the baseline and with no production impact, test loss of one sandbox app instance and observe ASG replacement. Avoid disruption outside sandbox.
**Checkpoint:** capture ASG Activity, instance version, target health, and real app response. Do not confuse “healthy” with functional correctness.
**Analogy:** 多請一位廚師、再恢復原班；確認客人始終吃得到餐點。

## Phase 9 — LFM comparison session / 對照真實 LFM（只讀）
Read actual LFM architecture/deployment documentation before drawing its production architecture. Compare responsibilities, **not implementation identity**:
- EC2 ASG replacement vs ECS task replacement.
- Launch Template/User Data vs container image/task definition/deployment pipeline.
- RDS sample story catalog vs LFM's actual authoritative content source (verify; do not assume).
- ALB health vs published-story and image retrieval checks.
- Security of family memories: access control, image privacy, and safe logs.
**Deliverable:** a two-column “Capstone sandbox vs verified LFM production” comparison, with links to source documentation and any differences clearly marked.
**Checkpoint:** explain how the same reliability principle can be implemented with different AWS services.

## Phase 10 — Cleanup and evidence / 清理與證據
**Evidence:** architecture diagram; sanitized settings/screenshots; VPC routes/SG rules; RDS query row count; real app output; Launch Template version; ASG 2→3→2; ALB health; lessons learned.
**Cleanup order (check dependencies):** remove ASG and EC2/target associations; ALB/listeners/target group; RDS and snapshots as intentionally selected; NAT gateways and EIPs; endpoints, unused EBS, AMIs/snapshots; security groups; subnets/routes/IGW/VPC; CloudWatch logs and budget checks. Verify no residual chargeable resources.
**Never delete LFM production resources.**
**Completion criteria:** deploy from zero, explain each component, demonstrate real request, recover/redeploy with versioned config, scale out/in, clean up, and document SAA-C03 concepts.

## Guided lab session template / 未來逐步教學格式
For **each** future session, record:
1. **Goal / 目標** — one small task.
2. **Analogy / 比喻** — third-grade explanation.
3. **AWS Console steps / 操作** — exact current UI clicks (verify current UI at that time).
4. **Why / 原因** — dependency and security.
5. **Expected evidence / 預期結果** — screenshot or safe CLI output.
6. **Troubleshooting / 排錯** — one symptom and one diagnostic step.
7. **SAA-C03 question / 考點** — explain rather than memorize.
8. **Checkpoint / 完成判斷** — PASS / NEEDS FIX / NOT STARTED.
9. **Cost + cleanup / 費用與清理** — what remains billable.
Do not claim a phase is built until the user has actually verified it.

## Personal AWS Rebuild decision
**YES — Very High Learning Value**, **future plan only**. Start with Phase 0 budget and sandbox isolation, not VPC creation. For real LFM improvements, use a separate, repository-first review and production change process.
