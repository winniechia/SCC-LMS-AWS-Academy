# Module 9 Knowledge Check — Quick Review

## Core memory map

```text
IAM Group       -> users share permissions; group has no credentials
RBAC            -> ROLE
ABAC            -> ATTRIBUTE / TAG
Explicit Deny   -> beats Allow
Federation      -> use an existing external identity
Cognito         -> web/mobile application identities
Organizations   -> central multi-account governance
SCP             -> organization/OU/account permission guardrail; does not grant access
Envelope        -> protect the data key
KMS             -> cryptographic key management and rotation
Macie           -> sensitive data in S3
Detective       -> investigate suspicious security activity
```

## 大宅院快速記憶

| AWS concept | 大宅院 |
| --- | --- |
| IAM Group | 部門；員工可加入多個部門，部門本身沒有員工證 |
| RBAC | 看職位 |
| ABAC | 看標籤／屬性 |
| Explicit Deny | 警衛看到明確禁止就不放行 |
| Federation | 使用外部已可信的身分證明 |
| Cognito | 網站／App 客人的身分入口 |
| AWS Organizations | 多座宅院的總管府 |
| SCP | 總管府的權限護欄 |
| Envelope encryption | 小鑰匙鎖資料，再保護小鑰匙 |
| KMS | 中央鑰匙管理室 |
| Macie | S3 倉庫敏感資料檢查員 |
| Detective | 安全事件偵探 |

## High-value distinctions

### IAM Group

```text
User -> can join multiple groups
Group -> permissions organization
Group -> NO security credentials
Group -> cannot contain another group
```

### ABAC vs RBAC

```text
RBAC = job role
ABAC = attributes/tags
```

ABAC can scale with fewer policy changes as tagged users/resources grow.

### IAM evaluation

```text
Allow + Explicit Deny = DENY
```

Do not say "group policy overrides user policy." The important rule is **explicit Deny**.

### Federation vs Cognito vs Identity Center

```text
Existing external identity -> Federation
Application/customer users -> Cognito
Workforce + many AWS accounts -> IAM Identity Center
```

### Organizations and SCP

```text
Many AWS accounts + central governance -> AWS Organizations
OU/account guardrail -> SCP
```

**SCP does not grant permissions.**

### Envelope encryption

```text
Data
  -> encrypted by Data Key
Data Key
  -> protected/encrypted by another key (for AWS patterns, often KMS)
```

Memory:

> **KMS protects the key that protects the data.**

### KMS

KMS can manage cryptographic keys, including symmetric/asymmetric key types and supported rotation workflows.

```text
KMS key != IAM access key
KMS != bulk encrypted-data storage
```

### Macie vs Detective

```text
S3 + sensitive data -> Macie
Security investigation/root cause -> Detective
```

## Exam trigger words

| Trigger | Think |
| --- | --- |
| user belongs to several permission sets/groups | IAM Group |
| attributes, tags, scalable authorization | ABAC |
| explicit Deny | Deny wins |
| external identity provider | Federation |
| web/mobile users | Cognito |
| many AWS accounts, central governance | Organizations |
| OU-wide restriction | SCP |
| protect a data key | Envelope encryption |
| encryption key creation/management/rotation | KMS |
| discover sensitive information in S3 | Macie |
| investigate suspicious activity | Detective |

## Final 10-second review

> **Group = users, no credentials.**
>
> **ABAC = tags.**
>
> **Explicit Deny wins.**
>
> **Federation = existing identity.**
>
> **Cognito = app users.**
>
> **Organizations = many accounts.**
>
> **SCP = guardrail, not permission grant.**
>
> **Envelope = protect the data key.**
>
> **KMS = cryptographic keys.**
>
> **Macie = sensitive S3 data; Detective = investigation.**
