# Lab 05 — Amazon EFS Cheat Sheet

[Study index](../README.md) | [Full notes and troubleshooting](../labs/lab-05-amazon-efs.md)

## Architecture and exam essentials

```text
EC2 -> Security Group -> Mount Target -> EFS
                  NFSv4.1 / TCP 2049
```

Backpack → security guard → library door → shared classroom bookshelf. A security group is a firewall rule set, not a routing hop. Mount = connect a backpack folder to the bookshelf（掛載）.

| EFS | EBS | S3 |
| --- | --- | --- |
| Shared Linux/network file storage | Block storage | Object storage |
| Shared bookshelf | Attached disk | Objects in a bucket |

- Mount target = VPC network access to EFS. Allow inbound **TCP 2049** from the EC2 client security group; allow corresponding client outbound traffic.
- Session Manager = managed EC2 shell. `amazon-efs-utils` = EFS utilities/mount helper.
- fio = Linux benchmark tool, **not an AWS service**. CloudWatch = AWS metrics monitoring.
- EFS capacity grows/shrinks with data; `8.0E` in `df` is not allocated storage. EBS has a provisioned size.

## Commands — run on the EC2 Linux client

Recorded relative paths require `/home/ec2-user` as the working directory; verify it first. Create the directory once, then mount. These are the original lab commands, wrapped with shell continuations for printing.

```bash
sudo yum install -y amazon-efs-utils
sudo mkdir efs
sudo mount -t nfs4 \
  -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport \
  fs-092652c2d9fc2a238.efs.us-east-1.amazonaws.com:/ efs
df -hT
```

**STOP and check:** EFS source, Type **`nfs4`**, Mounted on **`/home/ec2-user/efs`**. Only then:

```bash
sudo fio --name=fio-efs --filesize=10G \
  --filename=./efs/fio-efs-test.img --bs=1M --nrfiles=1 \
  --direct=1 --sync=0 --rw=write --iodepth=200 --ioengine=libaio
```

## Critical troubleshooting rule

> **Before big writes -> df -hT -> verify nfs4 -> then run workload.**
>
> **資料夾存在 ≠ EFS 已掛載。先確認，再大量寫入。**

Unmounted `./efs` still existed → fio wrote locally → 8 GB root disk filled. `df -hT`: `/dev/xvda1`, `xfs`, `8.0G` used, `100% /`. `ls -lh ~/efs`: accidental file ≈ **6.2G**.

Cleanup used **only after confirming EFS was absent and the file was local** (`~` = `/home/ec2-user`):

```bash
sudo rm ~/efs/fio-efs-test.img
```

Root usage returned to ≈ **22%** → remount → `df -hT` → confirm `nfs4` and exact mount → rerun fio. If EFS is mounted, that deletion path points into EFS instead; the local file underneath is hidden.

## CloudWatch calculation

Lab navigation: **CloudWatch → Metrics → Classic metrics → EFS → File System Metrics**. Region: `us-east-1`; select the file system above. `DataWriteIOBytes` appeared after valid EFS writes and a short population delay.

| Remember | Meaning |
| --- | --- |
| `PermittedThroughput` | Allowed road capacity, in bytes/second |
| `DataWriteIOBytes` | Actual write activity: boxes moved |
| **Sum**, **1-minute period** | Total bytes in a 60-second bucket |

```text
7,306,340,720 bytes / 60 seconds
= 121,772,345.3 bytes/s ≈ 121.8 MB/s
```

**Throughput = bytes / seconds.** Peak bucket average ≠ instantaneous peak. fio observed ≈ **127 MiB/s (about 134 MB/s)**; time-window aggregation and rounding can produce differences. MB is decimal; MiB is binary. See [full notes](../labs/lab-05-amazon-efs.md) for definitions and AWS references.
