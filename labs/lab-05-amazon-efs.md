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
