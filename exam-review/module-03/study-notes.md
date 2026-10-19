# Module 03 — Securing Access | AWS SAA-C03 Review

> Study session: October 18, 2026. Bilingual English / 繁體中文, print-friendly. Six review checkpoints and practice scenarios are original review exercises, **not** verbatim AWS Academy Student Guide questions.

## Learning goals / 學習目標

- Apply least privilege and secure AWS account access / 使用最小權限與保護 AWS 帳戶。
- Distinguish IAM User, Group, Role, and temporary credentials / 區分使用者、群組、角色及臨時憑證。
- Read IAM policy JSON and understand S3 ARNs / 讀懂 IAM 政策與 S3 資源名稱。
- Evaluate default deny, explicit deny, permissions boundaries, and SCPs / 判斷有效權限。
- Design secure EC2-to-S3 and cross-account access / 設計安全的 EC2–S3 與跨帳戶存取。

## Checkpoint 1 — AWS Security Principles / AWS 安全原則

**Least privilege（最小權限）:** Give a principal only the actions and resources needed for a job. 只給完成工作必要的操作與資源。**三年級比喻：**圖書館助理只需要借閱指定書籍的通行證，不需要全校的萬能鑰匙。

**Root user（根使用者）:** The AWS account owner identity has extensive privileges. Enable MFA, avoid routine root use, and do not use root access keys for everyday work. Root is reserved for the limited tasks that require it. 根使用者權限非常高；日常操作應避免使用。

**Shared Responsibility Model（共同責任模型）:** AWS protects security **of** the cloud; customers protect security **in** the cloud, including permissions and their data. AWS 保護雲端本身；客戶保護放在雲端裡的資料與設定。

**Exam trap / 考試陷阱:** `AdministratorAccess` is almost never the best answer when a narrow permission meets the requirement.

## Checkpoint 2 — IAM Users and Groups / 使用者與群組

- **IAM User（使用者）:** An identity in one AWS account that can have long-term credentials. A human user generally benefits from federation/IAM Identity Center and temporary credentials instead of long-term access keys.
- **IAM Group（群組）:** A collection of IAM Users for managing permissions. An IAM User may be in multiple groups, but groups cannot be nested and groups do not sign in.
- **Analogy / 比喻:** 學生 = User；讀書會 = Group。學生可以參加多個讀書會，讀書會本身不能登入 AWS。
- **Exam trap:** IAM Role is not an IAM Group.

## Checkpoint 3 — IAM Roles and STS / 角色與臨時憑證

**IAM Role（角色）:** An AWS identity that trusted principals can assume, usually with temporary credentials. 角色就像一個可領取的工作身分。

**Trust Policy（信任政策）** = *Who can assume this role?* 誰能擔任角色？

**Permissions Policy（權限政策）** = *What actions can this role perform?* 擔任角色後可以做什麼？

**AWS STS = Security Token Service（安全權杖服務）** issues temporary security credentials. `sts:AssumeRole` is used to assume a role. STS 好比發放有期限的工作證。

**EC2 Instance Profile（執行個體設定檔）:** Provides an IAM Role to an EC2 instance so the application can obtain automatically rotated temporary credentials. Do not hard-code long-term access keys on EC2.

**Cross-account role example / 跨帳戶角色範例:**
1. Account B creates a role with appropriate permissions and a trust policy for Account A's authorized principal.
2. The principal in Account A is allowed to call `sts:AssumeRole` for that role.
3. The workload obtains temporary credentials and accesses permitted resources in B.

**Exam trap:** Trust Policy determines who may assume; Permissions Policy determines what they may do. Both matter.

## Checkpoint 4 — IAM Policies and S3 ARNs / IAM 政策與 ARN

**ARN = Amazon Resource Name（Amazon 資源名稱）.** An ARN uniquely identifies a resource. ARN 就像 AWS 資源的地址。

| JSON key | Meaning / 意義 |
| --- | --- |
| `Version` | IAM policy language version / 政策語言版本 |
| `Statement` | One or more policy statements / 一個或多個敘述 |
| `Effect` | `Allow` or `Deny` / 允許或拒絕 |
| `Action` | AWS API operation / AWS 操作 |
| `Resource` | Target ARN(s) / 指定資源 |
| `Condition` | Optional additional requirements / 附加條件 |

**Important:** `"Version": "2012-10-17"` specifies policy language version; it is **not** the policy's creation date or expiration date.

### Bucket versus object / Bucket 與物件

**Analogy:** S3 Bucket = 圖書館建築；S3 Object = 裡面的一本書。

| Need / 需求 | IAM action | Resource ARN |
| --- | --- | --- |
| List keys in the bucket / 列出目錄 | `s3:ListBucket` | `arn:aws:s3:::photos` |
| Read objects / 讀取物件 | `s3:GetObject` | `arn:aws:s3:::photos/*` |
| Upload objects / 上傳物件 | `s3:PutObject` | `arn:aws:s3:::photos/*` |
| Delete objects / 刪除物件 | `s3:DeleteObject` | `arn:aws:s3:::photos/*` |

`*` is a wildcard（萬用字元）. For S3, `photos/*` covers object keys in the specified bucket. `ListBucket` does not use the object ARN.

### Example A — EC2 reads objects from only one bucket / EC2 只讀指定 Bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-data/*"
    }
  ]
}
```

**What this does:** Allows reading objects in `company-data`; it does not grant `s3:ListBucket`, `s3:PutObject`, or `s3:DeleteObject`.

### Example B — List objects AND read them / 可列出也可閱讀

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::photos"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::photos/*"
    }
  ]
}
```

### Example C — Upload only / 只允許上傳

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::company-uploads/*"
    }
  ]
}
```

**Production note / 實務注意:** Access can also be affected by bucket policies, KMS key permissions, applicable deny statements, and other guardrails. These snippets illustrate identity permissions only.

## Checkpoint 5 — Permission Evaluation / 權限判斷

1. **Default Deny（預設拒絕）:** If no applicable policy grants a requested action, it is denied. 沒有授權，就拒絕。
2. **Explicit Allow（明確允許）:** A policy grants a specified action and resource, subject to all other applicable controls. 政策允許指定操作。
3. **Explicit Deny（明確拒絕）:** An applicable explicit deny wins over allow. **Explicit Deny > Allow.**

**Permissions Boundary（權限邊界）:** Limits maximum effective permissions of an IAM User or Role; it **does not grant** permissions by itself. 可視為使用者／角色的「權限天花板」，不是額外發權限。

**SCP = Service Control Policy（服務控制政策）:** AWS Organizations guardrail limiting permissions available to IAM principals in **member accounts**. An applicable SCP explicit deny can block a member-account IAM administrator even with `AdministratorAccess`. SCP also **does not grant** permissions. SCP does not restrict IAM principals in the Organizations management account. Be aware that details involving resource-based policies and special principals have nuances beyond the simplified exam model.

| Policy type | Attached/evaluated where? / 用在哪裡？ | Grants access? / 自己會授權？ |
| --- | --- | --- |
| Identity policy | IAM user/role / 身分 | With Allow |
| Permissions Boundary | IAM user/role / 身分上限 | **No** |
| SCP | AWS Organizations member accounts / 組織上限 | **No** |
| S3 Bucket Policy | S3 bucket / 資源政策 | Can grant resource access |

**Three-grade analogy / 三年級比喻:** 老師說你可以進教室 = Allow；校長公告不可進實驗室 = Explicit Deny；即使你有萬能通行證，仍不可以違反校長的禁令。

## Checkpoint 6 — SAA-C03 Architecture Scenarios / 架構情境

### Scenario 1 — Private S3 access from EC2 / EC2 讀取私人 S3
**Requirements:** No long-term access keys; read only objects from one bucket; least privilege.  
**Best:** IAM Role on EC2 via instance profile with `s3:GetObject` on `arn:aws:s3:::company-data/*`.  
**Avoid:** Static IAM User keys, public bucket, `s3:*`, `AdministratorAccess`.

### Scenario 2 — Assume role across accounts / 跨帳戶擔任角色
**Best:** Trust policy in the target account plus appropriate `sts:AssumeRole` permission in the source; assumed role permissions grant target actions.  
**Analogy:** 去另一間學校工作，要先獲准領到當地臨時工作證。

### Scenario 3 — Grant cross-account S3 access directly / 直接允許跨帳戶讀取
**Best when the requirement specifically asks for permissions on the bucket:** S3 Bucket Policy, a **resource-based policy**. Cross-account callers typically also need appropriate source-side authorization and no relevant denies.  
**Do not confuse:** Permissions Boundary is an upper limit on an IAM principal, **not** an S3 bucket access policy.

### Scenario 4 — Prevent CloudTrail being stopped across member accounts / 禁止成員帳戶停用 CloudTrail
**Best:** Attach an AWS Organizations SCP with explicit deny of `cloudtrail:StopLogging` to the appropriate OU or member accounts.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "cloudtrail:StopLogging",
      "Resource": "*"
    }
  ]
}
```

This sample blocks `StopLogging` for affected IAM principals; a thorough guardrail may also restrict `cloudtrail:DeleteTrail` and `cloudtrail:UpdateTrail`. SCP restrictions do not apply to the AWS Organizations management account.

## Mistake review / 錯題重點

- **Bucket ARN vs Object ARN:** `ListBucket` → bucket ARN; `GetObject`/`PutObject` → object ARN ending with `/*`. 用「圖書館 vs 書本」記憶。
- **Bucket Policy vs Permissions Boundary:** Need access defined on the S3 resource? Bucket Policy. Need a maximum permission limit on user/role? Boundary.
- **SCP vs AdministratorAccess:** An SCP explicit deny overrides an IAM admin's allow in member accounts. Do not pick `Allow cloudtrail:StopLogging` when the question asks to **prevent** stopping logging.
- **Least privilege:** Read = `GetObject`; upload = `PutObject`; list = `ListBucket`; don't grant `s3:*` by habit.

## Quick self-test / 自我測驗

1. What does ARN stand for? → **Amazon Resource Name**.
2. What does STS stand for? → **Security Token Service**.
3. Who is allowed to assume a role? → **Trust Policy**.
4. What can the assumed role do? → **Permissions Policy**, subject to applicable constraints.
5. What happens if no policy allows an action? → **Default Deny**.
6. Which wins: Allow or Explicit Deny? → **Explicit Deny**.
7. IAM User/Role maximum permission limit? → **Permissions Boundary**.
8. Member account organizational guardrail? → **SCP**.
9. Policy attached directly to an S3 bucket? → **S3 Bucket Policy**.
10. Secure EC2 credentials without static access keys? → **IAM Role via Instance Profile**.

## Session results / 本次成績

| Checkpoint | Result |
| --- | --- |
| 1. AWS Security Principles | 2/3 |
| 2. IAM Users and Groups | 3/3 |
| 3. IAM Roles | 3/3 |
| 4. IAM Policies | 3/4 |
| 5. Permission Evaluation | 4/4 |
| 6. SAA-C03 Architecture Scenarios | 4/6 |
| **Total / 總分** | **19/23 = 83%** |

All six checkpoints completed. Additional explanatory mini-quizzes and the repeated final SCP answer were not counted again. / 六個關卡完成；額外練習和重答不重複計分。

## AWS documentation / 參考資料

- [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM policy evaluation logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [AWS STS](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html)
- [S3 actions, resources, and condition keys](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazons3.html)
- [AWS Organizations SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)

---
*Educational review notes / 本文件是複習教材，請以 AWS 官方文件及最新考綱為準。*
