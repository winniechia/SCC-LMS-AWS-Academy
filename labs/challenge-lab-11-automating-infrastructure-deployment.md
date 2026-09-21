# Challenge Lab 11 — Automating Infrastructure Deployment

[Study index](../README.md) | [Guided Lab 11 notes](lab-11-cloudformation.md) | [Printable cheat sheet](../cheat-sheets/challenge-lab-11-automating-infrastructure-deployment-cheat-sheet.md) | [Personal rebuild plan](../personal-labs/lab-11-cloudformation-companion-plan.md)

## Purpose and completed lab record

AWS Academy Challenge Lab: **Automating Infrastructure Deployment**.

The challenge moved beyond deploying supplied templates. I created and extended CloudFormation YAML, connected network and application stacks, used a CodeCommit/CodePipeline workflow to deploy changes, and then reused the same templates to duplicate the Café network and application in a second AWS Region.

**Class lab: completed.**

Temporary account IDs, ARNs, stack operation IDs, VPC/subnet/security-group/instance IDs, public IPs, DNS names, S3 bucket names, credentials, key material, and other classroom identifiers are intentionally omitted.

## 1. Architecture I built

```text
CodeCommit repository
       |
       v
CodePipeline
       |
       v
CloudFormation
       |
       +--> Network Stack
       |      VPC
       |      Public Subnet
       |      Outputs / Exports
       |
       +--> Application Stack
              Security Group
              EC2 Cafe Web Server
              UserData bootstrap

Same templates
       |
       +--> us-east-1
       |
       +--> us-west-2
```

The network stack exported VPC and subnet values. The application stack imported those values instead of hard-coding environment-specific resource IDs.

## 2. What AWS Academy prepared for me

**[AWS Academy prepared this]**

The challenge environment supplied substantial prerequisites, including the Lab IDE, classroom IAM permissions/roles, a CodeCommit repository, CodePipeline pipelines, S3 buckets used by the exercise, starter template material, application bootstrap assets, and other classroom infrastructure.

The Academy environment also provided the CI/CD framework that reacted to repository changes.

This matters because completing the challenge did **not** mean I created the entire CI/CD platform and classroom environment from zero.

## 3. What I built/configured

**[I built/configured this]**

- Created an S3 bucket through a small CloudFormation template.
- Updated that template to configure static website hosting.
- Added a Website URL Output using `!GetAtt`.
- Validated templates with `aws cloudformation validate-template`.
- Updated the S3 stack and verified the static website.
- Created `templates/cafe-network.yaml`.
- Defined network resources including a VPC, public subnet, routing, and subnet association.
- Committed and pushed the network template to CodeCommit.
- Observed the network pipeline deploy the CloudFormation stack.
- Added network Outputs and Exports for the public subnet and VPC.
- Created `templates/cafe-app.yaml`.
- Added Parameters for the AMI, network stack name, and EC2 instance type.
- Used an SSM public parameter for the latest Amazon Linux 2 AMI rather than hard-coding an AMI ID.
- Added a Region mapping for Region-specific key-pair selection.
- Created the application Security Group.
- Imported VPC and subnet values from the network stack.
- Created the EC2 Café web server.
- Added UserData to install/configure the web application during first boot.
- Validated, committed, and pushed the application template.
- Observed the application pipeline successfully deploy the stack.
- Verified the dynamic Café website.
- Reused `cafe-network.yaml` to create the network stack in `us-west-2` with the AWS CLI.
- Created the Region-specific Oregon key pair required by the template mapping.
- Copied `cafe-app.yaml` to the lab S3 template bucket.
- Used the same application template to create an Oregon application stack.
- Overrode the instance-type parameter to `t3.micro` without editing the template.
- Verified the Oregon Café EC2 instance was running with the expected instance type.
- Completed the lab submission.

## 4. CloudFormation template model

```text
Parameters = values chosen at deployment time
Mappings   = lookup tables inside the template
Resources  = AWS resources CloudFormation creates
Outputs    = useful values returned after deployment
Exports    = Outputs made available to other stacks
```

### 3rd-grade analogy / 三年級比喻

> Template = 施工設計圖.
>
> Parameters = 施工前問老闆要哪個規格.
>
> Mappings = 施工隊的地區對照表.
>
> Resources = 真正要蓋的東西.
>
> Outputs = 蓋完後的重要資訊.
>
> Export / ImportValue = 一個施工隊把地址登記出去，另一個施工隊拿來使用.

## 5. Intrinsic functions used

| Function | Meaning | Memory |
| --- | --- | --- |
| `!Ref` | use a parameter/resource reference | 拿這個值 |
| `!GetAtt` | retrieve a resource attribute | 拿 resource 的某個屬性 |
| `!Sub` | substitute values into a string | 把值代入文字 |
| `!FindInMap` | look up a value in a Mapping | 去對照表查答案 |
| `!ImportValue` | consume an exported stack value | 拿另一個 stack 登記的資訊 |

The application stack used `!ImportValue` to consume the network stack's exported VPC and subnet values.

## 6. Parameters vs Mappings

These solve different problems.

```text
Parameter
= caller chooses a value at deployment time

Mapping
= template chooses a predefined value based on a lookup key
```

In the challenge, the instance type could be overridden at stack creation. The Region mapping allowed the same application template to select the appropriate key-pair name based on `AWS::Region`.

### Memory

> Parameter = 老闆這次要選什麼？
>
> Mapping = 到這個地區時，施工圖規定查表用什麼？

## 7. Dynamic AMI selection with SSM

The application template used an SSM public parameter path for the Amazon Linux 2 AMI instead of embedding one fixed AMI ID.

```yaml
Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
```

**SAA memory:** avoid unnecessary hard-coded environment-specific values when CloudFormation can receive or resolve them dynamically.

## 8. Cross-stack references

The network template exported the VPC and public subnet values. The application template imported them.

```text
Network Stack
   |
   +--> Export VPC
   +--> Export Subnet
             |
             v
Application Stack
   |
   +--> Import VPC -> Security Group
   +--> Import Subnet -> EC2 network interface
```

This creates an explicit relationship between infrastructure layers without copying temporary resource IDs into the application template.

## 9. UserData = first-day work list

The EC2 resource used UserData to bootstrap the Café application.

Conceptually:

```text
CloudFormation creates EC2
          |
          v
EC2 boots
          |
          v
UserData runs
          |
          +--> update/install packages
          +--> start services
          +--> download Café setup script
          +--> configure application
```

### 3rd-grade analogy

> CloudFormation hires the new Café worker. UserData is the worker's first-day checklist telling them how to prepare the shop.

CloudFormation creates infrastructure; UserData configures software inside the new instance at boot.

## 10. CI/CD infrastructure workflow

The challenge used a repository and pipelines so a template change could trigger infrastructure deployment.

```text
edit YAML
   |
git commit / push
   |
CodeCommit
   |
CodePipeline
   |
CloudFormation
   |
create/update AWS resources
```

This is infrastructure delivery automation: the version-controlled template is the source definition, and CloudFormation performs the infrastructure change.

## 11. Multi-Region reuse

The final task demonstrated the strongest benefit of the challenge: the same templates were reused in a second Region.

For the network layer, the AWS CLI specified:

```text
--region us-west-2
```

For the application layer, the same `cafe-app.yaml` was supplied to CloudFormation in Oregon. The template adapted through Parameters, `AWS::Region`, and the Region mapping.

```text
same cafe-network.yaml
   +--> us-east-1
   +--> us-west-2

same cafe-app.yaml
   +--> Region-specific mapping
   +--> deployment-time instance type
   +--> Region-local network exports
```

This demonstrated **repeatability and portability**, not automatic cross-Region replication.

## 12. Important Region scope lesson

Creating the second Region deployment did not magically make the original resources global.

CloudFormation stacks and most resources in this exercise are Region-scoped. The network stack had to be created in Oregon, and the Oregon application stack consumed the exports available in that Region.

**Memory:** same blueprint can build another house in another city; the first house does not move there automatically.

## 13. S3 Object URL lesson

The application template was uploaded to a private lab S3 bucket and its Object URL was supplied to CloudFormation.

Opening that Object URL directly in an unauthenticated browser returned `AccessDenied`. That did **not** mean the upload failed.

> Having an S3 Object URL does not mean the object is public.

Do not make a bucket public merely to fix an expected anonymous-access denial.

## 14. IAM role distinction

Two role concepts must stay separate:

```text
CloudFormation execution role
= permissions used to create/update AWS infrastructure

EC2 instance role / instance profile
= permissions used by the running EC2 instance
```

### Memory

> CloudFormation role = 施工隊工作證.
>
> EC2 role = 店員自己的員工證.

## 15. Git history as infrastructure history

Because templates were committed to source control, the repository recorded which files and lines changed.

That means IaC + version control provides a reviewable history of infrastructure definitions.

This does not replace CloudFormation Events: Git explains **what definition changed**, while Stack Events explain **what happened during deployment**.

## 16. Grading correction worth remembering

One question asked how many lines **in the Resources section** are required to define the minimal S3 bucket resource.

The correct count is **2**:

```yaml
S3Bucket:
  Type: AWS::S3::Bucket
```

The `Resources:` section heading is not part of the two-line resource definition.

**Exam-reading lesson:** answer the exact scope of the wording; do not count the section heading when the question asks for lines that define the resource *inside* the section.

## 17. SAA-C03 takeaways

- Infrastructure as Code makes deployments repeatable and consistent.
- CloudFormation templates can separate network and application layers.
- Parameters provide deployment-time customization.
- Mappings provide predefined lookup logic.
- `AWS::Region` can drive Region-aware template behavior.
- SSM public parameters can reduce hard-coded AMI IDs.
- Outputs/Exports plus `ImportValue` connect stacks without hard-coded resource IDs.
- `!Ref`, `!GetAtt`, `!Sub`, and `!FindInMap` solve different value-resolution problems.
- UserData bootstraps software after EC2 creation.
- Version-controlled IaC provides change history.
- CodePipeline can automate CloudFormation deployments after source changes.
- The same template can be reused in multiple Regions when Region-specific dependencies are handled correctly.
- Multi-Region reuse is not the same as automatic replication or failover.
- S3 Object URL does not imply public access.
- CloudFormation execution roles and EC2 instance roles serve different principals.
- Stack Events remain the evidence source for deployment troubleshooting.

## 18. Lab Completion Checkpoint / Lab 結束檢查點

- [x] Class Lab Complete
- [x] S3 resource created through CloudFormation
- [x] Static website configuration added
- [x] Network template created and validated
- [x] Network pipeline deployment verified
- [x] Network Outputs/Exports added
- [x] Application template created and validated
- [x] Parameter, Mapping, ImportValue, and UserData behavior understood
- [x] Application pipeline deployment verified
- [x] Dynamic Café website verified
- [x] Second-Region network stack created
- [x] Region-specific key-pair mapping verified
- [x] Same application template reused in Oregon
- [x] `t3.micro` parameter override verified
- [x] Oregon EC2 verified running
- [x] Grading wording correction captured: minimal S3 resource definition = 2 lines inside Resources
- [x] SAA-C03 takeaways captured
- [x] 3rd-grade analogies captured
- [x] Sensitive temporary AWS identifiers omitted

## 19. Personal AWS Rebuild Decision

**Yes — Very High Learning Value.**

The guided Lab 11 rebuild plan already covers writing layered CloudFormation templates from zero. This challenge adds important future rebuild goals:

- create the source-control and deployment workflow independently rather than relying on Academy-prepared pipelines;
- write Parameters, Mappings, Outputs, Exports, and imports from zero;
- use an SSM public parameter for AMI selection;
- bootstrap a small application with UserData;
- deploy the same architecture into a second Region deliberately;
- prove what is Region-scoped;
- troubleshoot from Git history plus CloudFormation Events;
- clean up both Regions.

See the updated [Lab 11 personal rebuild plan](../personal-labs/lab-11-cloudformation-companion-plan.md).
