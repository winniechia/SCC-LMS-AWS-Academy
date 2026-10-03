# Challenge Lab 14 — Serverless Daily Sales Reporting

## Status

**Completed — Full score**

This challenge built and troubleshot a serverless daily sales-report workflow with AWS Lambda, Amazon RDS, Amazon SNS, Amazon EventBridge, IAM, VPC networking, and Amazon CloudWatch Logs.

## Architecture

```text
Amazon RDS café sales data
        |
        v
salesAnalysisReportDataExtractor (Lambda)
        |
        v
salesAnalysisReport (Lambda)
        |
        v
Amazon SNS — SalesReportTopic
        |
        v
Email subscription

EventBridge scheduled rule
        |
        v
salesAnalysisReport
```

## What AWS Academy prepared for me

- Lab VPC and private subnets
- Amazon RDS database and its existing database security group
- IAM execution roles required by the challenge
- Lambda deployment ZIP files
- Temporary lab resources and data

## What I built/configured

- Created `LambdaSG` with no inbound rules and outbound access.
- Added a MySQL/Aurora TCP 3306 inbound rule to the database security group with `LambdaSG` as the source.
- Created `salesAnalysisReportDataExtractor` in the Lab VPC using both private subnets and `LambdaSG`.
- Uploaded the supplied Data Extractor deployment package and configured its handler.
- Created `salesAnalysisReport`, uploaded its deployment package, and configured its handler.
- Created the standard SNS topic `SalesReportTopic`.
- Configured the report Lambda with the SNS topic ARN through the `topicARN` environment variable.
- Created and confirmed an SNS email subscription.
- Tested `salesAnalysisReport` successfully.
- Created an enabled EventBridge **Scheduled Standard rule** targeting `salesAnalysisReport` and using the supplied scheduler role.
- Used CloudWatch Logs to troubleshoot Lambda execution.
- Answered the challenge knowledge questions and received a **full score**.

## Current-console differences observed

### Python runtime

The lab instructions referenced Python 3.11, but that runtime was not available in the current Lambda console during the lab. Python **3.14** was used successfully.

This is an important operational lesson: use the current supported runtime while preserving compatibility with the supplied code, rather than assuming an older lab screenshot still matches the console.

### Lambda console

Runtime and handler configuration were found under **Runtime settings** in the current Lambda console. Deployment-package changes were applied through the current console workflow rather than older save-menu instructions.

### Database security-group troubleshooting

Attempting to modify the existing MySQL rule produced an IAM authorization error. The intended task was to **keep the existing rule and add a second inbound rule**:

```text
Type: MYSQL/Aurora
Protocol: TCP
Port: 3306
Source: LambdaSG
```

That succeeded and preserved the existing database access rule.

### EventBridge: Scheduler vs scheduled rule

The current EventBridge console prominently offers **EventBridge Scheduler**, and a Scheduler schedule was initially created. However, the challenge specifically asked for an **EventBridge rule**.

The final graded implementation used:

```text
EventBridge
  -> Scheduled rules (legacy)
  -> Scheduled Standard rule
  -> Lambda target: salesAnalysisReport
```

This distinction matters when following older AWS Academy instructions or when a grader expects a particular AWS resource type.

## Troubleshooting evidence

### Lambda

A manual test of `salesAnalysisReport` completed successfully. CloudWatch Logs showed the normal Lambda lifecycle:

```text
START
...
END
REPORT
```

with no Lambda error or timeout.

### Scheduled reporting

CloudWatch Logs and EventBridge monitoring were used as the primary troubleshooting tools. The key exam lesson is:

> When a Lambda-based scheduled workflow is not producing the expected result, check the EventBridge schedule/rule and CloudWatch Logs before guessing at the cause.

### SNS observation

During the lab, the email subscription was observed as confirmed and later deactivated. The exact external cause was **not established**, so it should not be documented as a proven email-security or link-scanner issue.

The lab's core resources and configuration received a full score.

## Challenge questions — learning takeaways

- A Lambda deployment ZIP can contain a `package` folder because third-party Python packages/dependencies used by the function may be bundled there.
- A Lambda function that must communicate directly with an RDS database in a VPC needs appropriate VPC networking and security-group access.
- An SNS topic ARN can technically be stored elsewhere and retrieved by code, but an ARN itself is normally an identifier rather than a secret.
- An unconfirmed SNS email subscription does not receive topic notifications.
- For missing scheduled email reports, **review Amazon CloudWatch Logs for errors**.

## SAA-C03 takeaways

1. **Lambda + RDS networking** — Lambda needs VPC connectivity when accessing private RDS resources.
2. **Security groups reference security groups** — allowing `LambdaSG` on database port 3306 is more precise than opening the database broadly.
3. **Lambda is event driven** — a function can be invoked manually, by another AWS service, or on a schedule.
4. **SNS is pub/sub notification** — publishers send to a topic; confirmed subscribers receive notifications.
5. **EventBridge can schedule work** — scheduled rules can invoke Lambda without a continuously running server.
6. **CloudWatch Logs are primary Lambda troubleshooting evidence** — inspect execution logs for errors and timeouts.
7. **IAM roles define service permissions** — the scheduling service must be allowed to invoke its Lambda target.

## 3rd-grade analogy

Imagine a café that prepares a sales report every evening:

- **RDS** = the café's sales notebook.
- **Data Extractor Lambda** = the worker who reads numbers from the notebook.
- **Report Lambda** = the worker who turns the numbers into a daily report.
- **SNS** = the mailroom that sends the report.
- **Email subscription** = the approved recipient address.
- **EventBridge scheduled rule** = the alarm clock that tells the report worker when to start.
- **CloudWatch Logs** = the work diary showing what happened when something goes wrong.
- **Security Group** = the guard deciding who may enter the database room.

## Completion checkpoint

### Class Lab Complete

- Challenge submitted successfully.
- Full score received.

### Personal AWS Rebuild Decision

**Yes — High Learning Value.**

A future personal-account rebuild should create the complete architecture from zero rather than relying on Academy-prepared VPC, database, roles, or deployment packages. Use the newest AWS Console GUI available at rebuild time and explicitly document any differences from the Academy material.

### Cleanup

AWS Academy lab resources are temporary. No temporary account IDs, ARNs, IP addresses, endpoints, email addresses, or other lab-specific identifiers are recorded in this repository.
