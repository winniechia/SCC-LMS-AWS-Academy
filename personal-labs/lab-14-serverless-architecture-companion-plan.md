# Personal Companion Lab 14 — Serverless Inventory Workflow From Zero

[Study index](../README.md) | [Class Lab 14 notes](../labs/lab-14-serverless-architecture.md) | [Lab 14 cheat sheet](../cheat-sheets/lab-14-serverless-architecture-cheat-sheet.md)

## Status

**Planned only — do not build automatically.**

## Decision

**🟢 Yes — Very High Learning Value**

AWS Academy supplied important scaffolding such as roles, table/dashboard components, Cognito configuration, sample data, and starter code. The personal rebuild should test the missing skill: design and build the serverless event chain **from zero**.

## Target architecture

```text
CSV -> S3 -> Lambda -> DynamoDB -> DynamoDB Stream -> Lambda -> SNS
```

Build a minimal verification client/query rather than reproducing the Academy dashboard unless the UI itself adds learning value.

## Phase 0 — Cost and safety

- choose one Region deliberately;
- use non-production names and tags;
- define cleanup before creation;
- never reuse Academy credentials, account IDs, ARNs, bucket names, email addresses, or temporary identifiers;
- check current Lambda runtime support and AWS pricing at rebuild time.

## Phase 1 — DynamoDB from zero

Create the inventory table yourself.

Define a key design suitable for store/item inventory and explain why it supports the access pattern.

Enable DynamoDB Streams with the view type required by the stock-check function.

Checkpoint:

> Can I explain what a DynamoDB Stream record represents?

## Phase 2 — S3 event source

Create a private S3 bucket.

Configure an ObjectCreated event to invoke the ingest Lambda only after the Lambda and permissions are ready.

Test with a tiny UTF-8 CSV.

## Phase 3 — IAM from zero

Create least-privilege execution roles rather than using Academy-provided roles.

Ingest Lambda needs only the permissions required to read the uploaded object, write inventory records, and write logs.

Stock-check Lambda needs only the permissions required for its stream/event-source operation, SNS publication, and logging.

Checkpoint:

> Role = Lambda's employee badge. Policy = what the badge permits.

## Phase 4 — Write Load-Inventory from zero

Write a small Lambda function that:

1. receives the S3 event;
2. identifies bucket/key;
3. reads the object safely;
4. parses CSV;
5. validates required fields;
6. writes inventory records to DynamoDB;
7. logs useful non-sensitive evidence.

Test malformed input as well as valid input.

## Phase 5 — DynamoDB Stream to Check-Stock

Write the second Lambda independently.

It should:

1. process stream records;
2. handle the event types intentionally;
3. inspect the new inventory count;
4. detect zero stock;
5. publish an alert to SNS;
6. avoid duplicate side effects where practical.

## Phase 6 — SNS notification

Create the SNS topic and a test subscriber.

First test SNS directly. Then test the complete Lambda publication path.

If email delivery behaves unexpectedly, distinguish:

```text
publish failed
subscription inactive
email delivery/filtering issue
```

Do not change upstream resources until evidence points upstream.

## Phase 7 — Observability drill

Use CloudWatch logs to prove each boundary:

```text
S3 event received?
CSV parsed?
DynamoDB write succeeded?
Stream record received?
Count == 0 detected?
SNS publish succeeded?
```

Repeat the Academy CSV-encoding lesson safely by uploading a deliberately invalid-encoding test file and identifying the first failure from logs.

## Phase 8 — Concurrency/scaling thought experiment

Upload several inventory files close together and observe Lambda invocations.

Explain:

- Lambda concurrency;
- event-driven scaling;
- why serverless does not mean unlimited capacity;
- service quotas and downstream capacity;
- why idempotency matters when events can be retried.

## Phase 9 — SAA reconstruction test

Without notes, draw and explain:

```text
S3 ObjectCreated
 -> Lambda
 -> DynamoDB
 -> DynamoDB Stream
 -> Lambda
 -> SNS
```

Then answer:

- Why Lambda instead of an always-running EC2 instance?
- Why DynamoDB Streams rather than polling the table?
- What permissions does each function need?
- Where would SQS fit if work needed durable buffering?
- How would you troubleshoot a missing notification?

## Cleanup

Delete all temporary personal-lab resources:

- S3 objects/bucket;
- Lambda functions;
- DynamoDB table;
- event source mappings/triggers;
- SNS topic/subscriptions;
- IAM roles/policies created for the exercise;
- CloudWatch log groups if no longer needed.

## Completion standard

The rebuild is complete only when I can:

- create every required resource without Academy scaffolding;
- explain each event boundary;
- write both minimal functions myself;
- use least-privilege IAM;
- prove the full workflow with logs;
- diagnose a controlled failure from evidence;
- explain where SQS would or would not belong;
- clean up all temporary resources.
