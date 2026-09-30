# Lab 13 — Decoupled Applications with Amazon SQS Cheat Sheet

[Full notes](../labs/lab-13-decoupled-applications-sqs.md) | [Study index](../README.md) | [Personal rebuild plan](../personal-labs/lab-13-decoupled-applications-sqs-companion-plan.md)

## Core architecture

```text
Upload -> S3 -> SNS -> SQS -> App Server -> process -> S3
                    |
                    +--> Email
```

## Core split

| Service | Think |
| --- | --- |
| S3 | object storage + event source |
| SNS | publish / subscribe / fan-out |
| SQS | queue / buffer / decouple |
| App Server | consumer / worker |
| DynamoDB | application metadata/status |

## Fast SAA recognition

| Exam clue | Think |
| --- | --- |
| Decouple application components | SQS |
| Buffer asynchronous work | SQS |
| Producer faster than consumer | SQS |
| Publish one event to multiple subscribers | SNS |
| Fan-out with independent buffering | SNS + SQS |
| Trigger workflow when object is created | S3 Event Notification |
| Consumer does not process queue | Check consumer config/permissions/logs |
| Config file changed but app still uses old value | Restart/reload process if required |

## Lab message path

```text
S3 object created
      |
      v
SNS topic
   /     \
  v       v
SQS      Email
  |
  v
App Server polls
  |
  v
Process image
  |
  v
Save adjusted image to S3
```

## Successful image status

```text
1. Sent to S3
2. App server received image URL
3. App server got original image buffer
4. Processed image
5. Saved adjusted to S3
6. Complete
```

## Troubleshooting evidence from the lab

### Wrong SNS topic setup

```text
uploadnotification.fifo  -> mistaken lab setup
uploadnotification       -> intended lab topic
```

### Invalid SQS URL

Observed app-server error:

```text
ERR_INVALID_URL
https://[Phase2 SQS URL]
```

Fix pattern:

```text
Get actual queue URL
 -> update app_server_2 config
 -> save
 -> restart Node process
 -> Poll SQS
 -> verify processed image
```

## 3rd-grade memory

> **S3 = 收件櫃台**

> **SNS = 廣播員**

> **SQS = 待辦工作箱**

> **App Server = 廚師**

> **Poll SQS = 廚師去工作箱拿訂單**

## One-line memory

> **SNS spreads the message; SQS holds the work.**

> **SNS 負責廣播；SQS 負責讓工作安全排隊。**

## Completion evidence

- S3 event reached SNS
- SQS subscribed to SNS
- App server used the real SQS queue URL
- Polling consumed queued work
- Image was processed
- Adjusted 300x300 tinted image displayed
- Temporary AWS identifiers intentionally omitted
