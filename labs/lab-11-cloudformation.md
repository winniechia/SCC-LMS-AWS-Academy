# Lab 11 — Automating Infrastructure Deployment with AWS CloudFormation

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-11-cloudformation-cheat-sheet.md) | [Personal rebuild plan](../personal-labs/lab-11-cloudformation-companion-plan.md)

## Purpose and completed lab record

AWS Academy Guided Lab: **Automating Infrastructure Deployment with AWS CloudFormation**.

The lab demonstrated how infrastructure can be defined in templates and deployed consistently as CloudFormation stacks. It covered a networking layer, an application layer that references the network stack, a stack update, Infrastructure Composer, and deletion behavior controlled by a deletion policy.

**Class lab: completed.**

Temporary account IDs, ARNs, VPC/subnet/resource IDs, public IPs, DNS names, credentials, and other classroom identifiers are intentionally omitted.

## 1. Architecture and stack relationship

```text
lab-network.yaml
      |
      v
lab-network Stack
  VPC + Public Subnet
      |
      | Outputs / Exports
      v
 VPC ID + Subnet ID
      |
      | Fn::ImportValue
      v
lab-application Stack
  Security Group + EC2
```

The important design idea is **layering**. Network, database, and application infrastructure can be separated so that templates and layers can be reused independently.

## 2. What AWS Academy prepared for me

**[AWS Academy prepared this]**

AWS Academy supplied the CloudFormation templates used by the guided lab:

- `lab-network.yaml`
- `lab-application.yaml`
- `lab-application2.yaml`

The templates already contained the infrastructure definitions and CloudFormation relationships needed by the exercise.

This means completing the guided lab is **not the same as writing the CloudFormation architecture from zero**.

## 3. What I deployed/configured

**[I deployed/configured this]**

- Deployed `lab-network.yaml` as the `lab-network` stack.
- Added the lab tag `application=inventory`.
- Waited for `CREATE_COMPLETE`.
- Inspected the stack's Resources, Events, Outputs, and Template.
- Observed exported VPC and public-subnet values from the network stack.
- Deployed `lab-application.yaml` as the `lab-application` stack.
- Used the `NetworkStackName` parameter to identify `lab-network`.
- Observed the application template import VPC and subnet values from the network stack.
- Verified the application through the URL produced in the stack Outputs.
- Examined the application Security Group before the update.
- Updated the application stack with `lab-application2.yaml`.
- Reviewed the Change Set preview before submitting the update.
- Observed that the WebServerSecurityGroup was modified with **Replacement = False**.
- Verified the additional inbound HTTPS/TCP 443 rule.
- Explored the application and network templates with Infrastructure Composer.
- Deleted the application stack.
- Verified that the network stack remained separate.
- Verified that the EBS volume's `DeletionPolicy: Snapshot` caused a snapshot to be created before volume deletion.

## 4. CloudFormation mental model

```text
Template       = construction blueprint / 施工設計圖
CloudFormation = construction team / 施工團隊
Stack          = managed group of built resources / 被管理的一整組建築
```

CloudFormation does not replace architecture judgment. It automates the architecture described by the template.

## 5. Template reading model

| CloudFormation concept | Meaning | 三年級記法 |
| --- | --- | --- |
| Parameters | deployment-time inputs | 施工前問老闆 |
| Resources | AWS resources to create | 真正要蓋的東西 |
| Properties | resource configuration | 建築規格 |
| Outputs | useful values after deployment | 蓋完後的重要資訊 |
| Export | expose an output for another stack | 把資訊登記給別的施工隊 |
| ImportValue | consume an exported value | 另一個施工隊把資訊拿進來 |

The lab's application stack used values exported by the network stack rather than hard-coding the VPC and subnet identifiers.

## 6. Intrinsic functions memory

```text
!Ref     = get a value/reference / 拿值或 reference
!GetAtt  = get a resource attribute / 拿 resource 的某個屬性
!Sub     = substitute values into text / 把值代入文字
```

The lab specifically demonstrated `!Sub` together with `Fn::ImportValue` to construct export names based on `NetworkStackName`.

Example concept:

```yaml
VpcId:
  Fn::ImportValue:
    !Sub ${NetworkStackName}-VPCID
```

Meaning: use the network stack name to locate the exported VPC value, then use that VPC for the application Security Group.

## 7. Dependencies and stack lifecycle

CloudFormation evaluates resource relationships rather than simply treating YAML as a top-to-bottom click script.

Useful lifecycle states:

```text
CREATE_IN_PROGRESS -> CREATE_COMPLETE
UPDATE_IN_PROGRESS -> UPDATE_COMPLETE
failure -> rollback behavior
DELETE -> stack resources handled according to template policies
```

**Events = construction log / 施工日誌.**

When a deployment fails, inspect Stack Events and the failing resource/reason before making random changes.

## 8. Updating a stack and Change Set

The lab updated the application Security Group by replacing the current template with `lab-application2.yaml`.

The new template added inbound HTTPS/TCP port 443 for demonstration. The Change Set preview showed:

```text
Action: Modify
Resource: WebServerSecurityGroup
Replacement: False
```

This means the existing Security Group could be modified without replacing the resource.

> **Change Set = renovation plan before construction / 動工前的改建清單.**

A Change Set helps review proposed infrastructure changes, but the architect still needs to understand their impact.

## 9. CloudFormation vs Auto Scaling

Do not confuse two different kinds of automation:

```text
CloudFormation
= infrastructure deployment automation
= 自動蓋／修改 infrastructure

Auto Scaling
= runtime capacity automation
= 營業時依需求增減 EC2 capacity
```

Lab 10 demonstrated runtime HA/scaling behavior. Lab 11 demonstrated repeatable infrastructure deployment and updates.

## 10. Infrastructure Composer

Infrastructure Composer is a visual editor for CloudFormation templates. In the lab it was used to:

- open the application template;
- view resources and their relationships graphically;
- inspect configurable resource properties;
- switch between the visual canvas and YAML/JSON template;
- experiment with adding resources and relationships;
- inspect the network template.

Important distinction:

> Infrastructure Composer visualizes and edits the **template definition**; it is not merely a traditional architecture diagram.

## 11. DeletionPolicy

The application template defined an EBS volume with:

```yaml
DeletionPolicy: Snapshot
```

When the application stack was deleted, CloudFormation created a snapshot before deleting the EBS volume.

### 3rd-grade analogy

> 拆餐廳之前，施工隊先把重要倉庫拍一份可以恢復的備份，再拆掉原本的倉庫。

Deletion policies matter for stateful resources because deleting infrastructure and deleting valuable data are not always the same decision.

## 12. 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

Imagine two construction teams:

```text
Network team
  builds the estate and road
       |
       | Export: "Here is the land location"
       v
Application team
  ImportValue: "I'll build the Café on that land"
```

Memory shortcuts:

- Template = 施工設計圖
- CloudFormation = 施工團隊
- Stack = 施工隊管理的一整組建築
- Parameter = 施工前問老闆
- Resource = 真正要蓋的東西
- Output = 蓋完後的重要資訊
- Export = 把資訊登記給其他施工隊
- ImportValue = 另一隊把資訊拿進來
- Change Set = 動工前先看的改建清單
- Events = 施工日誌
- DeletionPolicy = 拆除時的重要資產處理規則

## 13. SAA-C03 takeaways

- Infrastructure as Code improves repeatability and consistency.
- CloudFormation templates can be written in YAML or JSON.
- Stacks manage related AWS resources as a unit.
- Separating infrastructure into layers can improve reuse and independent lifecycle management.
- Outputs can expose useful resource values.
- Exports and `Fn::ImportValue` enable cross-stack references.
- Parameters allow deployment-time input rather than hard-coding every value.
- Change Sets help preview stack updates.
- An update can modify some resources without replacing them; replacement behavior matters.
- Stack Events are a primary troubleshooting source.
- Infrastructure Composer provides a visual way to inspect and edit CloudFormation templates.
- `DeletionPolicy` can control what happens to important resources when a stack is deleted.
- Stateful resources require deliberate deletion/retention decisions.
- CloudFormation automation is different from Auto Scaling runtime capacity automation.

## 14. Lab Completion Checkpoint / Lab 結束檢查點

- [x] Class Lab Complete
- [x] Network stack deployed
- [x] Resources / Events / Outputs inspected
- [x] Cross-stack Export -> ImportValue understood
- [x] Application stack deployed and verified
- [x] Stack update performed
- [x] Change Set preview reviewed
- [x] HTTPS/TCP 443 Security Group update verified
- [x] Infrastructure Composer explored
- [x] Application stack deleted
- [x] EBS snapshot behavior verified
- [x] Network stack independence observed
- [x] SAA-C03 takeaways captured
- [x] 3rd-grade analogies captured
- [x] Sensitive temporary AWS identifiers omitted

## 15. Personal AWS Rebuild Decision

**Yes — High Learning Value.**

The strongest learning gap is that AWS Academy supplied the CloudFormation templates. A future personal rebuild should therefore start from zero and **write the templates**, not simply upload the Academy files again.

See [Lab 11 personal rebuild plan](../personal-labs/lab-11-cloudformation-companion-plan.md).
