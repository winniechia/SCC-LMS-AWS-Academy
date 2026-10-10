# Lab 16 — S3 File Gateway, Hybrid Storage, and Cross-Region Replication

**Date:** 2026-10-10  
**Status:** BLOCKED — AWS Academy SCP restriction; not completed  
**Source:** AWS Academy Guided Lab: *Configuring Hybrid Storage and Migrating Data with AWS Storage Gateway S3 File Gateway*.

## Intended architecture / 原始架構

- **us-east-1 (N. Virginia):** simulated on-premises Linux EC2 server, NFS client, S3 File Gateway EC2 appliance.
- **us-east-2 (Ohio):** primary/source S3 bucket.
- **us-west-2 (Oregon):** secondary/destination S3 bucket.
- Data path: Linux files → NFS → S3 File Gateway → Ohio S3 → S3 Cross-Region Replication → Oregon S3.
- **Do not change the three-Region design without instructor approval.**

## Attempted work and observed evidence / 實作紀錄

- On-premises Linux EC2 server and pre-provisioned FileGatewayAccess security group were present in N. Virginia.
- Pre-created IAM roles `S3-CRR-Role` and a role containing `FgwRole` were observed.
- Destination bucket `lab-16-secondary-wch` was created in Oregon with versioning enabled.
- Attempted to create the source bucket in Ohio as explicitly required by Task 2, Step 6. AWS returned an **explicit deny in a Service Control Policy** for `s3:CreateBucket`.
- Retrying with another bucket name did not resolve the denial. A temporary bucket could be created in Virginia, but was deleted; **Virginia is not an approved substitute for Ohio in the original lab**.
- Instructor was contacted. No claim of completion, successful replication, NFS mount, or lifecycle configuration is made.

## Minimum correction review / 教材最小修正審查

| Location | Finding | Action |
| --- | --- | --- |
| Task 2, Step 6 | Required Ohio source bucket creation blocked by Organizations SCP | **Lab administrator action needed:** investigate permission for `s3:CreateBucket` in `us-east-2`. Not proven to be a typo. |
| Task 4, Step 46 | NFS mount command uses illustrative IP/export path | Use the **actual** mount command produced by the Storage Gateway console; do not paste the example literally. |
| Task 6, Step 57 | Destination is described as Virginia, contradicting Tasks 1–3 | Correct destination to **Oregon (`us-west-2`)**. |
| Lab overview/objectives | Includes S3 Lifecycle Policy, but Tasks 1–6 contain no lifecycle-rule creation procedure | Request instructor clarification; do not invent an official step. |

### Root cause and boundary

An AWS Organizations **SCP explicit deny** cannot be overridden by an IAM identity-policy allow. This is an authorization boundary, not a bucket naming problem. Only an authorized administrator can adjust the relevant organization-level policy, or provide an approved alternate lab environment. The exact SCP policy statement has not been inspected.

## Third-grade analogy / 三年級比喻

Think of three locations: an office in Virginia, the main storage room in Ohio, and the backup storage room in Oregon. The File Gateway is the delivery counter; NFS is the way the office hands over files; S3 replication automatically sends copies from Ohio to Oregon. A company-wide security rule locked the Ohio storage room. Giving the delivery person another key (IAM Allow) cannot override that company-wide lock (SCP explicit deny).

## SAA-C03 review / 考試重點

- **S3 File Gateway:** file-oriented hybrid access using NFS or SMB, backed by S3 objects.
- **S3 Versioning:** must be enabled on both source and destination buckets for S3 replication.
- **Cross-Region Replication (CRR):** source and destination S3 buckets are in different Regions; a replication IAM role is used.
- **SCP vs IAM:** explicit deny wins; IAM Allow cannot bypass an SCP.
- **S3 Lifecycle vs CRR:** lifecycle transitions/expires objects according to policy; replication copies objects between buckets. These are different capabilities.
- **Troubleshooting sequence:** confirm intended Region → capture exact error/action → distinguish permission issue from instructions issue → document → escalate.

## Completion checkpoint

- [x] Reviewed architecture and instructions.
- [x] Captured `s3:CreateBucket` SCP denial.
- [x] Created destination bucket in Oregon.
- [x] Documented two instruction clarifications and lifecycle-objective gap.
- [ ] Source bucket in Ohio — **blocked**.
- [ ] CRR configured and tested.
- [ ] S3 File Gateway activated and file share mounted.
- [ ] Migrated 20 PNG files and verified replication.
- [ ] Lifecycle rule — not specified in official task steps.

**Personal AWS Rebuild?** **Yes, later**, after defining a budget and clean-up plan. Preserve the original three-Region architecture and include a separate lifecycle-rule exercise in the personal version. See `personal-labs/lab-16-hybrid-storage-lifecycle-companion-plan.md`.
