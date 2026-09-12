# Module 9 — Knowledge Check Study Notes

## Status

**Knowledge Check: Completed and submitted**

These are concept-focused study notes from the Module 9 review session. They summarize the learning rather than reproduce the quiz verbatim.

---

## Module 9 concept map

```text
IAM users/groups/policies
        |
        +--> IAM permission evaluation / explicit Deny
        |
        +--> RBAC vs ABAC

Federation
        |
        +--> workforce external IdP
        +--> Amazon Cognito for application users

AWS Organizations
        |
        +--> OUs
        +--> SCP guardrails

Encryption
        |
        +--> envelope encryption
        +--> AWS KMS

Sensitive data in S3
        |
        +--> Amazon Macie
```

---

## 1. IAM groups

Important characteristics:

- An IAM user can belong to more than one IAM group.
- A user receives the permissions associated with the groups that the user joins.
- IAM groups do not have their own security credentials.
- IAM groups cannot be nested inside other IAM groups.

### Key lesson

An IAM group is a convenient permissions-management container for IAM users. It is not itself a login identity.

### Mansion analogy / 大宅院

An IAM group is like a department. One worker can belong to more than one department, and joining a department gives the worker the department's applicable rules. The department itself does not carry an employee badge.

---

## 2. ABAC vs RBAC

### RBAC — Role-Based Access Control

Permissions are organized around roles/job functions.

```text
Developer -> developer permissions
DB admin  -> database permissions
Auditor   -> audit permissions
```

### ABAC — Attribute-Based Access Control

Permissions are based on attributes, commonly AWS tags.

Conceptual example:

```text
Principal tag: Project = LFM
Resource tag:  Project = LFM
             -> policy can allow access
```

### Why ABAC can scale well

ABAC can reduce the need to create or continually update many policies as users and resources grow. New tagged users/resources can match existing attribute-based rules.

### Memory

> **RBAC = ROLE. ABAC = ATTRIBUTE/TAG.**
>
> **RBAC 看職位；ABAC 看屬性／標籤。**

---

## 3. IAM permission evaluation — explicit Deny

A user can receive permissions from multiple applicable identity policies, including user and group policies.

The critical rule from the review:

> **Explicit Deny overrides Allow.**
>
> **明確 Deny 優先於 Allow。**

Example:

```text
Group policy -> Allow S3
Group policy -> Allow EC2
Group policy -> Deny ECS
User policy  -> Allow ECS
User policy  -> Allow CloudFront

Final:
S3         -> Allow
EC2        -> Allow
ECS        -> DENY
CloudFront -> Allow
```

Do not memorize this as "group policy beats user policy." The important reason is the **explicit Deny**.

---

## 4. Identity federation

Identity federation lets AWS trust identity information from an external identity provider (IdP), allowing users to use an existing identity rather than requiring a separate long-term IAM user identity for every person.

Conceptually:

```text
Workforce user
      |
External IdP
      |
Federation
      |
AWS access according to authorization
```

Federation solves the authentication/identity integration problem; AWS permissions are still required for authorization.

### Memory

> **Federation = bring an existing identity to AWS.**
>
> **Federation = 用原本的身分進 AWS。**

---

## 5. Amazon Cognito

For web/mobile application users, Amazon Cognito is a key service for authentication and federation scenarios.

From the hands-on lab:

```text
Cognito User Pool
    -> authentication
    -> token

Cognito Identity Pool
    -> temporary AWS credentials

IAM role/policy
    -> what AWS actions/resources are allowed
```

### Important distinction

For workforce users centrally accessing multiple AWS accounts, think **IAM Identity Center**.

For application/customer users, think **Amazon Cognito**.

---

## 6. AWS Organizations

AWS Organizations helps centrally manage and govern an environment containing multiple AWS accounts.

Exam clue:

> **central management + multiple AWS accounts -> AWS Organizations**

Do not confuse this with IAM, which focuses on identities and access control within AWS authorization contexts.

---

## 7. Service control policies (SCPs)

SCPs are AWS Organizations guardrails that can be applied at organization, OU, or account scope as appropriate.

Example scenario:

```text
AWS Organizations
       |
Production OU
       |
SCP: prevent deletion of CloudTrail logs
       |
Multiple production accounts
```

### Critical SAA concept

> **SCPs do not grant permissions.**

They define permission guardrails / the maximum available permission space for affected accounts/principals. Required permissions still need to be granted by the relevant authorization mechanism.

### Mansion analogy

IAM policy = a worker's job permission rules.

SCP = headquarters' company-wide/OU guardrail.

---

## 8. Envelope encryption

If sensitive data is encrypted with a data key, the data key itself must also be protected.

Envelope encryption conceptually works like this:

```text
Sensitive data
      |
 encrypted by
      v
Data key
      |
 protected/encrypted by
      v
Key-encryption key / KMS key
```

This connects directly to the encryption-at-rest lab:

> **KMS protects the key that protects the data.**
>
> **KMS 保護「保護資料的鑰匙」。**

If an exam question emphasizes a data key and asks how to protect that key, think **envelope encryption**.

---

## 9. AWS KMS

AWS Key Management Service manages cryptographic keys and cryptographic operations.

Key functions emphasized in this review:

- Create/manage symmetric and asymmetric KMS keys.
- Support key rotation where applicable.

Do not confuse:

```text
KMS cryptographic key != IAM access key
```

KMS is also not the primary storage location for the application's encrypted bulk data. Services such as S3/EBS store the data; KMS manages/protects cryptographic keys used in encryption workflows.

---

## 10. Amazon Macie vs Amazon Detective

### Amazon Macie

Think:

> **Amazon S3 + discover sensitive data -> Amazon Macie**

Macie helps discover sensitive information in Amazon S3.

### Amazon Detective

Think:

> **security investigation / suspicious activity / root-cause investigation -> Amazon Detective**

### Memory

> **Macie finds sensitive data. Detective investigates suspicious activity.**
>
> **Macie 找敏感資料；Detective 查可疑事件。**

---

## Weak spots repaired / 本次修正的易錯點

### IAM Group

Incorrect mental model: a group can have security credentials.

Correct model: **groups organize users and permissions; the group itself does not have credentials.**

### ABAC

Incorrect mental model: ABAC's advantage is explicitly listing protected resources.

Correct model: **attributes/tags allow scalable policy matching and can reduce policy-management overhead.**

### AWS Organizations vs IAM

Incorrect mental model: IAM centrally governs billing/compliance/security across many AWS accounts.

Correct model: **multi-account central governance -> AWS Organizations.**

### Symmetric encryption vs envelope encryption

Incorrect mental model: if a data key might be stolen, simply choose symmetric encryption.

Correct model: **protect the data key itself -> envelope encryption.**

### KMS

Incorrect mental model: KMS stores encrypted application data or creates IAM access keys.

Correct model: **KMS manages cryptographic keys; IAM access keys are authentication credentials.**

### Detective vs Macie

Incorrect mental model: Detective discovers sensitive information in S3.

Correct model: **Macie -> sensitive S3 data; Detective -> security investigation.**

---

## SAA-C03 rapid-recognition map

```text
IAM users with shared job permissions
 -> IAM Group

Tags/attributes drive permissions
 -> ABAC

Allow + explicit Deny
 -> DENY wins

Existing external identity
 -> Federation

Web/mobile app identities
 -> Amazon Cognito

Workforce + centralized multi-account access
 -> IAM Identity Center

Central governance of many AWS accounts
 -> AWS Organizations

OU/account permission guardrail
 -> SCP

Protect a data key
 -> Envelope encryption

Cryptographic key management / rotation
 -> AWS KMS

S3 + sensitive information discovery
 -> Amazon Macie

Security finding investigation / root cause
 -> Amazon Detective
```

---

## 3rd-grade mansion story / 三年級大宅院故事

A company owns several mansions.

Inside each mansion, workers belong to departments (**IAM groups**) and receive department rules. Some rules depend on their job (**RBAC**), while other rules can depend on labels on the worker and room (**ABAC**).

If a guardbook explicitly says **DENY**, another permission slip saying Allow cannot override that explicit Deny.

Visitors who already have trusted identification from another organization can use **federation**. Customers entering an application can use **Cognito**.

The company headquarters uses **AWS Organizations** to organize all the mansions. It can place an **SCP guardrail** around an entire production district so individual mansions cannot exceed the corporate boundary.

Important documents are encrypted with a working **data key**. Instead of leaving that key unprotected, another key protects it — **envelope encryption**. **KMS** is the secure key-management office.

Finally, **Macie** inspects the S3 warehouse for sensitive documents, while **Detective** investigates suspicious security activity.

---

## Final memory chain

> **Identity -> Permissions -> Multi-account guardrails -> Encryption -> Sensitive-data discovery**
>
> **IAM/Federation/Cognito -> Organizations/SCP -> Envelope/KMS -> Macie**
