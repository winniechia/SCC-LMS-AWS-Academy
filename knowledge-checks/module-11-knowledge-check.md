# Module 11 — Knowledge Check Study Notes

## Status

**Knowledge Check: Completed and reviewed**

These are concept-focused study notes from the Module 11 review session. They summarize the learning and corrections rather than reproduce the quiz verbatim.

---

## Module 11 concept map

```text
Automation / IaC
      |
      +--> repeatability + configuration consistency
      +--> version-controlled infrastructure changes
      |
CloudFormation
      |
      +--> Template = blueprint
      +--> Service = construction team
      +--> Stack = managed build
      |
      +--> Parameters / Mappings / Conditions
      +--> Change Sets
      +--> Drift Detection
      |
AWS Quick Start
      +--> vetted rapid deployment / best-practice reference architecture

Amazon Q Developer
      +--> AI coding companion
      +--> accelerate coding tasks
      +--> enhance application security
```

---

## 1. Why automate infrastructure provisioning?

Automation helps manage infrastructure change reliably and supports repeatable, version-controlled processes.

### Important correction

Do not confuse **automation helps high availability** with **automation is required for high availability**.

Automation can help build and manage highly available architectures, but high availability is not itself a requirement to use automation.

A major IaC advantage over manual work is that infrastructure definitions can be version controlled.

### Memory

> **Automation = repeatable change, not an HA requirement.**
>
> **自動化 = 可重複、可追蹤的變更；不是 HA 的必要條件。**

---

## 2. Infrastructure as Code (IaC)

Key benefits emphasized in this module:

- Reuse infrastructure definitions across environments.
- Deploy environments with configuration consistency.
- Propagate controlled infrastructure updates through a common definition/process.
- Keep infrastructure definitions in version control.

This connects directly to Challenge Lab 11, where the same CloudFormation architecture was reused in more than one Region while deployment-specific values could vary.

### 3rd-grade analogy / 三年級比喻

Instead of rebuilding every café from memory, keep one master construction blueprint. Each new café starts from the same blueprint, so the buildings are less likely to become accidentally different.

> **IaC = do not rely on human memory; write down how to build the infrastructure.**
>
> **IaC = 不靠人腦記住怎麼蓋，把「怎麼蓋」寫成可重複使用的施工圖。**

---

## 3. AWS Quick Start vs an arbitrary internet template

When the goal is to deploy a solution quickly using a prebuilt approach designed around AWS best practices, the course points to **AWS Quick Start**.

A CloudFormation template found randomly on the internet might automate deployment, but its security, quality, provenance, and alignment with AWS best practices are not guaranteed.

### Memory

> **Quick deployment + AWS best-practice reference architecture -> AWS Quick Start**
>
> **快速部署 + AWS best practices -> AWS Quick Start**

Do not confuse this with CloudFormation Designer, which is used to author templates, or an AMI, which is a machine image rather than a complete reference architecture.

---

## 4. Amazon Q Developer

Amazon Q Developer is an **AI-powered coding companion** that integrates with a developer's coding workflow/IDE.

Key reasons emphasized in this Knowledge Check:

- **Accelerate coding tasks**
- **Enhance application security**

### Important corrections

Do not treat Amazon Q Developer as:

- an HA automation service,
- a compliance auditor that writes compliance tests,
- a mechanism that shares the code you write as open source,
- or an IDE itself.

### Memory

> **Amazon Q Developer = AI coding companion: code faster + help improve application security.**
>
> **Amazon Q Developer = AI 程式開發助手：加快 coding + 協助改善 application security。**

---

## 5. AWS CloudFormation — service vs template vs stack

This was one of the most important corrections in the review.

### AWS CloudFormation

CloudFormation is the **AWS service** used to model, create, and manage AWS resources.

### CloudFormation template

The template is the **infrastructure definition / blueprint** that describes what CloudFormation should build.

### CloudFormation stack

The stack is the **managed set of resources** created and managed from the template.

### 3rd-grade construction analogy

```text
CloudFormation Template = construction blueprint / 施工圖
AWS CloudFormation      = construction team / 施工隊
CloudFormation Stack    = managed build / 被管理的一整組建築
```

### Exam trap

If the question asks **"What is AWS CloudFormation?"**, do not answer "a template that describes your infrastructure." That describes a **CloudFormation template**, not the CloudFormation service.

---

## 6. AWS CloudFormation Designer

CloudFormation Designer is a **graphical design interface for authoring CloudFormation templates**.

```text
Designer
   |
   v
author/design Template
   |
   v
CloudFormation service
   |
   v
creates/manages Stack resources
```

### Memory

> **Designer = drawing/design desk for the blueprint.**
>
> **Designer = 畫施工圖的設計桌。**

It is not the deployment engine, a source-code repository, or a collection of reusable templates.

---

## 7. Parameters, Mappings, Conditions, and Change Sets

These concepts solve different problems.

| Concept | Question it answers | Simple analogy |
| --- | --- | --- |
| Parameter | What value should this deployment use? | 老闆這次選 Small 還是 Large？ |
| Mapping | What value should I look up for this key/Region? | 去對照表查這個地區用哪個材料 |
| Condition | Should this resource/property be created or applied? | 如果是 Production 才蓋第二個房間 |
| Change Set | What would change if I apply this stack update? | 裝修前先看變更預覽 |

Challenge Lab 11 reinforced Parameters and Mappings:

- a deployment parameter could select a different EC2 instance type without rewriting the template;
- `AWS::Region` plus a mapping could select a Region-specific value.

The Knowledge Check adds the important role of **Conditions** for deployment-specific differences.

### Memory

> **Parameter = choose a value. Mapping = look up a value. Condition = yes/no decision. Change Set = preview proposed changes.**
>
> **Parameter = 選值；Mapping = 查值；Condition = 決定要不要；Change Set = 先看將要改什麼。**

---

## 8. Conditions

A CloudFormation **Condition** can control whether a resource or property is created/applied based on an input or environment-specific rule.

Conceptual example:

```text
Same template
     |
Environment?
   /     \
 Dev     Prod
  |        |
skip     create
extra    extra
resource resource
```

This supports deployment-specific differences while retaining a common template.

### Exam clue

> **Different resources depending on environment -> Conditions**

---

## 9. Change Sets

A **Change Set** previews proposed stack changes before they are implemented.

Think:

```text
Current stack
     +
updated template
     |
     v
Change Set
     |
     v
review Add / Modify / Delete / Replace
     |
     v
execute if acceptable
```

### Memory

> **Change Set = WILL change.**
>
> **Change Set = 還沒施工，先看施工隊準備改什麼。**

A visual inspection of YAML is not the same as asking CloudFormation to calculate the proposed stack changes.

---

## 10. Drift Detection

**Drift** occurs when the actual configuration of a CloudFormation-managed resource differs from the configuration expected by the stack/template, such as after an out-of-band manual change.

Drift Detection helps identify those differences.

Conceptual example:

```text
Template expected state
        !=
Actual AWS resource after manual change
        |
        v
      DRIFT
```

### Change Set vs Drift Detection

| Tool | Core question |
| --- | --- |
| Change Set | **What WILL change if I update the stack?** |
| Drift Detection | **What HAS changed outside the expected stack configuration?** |

### 3rd-grade analogy

The blueprint says the café door is green. Someone manually paints it red. Drift Detection is the inspector who compares the real building with the blueprint and notices the difference.

### SAA trigger

> **manually modified / changed outside CloudFormation / actual vs expected configuration -> Drift Detection**

---

## Weak spots repaired / 本次修正的易錯點

### Automation vs high availability

Incorrect mental model: automation is required for high availability.

Correct model: **automation supports reliable change and can help HA, but HA does not require automation by definition.**

### Quick Start vs internet template

Incorrect mental model: any downloadable CloudFormation template is the safest fast path to an AWS best-practice implementation.

Correct model: **for the course scenario emphasizing rapid deployment plus AWS best practices, think AWS Quick Start.**

### Amazon Q Developer

Incorrect mental model: Amazon Q Developer writes compliance tests or automates HA.

Correct model: **AI coding companion -> accelerate coding tasks + enhance application security.**

### CloudFormation vs CloudFormation template

Incorrect mental model: AWS CloudFormation is the template.

Correct model: **CloudFormation = service; Template = blueprint; Stack = managed resources.**

### Conditions vs Change Sets

Incorrect mental model: Change Sets create deployment-specific differences.

Correct model: **Conditions control conditional deployment behavior; Change Sets preview proposed updates.**

---

## SAA-C03 rapid-recognition map

```text
Repeatable / consistent / version-controlled infrastructure
 -> IaC / CloudFormation

Rapid AWS best-practice reference deployment
 -> AWS Quick Start

AI-powered coding companion
 -> Amazon Q Developer

AWS service to model/create/manage resources
 -> AWS CloudFormation

Infrastructure blueprint
 -> CloudFormation Template

Graphical template authoring
 -> CloudFormation Designer

Deployment-time input value
 -> Parameter

Region/key lookup table
 -> Mapping

IF this environment, create/apply this
 -> Condition

Preview proposed stack update
 -> Change Set

Find manual/out-of-band resource changes
 -> Drift Detection
```

---

## 3rd-grade café story / 三年級 Café 故事

A company wants to build several cafés without relying on people to remember every construction step.

It writes the infrastructure as a reusable **CloudFormation template**, the construction blueprint. **AWS CloudFormation** is the construction team that reads the blueprint and creates a **stack**, the managed café infrastructure.

Before construction, **Parameters** let the owner choose deployment values. A **Mapping** is a lookup table for values such as Region-specific choices. A **Condition** says, "If this is the Production café, build this extra room."

When the blueprint changes, a **Change Set** shows what the construction team would change before work begins.

If someone later walks into the AWS Console and manually changes something without updating the blueprint, **Drift Detection** is the inspector that discovers the real café no longer matches the blueprint.

For a ready-made, best-practice deployment approach, the course points to **AWS Quick Start**. During coding, **Amazon Q Developer** is the AI coding companion.

---

## Final memory chain

> **IaC -> Template -> CloudFormation -> Stack -> Condition -> Change Set -> Drift Detection**
>
> **施工方法寫成 code -> 施工圖 -> 施工隊 -> 實際資源 -> 要不要蓋 -> 施工前預覽 -> 找出已被手動改掉的地方**

### One-line exam memory

> **Condition = SHOULD it be built? Change Set = WHAT WILL change? Drift Detection = WHAT HAS changed?**
