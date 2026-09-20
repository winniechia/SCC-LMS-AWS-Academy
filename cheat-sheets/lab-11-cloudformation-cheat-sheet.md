# Lab 11 — AWS CloudFormation Cheat Sheet

[Full notes](../labs/lab-11-cloudformation.md) | [Study index](../README.md) | [Personal rebuild plan](../personal-labs/lab-11-cloudformation-companion-plan.md)

## Core mental model

| CloudFormation | Memory |
| --- | --- |
| Template | construction blueprint / 施工設計圖 |
| CloudFormation | construction team / 施工團隊 |
| Stack | managed group of resources / 被管理的一整組建築 |
| Parameter | input before construction / 施工前問老闆 |
| Resource | AWS thing to build / 真正要蓋的東西 |
| Output | useful result after deployment / 蓋完的重要資訊 |
| Events | construction log / 施工日誌 |
| Change Set | renovation preview / 動工前改建清單 |
| DeletionPolicy | deletion/retention rule / 拆除時資產處理規則 |

## Lab architecture

```text
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

**Memory:** Network Stack exports -> Application Stack imports.

## Template structure

```text
Parameters = inputs
Resources  = things to build
Properties = configuration
Outputs    = useful results
```

Avoid hard-coding environment-specific identifiers when the template can receive or derive them.

## Intrinsic functions

```text
!Ref     = get a value/reference
!GetAtt  = get a resource attribute
!Sub     = substitute dynamic values into text
```

Cross-stack example pattern:

```yaml
Fn::ImportValue:
  !Sub ${NetworkStackName}-VPCID
```

## Stack lifecycle

```text
Template -> Create Stack -> Events -> CREATE_COMPLETE
                    |
                 update
                    v
              Change Set
                    |
                  review
                    v
             UPDATE_COMPLETE
```

Failure? **Check Events first.**

## Change Set lesson

Lab update:

```text
WebServerSecurityGroup
Action: Modify
Replacement: False
```

The template added HTTPS/TCP 443 for demonstration. The application itself remained accessed on port 80 in the lab.

**Replacement=False** = modify the existing resource rather than replace it.

## CloudFormation vs Auto Scaling

- **CloudFormation:** deploy/update infrastructure.
- **Auto Scaling:** adjust runtime EC2 capacity.

> CloudFormation = 蓋餐廳；Auto Scaling = 營業時調整員工數量。

## Infrastructure Composer

Visual editor for CloudFormation templates:

- displays template resources and relationships;
- lets you inspect properties;
- connects visual changes with YAML/JSON;
- can add resources/relationships visually.

It visualizes the **template**, not just a generic architecture diagram.

## DeletionPolicy

```yaml
DeletionPolicy: Snapshot
```

Lab behavior:

```text
Delete application stack
       |
       +--> EBS volume -> snapshot first
       |
       +--> network stack remains separate
```

## SAA triggers

- **repeatable infrastructure / IaC** -> CloudFormation
- **preview infrastructure update** -> Change Set
- **find deployment failure reason** -> Stack Events
- **share values between stacks** -> Outputs/Export + ImportValue
- **deployment-time choice** -> Parameter
- **preserve/snapshot stateful resource during deletion** -> DeletionPolicy
- **visual CloudFormation template relationships** -> Infrastructure Composer

## 3rd-grade memory

> Template = 施工圖; CloudFormation = 施工隊; Stack = 蓋好並管理的一整組建築.

> Export = 我把地址登記出去; ImportValue = 另一個施工隊把地址拿來用.

> Change Set = 改建前先看施工清單; Events = 施工日誌.

## Completed verification

- Network stack deployed
- Application stack imported network values
- Application URL verified
- Stack update completed
- Security Group HTTPS rule verified
- Infrastructure Composer explored
- Application stack deleted
- EBS snapshot verified
- Sensitive temporary identifiers intentionally omitted
