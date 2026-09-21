# Module 11 — Knowledge Check Cheat Sheet

## Core chain

```text
IaC -> CloudFormation Template -> CloudFormation -> Stack
```

> **Template = blueprint. CloudFormation = construction team. Stack = managed build.**
>
> **Template = 施工圖；CloudFormation = 施工隊；Stack = 被管理的一整組資源。**

## Fast recognition

| Exam clue | Answer |
| --- | --- |
| Repeatable, consistent, version-controlled infrastructure | IaC / CloudFormation |
| Rapid AWS best-practice reference deployment | AWS Quick Start |
| AI-powered coding companion | Amazon Q Developer |
| Service to model/create/manage AWS resources | AWS CloudFormation |
| Infrastructure blueprint | CloudFormation Template |
| Graphical template authoring | CloudFormation Designer |
| Deployment-time choice | Parameter |
| Lookup value by Region/key | Mapping |
| IF environment/value, create/apply something | Condition |
| Preview proposed stack update | Change Set |
| Find manual/out-of-band changes | Drift Detection |

## The three-question test

```text
Condition
= SHOULD this be built/applied?
= 要不要蓋？

Change Set
= WHAT WILL change?
= 將要改什麼？

Drift Detection
= WHAT HAS changed?
= 已經被手動改了什麼？
```

## Amazon Q Developer

Correct mental model:

- AI coding companion
- Accelerates coding tasks
- Helps enhance application security

Do **not** treat it as an HA automation service, compliance auditor, or IDE.

## Automation / IaC trap

> **Automation can help HA, but automation is not a requirement for HA.**

IaC strengths:

- repeatability
- configuration consistency
- controlled updates
- version-controlled infrastructure definitions

## Quick Start trap

> **Quick deployment + AWS best-practice reference architecture -> AWS Quick Start**

A random internet CloudFormation template is not automatically trusted or aligned with AWS best practices.

## Lab 11 connection

```text
Same template
   + Parameter -> deployment-specific input
   + Mapping   -> lookup Region-specific value
   + Condition -> conditional behavior
```

## 3rd-grade memory

> **Parameter = 選值**
>
> **Mapping = 查值**
>
> **Condition = 決定要不要**
>
> **Change Set = 施工前先看**
>
> **Drift Detection = 檢查有沒有被偷偷改**
