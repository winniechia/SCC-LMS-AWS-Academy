# Personal Companion Lab 13 — Decoupled Image Processing From Zero

[Study index](../README.md) | [Class Lab 13 notes](../labs/lab-13-decoupled-applications-sqs.md) | [Lab 13 cheat sheet](../cheat-sheets/lab-13-decoupled-applications-sqs-cheat-sheet.md)

## Status

**Planned only — do not build automatically.**

## Decision

**🟢 Yes — Very High Learning Value**

AWS Academy supplied starter application code and classroom scaffolding. The personal rebuild should test the missing skill: create the event-driven architecture **from zero** in a personal AWS account and understand every policy, event, queue, and consumer dependency.

## Target architecture

```text
Upload object
    |
    v
S3 bucket
    |
    | ObjectCreated
    v
SNS topic
   /      \
  v        v
SQS       optional notification subscriber
  |
  v
Consumer / worker
  |
  v
Processed result
```

The personal version does not need to reproduce the Academy UI. A minimal image/text job is enough if it proves the architecture.

## Phase 0 — Cost and safety

Before building:

- choose one Region intentionally;
- use unique non-production names;
- define tags;
- set a cleanup deadline;
- never reuse Academy credentials, account IDs, ARNs, IPs, queue URLs, or temporary resource names;
- check current AWS pricing and service behavior at rebuild time.

## Phase 1 — Build the event source

Create an S3 bucket from zero.

Verify:

- object upload works;
- Block Public Access remains deliberate rather than being disabled casually;
- no broad public-read/write policy is added merely to imitate the classroom lab.

Checkpoint:

> Can I explain who needs access to the object and why?

## Phase 2 — Create SNS

Create an SNS topic intentionally.

Explain:

- why Standard is appropriate for the chosen integration;
- publisher vs subscriber;
- topic policy;
- what principal is allowed to publish.

Checkpoint:

> SNS = broadcast/distribution, not a work buffer.

## Phase 3 — Create SQS

Create an SQS queue from zero.

Configure the required queue policy/subscription relationship and subscribe it to SNS.

Verify that an SNS publication results in an SQS message.

Checkpoint:

> SQS = durable work waiting for a consumer.

## Phase 4 — Configure S3 event notification

Configure an object-created event from S3 to SNS.

Test with one object.

Verify the complete event path:

```text
S3 -> SNS -> SQS
```

Do not add the consumer until this path is independently proven.

## Phase 5 — Build a minimal consumer

Write a small consumer from zero using an AWS SDK.

It should:

1. long-poll SQS;
2. parse the event message;
3. identify the S3 object;
4. retrieve/process the object;
5. write a result if appropriate;
6. delete the SQS message only after successful processing.

Checkpoint:

> Receive != successful completion.

## Phase 6 — Visibility and failure experiment

In the isolated personal lab, deliberately make one job take longer or fail safely.

Observe:

- visibility timeout;
- message reappearance when work is not completed/deleted;
- why idempotent processing matters.

## Phase 7 — Add a DLQ

Create a dead-letter queue and redrive policy.

Generate a controlled poison/failing message and verify that repeated failures eventually move it to the DLQ.

SAA goal:

> Know the difference between buffering normal work and isolating repeatedly failing work.

## Phase 8 — Scaling thought experiment

Measure queue backlog and explain how worker capacity could scale based on queue depth.

Connect to Module 10:

```text
SQS backlog = unfinished orders
Auto Scaling = add/remove workers
```

Building an ASG is optional unless it adds learning value at rebuild time.

## Phase 9 — Troubleshooting drills

Practice evidence-first diagnosis:

- wrong queue URL;
- consumer lacks permission;
- S3 event not configured;
- SNS subscription missing;
- queue policy blocks delivery;
- worker receives but does not delete messages.

For each failure:

1. predict the symptom;
2. inspect logs/metrics/configuration;
3. identify the first causal failure;
4. fix only that cause;
5. retest end-to-end.

## Phase 10 — SAA reconstruction test

Without notes, explain:

```text
S3 event
 -> SNS distribution
 -> SQS buffering
 -> consumer polling
 -> visibility timeout
 -> successful processing
 -> message deletion
 -> DLQ for repeated failure
```

Also explain:

- Standard vs FIFO;
- short vs long polling;
- SNS vs SQS;
- fan-out;
- why decoupling improves resilience.

## Cleanup

Verify deletion of all temporary resources:

- S3 objects and bucket;
- SNS topic/subscriptions;
- SQS queue;
- DLQ;
- consumer compute/resources;
- IAM roles/policies created for the exercise;
- logs or other retained artifacts if no longer needed.

## Completion standard

The rebuild is complete only when I can:

- build S3 -> SNS -> SQS without Academy scaffolding;
- write a minimal consumer;
- explain every permission boundary;
- prove a message waits safely while the consumer is stopped;
- explain visibility timeout and deletion;
- demonstrate a controlled failure and DLQ behavior;
- troubleshoot from evidence;
- clean up every temporary resource.
