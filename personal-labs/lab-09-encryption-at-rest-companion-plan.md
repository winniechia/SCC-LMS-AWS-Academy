# Lab 09 — Encryption at Rest Personal AWS Rebuild Plan

## Decision

**🟢 YES — Very High Learning Value**

This document is a **future independent rebuild plan**, not a record of resources already created in a personal AWS account.

## Objective

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把 AWS Academy 事先準備好的架構，從零自己建立一次。**

The rebuild must use a personal AWS account without Academy pre-created resources. ChatGPT should guide it concept-first and step-by-step.

At rebuild time, use the **newest AWS Console GUI and current AWS behavior/documentation**. Do not treat classroom screenshots, menu positions, or button names as permanent instructions.

---

## Architecture to create from zero

```text
Personal AWS account

S3 test bucket
  -> default encryption review
  -> small test object

EC2 test instance
  |
  +--> root EBS volume
  |
  +--> separate encrypted EBS data volume
          |
          v
       AWS KMS
       customer managed symmetric key
          |
          v
       key policy / IAM permissions / grants

CloudTrail Event history
  -> observe KMS and EC2 API activity
```

The goal is not to reproduce Academy names. The goal is to rebuild and understand every dependency.

---

## Phase 0 — Safety and cost plan

Before creating anything:

- Check current AWS pricing/free-tier behavior for EC2, EBS, KMS, S3, and related services.
- Pick one Region deliberately.
- Create temporary lab naming/tags.
- Define a cleanup checklist first.
- Use the smallest practical EC2 instance and EBS volume for the learning goal.
- Never place access keys, account IDs, ARNs, key IDs, instance IDs, or other temporary identifiers in the public GitHub notes.

---

## Phase 1 — Build the S3 side from scratch

Create a new test S3 bucket deliberately.

Study and validate:

- Current default bucket encryption behavior
- Server-side encryption settings
- Upload a harmless small test object
- Inspect the object's encryption properties
- Confirm authorized retrieval works normally

Do not assume the future console or AWS defaults are identical to the classroom lab; verify current AWS documentation at rebuild time.

### Understanding checkpoint

Explain:

> Encryption at rest does not mean an authorized application must manually decrypt every S3 object before reading it.

---

## Phase 2 — Build the EC2 foundation from scratch

Academy provided the instance. The personal rebuild should create it.

Create:

- VPC/subnet choice appropriate for the lab
- Security group with minimal required access
- Small EC2 instance
- Root EBS volume
- IAM role only if the chosen validation method requires it

Prefer a secure management method available/recommended at rebuild time rather than blindly copying an old SSH workflow.

---

## Phase 3 — Create a customer managed KMS key

Create a symmetric customer managed KMS key deliberately.

Understand:

- Key alias vs key ID
- Key administrators
- Key users
- Key policy
- IAM permissions
- Grants used by integrated AWS services
- Enabled/disabled key states
- Current automatic rotation options

### 3rd-grade checkpoint

> Who manages the key is not automatically the same question as who may use the key to encrypt/decrypt application data.

---

## Phase 4 — Create encrypted EBS from scratch

Create a small data EBS volume:

- Same Availability Zone as the EC2 instance
- Encryption enabled
- Customer managed KMS key selected

Attach it to the instance.

Optionally, if useful for the future learning goal:

- Create a filesystem
- Mount the volume
- Write a harmless test file
- Confirm it remains available through the expected attach/mount lifecycle

The personal rebuild should distinguish **AWS-level EBS encryption** from filesystem-level operations.

---

## Phase 5 — KMS disable/enable failure experiment

Reproduce the classroom learning experiment safely:

1. Confirm encrypted volume works while the key is enabled.
2. Unmount if a filesystem was mounted.
3. Detach the data volume safely.
4. Disable the customer managed KMS key.
5. Attempt a fresh attach.
6. Observe and record the expected failure.
7. Re-enable the KMS key.
8. Retry attach.
9. Confirm recovery.

### Critical understanding

```text
Disabled KMS key
   != data deleted

Disabled KMS key
   = key cannot be used for required cryptographic operations
```

Also explain why a fresh detach/attach is important: an already attached encrypted volume may already have the usable data key in memory.

---

## Phase 6 — Observe CloudTrail

Use CloudTrail Event history to investigate the actions generated during the experiment.

Look for current equivalents of events such as:

- `DisableKey`
- `EnableKey`
- `AttachVolume`
- `CreateGrant`
- `Decrypt`
- `GenerateDataKeyWithoutPlaintext`

Do not force the future environment to produce an identical event list if AWS implementation details change. The learning goal is to trace which principal/service made which API request and whether it succeeded.

### Investigation questions

- Who initiated the key-state change?
- Which service attempted to use the KMS key?
- What event correlates with the failed attach?
- Which events appear after the key is enabled again?

---

## Phase 7 — Key rotation

Review the current KMS rotation configuration for the selected symmetric customer managed key.

Enable automatic rotation if it remains appropriate for the chosen key type and learning exercise.

Understand the concept rather than memorizing a console button location.

---

## Phase 8 — Failure drills

After the successful build, deliberately diagnose safe failures:

### Drill A — Key disabled

Expected mental path:

```text
Encrypted EBS attach fails
 -> check EBS encryption/key
 -> check KMS key state
 -> check key permissions/grants
 -> inspect CloudTrail
```

### Drill B — Permission problem

Use a controlled least-privilege test to distinguish:

```text
Key exists and is enabled
but
principal/service is not authorized to use it
```

This is different from a disabled key.

### Drill C — Wrong Availability Zone

Attempting to attach an EBS volume to an EC2 instance in another AZ should be recognized as an EBS architecture issue, not a KMS issue.

---

## Phase 9 — SAA-C03 explanation test

The rebuild is not complete until I can explain these without notes:

1. What is encryption at rest?
2. What is the difference between a KMS key and a data key?
3. What is envelope encryption?
4. Why can disabling a KMS key make an encrypted EBS volume unusable for a fresh attach?
5. Why does disabling a key not mean the encrypted data was erased?
6. What does CloudTrail tell me that KMS itself does not?
7. What is the difference between a key administrator and a key user?
8. Why must an EBS volume and EC2 instance be in the same AZ for attachment?

---

## Phase 10 — Cleanup

This step is mandatory in personal AWS.

Suggested dependency-aware cleanup:

1. Unmount/detach the temporary data volume safely.
2. Delete the lab-only EBS data volume.
3. Terminate the lab-only EC2 instance.
4. Delete the S3 test objects and bucket if created only for this lab.
5. Review and remove lab-only IAM resources where appropriate.
6. Schedule deletion of the customer managed KMS key only after confirming no retained resource depends on it; respect the required KMS deletion waiting period/current AWS behavior.
7. Remove other lab-only networking resources if they were created specifically for this exercise.
8. Check billing/cost tools and resource views for anything unintentionally left running.

Never schedule deletion of a KMS key until its dependencies are understood.

---

## Final mansion story / 最後的大宅院故事

> EBS 是鎖住的資料房。真正每天鎖資料的是 data key。KMS 是中央鑰匙庫，保護這把 data key。EC2 要使用資料房時，AWS 必須能透過 KMS 讓 data key 可用。如果老爺把 KMS 主鑰匙停用，資料房沒有消失，但新的開門流程會失敗。CloudTrail 則把誰關掉鑰匙、誰嘗試開門、何時失敗全部記在警衛紀錄簿。

---

## Completion definition

The future personal rebuild is complete only when I can:

- Create all prerequisites without Academy setup
- Build the KMS + encrypted EBS architecture from zero
- Explain envelope encryption
- Reproduce and diagnose the disabled-key failure
- Use CloudTrail as evidence rather than guessing
- Explain permissions/key-state failures separately
- Clean up the environment safely and verify costs/resources
