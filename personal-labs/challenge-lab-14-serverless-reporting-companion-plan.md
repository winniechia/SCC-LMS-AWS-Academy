# Challenge Lab 14 — Personal Serverless Reporting Companion Plan

## Decision

**🟢 Yes — High Learning Value; rebuild from zero in a personal AWS account.**

The classroom challenge depended on Academy-prepared networking, database, roles, data, and deployment packages. A personal rebuild should recreate every important dependency to prove the architecture is understood rather than merely configured.

## Goal

Build a small scheduled reporting system:

```text
Private RDS
   |
   v
Data-extraction Lambda
   |
   v
Report Lambda -> SNS -> confirmed email
   ^
   |
EventBridge scheduled rule
```

## Build from zero

1. Create a VPC with at least two private subnets.
2. Create an RDS database in private networking.
3. Create separate Lambda and database security groups.
4. Allow the database port only from the Lambda security group.
5. Create least-privilege Lambda execution roles.
6. Write a small data-extraction Lambda.
7. Write a report-generation Lambda.
8. Create an SNS topic and confirmed test subscription.
9. Create an EventBridge scheduled rule targeting the report Lambda.
10. Use CloudWatch Logs to prove each stage works.
11. Intentionally break one dependency at a time and diagnose it from evidence.
12. Delete resources after the exercise to control cost.

## Required learning experiments

- Remove the DB inbound rule and observe the connection failure.
- Restore the SG-to-SG rule and verify recovery.
- Give EventBridge an insufficient role and observe invocation failure.
- Use a near-future cron time and prove the scheduled invocation in CloudWatch.
- Compare EventBridge **Scheduler** with a classic **scheduled rule** and record when each is appropriate.
- Confirm and then intentionally leave an SNS subscription unconfirmed to observe delivery behavior.

## Modern-console rule

Use the **newest AWS Console GUI available at rebuild time**. Do not copy stale Academy screenshots mechanically. Record console/runtime differences explicitly.

## Security and cost guardrails

- Do not make RDS publicly accessible just to simplify the lab.
- Do not use `0.0.0.0/0` for database inbound access.
- Do not commit secrets, database passwords, account IDs, ARNs, endpoints, or email addresses.
- Use small/free-tier-eligible resources where appropriate.
- Clean up EventBridge, Lambda, SNS, RDS, networking, and log resources when finished.

## Completion proof

The rebuild is complete only when:

- Lambda can read from the private database.
- The report Lambda succeeds.
- EventBridge automatically invokes the report Lambda.
- CloudWatch contains the scheduled invocation.
- SNS delivery works to a confirmed subscriber.
- A deliberate failure can be diagnosed from CloudWatch/IAM/network evidence.
