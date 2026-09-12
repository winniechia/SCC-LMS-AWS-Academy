# Lab 09 — Encrypting Data at Rest by Using AWS Encryption Options

## Lab status

**Class Lab: Complete and submitted**

This lab focused on encryption at rest across Amazon S3 and Amazon EBS, customer managed AWS KMS keys, the effect of disabling a KMS key, CloudTrail audit events, and key rotation.

> **KMS manages keys. S3/EBS store encrypted data. CloudTrail records key activity.**
>
> **KMS 管鑰匙；S3/EBS 放加密資料；CloudTrail 記錄鑰匙活動。**

---

## What the classroom lab demonstrated / 課堂實驗做了什麼

### 1. Amazon S3 default encryption

The lab reviewed the bucket's default encryption and confirmed that new S3 objects use server-side encryption with Amazon S3 managed keys (SSE-S3) as the base encryption behavior in the lab.

An image object was uploaded and opened through its object URL. The important lesson was that encrypted data can still be served normally to an authorized request because S3 performs the required decryption transparently.

### 2. Customer managed AWS KMS key

A symmetric customer managed KMS key named `MyKMSKey` was created.

The lab separated two concepts:

- **Key administrator** — manages the KMS key and its configuration.
- **Key user** — is allowed to use the key for cryptographic operations such as encrypt/decrypt according to policy.

### 3. Encrypted EBS data volume

A new 1-GiB EBS data volume was created in the same Availability Zone as the existing EC2 instance.

The volume was configured as encrypted and associated with `MyKMSKey`, then attached to `LabInstance`.

The instance then had:

```text
Root EBS volume  -> unencrypted (classroom starting state)
Data EBS volume  -> encrypted with MyKMSKey
```

### 4. Disable KMS key and observe failure

The key was disabled intentionally.

The encrypted EBS volume was detached and an attempt was made to attach it again. The attach operation failed with the expected message that the encrypted volume could not access the KMS key.

This failure was the desired experiment result, not a configuration mistake.

After re-enabling `MyKMSKey`, the volume could be attached successfully again.

### 5. CloudTrail audit trail

CloudTrail Event history was used to inspect KMS-related events and the failed attach sequence.

Important events examined included:

- `DisableKey`
- `AttachVolume`
- `CreateGrant`
- `Decrypt`
- `GenerateDataKeyWithoutPlaintext`
- optionally `RetireGrant`

### 6. KMS key rotation

The lab reviewed automatic key rotation for the symmetric customer managed KMS key and enabled automatic rotation.

---

## Core architecture / 核心架構

```text
                     AWS KMS
                 Customer managed key
                    MyKMSKey
                       |
                       | protects / decrypts data keys
                       v
EC2 LabInstance <--> Encrypted EBS data volume
       |
       +------------------> CloudTrail records API activity

S3 ImageBucket
       |
       +--> SSE-S3 default server-side encryption
```

---

## The most important mental model: envelope encryption

The lab's architecture shows that KMS is not used to decrypt every byte of the EBS volume directly.

Conceptually:

```text
Data on EBS
   |
   | encrypted/decrypted with
   v
Data key
   |
   | data key is protected by
   v
KMS key
```

When the encrypted EBS volume is attached, AWS needs access to the KMS key so the encrypted data key can be made usable for the instance. The usable data key is then kept in memory for encryption/decryption operations.

### Mansion analogy / 大宅院比喻

- **EBS volume** = locked records room
- **Data key** = the key that actually opens the records cabinet
- **KMS key** = the master key that protects the cabinet key
- **KMS** = central secure key vault

> KMS protects the key that protects the data.
>
> KMS 管的是「保護資料鑰匙的鑰匙」。

---

## Why disabling the key matters

A key concept from the lab:

```text
Encrypted data still exists
        +
KMS key disabled
        =
required cryptographic operation cannot complete
```

Disabling the KMS key did **not** erase the EBS data. It prevented AWS from using the key to complete the required cryptographic process during the fresh attach operation.

### Important nuance

The lab deliberately detached the volume before testing again because an already attached encrypted volume can have the usable data key in memory. A fresh attach forces AWS to perform the key-access flow again.

### Mansion analogy

The locked room is still physically there, but the central key office refuses to release/use the master key. The contents are not deleted; access is blocked.

---

## CloudTrail lesson / CloudTrail 學習重點

CloudTrail is the audit record of AWS API activity.

For this lab it answered questions such as:

- Who disabled the key?
- When was it disabled?
- Which KMS API operations were requested?
- What happened when EC2 tried to attach the encrypted EBS volume?

### Mansion analogy

> **CloudTrail = 大宅院警衛的操作紀錄簿。**

It records actions; it does not itself perform encryption.

---

## SAA-C03 takeaways

### S3 encryption

Remember the distinction between data storage and key management.

```text
S3 object
   -> server-side encryption at rest
```

The lab reviewed SSE-S3 behavior. For exam questions, always read which party must control/manage the encryption key before choosing among S3 encryption options.

### EBS encryption

Encrypted EBS volumes integrate with AWS KMS.

Exam clues:

- encrypted volume
- customer managed KMS key
- key permissions
- disabling the KMS key blocks future cryptographic use

### KMS key policy / permissions matter

Having an encrypted resource does not mean every principal can use its KMS key. The caller/service must be allowed to use the relevant key according to the applicable IAM/key-policy/grant model.

### Key administrators vs key users

Do not assume that permission to manage a key automatically means the same thing as permission to use it for application cryptographic operations.

### CloudTrail

Think **auditability / who called what API and when**.

### Key rotation

The lab reviewed automatic rotation for an AWS KMS-created symmetric customer managed key.

---

## 3rd-grade story / 三年級故事

The mansion has a records room full of important documents.

The documents are locked using a small working key called the **data key**.

Instead of leaving that small key on the table, the mansion locks that key inside a central key vault controlled by **KMS**.

When EC2 needs the records room:

1. EC2 asks the central key office for help.
2. KMS checks whether the request is allowed.
3. If allowed, the working data key becomes usable.
4. EC2 uses that key while working with the encrypted volume.

If the master KMS key is disabled, the records room does not disappear. The central key office simply refuses to unlock the working key.

CloudTrail is the notebook showing that the request happened.

---

## Classroom-environment dependency / 課堂預建依賴

AWS Academy started the lab with important resources already prepared, including the S3 bucket, EC2 instance, surrounding IAM/lab access, and other lab scaffolding. The lab instructions also guided resource names and permissions.

Therefore:

> **Completing the classroom lab is not the same as independently designing and building the architecture from zero.**

This is especially important for KMS because a personal rebuild should deliberately create the IAM permissions, KMS key policy/usage model, EC2 instance, EBS volumes, CloudTrail observations, and cleanup process instead of relying on Academy defaults.

---

## Personal AWS rebuild decision

### 🟢 YES — Very High Learning Value

This lab should be rebuilt later in a personal AWS account from scratch.

> **Do not merely replay the classroom steps. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把 AWS Academy 事先準備好的架構，自己從零建立。**

Future rebuild should use the **newest AWS Console GUI available at rebuild time** rather than depending on today's screenshots or button locations.

The future personal rebuild plan is documented separately in:

`personal-labs/lab-09-encryption-at-rest-companion-plan.md`

---

## Lab completion checkpoint

- [x] Reviewed S3 default server-side encryption
- [x] Accessed an encrypted S3 object
- [x] Created a customer managed symmetric KMS key
- [x] Created an encrypted EBS volume
- [x] Attached encrypted EBS volume to EC2
- [x] Disabled KMS key and observed expected attach failure
- [x] Re-enabled KMS key and restored attach capability
- [x] Reviewed CloudTrail events related to KMS/EBS activity
- [x] Reviewed/enabled KMS key rotation
- [x] Submitted classroom lab

**Class Lab: COMPLETE ✅**
