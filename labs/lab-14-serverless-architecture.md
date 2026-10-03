# Lab 14 — Implementing a Serverless Architecture on AWS

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-14-serverless-architecture-cheat-sheet.md) | [Personal rebuild plan](../personal-labs/lab-14-serverless-architecture-companion-plan.md)

## Purpose and completed lab record

AWS Academy Guided Lab: **Implementing a Serverless Architecture on AWS**.

The lab implemented an event-driven inventory workflow without provisioning EC2 instances. Inventory CSV files uploaded to Amazon S3 invoked AWS Lambda, data was stored in Amazon DynamoDB, DynamoDB changes invoked a second Lambda function, and Amazon SNS was used for out-of-stock notifications. A supplied dashboard used Amazon Cognito to obtain permission to read inventory data from DynamoDB.

**Class lab: completed and submitted.**

Temporary account IDs, ARNs, bucket names, email addresses, endpoints, credentials, and other classroom identifiers are intentionally omitted.

## 1. Architecture

```text
Inventory CSV
    |
    v
Amazon S3
    |
    | ObjectCreated event
    v
Load-Inventory Lambda
    |
    v
DynamoDB Inventory
    |
    | DynamoDB Stream
    v
Check-Stock Lambda
    |
    | Count == 0
    v
Amazon SNS
    |
    v
Email subscription
```

Dashboard path:

```text
Static dashboard -> Amazon Cognito identity -> DynamoDB Inventory
```

## 2. What AWS Academy prepared for me

**[AWS Academy prepared this]**

The guided environment supplied classroom scaffolding including the Inventory DynamoDB table, Lambda execution roles, dashboard/application components, Cognito access configuration, sample inventory files, and starter Lambda code.

This means completing the guided lab is **not the same as building the complete architecture from zero**.

## 3. What I built/configured

**[I built/configured this]**

- Created the `Load-Inventory` Lambda function.
- Used the supplied `Lambda-Load-Inventory-Role`.
- Deployed the inventory-loading Python code.
- Created an S3 inventory bucket and configured an ObjectCreated event notification to invoke `Load-Inventory`.
- Uploaded lab inventory CSV files and verified inventory data in the dashboard/DynamoDB.
- Created the Standard SNS topic `NoStock`.
- Created and confirmed an email subscription.
- Created the `Check-Stock` Lambda function.
- Used the supplied `Lambda-Check-Stock-Role`.
- Deployed the stock-checking Python code.
- Added the DynamoDB Inventory stream as the trigger for `Check-Stock`.
- Uploaded additional store inventory and verified that the second Lambda detected an out-of-stock item.
- Submitted the AWS Academy lab.

## 4. Current-console differences from the lab guide

The lab guide referenced **Python 3.8**, but that runtime was not available in the current Lambda console during this lab. The functions were created with **Python 3.14**, and the supplied lab code executed successfully.

The older guide also referenced **File -> Save** in the Lambda editor. The current console workflow used **Deploy** after editing the function code.

### Lesson

Console interfaces and runtime availability change. Preserve the architectural requirement, then adapt the implementation to currently supported options when the classroom guide is outdated.

## 5. Event-driven serverless mental model

```text
Event -> Function -> Action
```

In this lab:

```text
S3 object created
 -> Lambda runs
 -> DynamoDB updated

DynamoDB item changed
 -> Lambda runs
 -> stock checked
 -> SNS notification when Count == 0
```

**Serverless does not mean there are no servers.** AWS manages the underlying compute infrastructure, while the application supplies code, permissions, and event triggers.

## 6. Why two Lambda functions?

The lab intentionally separated responsibilities:

- `Load-Inventory` = ingest inventory data.
- `Check-Stock` = evaluate stock changes and initiate alerts.

This keeps each function focused on one job and makes the workflow easier to maintain and troubleshoot.

> **One function, one clear responsibility. / 一個 Lambda，負責一個清楚的工作。**

## 7. DynamoDB Streams

DynamoDB Streams provided the change events that invoked `Check-Stock`.

The stock-checking code inspected each incoming record's new image and checked the numeric `Count`. When the count was zero, it constructed an out-of-stock message and published it to the SNS topic.

```text
DynamoDB change
      |
      v
DynamoDB Stream record
      |
      v
Check-Stock Lambda
      |
      v
Count == 0 ?
   yes -> SNS
```

## 8. Troubleshooting lesson 1 — CSV encoding

An initial uploaded file invoked `Load-Inventory`, but the Lambda log showed a Unicode decoding failure while Python attempted to read the file as UTF-8.

Important evidence:

- the Lambda had been invoked;
- therefore the S3 event notification was working;
- the failure occurred while reading the uploaded file;
- DynamoDB remained empty because processing stopped before the writes.

Using an original lab-provided inventory CSV resolved the problem and the dashboard populated correctly.

### Lesson

> **A Lambda error after invocation does not mean the trigger is broken. Find the first failing operation.**

## 9. Troubleshooting lesson 2 — prove each stage separately

After additional inventory files were uploaded, the dashboard displayed multiple stores but no alert email arrived.

CloudWatch logs for `Check-Stock` proved that:

- DynamoDB Streams invoked the function;
- the function received the inventory changes;
- it detected an item whose `Count` was zero;
- the function completed without an execution error.

This narrowed the investigation to the notification/subscription portion rather than S3, DynamoDB, or the Lambda trigger.

### Evidence-first path

```text
Dashboard updated
 -> Load path works

Check-Stock CloudWatch invocation exists
 -> DynamoDB Stream trigger works

Out-of-stock message appears in log
 -> business condition works

Email absent
 -> investigate SNS subscription/delivery
```

## 10. Troubleshooting lesson 3 — SNS email subscription deactivated

The SNS email subscription was successfully confirmed and received a subscription ID, but it was subsequently shown as deleted/deactivated and the expected inventory alert email was not delivered.

The exact external cause was **not established during the Academy lab**. The important troubleshooting conclusion is narrower: the Lambda processing path had already been proven by CloudWatch, while the email subscription was no longer active.

Do not misdiagnose this as a failure of the S3 -> Lambda -> DynamoDB -> DynamoDB Streams -> Lambda processing chain.

## 11. 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

Imagine a store:

- **S3 = 收貨箱** — stores send their inventory list here.
- **S3 Event = 門鈴** — a new file rings the bell.
- **Load-Inventory Lambda = 第一位店員** — reads the list and writes the numbers into the inventory book.
- **DynamoDB = 庫存記錄簿** — remembers how many items each store has.
- **DynamoDB Stream = 記錄簿的變更通知** — tells us something in the book changed.
- **Check-Stock Lambda = 第二位店員** — checks whether any count became zero.
- **SNS = 廣播員** — sends an alert to subscribers.
- **Cognito = 臨時通行證櫃台** — gives the dashboard permission to read the inventory.

The important idea:

> **Nobody has to keep an EC2 employee sitting at a desk waiting all day. An event rings the bell, Lambda comes to work, finishes the job, and stops.**

## 12. SAA-C03 takeaways

| Exam clue | Think |
| --- | --- |
| Run code without provisioning/managing servers | AWS Lambda |
| Run code when an S3 object is created | S3 Event Notification + Lambda |
| React to changes in DynamoDB items | DynamoDB Streams + Lambda |
| Notify subscribers | Amazon SNS |
| Event-driven processing | Event source -> Lambda -> action |
| Web client needs AWS resource access | Cognito + appropriate IAM permissions |
| Troubleshoot a serverless chain | Verify each event boundary with logs/evidence |

Fast recognition:

> **Event -> Lambda -> Action**

> **S3 upload -> Lambda**

> **DynamoDB change -> Stream -> Lambda**

> **Notification -> SNS**

## 13. Connection to Module 13

Module 13 focused on **decoupling and buffering**:

```text
S3 -> SNS -> SQS -> consumer
```

Module 14 focuses on **event-driven serverless execution**:

```text
S3 -> Lambda -> DynamoDB -> Stream -> Lambda -> SNS
```

SQS answers: **Where can work wait safely?**

Lambda answers: **What code should run when an event occurs?**

SNS answers: **Who should receive this notification?**

## 14. Lab Completion Checkpoint

- [x] Class Lab Complete
- [x] Lab submitted
- [x] `Load-Inventory` Lambda created/deployed
- [x] S3 ObjectCreated trigger configured
- [x] Inventory data loaded into DynamoDB
- [x] Dashboard displayed inventory
- [x] SNS topic created
- [x] Email subscription confirmation performed
- [x] `Check-Stock` Lambda created/deployed
- [x] DynamoDB Stream trigger configured
- [x] Multiple store inventory tested
- [x] Out-of-stock condition observed in CloudWatch
- [x] CSV encoding troubleshooting captured
- [x] SNS subscription deactivation captured without claiming an unproven cause
- [x] Current Lambda console/runtime differences captured
- [x] SAA-C03 takeaways captured
- [x] 3rd-grade analogy captured
- [x] Sensitive temporary AWS identifiers omitted

## 15. Personal AWS Rebuild Decision

**Yes — Very High Learning Value.**

A personal rebuild should create the serverless architecture from zero, including the pieces AWS Academy prepared in advance. It should independently create the DynamoDB table and stream, IAM roles/policies, S3 event configuration, Lambda functions, SNS notification path, and a minimal verification client or query path.

See [Lab 14 personal rebuild plan](../personal-labs/lab-14-serverless-architecture-companion-plan.md).
