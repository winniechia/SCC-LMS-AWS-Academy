# Personal AWS Rebuild — Lab 16 Hybrid Storage + S3 Lifecycle

**Status:** PLANNED ONLY — not deployed  
**Scope:** A personal learning lab, **not** a claim about the AWS Academy lab instructions or grader.

## Architecture / 架構

- **us-east-1 (N. Virginia):** Linux EC2 NFS client + S3 File Gateway appliance.
- **us-east-2 (Ohio):** versioned primary/source S3 bucket.
- **us-west-2 (Oregon):** versioned secondary/destination S3 bucket.
- Data flow: Linux → NFS → File Gateway → Ohio S3 → CRR → Oregon S3.
- Keep all **three** Regions. Do not use a two-Region shortcut.

## Proposed build sequence / 個人實驗步驟

1. Set a cost budget and alerts; plan teardown. Check EC2, EBS, Storage Gateway, S3 storage, requests, and replication transfer costs before provisioning.
2. Verify permissions for EC2, IAM, Storage Gateway, S3, replication, and lifecycle actions in each Region.
3. Create Ohio source and Oregon destination buckets; enable versioning and default encryption; keep Block Public Access on.
4. Configure least-privilege replication permissions and **CRR Ohio → Oregon**. Upload a harmless test object and verify its replica.
5. Deploy and activate S3 File Gateway in Virginia, with appropriately restricted NFS network access and cache disk.
6. Create an NFS file share targeting the Ohio source bucket, using a scoped IAM role. Mount from the Linux instance using the **console-generated** export path.
7. Copy a small test dataset; verify objects in Ohio and replicated copies in Oregon. Record timestamps and screenshots.

## Required addition: S3 Lifecycle Policy / 必須加入生命週期規則

The AWS Academy lab **mentions lifecycle in its objectives but does not give a corresponding procedure**. Our personal lab **must** include an explicit, independently designed lifecycle task:

8. Create a **dedicated test prefix**, such as `lifecycle-demo/`, in the Ohio source bucket; do not apply an experimental expiration rule to all objects.
9. Create a lifecycle rule **scoped only to `lifecycle-demo/`**. For a demonstrative policy, choose a transition for current objects after **30 days** to a compatible storage class, and optionally expire noncurrent versions after a separately chosen retention period. Confirm current AWS minimum storage-duration charges and lifecycle eligibility before saving.
10. Validate that the rule is **Enabled** and that its prefix/filter and actions match the design. **Do not claim the transition has executed immediately**; lifecycle actions are asynchronous and require time.
11. Document the interaction between **versioning, replication, and lifecycle**: lifecycle configuration does not automatically copy to the destination bucket; destination lifecycle behavior must be designed separately. Test and document replication of new objects independently from lifecycle transitions.
12. Record expected behavior, cost implications, screenshots, and a clean-up plan. Remove test resources to avoid ongoing charges.

### Lifecycle learning goals

- Distinguish **Transition** from **Expiration**.
- Distinguish **current object versions** from **noncurrent versions**.
- Explain why versioning and CRR do not replace lifecycle management.
- Explain why retention/expiration decisions must be explicit and scoped.

## Third-grade analogy / 三年級比喻

CRR is a delivery truck that makes a backup copy in another city. Lifecycle is the warehouse rulebook: after enough time, move boxes to a cheaper shelf, or dispose of approved old boxes. A truck and a rulebook solve different problems.

## Validation and checkpoints

- [ ] Three-Region architecture verified.
- [ ] Both S3 buckets versioned.
- [ ] CRR tested with a new object.
- [ ] NFS mount and data migration verified.
- [ ] **Lifecycle rule configured, scope reviewed, and enabled.**
- [ ] Lifecycle behavior documented without claiming immediate execution.
- [ ] SAA-C03 notes and screenshots saved.
- [ ] Resources removed / cost check completed.

**Note:** The lifecycle policy above is a **proposed personal-lab design**, not a correction to the official AWS Academy instructions. Confirm instructor guidance for the original lab separately.
