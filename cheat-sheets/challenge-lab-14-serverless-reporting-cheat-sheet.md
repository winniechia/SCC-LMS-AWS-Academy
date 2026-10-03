# Challenge Lab 14 — Serverless Daily Sales Reporting Cheat Sheet

## Core flow

```text
RDS -> Data Extractor Lambda -> Report Lambda -> SNS -> Email
                                      ^
                                      |
                         EventBridge scheduled rule
```

## Fast recall

| Service | Job | Memory cue |
| --- | --- | --- |
| Amazon RDS | Stores café sales data | Sales notebook |
| Lambda | Extracts/processes report data | On-demand worker |
| Security Group | Controls RDS network access | Guard |
| SNS | Publishes report notification | Mailroom |
| EventBridge | Runs report on schedule | Alarm clock |
| CloudWatch Logs | Shows execution/error evidence | Work diary |
| IAM role | Grants AWS-service permissions | Employee badge |

## Critical configuration

**Database access**

```text
DB security group inbound:
MYSQL/Aurora
TCP 3306
Source = LambdaSG
```

Do not open the database broadly when a security-group reference can identify the allowed caller.

**Data Extractor Lambda**

- Private subnets
- `LambdaSG`
- Supplied execution role
- 128 MB
- 30-second timeout
- Handler points to the supplied extractor module

**Report Lambda**

- Supplied execution role
- 128 MB
- 30-second timeout
- `topicARN` environment variable points to the SNS topic

**SNS**

- Standard topic: `SalesReportTopic`
- Email subscriber must confirm subscription before receiving notifications

**EventBridge**

- Challenge expects an **EventBridge Scheduled Standard rule**
- Target: `salesAnalysisReport`
- Use the supplied scheduler role
- Cron schedules are evaluated in UTC in this lab workflow

## Current-console lesson

Academy instructions can lag behind AWS Console changes.

Observed during this lab:

- Instructions referenced Python 3.11; current console required a newer available runtime and Python 3.14 worked.
- Current EventBridge UI promotes **Scheduler**, but the challenge asked for a **scheduled rule**.
- For grader-sensitive labs, match the requested AWS resource type, not merely a functionally similar newer service.

## Troubleshooting order

```text
Expected email missing
  -> Did EventBridge trigger?
  -> Did Lambda run?
  -> Check CloudWatch Logs
  -> Check Lambda errors/timeouts/config
  -> Check SNS topic/subscription
  -> Check IAM permissions
```

For RDS connection errors:

```text
Lambda VPC/subnets
  -> LambdaSG
  -> DB security-group inbound TCP 3306
```

## SAA-C03 triggers

- **Lambda cannot reach private RDS** -> VPC/subnet/security-group path.
- **Scheduled Lambda** -> EventBridge.
- **Lambda application error** -> CloudWatch Logs.
- **Fan-out / notification** -> SNS.
- **Service cannot invoke target** -> IAM permissions/role.
- **Email subscription unconfirmed** -> no SNS email delivery.

## 3rd-grade analogy

> EventBridge is the alarm clock. Lambda is the worker. RDS is the notebook. SNS is the mailroom. CloudWatch Logs are the worker's diary.

## Result

**Challenge Lab 14: COMPLETE — Full score.**
