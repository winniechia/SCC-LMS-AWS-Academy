# Personal Companion Lab 11 — CloudFormation From Zero

[Study index](../README.md) | [Class Lab 11 notes](../labs/lab-11-cloudformation.md) | [Lab 11 cheat sheet](../cheat-sheets/lab-11-cloudformation-cheat-sheet.md)

## Status

**Planned only — do not build automatically.**

## Decision

**🟢 Yes — High Learning Value**

AWS Academy supplied the CloudFormation templates in the guided lab. The personal rebuild should therefore test the missing skill: **design and write the infrastructure templates from zero**, without copying the Academy templates.

## Learning goal

Build a small layered architecture from an empty personal lab environment:

```text
Network Stack
├── VPC
├── Public Subnet
├── Internet Gateway
├── Route Table / Route
└── Outputs + Exports
          |
          | ImportValue
          v
Application Stack
├── Security Group
└── Small EC2 instance
```

Then update the application stack, review the proposed change, and test deletion/retention behavior.

## Phase 0 — Cost and safety

Before creating resources:

- choose one Region intentionally;
- use a small non-production configuration;
- define naming/tags;
- set a cleanup deadline;
- never copy Academy credentials, IDs, endpoints, or temporary resources;
- check current AWS pricing and Console behavior at rebuild time.

## Phase 1 — Write the network template

Create a YAML template from zero that defines:

- VPC
- public subnet
- Internet Gateway and attachment
- public route table
- default route
- subnet association
- useful Outputs
- exported VPC and subnet values

Checkpoint:

> Can I explain every `Resource`, `Property`, `!Ref`, and Output without looking at the Academy template?

## Phase 2 — Deploy and inspect the network stack

Deploy the template and verify:

- stack reaches `CREATE_COMPLETE`;
- Resources match the intended design;
- Events show the deployment sequence;
- Outputs expose the intended values;
- exports are available for the application layer.

## Phase 3 — Write the application template

Create a second YAML template from zero with:

- `NetworkStackName` parameter;
- Security Group;
- small EC2 instance;
- imports of the network VPC/subnet values;
- useful application Output.

Use cross-stack references rather than hard-coded VPC/subnet IDs.

Checkpoint:

```text
Network Stack exports
        ->
Application Stack imports
```

## Phase 4 — Practice intrinsic functions

Use and explain:

- `!Ref`
- `!GetAtt`
- `!Sub`
- `Fn::ImportValue`

Checkpoint: explain each function in plain language and identify what value it produces in the template.

## Phase 5 — Update safely

Modify the application template in a controlled way, such as adding a Security Group rule.

Before execution:

1. review the proposed change;
2. identify Modify vs Replacement behavior;
3. predict impact;
4. execute only after the prediction is understood;
5. verify `UPDATE_COMPLETE`.

Checkpoint:

> Change Set = preview; it does not replace architecture judgment.

## Phase 6 — Explore with Infrastructure Composer

Open both templates and verify that the visual resource relationships match the intended YAML.

Use Composer to reinforce understanding, not to avoid learning the template structure.

## Phase 7 — DeletionPolicy experiment

Add a small stateful resource appropriate for the exercise and deliberately test a safe deletion policy such as snapshot/retain behavior where supported.

Before deletion, predict:

- what CloudFormation will delete;
- what should remain;
- what evidence will prove the policy worked.

## Phase 8 — Troubleshooting drills

In the isolated personal lab only:

- introduce one safe template validation/configuration error;
- inspect Stack Events;
- identify the first causal failure;
- correct it without random changes;
- redeploy and verify.

Goal: troubleshoot from evidence.

## Phase 9 — SAA reconstruction test

Without notes, explain:

```text
Template -> Stack -> Resources
Parameters -> deployment inputs
Outputs/Export -> values exposed
ImportValue -> cross-stack dependency
Change Set -> update preview
Events -> troubleshooting evidence
DeletionPolicy -> stateful deletion behavior
```

Also explain:

> CloudFormation = infrastructure deployment automation.

> Auto Scaling = runtime capacity automation.

## Cleanup

Cleanup is part of the exercise.

Verify all temporary resources and retained artifacts intentionally:

- application stack;
- network stack;
- EC2 instance;
- EBS volumes/snapshots if created;
- Security Groups;
- networking resources;
- any retained resource created by the deletion-policy test.

Do not assume stack deletion removes resources configured to remain.

## Completion standard

The rebuild is complete only when I can:

- write both templates without copying Academy templates;
- deploy the network and application layers successfully;
- explain cross-stack exports/imports;
- predict Change Set impact before execution;
- troubleshoot from Events;
- explain deletion-policy behavior;
- remove all temporary personal-lab resources safely.
