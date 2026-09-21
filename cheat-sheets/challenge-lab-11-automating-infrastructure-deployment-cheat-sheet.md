# Challenge Lab 11 — CloudFormation Automation Cheat Sheet

[Full notes](../labs/challenge-lab-11-automating-infrastructure-deployment.md) | [Guided Lab 11](../labs/lab-11-cloudformation.md) | [Study index](../README.md) | [Personal rebuild plan](../personal-labs/lab-11-cloudformation-companion-plan.md)

## Architecture memory

```text
Git commit/push
     |
CodeCommit
     |
CodePipeline
     |
CloudFormation
     |
     +--> Network Stack -> Exports
     |                     |
     +--> Application Stack <- ImportValue

Same templates -> us-east-1 + us-west-2
```

## Core mental model

| Concept | Memory |
| --- | --- |
| Template | 施工設計圖 |
| CloudFormation | 施工團隊 |
| Stack | 被管理的一整組建築 |
| Parameter | 施工前問老闆 |
| Mapping | 地區/條件對照表 |
| Output | 蓋完的重要資訊 |
| Export | 對外登記值 |
| ImportValue | 拿另一 stack 的值 |
| UserData | EC2 第一天工作清單 |
| CloudFormation role | 施工隊工作證 |
| EC2 instance role | 店員員工證 |

## Intrinsic functions

```text
!Ref         = use a value/reference
!GetAtt      = get a resource attribute
!Sub         = substitute values into text
!FindInMap   = look up a Mapping
!ImportValue = consume another stack's Export
```

## Parameter vs Mapping

**Parameter:** deployment-time choice.

> 這次老闆要選哪個規格？

**Mapping:** predefined lookup.

> 到這個 Region，查表應該用哪個值？

Challenge example:

```text
Instance type -> Parameter override
Region key pair -> Mapping + AWS::Region
```

## Cross-stack pattern

```text
Network Stack
  Export VPC + Subnet
          |
          v
Application Stack
  ImportValue
```

Avoid hard-coded temporary VPC/subnet IDs.

## Dynamic AMI

The template used an SSM public parameter path for the latest Amazon Linux 2 AMI rather than one hard-coded AMI ID.

**Trigger:** environment-specific AMI changes -> consider dynamic parameter resolution.

## UserData

```text
CloudFormation creates EC2
 -> EC2 boots
 -> UserData runs
 -> packages/services/app configured
```

CloudFormation builds the machine; UserData prepares software inside it.

## CI/CD IaC flow

```text
YAML change
 -> git commit/push
 -> CodeCommit
 -> CodePipeline
 -> CloudFormation
 -> stack create/update
```

Git history = what definition changed.

CloudFormation Events = what happened during deployment.

## Multi-Region lesson

```text
same template
   +--> us-east-1
   +--> us-west-2
```

The template was reusable because Region-specific differences were handled through parameters/mappings and each Region had its required dependencies.

**Important:** reuse is not automatic replication or failover.

## S3 URL lesson

```text
S3 Object URL != public object
```

Browser `AccessDenied` on a private object does not prove the upload failed. Do not make the bucket public just to make the URL open anonymously.

## Role distinction

- CloudFormation execution role -> permissions for infrastructure deployment.
- EC2 instance role/profile -> permissions for the running EC2 workload.

## Exam wording correction

Minimal S3 resource definition **inside the Resources section = 2 lines**:

```yaml
S3Bucket:
  Type: AWS::S3::Bucket
```

Do not count the `Resources:` section heading when the question scopes the count to the resource definition inside that section.

## SAA triggers

- repeatable infrastructure -> CloudFormation / IaC
- deployment-time choice -> Parameter
- predefined lookup by Region -> Mapping + `AWS::Region`
- cross-stack value -> Output/Export + ImportValue
- current AMI via public parameter -> SSM parameter
- first-boot configuration -> UserData
- automated template deployment -> source control + pipeline + CloudFormation
- deployment failure -> Stack Events
- duplicate architecture in another Region -> reuse template while handling Region-scoped dependencies
- S3 URL returns AccessDenied -> check permissions/context; URL does not imply public access

## Completed verification

- Network and application templates validated
- Pipeline deployments succeeded
- Café application worked
- Network and application duplicated to Oregon
- Oregon EC2 used `t3.micro`
- Sensitive temporary identifiers intentionally omitted
