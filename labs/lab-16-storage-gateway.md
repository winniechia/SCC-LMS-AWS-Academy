# Lab 16 — Configuring Hybrid Storage and Migrating Data with AWS Storage Gateway S3 File Gateway

[Study index](../README.md) | [Quick review](../cheat-sheets/lab-16-storage-gateway-cheat-sheet.md) | [Personal rebuild plan](../personal-labs/lab-16-storage-gateway-companion-plan.md)

## Outcome / 成果

**AWS Academy Guided Lab: Completed — 40/40 (100%).** All eight grading checks received **5/5**: buckets, East/West versioning, CRR, gateway, file share, Linux file copy, and replication to West Region.

**Hands-on evidence:** Copied 20 PNG images from the supplied on-premises Linux server to an NFS mount, verified all 20 objects in the primary S3 bucket, and verified all 20 replicated to the secondary bucket. An earlier small test object also replicated successfully.

**Scope:** This is a completed **AWS Academy classroom lab**, not yet a deployment in the personal AWS account. Temporary resource identifiers, public IP addresses, and account-specific ARNs are omitted.

## Architecture / 架構

```text
On-premises Linux EC2 (provided by Academy)
  /media/data/{1..20}.png
          |
          | cp to /mnt/nfs/s3
          v
NFS mount (private network)
          |
          v
S3 File Gateway appliance (EC2 m5.xlarge)
  local EBS cache (150 GiB)
          |
          v
Primary S3 bucket — Virginia (us-east-1)
  Versioning enabled
          |
          | S3 Cross-Region Replication (CRR)
          v
Secondary S3 bucket — Oregon (us-west-2)
  Versioning enabled
          |
          v
20 PNG objects verified
```

**Two separate jobs / 兩個不同的工作:**
- **S3 File Gateway** bridges file-protocol access (NFS here; SMB is another supported option) to S3 object storage.
- **CRR** copies eligible S3 objects from one bucket to a bucket in another Region. CRR does **not** replace File Gateway or mount an NFS share.

## Academy-provided vs personally configured

**[AWS Academy provided]** A temporary lab account/environment, on-premises Linux server and 20 sample PNG files, VPC/subnet and security-group scaffolding, and lab-specific IAM roles.

**[I configured and verified]**
1. Created primary and secondary S3 buckets and enabled **Versioning** on both.
2. Configured **full-bucket CRR** with an existing classroom IAM replication role; verified a small test object appeared in the destination.
3. Launched a File Gateway EC2 appliance in the on-premises VPC: **m5.xlarge**, **80 GiB root EBS**, **150 GiB cache EBS**.
4. Activated **S3 File Gateway**, selected the cache disk, and created a read/write **NFS file share** mapped to the primary bucket.
5. Connected to the provided Linux server through the Academy instance session, inspected `/media/data`, and confirmed 20 PNG files.
6. Mounted the NFS share and copied the files.
7. Independently checked that all 20 objects reached primary S3 and then Oregon through CRR.
8. Submitted the lab and received **40/40**.

## Representative Linux commands

Use the **current gateway private IP** and **current share name**; the values below are illustrative, not reusable endpoints.

```bash
sudo su -l ec2-user
ls /media/data
sudo mkdir -p /mnt/nfs/s3
sudo mount -t nfs -o nolock,hard <GATEWAY_PRIVATE_IP>:/<SHARE_NAME> /mnt/nfs/s3
df -h
cp -v /media/data/*.png /mnt/nfs/s3
```

**Verification:** `df -h` showed the gateway's NFS export mounted at `/mnt/nfs/s3`. `cp -v` showed 20 successful file copies. **Copy success alone is not proof of completed S3 ingestion or CRR**; verify objects in both S3 buckets separately. The very large NFS capacity displayed by `df` is a virtual reported capacity, not actual allocated S3 storage.

## Troubleshooting: Region blocked by SCP

The original guide specified the primary bucket in **Ohio (us-east-2)** and the destination in **Oregon (us-west-2)**. In this Academy environment, an AWS Organizations **Service Control Policy (SCP)** explicitly denied `s3:CreateBucket` in Ohio.

**Resolution:** Used **Virginia (us-east-1)** for the primary bucket and kept **Oregon (us-west-2)** for the replica. Confirmed the end-to-end workflow and obtained **40/40**, so this particular lab grader accepted the adaptation.

**Lesson:** An explicit SCP deny cannot be overridden by adding IAM Allow permissions. Identify the actual policy boundary before troubleshooting an IAM role. An alternate Region is appropriate only when the exercise requirements and applicable organizational rules permit it.

## Concepts / 核心觀念

| English | 繁體中文 |
| --- | --- |
| Hybrid storage | 本地端系統透過 Gateway 使用雲端物件儲存。 |
| NFS file share | Linux 使用熟悉的資料夾路徑讀寫檔案。 |
| S3 File Gateway | 將檔案介面連接到 S3 物件；不等同於一般完整 POSIX 檔案系統。 |
| Cache disk | Gateway 的本地快取，有助於檔案存取與資料傳輸。 |
| S3 Versioning | 保留物件版本；是 S3 CRR 的必要設定之一。 |
| Cross-Region Replication | 自動將符合條件的 S3 物件複寫至另一個 Region；非同步完成。 |
| SCP explicit deny | 組織層級限制，IAM Allow 無法覆蓋。 |
| Verification | Linux 複製成功、來源 S3 存在、目的地 S3 存在，三者分開驗證。 |

## Third-grade analogy / 三年級比喻

A Linux computer is a classroom with photo folders. **NFS** is the familiar classroom doorway; **File Gateway** is the delivery counter; **S3 Virginia** is the first warehouse; **CRR** is a scheduled copy truck; **S3 Oregon** is the second warehouse.

Linux 電腦像教室，照片放在資料夾裡。**NFS** 是原本熟悉的門；**File Gateway** 是轉運站；**Virginia S3** 是第一座倉庫；**CRR** 是自動複製的貨車；**Oregon S3** 是第二座倉庫。

> **Gateway = local files to S3. CRR = S3 to S3. / Gateway 把本地檔案送到 S3；CRR 把 S3 物件複製到另一個 S3。**

## SAA-C03 recognition / 考試重點

- **On-premises file workloads + NFS/SMB + S3 + minimal app changes** → consider **S3 File Gateway**.
- **Replicate objects to another AWS Region** → **S3 Cross-Region Replication**; enable versioning on both buckets and configure replication permissions.
- **Linux mounts file share** → **NFS**; **Windows file share** scenarios often suggest **SMB**.
- **AccessDenied with explicit SCP deny** → check **AWS Organizations policy**; an IAM Allow cannot bypass the deny.
- **Data protection**: replication is not a complete backup strategy; understand versioning, deletion behavior, and recovery requirements.

## Cost awareness / 成本學習

The classroom lab used **m5.xlarge**, **80 GiB root EBS**, and **150 GiB cache EBS**. Copying **one** picture instead of 20 reduces data usage but **does not reduce the running EC2 instance cost or provisioned EBS capacity**. A personal rebuild also needs a Linux NFS client, networking, S3 buckets, and appropriate permissions.

**Stop vs delete:** Stopping EC2 stops instance compute charges, but attached EBS volumes can still incur storage charges. A running NAT Gateway and allocated public IPv4 addresses can incur separate charges. S3 objects/versions, requests, and replication can also incur charges. Gateway service usage may have additional charges. Check current regional pricing and credit eligibility before starting.

**Before leaving the lab:** inspect EC2 instances (both gateway and Linux client), EBS volumes/snapshots, Storage Gateway and file shares, S3 objects and versions in **both Regions**, replication rules, public IPv4/Elastic IPs, NAT Gateways, CloudWatch logs, and lab-created IAM/networking resources. **Never delete a mounted or active gateway prematurely**; unmount/finish writes, verify data, and follow the service's supported gateway deletion sequence. Confirm cleanup in the console and later in billing; billing data may lag.

## Personal AWS rebuild decision

**YES — planned for later, not started.** Rebuild the complete architecture in a personal AWS account using **one small test image**, explicitly practicing cost visibility, budget alerts, and resource cleanup. Credits are available but the remaining balance and applicability have **not** been checked yet. Budget alerts are not real-time hard spending caps.

See the [personal companion plan](../personal-labs/lab-16-storage-gateway-companion-plan.md) for the phased rebuild and shutdown checklist.
