# Lab 16 — S3 File Gateway + CRR Cheat Sheet

[Full notes](../labs/lab-16-storage-gateway.md) | [Study index](../README.md) | [Personal rebuild](../personal-labs/lab-16-storage-gateway-companion-plan.md)

**AWS Academy score: 40/40 (100%).** Verified 20 PNG files in primary Virginia S3 and replicated Oregon S3.

## One-line architecture

```text
Linux -> NFS -> S3 File Gateway (EC2 + EBS cache) -> S3 Virginia -> CRR -> S3 Oregon
```

## SAA-C03 clues

| Question clue | Answer |
| --- | --- |
| On-prem Linux file interface to S3 | S3 File Gateway + NFS |
| Windows file sharing to S3 | S3 File Gateway + SMB |
| Automatically copy S3 objects across Regions | S3 Cross-Region Replication |
| CRR prerequisites | Versioning enabled on source and destination; replication role/permissions |
| IAM Allow but explicit organization deny | SCP explicit Deny wins |
| `cp` succeeds but objects not in destination yet | Verify S3 upload separately; CRR is asynchronous |
| Stopped EC2 but still billed | EBS storage, public IPv4, S3, NAT Gateway, other resources |

## Hands-on checkpoints

1. NFS mount visible in `df -h`.
2. `cp -v` confirms files copied to mounted directory.
3. Primary S3 contains **all 20** PNG objects.
4. Secondary S3 contains **all 20** replicated PNG objects.
5. Academy grade **8 x 5/5 = 40/40**.

## Three-year-old / 三年級比喻

**Gateway = transfer counter; S3 = warehouse; CRR = copy truck.**

**Gateway 是轉運站，S3 是倉庫，CRR 是複製貨物的卡車。**

## Academy-specific troubleshooting

Original primary Region Ohio (`us-east-2`) blocked by SCP `s3:CreateBucket` explicit deny. Primary changed to Virginia (`us-east-1`); replica remained Oregon (`us-west-2`). Workflow verified and grader awarded full points.

## Personal rebuild decision

**YES, LATER — full hybrid rebuild, 1 picture, cost monitoring + complete cleanup.** Not yet deployed. One photo saves data-transfer/storage usage, **not** m5.xlarge hourly or EBS provisioned-capacity costs. Verify current prices and AWS Credits before deployment.
