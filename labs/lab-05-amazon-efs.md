# Lab 05 — Introducing Amazon Elastic File System (Amazon EFS)

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-05-amazon-efs-cheat-sheet.md)

## Purpose and lab record

Learn to mount shared network storage on Linux, verify where writes go, benchmark with fio, and interpret CloudWatch throughput. Commands and measurements below come from the supplied hands-on record; explanations provide study context. No throughput mode, security group IDs, or unrecorded setup results are assumed.

| Resource | Recorded value |
| --- | --- |
| EFS file system | My First EFS File System |
| File system ID | `fs-092652c2d9fc2a238` |
| Region | `us-east-1` |
| EC2 instance | EFS Client |
| Instance ID | `i-013d5d878ffb60330` |

## 1. Picture the architecture

```text
EC2 -> Security Group -> Mount Target -> EFS
        NFS traffic: TCP 2049
```

This is a memory aid: a security group is a firewall rule set attached to network resources, not a separate routing hop.

| Concept | Simple analogy | Meaning |
| --- | --- | --- |
| EC2 | Student's backpack | The computer running the workload |
| EFS | Shared classroom bookshelf | Shared files accessible by multiple clients |
| Mount（掛載） | Connect a backpack folder to the bookshelf | Make a remote file system accessible at a Linux directory |
| Mount target（掛載目標） | Library door | A network endpoint with an IP address in a VPC subnet |
| Security group（安全群組） | Security guard | Controls allowed network traffic |
| TCP 2049 | Special NFS door number | Port used for EFS NFS access |
| fio | Fast student carrying boxes | A Linux storage benchmarking tool |
| CloudWatch | Teacher with a clipboard measuring activity | AWS metrics monitoring |

EFS is shared network **file storage（檔案儲存）**. Linux clients commonly access it through NFS; this lab used **NFSv4.1**. Multiple clients can work with the same file system. See [AWS EFS mounting guidance](https://docs.aws.amazon.com/efs/latest/ug/mounting-fs.html).

### EFS vs. EBS vs. S3

| Service | Storage model | Remember it as | Typical use |
| --- | --- | --- | --- |
| EFS | Shared network file storage | Shared classroom bookshelf | Shared Linux directories and files |
| EBS | Block storage（區塊儲存） | A disk attached to a computer | EC2 boot/data volumes; provisioned capacity can be resized |
| S3 | Object storage（物件儲存） | Labeled objects in a bucket | Backups, media, and objects accessed through APIs |

EBS is normally attached to one EC2 instance; supported Multi-Attach configurations are a special case and do not automatically provide a shared file system. S3 is not a native NFS file system. See the [AWS storage overview](https://aws.amazon.com/products/storage/).

### Network and session access

The mount target provides access to EFS inside a VPC. Its security group must allow **inbound TCP 2049 from the EC2 client's security group**; the client's security group must allow corresponding outbound NFS traffic. DNS and network connectivity must also work. See [AWS EFS security group rules](https://docs.aws.amazon.com/efs/latest/ug/network-access.html).

**AWS Systems Manager Session Manager** provides an interactive shell on a managed EC2 instance. It requires an appropriately configured SSM Agent, IAM permissions, and service connectivity. It does not require inbound SSH port 22 for a normal shell session. Opening a session does not mount EFS or replace its NFS network requirements. See [Session Manager documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html).

## 2. Install, mount, and verify

These commands run on the **EC2 Linux instance**, not on the local Windows computer. The recorded relative paths assume the working directory is `/home/ec2-user`. A Session Manager shell may start elsewhere: check your location and use `cd /home/ec2-user` before reproducing them. Likewise, `~` means the current user's home and must resolve to `/home/ec2-user` for the recorded cleanup path.

Install the EFS utilities package:

```bash
sudo yum install -y amazon-efs-utils
```

`amazon-efs-utils` includes the EFS mount helper. The actual command below uses the Linux NFS client (`-t nfs4`), rather than invoking the EFS helper (`-t efs`). It does not request the helper's TLS encryption option.

Create the local mount-point directory:

```bash
sudo mkdir efs
```

Mount using the recorded command (shown unchanged):

```bash
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-092652c2d9fc2a238.efs.us-east-1.amazonaws.com:/ efs
```

| Part | Meaning |
| --- | --- |
| `-t nfs4`, `nfsvers=4.1` | Use NFS version 4.1 |
| `rsize=1048576`, `wsize=1048576` | Request 1 MiB read/write transfer sizes |
| `hard` | Keep retrying requests during server interruptions |
| `timeo=600`, `retrans=2` | 60-second initial timeout; two retransmissions before further recovery action; `hard` continues retrying |
| `noresvport` | Allow a new nonprivileged source port when reconnecting |
| DNS name followed by `:/` | EFS endpoint and remote root directory |
| `efs` | Local directory where EFS becomes accessible |

Verify:

```bash
df -hT
```

`-h` displays human-readable sizes; `-T` displays file system type. The successful lab result showed **Type = `nfs4`** and **Mounted on = `/home/ec2-user/efs`**. Check the EFS source as well as the type and exact mount location.

### Why did capacity look like 8.0E?

EFS grows and shrinks as files are added or removed; it has no fixed volume size to provision. The approximately `8.0E` reported through NFS is an extremely large capacity representation, not storage actually allocated to this account or a promise of unlimited throughput. An EBS volume instead reports its provisioned disk capacity. Bookshelf analogy: EFS can add shelf space as needed; an EBS disk starts with a chosen number of shelves. See [Amazon EFS FAQ](https://aws.amazon.com/efs/faq/).

## 3. Most important incident: the local disk filled up

> **Before big writes -> df -hT -> verify nfs4 -> then run workload.**
>
> **資料夾存在 ≠ EFS 已掛載。大量寫入前，先確認檔案系統類型與掛載位置。**

At one point EFS was **not mounted**, but `./efs` still existed. fio wrote `fio-efs-test.img` into that local directory on the EC2 root disk. A mount-point directory can remain after a remote file system is unmounted; writes then silently go to the underlying local file system.

The root disk was only **8 GB**, smaller than the requested 10G test file. Recorded observations:

| Check | Observation |
| --- | --- |
| `df -hT` | `/dev/xvda1`, type `xfs`, approximately `8.0G` used, `100%`, mounted at `/` |
| `ls -lh ~/efs` | `fio-efs-test.img` was approximately `6.2G` |

The test file did not need to reach 10G to fill the disk because the root disk already contained the operating system and other files.

### Recovery performed in the lab

With EFS absent and the file confirmed to be the accidental **local** file, we removed it:

```bash
sudo rm ~/efs/fio-efs-test.img
```

Root usage returned to approximately **22%**. We then:

1. Remounted EFS with the recorded mount command.
2. Ran `df -hT`.
3. Verified the file system type was `nfs4`.
4. Verified the mount location was `/home/ec2-user/efs`.
5. Only then reran fio.

**Recovery context matters:** if EFS is mounted, that same path refers to the EFS file, and the local file underneath is hidden. Confirm which file system owns the path before deleting anything. Do not blindly repeat the cleanup command after remounting.

## 4. Run fio only after verifying the mount

The lab used this exact command:

```bash
sudo fio --name=fio-efs --filesize=10G --filename=./efs/fio-efs-test.img --bs=1M --nrfiles=1 --direct=1 --sync=0 --rw=write --iodepth=200 --ioengine=libaio
```

| Option | Meaning |
| --- | --- |
| `--name=fio-efs` | Job name |
| `--filesize=10G`, `--nrfiles=1` | One test file with a target size of 10 GiB using fio's default units |
| `--filename=./efs/fio-efs-test.img` | Destination relative to the current directory |
| `--bs=1M`, `--rw=write` | Sequential writes using 1 MiB blocks |
| `--direct=1` | Request direct I/O to bypass the client page cache |
| `--sync=0` | Do not request synchronous I/O for every write |
| `--iodepth=200`, `--ioengine=libaio` | Request up to 200 outstanding I/Os using Linux asynchronous I/O; achieved depth can vary |

fio is a **Linux benchmarking tool, not an AWS service**. Its installation command was not included in the lab record. See the [fio manual](https://fio.readthedocs.io/en/latest/fio_doc.html) for option definitions.

One recorded result was approximately **WRITE bandwidth = 127 MiB/s**, reported as approximately **134 MB/s**. These are rounded observations: 1 MiB = 1,048,576 bytes and 1 MB = 1,000,000 bytes, so exactly 127 MiB/s converts to about 133.2 MB/s. This measurement alone does not prove the destination was EFS; always verify the mount.

## 5. Observe CloudWatch metrics

Recorded console navigation:

```text
CloudWatch -> Metrics -> Classic metrics -> EFS -> File System Metrics
```

Select the file system `fs-092652c2d9fc2a238` in `us-east-1` and a time range covering the test. The navigation above records the lab UI; console labels can change.

Initially **DataWriteIOBytes** was not visible. After valid EFS write activity and a wait for metric population, it appeared. CloudWatch metrics can have a short delay, and activity-related metrics may not appear immediately. A write to the EC2 root disk does not create EFS write activity.

| Metric | Meaning | Simple analogy |
| --- | --- | --- |
| `PermittedThroughput` | Throughput EFS is permitted to drive, in bytes/second; depends on throughput configuration and relevant limits | Road width / allowed road capacity |
| `DataWriteIOBytes` | Bytes associated with EFS data writes; use `Sum` for total write bytes in a period | Boxes that actually traveled down the road |

Permitted throughput is a capacity metric, not a guarantee that a client will achieve that rate. See [EFS metric definitions](https://docs.aws.amazon.com/efs/latest/ug/efs-metrics.html).

### Sum over one minute → average write throughput

For **DataWriteIOBytes**, the lab selected:

- **Statistic = Sum**: add the bytes reported during the selected period（加總）.
- **Period = 1 minute**: each bucket covers 60 seconds.
- **Observed peak bucket = 7,306,340,720 bytes**.

```text
Throughput = bytes / seconds
7,306,340,720 / 60
= 121,772,345.3 bytes/second
≈ 121.8 MB/s (decimal)
```

Sum over one minute is all the boxes moved during that minute. Divide by 60 to estimate boxes per second. This is the **average rate within the peak one-minute bucket**, not an instantaneous peak. See [AWS EFS metric math](https://docs.aws.amazon.com/efs/latest/ug/monitoring-metric-math.html).

CloudWatch's approximately **121.8 MB/s** can differ from fio's approximately **134 MB/s** because CloudWatch aggregates into time windows, while fio reports over its own run interval. A bucket may include time before or after active writing. Without aligned timestamps, these observations do not establish an exact cause for the difference.

## 6. AWS exam takeaways

- **EFS = shared Linux file system / network file storage. EBS = block storage. S3 = object storage.**
- EFS commonly uses **NFS**; this lab used **NFSv4.1 over TCP 2049**.
- Mount targets provide network access to EFS inside a VPC; security groups control network access.
- Session Manager provides a managed shell; it does not establish the EFS mount.
- `df -hT` verifies file system type and mount location. A directory alone proves nothing about a remote mount.
- fio benchmarks storage; CloudWatch monitors AWS metrics.
- `PermittedThroughput` is allowed capacity; `DataWriteIOBytes` measures write activity.
- **Sum** aggregates values across the selected period. **Throughput = bytes / seconds**; one minute means divide by **60**.

> **Before big writes -> df -hT -> verify nfs4 -> then run workload.**

## Lab Completion Checkpoint / Lab 結束檢查點

This checkpoint applies the standard end-of-lab workflow retroactively to Lab 05 Amazon EFS. It evaluates the work recorded above without adding new lab steps or changing the observations.

### 1. Class Lab Complete

| Check | Lab record |
| --- | --- |
| Lab completed successfully | Yes — EFS mounted, verified, and exercised with fio; CloudWatch write activity reviewed |
| Important troubleshooting captured | Yes — an unmounted directory redirected fio writes to the local root disk |
| Full notes documented | Yes |
| Cheat sheet documented | Yes |
| GitHub documentation | Already committed/pushed before this checkpoint update |

The recorded recovery included removing the accidental local file, remounting EFS, verifying `nfs4` at the expected mount location, and only then rerunning fio. This completion record does not imply that cloud-resource cleanup was verified.

### 2. SAA Takeaways

- **EFS vs EBS vs S3:** EFS provides shared network file storage, EBS provides block storage, and S3 provides object storage.
- **EC2-to-EFS networking:** mount targets provide network access to EFS in the VPC. Security groups must permit the client-to-mount-target NFS traffic.
- **NFSv4.1 / TCP 2049:** this lab used NFSv4.1; the EFS NFS destination port is TCP 2049.
- **Linux mounting and verification:** a local directory can exist without a remote mount. Check the EFS source, `nfs4` type, and exact mount location with `df -hT` before large writes.
- **Elastic capacity:** the large `8.0E` display is not preallocated storage or a throughput guarantee.
- **fio vs CloudWatch:** fio is a storage benchmark tool; CloudWatch records AWS metrics. A benchmark result alone does not prove the write destination was EFS.
- **Capacity vs activity:** `PermittedThroughput` describes permitted throughput; `DataWriteIOBytes` describes write bytes. `Sum` over one minute gives the period's write-byte total; divide by 60 for its average bytes/second.
- **Measurement windows:** fio and CloudWatch can report different rates because their measurement intervals differ. Preserve the recorded results without treating the difference as an error.

> **Before big writes -> df -hT -> verify nfs4 -> then run workload.**
>
> **資料夾存在不代表 EFS 已掛載；大量寫入前，先確認資料真正寫到哪裡。**

### 3. 🧒 3rd-Grade Understanding Check

| Concept | Explain it simply |
| --- | --- |
| EC2 | The student's backpack/computer |
| EFS | The shared classroom bookshelf |
| Mount | Connecting the backpack folder to the bookshelf |
| Mount Target | The library door |
| Security Group | The security guard |
| TCP 2049 | The special NFS door number |
| fio | The fast student carrying boxes |
| CloudWatch | The teacher measuring activity |

If the backpack folder is not connected to the bookshelf, the boxes stay in the backpack and can fill it up. The folder's label does not prove the connection exists. `df -hT` helps check that the folder really reaches the shared bookshelf before the student carries more boxes.

**理解重點：先確認書包資料夾連到共享書架，再搬大量箱子。** The operational check is to explain why an existing directory is insufficient and identify the expected `nfs4` mount before writing.

### 4. What AWS Academy Prepared for Me

The recorded work took place in an AWS Academy classroom environment with controlled access and lab prerequisites. A personal account must independently provide the network, EC2 client, access permissions, and supporting setup that the classroom workflow relied on.

The existing notes identify the EFS file system and EC2 client but do not establish exactly which underlying resources were precreated versus configured during the lab. Do not assume a personal account already contains the classroom's VPC/subnets, client instance, security groups, or Session Manager prerequisites. The recorded `amazon-efs-utils` installation remains a step we performed, not a claimed preinstalled classroom component.

**These notes are a Class Lab Record, not a guaranteed from-scratch runbook for a personal AWS account.**

**這是課堂操作紀錄；個人帳號的網路、主機、權限與連線方式需要自行建立及驗證。**

### 5. Personal AWS Rebuild Decision

**🟢 YES — Medium Priority**

This lab is worth rebuilding in a personal AWS account because it provides hands-on practice with EFS, EC2-to-EFS networking, mount targets, Security Groups, NFS/TCP 2049, Linux mounting and verification, and CloudWatch EFS metrics.

The accidental local-disk-full incident is an especially valuable operational lesson: a mount-point directory can exist even when EFS is not mounted. Run `df -hT` and verify `nfs4` at the intended mount location before large writes. A future exercise should teach this failure safely rather than require filling the root disk.

The rebuild is **Medium Priority** because its architecture is narrower than Challenge Lab 05 Café. It remains valuable as a focused storage and troubleshooting exercise. This is a learning decision, not a request to provision resources now.

Before rebuilding, evaluate:

| Decision area | What to establish first |
| --- | --- |
| Expected AWS cost | Estimate EC2 runtime, EBS storage, EFS storage/throughput, and any additional networking or monitoring resources chosen |
| Security differences | Scope NFS access to the intended client; choose an appropriate administration method and review encryption and permissions |
| Time required | Allow time for prerequisites, mount verification, bounded test writes, metric collection, and cleanup |
| Classroom resources to recreate | Plan the VPC/subnets, connectivity, security groups, EC2 client, access prerequisites, EFS file system, and mount targets |
| Cleanup requirements | Inventory all created resources and define removal and verification steps before provisioning |

### 6. Follow-up Companion Lab

Planned path: `personal-labs/lab-05-amazon-efs-personal-rebuild.md`

**Planned only; the Personal Rebuild has not been created.** This path is not a link to an existing file.

A future Personal Rebuild must:

- Work without AWS Academy resources.
- Create the necessary prerequisites in a personal AWS account from scratch.
- Explain why each resource exists and how the client reaches EFS.
- Use safer real-world defaults where practical and distinguish classroom shortcuts from recommendations.
- Include cost and security checkpoints before provisioning and before running workloads.
- Use bounded test data and verify the mount before writing.
- Include complete cleanup instructions and a final resource audit.

### 7. Cleanup Check

Before declaring a Personal Rebuild complete, explicitly review all resources created in every Region used: EC2 instances, EBS volumes and any snapshots, the EFS file system and mount targets, any EFS backups, and any additional networking, monitoring, or supporting resources. Record deletion results and any intentionally retained resources with their ongoing cost implications.

Removing the fio test file is not the same as removing the lab infrastructure. Stopping EC2 is not a complete cleanup procedure; storage and other retained resources still need review.

**This retroactive checkpoint does not claim that cloud-resource cleanup was performed.** The existing record confirms local test-file cleanup and recovery, not a final inventory of remaining AWS resources. No Personal Rebuild resources were created by this documentation update.

**清除測試檔案不等於清除全部 AWS 資源；完成前要逐項檢查。**

Class Lab -> Documentation -> SAA Review -> 3rd-Grade Check -> Personal Rebuild Decision -> Cleanup Review

上課 Lab -> 文件化 -> SAA 複習 -> 三年級理解檢查 -> Personal Rebuild 判斷 -> Cleanup 檢查
