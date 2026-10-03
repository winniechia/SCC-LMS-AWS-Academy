# Lab 14 — Serverless Architecture Cheat Sheet

[Full notes](../labs/lab-14-serverless-architecture.md) | [Study index](../README.md) | [Personal rebuild plan](../personal-labs/lab-14-serverless-architecture-companion-plan.md)

## Core architecture

```text
CSV -> S3 -> Load-Inventory Lambda -> DynamoDB
                                      |
                                      v
                                DynamoDB Stream
                                      |
                                      v
                               Check-Stock Lambda
                                      |
                               Count == 0
                                      |
                                      v
                                     SNS
```

## Fast SAA recognition

| Exam clue | Think |
| --- | --- |
| Run code without managing servers | Lambda |
| Object uploaded to S3 should run code | S3 Event Notification + Lambda |
| React to DynamoDB item changes | DynamoDB Streams + Lambda |
| Send notification to subscribers | SNS |
| Browser/app needs AWS credentials | Cognito |
| Serverless troubleshooting | Follow event chain and inspect logs |

## One-line mental model

> **Event -> Function -> Action**

## Service memory

- **S3 = 收貨箱**
- **S3 Event = 門鈴**
- **Lambda = event 來才工作的 AWS 店員/廚師**
- **DynamoDB = 庫存記錄簿**
- **DynamoDB Stream = 變更通知**
- **SNS = 廣播員**
- **Cognito = 臨時通行證櫃台**

## Lab functions

```text
Load-Inventory
S3 event -> read CSV -> write DynamoDB

Check-Stock
DynamoDB Stream -> inspect Count -> if 0 -> publish SNS
```

## Troubleshooting evidence

### CSV problem

```text
S3 trigger invoked Lambda
 -> Unicode decoding error while reading file
 -> DynamoDB remained empty
```

Conclusion: trigger worked; input processing failed. Original lab CSV worked.

### Missing email

```text
Dashboard updated                    OK
DynamoDB Stream invoked Check-Stock  OK
Count == 0 detected                  OK
Lambda completed                     OK
SNS email subscription               deactivated/deleted
```

Do not blame an earlier stage after logs have already proved it works.

## Current-console difference

Lab guide: Python 3.8 and older File -> Save workflow.

Lab execution: Python 3.14 was available/used; code was deployed with the current **Deploy** workflow.

## Module 13 vs Module 14

```text
SQS    = work can wait
Lambda = code runs on event
SNS    = notification/fan-out
```

## Personal rebuild

**Yes — Very High Learning Value.**

Rebuild from zero: S3, DynamoDB + Stream, IAM, both Lambda functions, SNS, permissions, logging, tests, cleanup.
