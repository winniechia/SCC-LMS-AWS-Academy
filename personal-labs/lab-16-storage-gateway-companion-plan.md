# Personal Companion Lab 16 — S3 File Gateway Hybrid Storage From Zero

[Study index](../README.md) | [Completed Academy lab](../labs/lab-16-storage-gateway.md) | [Quick review](../cheat-sheets/lab-16-storage-gateway-cheat-sheet.md)

## Status and intent

**PLANNED — not deployed. Return later.** The AWS Academy version was completed with **40/40**. The personal rebuild is approved in principle because AWS Credits may help offset eligible charges. **Do not create resources until the user resumes and confirms cost safeguards.**

**Learning goals:** (1) independently build the full hybrid architecture without Academy scaffolding; (2) test with **one small non-sensitive image**; (3) understand *which resources keep billing after a session*; (4) practice a verifiable teardown.

## Architecture

```text
Personal AWS account
Linux EC2 NFS client
   | mount + copy 1 image
   v
S3 File Gateway appliance on EC2 (suitable xlarge instance)
   | 150 GiB cache EBS + root EBS per current requirements
   v
Primary S3 (Virginia us-east-1, versioning)
   | CRR
   v
Secondary S3 (Oregon us-west-2, versioning)
   | verify the 1 image
   v
Unmount -> supported gateway shutdown/delete -> remove lab resources -> verify billing
```

Use private NFS networking and least-privilege IAM. Prefer no NAT Gateway if the validated design can avoid it; do not expose NFS publicly. Do not copy temporary Academy roles or credentials.

## Phase 0 — Cost and credit safety (mandatory, before deployment)

- Check **Billing and Cost Management → Credits**: remaining credit, expiry, eligible services; do not assume every charge is covered.
- Check current **m5.xlarge regional On-Demand price**, EBS gp3 price for root/cache, Storage Gateway charges, S3 storage/requests/CRR/inter-Region transfer, public IPv4 and potential networking costs.
- Set a small experiment budget (proposed **$5 alert threshold**, subject to confirmation), with alerts at 50%, 80%, and 100%. **Budgets are delayed alerts, not a hard cap or automatic shutdown.**
- Tag all created resources consistently, e.g. `Project=aws-lab16-personal` and `Owner=learning`; use clearly temporary names.
- Record **start time, expected runtime, cost estimate, planned end time, and cleanup inventory**. Schedule sufficient time for teardown, not just setup.
- Avoid unnecessary NAT Gateway, Elastic IP, load balancer, idle instances, and large files. A single photo does **not** make m5.xlarge cheaper per hour.
- Use AWS Cost Explorer/Billing for follow-up; understand that charges and credit consumption can appear with a delay.

## Phase 1 — Network and Linux client

- Design VPC, subnets, route tables, security groups, and connectivity needed for the NFS client and gateway.
- Launch a small, appropriately supported Linux EC2 client with private NFS connectivity.
- Verify access method, basic Linux commands, and security-group restrictions.
- Record: **resource created / cost category / teardown action**.

## Phase 2 — S3 + replication

- Create uniquely named private source bucket in Virginia and destination bucket in Oregon.
- Enable versioning on both.
- Configure CRR with appropriate IAM role and full-bucket rule; upload a tiny test object and verify replication.
- Avoid accidental public bucket access. Check versioned object cleanup requirements.

## Phase 3 — Gateway appliance and file share

- Confirm current AWS Storage Gateway S3 File Gateway EC2 hardware and cache requirements in official documentation before launch.
- Deploy a supported appliance EC2 instance (planned **m5.xlarge**) with appropriate root disk and **150 GiB cache** if still required.
- Activate gateway, assign cache, configure a private NFS file share pointing to source S3, and restrict client access.
- Verify gateway/share health. **Do not assume stopping the appliance is the same as deleting the gateway resource.**

## Phase 4 — One-file migration

```bash
sudo mkdir -p /mnt/nfs/s3
sudo mount -t nfs -o nolock,hard <GATEWAY_PRIVATE_IP>:/<SHARE_NAME> /mnt/nfs/s3
df -h
cp -v <ONE_TEST_IMAGE>.png /mnt/nfs/s3/
```

- Verify the single image appears in primary S3.
- Wait for asynchronous CRR and verify the same image appears in secondary S3.
- Record evidence at each boundary. No need for 20 photos.

## Phase 5 — Cost-awareness exercise

Create a table during the session:

| Resource | Charges while active? | Charges after stopping EC2? | End-of-lab action |
| --- | --- | --- | --- |
| Gateway EC2 m5.xlarge | Instance compute while running | Instance compute stops; storage can remain | Follow gateway shutdown/delete procedure; terminate instance |
| Linux client EC2 | Instance compute while running | Storage can remain | Terminate instance |
| EBS root + cache | Provisioned storage billed | **Yes**, if volumes remain | Delete unneeded volumes; check termination settings |
| Public IPv4 / Elastic IP | Potential hourly charges | **Possibly** | Release unused addresses |
| NAT Gateway (only if needed) | Hourly + data processing | **Yes** | Delete |
| S3 source + replica | Storage/requests/transfer | **Yes**, if objects/versions remain | Delete all object versions and buckets |
| CRR rule | Replication-related charges when processing | Rules may remain | Disable/delete and confirm no pending transfers |
| Storage Gateway | Usage-dependent service charges | Depends on usage/resources | Delete share/gateway through supported procedure |
| Logs/snapshots | Storage charges possible | **Yes** | Remove lab-specific leftovers if not needed |

Prices, minimums, billing behavior, and credit applicability must be rechecked on rebuild day.

## Phase 6 — Mandatory checkout before leaving

**Do not end the session until the following have been verified:**

- [ ] Confirm the single test image arrived in both Regions; no unfinished writes.
- [ ] Unmount NFS client; follow current AWS guidance for safe gateway/file-share deletion and data flush.
- [ ] Remove NFS share and Storage Gateway resource using supported deletion workflow.
- [ ] Terminate **both** EC2 instances (gateway appliance and Linux client); check state is **terminated**, not just stopped.
- [ ] Inspect **EBS Volumes** in Virginia; delete orphaned root/cache volumes; inspect snapshots.
- [ ] Inspect **Public IPv4 / Elastic IPs**, NAT Gateways, and other networking components for ongoing charges.
- [ ] Disable/delete CRR rule and empty **all versions and delete markers** in both versioned S3 buckets; delete buckets when safe.
- [ ] Clean up lab-specific IAM roles/policies, log groups, and other intentionally created resources where appropriate.
- [ ] Check **both Regions**, resource tags, and Billing/Cost Explorer; expect billing delays.
- [ ] Record completion time and cleanup evidence. If any resource remains, explicitly record its owner, reason, and expected cost.

## Completion standard

Rebuild counts as completed only when (1) one image has traveled Linux → NFS → File Gateway → primary S3 → CRR → secondary S3; (2) the user can explain the cost drivers; and (3) the cleanup checklist is verified.

**Next action when returning:** Phase 0, Step 1 — inspect AWS Credits. **Nothing has been launched in the personal account yet.**
