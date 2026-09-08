# Module 5 Knowledge Check — EC2 and Storage

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/module-05-knowledge-check-cheat-sheet.md)

## Study-session record

This completed study session reviewed EC2, machine images, instance families, storage, and purchasing options. The sections summarize tested concepts and the learning corrections supplied for this session, rather than reproducing the quiz. No question numbering, score, or verbatim feedback is invented. The supplied AWS Academy default-behavior guidance is identified explicitly; additional qualifications provide SAA study context.

## 1. Why choose EC2?

EC2 supports a broad range of workloads and gives control over computing resources: instance size/type, operating system, installed software, and application configuration. **EC2 is not serverless.** AWS operates the underlying physical infrastructure, but using EC2 still involves managing instances and their software.

**選 EC2：需要多種工作負載支援，以及對運算環境的控制。** “AWS runs the hardware” does not mean “my application uses a serverless service.”

## 2. AMIs: repeatable starting configurations

An Amazon Machine Image (AMI) provides the image/configuration needed to launch an EC2 instance. Useful purposes reviewed:

- **Repeatable configuration:** launch machines from a known starting image.
- **Backup/image use:** preserve an image for later recovery or rebuilding; a complete backup strategy must also address application consistency and data outside the image.
- **Packaging/sharing software:** distribute a prepared operating system and software environment through an image, subject to permissions and licensing.

**Correction:** an AMI does not itself provide high availability or a live shared file system. Launching from an image creates an instance; it does not make ongoing file changes synchronize between instances. High availability needs an appropriate deployment design; shared files call for a shared storage service such as EFS.

**AMI = “Who should I start like?” / 我啟動時要長得像誰？** See [AWS AMI overview](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html).

## 3. Change an EBS-backed instance type

The reviewed workflow is:

```text
Stop -> Change instance type -> Start
```

This applies to an EBS-backed instance and a **compatible target instance type**. Plan for downtime and check architecture, drivers, and other compatibility requirements. It is not a promise that every instance can change to any type. See [AWS instance-type change guidance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html).

**先停止，再改規格，最後啟動。** Stopping is different from terminating the instance.

## 4. Instance families: identify the resource doing the work

> **Ask “WHAT resource is doing the hard work?” rather than “does it need to be fast?”**
>
> **問「哪一種資源正在做最吃力的工作？」不要只問「需要快嗎？」**

Every workload wants speed. The useful distinction is whether its limiting resource is CPU, RAM, local storage I/O, or specialized processing.

| Family | Workload clue | Simple example |
| --- | --- | --- |
| Compute Optimized | CPU-heavy calculations | Large amounts of general-purpose computation |
| Memory Optimized | RAM-heavy working sets | Keeping large datasets in memory |
| Storage Optimized | High local storage read/write demand | Intensive local disk access and data processing |
| Accelerated Computing | Specialized hardware acceleration | GPU or other accelerator-suited processing |

The examples explain the categories; they are not reconstructed quiz scenarios. Storage Optimized refers to storage characteristics of the instance, especially local storage; it is not the same choice as selecting an EBS volume type. Accelerated Computing does not mean “anything that should run faster.” See [AWS instance-type categories](https://docs.aws.amazon.com/ec2/latest/instancetypes/ec2-instance-type-specifications.html).

## 5. AMI vs Instance Metadata vs User Data

| Concept | Question it answers | Purpose |
| --- | --- | --- |
| AMI | “Who should I start like?” | Starting machine image |
| Instance Metadata | “Who am I right now?” | Information about the running instance and its environment |
| User Data | “What setup instructions should I follow?” | Bootstrap/setup instructions supplied for the instance |

**AWS Academy Module 5 guidance supplied in this session:** User Data runs **once at instance launch by default**. For scripts, think initial boot, not every reboot. Execution behavior can be configured differently; this is a default, not an absolute limitation. See [AWS User Data documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html).

Metadata describes the instance; it is not the reusable starting image. User Data supplies setup instructions; it is not a synonym for metadata. **映像是起點，Metadata 是目前身分，User Data 是開機設定指示。**

## 6. EBS performance: IOPS vs throughput

| Measure / choice | Meaning | Workload clue |
| --- | --- | --- |
| IOPS | I/O operations per second | Many small, frequent operations |
| Throughput | Amount of data transferred per second | Large sequential transfers |
| Provisioned IOPS SSD | EBS storage for demanding, consistent transactional I/O | A workload requiring predictably high IOPS |

**IOPS 問「每秒幾次？」Throughput 問「每秒多少資料？」** Both measures matter, and actual performance also depends on I/O size and instance/volume limits. Conceptually, throughput is operations per second multiplied by bytes per operation; a high operation count alone does not describe the size of each transfer.

Do not interpret “transactional I/O” as a rule that every database always needs Provisioned IOPS SSD. The tested distinction is the fit for consistently high transactional demand. See [AWS EBS volume types](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html).

## 7. Managed file storage: EFS and FSx

**Amazon EFS** provides managed shared file storage and grows/shrinks with stored data. Compared with a single EC2-hosted NFS server, EFS removes the need to operate that file-server instance and provides managed availability. **Regional EFS** stores data across multiple Availability Zones; **One Zone EFS** is a distinct option and should not be described as having the same multi-AZ resilience. See [AWS EFS features](https://docs.aws.amazon.com/efs/latest/ug/features.html).

**Amazon FSx for Windows File Server** provides fully managed Windows file servers, including Windows-oriented file sharing through SMB. Match the named service to the workload rather than treating all FSx offerings as identical. See [FSx for Windows File Server](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html).

**共同使用檔案 → shared file storage；複製啟動環境 → AMI。** An image is not a substitute for live shared storage.

## 8. EC2 purchasing options

| Option | Main idea | What not to confuse it with |
| --- | --- | --- |
| On-Demand | Usage-based pricing without a long-term commitment | A commitment discount |
| Spot | Discounted spare capacity; interruptible | Guaranteed uninterrupted capacity |
| Reserved Instances | Commitment-based discounts for matching usage | Ownership of a dedicated physical server |
| Savings Plans | Discounts for a committed amount of eligible usage/spend over a term | Dedicated hardware |
| Dedicated Host | A physical server dedicated to the customer | Merely a billing discount |

Reserved Instances and Savings Plans typically use one- or three-year commitments. Details differ: a zonal Reserved Instance can include capacity reservation, but that still does not mean a dedicated physical host. Savings Plans do not themselves reserve capacity. See [AWS EC2 purchasing options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html).

**承諾換折扣 ≠ 整台實體主機專用。** Choose based on commitment tolerance, interruption tolerance, and hardware-isolation needs; no current rates or savings percentages are assumed here.

## Weak Spots Repaired / 已修正觀念

| Confusion from this session | Corrected distinction |
| --- | --- |
| AMI confused with high availability/shared files | AMI supplies a starting image; availability architecture and shared storage are separate needs |
| Compute Optimized confused with Storage Optimized | CPU work points to Compute; local storage reads/writes point to Storage |
| Accelerated Computing confused with Storage Optimized | Specialized processing hardware points to Accelerated; disk I/O points to Storage |
| “It needs to be fast” used as the selection rule | Identify the resource doing the hard work before choosing a family |

**修正方式：先圈出需求中的資源，再選服務或家族。** These record the learning corrections, not a new assessment score.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| Concept | Classroom analogy |
| --- | --- |
| EC2 | A computer you can set up for many different jobs |
| AMI | A recipe for preparing another computer the same way |
| Instance Metadata | The computer's current name tag |
| User Data | The setup checklist for its first day |
| Compute Optimized | A student doing lots of calculations |
| Memory Optimized | A bigger desk to keep more pages open |
| Storage Optimized | A worker taking files in and out of local drawers |
| Accelerated Computing | A special-purpose machine for a particular job |
| IOPS / throughput | Trips per second / total boxes carried per second |
| EFS | A shared classroom bookshelf that can grow |
| Dedicated Host | A whole physical computer reserved for your use |

A recipe makes a new starting copy. A bookshelf lets students share current books. A special machine is useful only when the job fits it. **食譜、共享書架、專用工具，各有不同用途。**

## SAA exam takeaways

- Choose EC2 for broad workload support and control; EC2 is not serverless.
- Separate **image**, **runtime information**, and **setup instructions**: AMI, Metadata, User Data.
- Remember **stop → change type → start** for compatible EBS-backed instances.
- Identify the hard-working resource: **CPU / RAM / local storage I/O / accelerator**.
- Match small frequent I/O to IOPS, and large sequential transfer to throughput.
- Recognize EFS shared files and FSx for Windows managed Windows file servers.
- Distinguish pricing commitments, interruptible capacity, and physical host dedication.

## Final memory map

```text
EC2: workload flexibility + control (not serverless)
AMI: start like | Metadata: am now | User Data: setup
EBS-backed resize: stop -> change type -> start
CPU -> Compute | RAM -> Memory
Local disk I/O -> Storage | Special hardware -> Accelerated
IOPS -> operations/s | Throughput -> data/s
Shared files -> EFS | Windows file server -> FSx for Windows
No commitment -> On-Demand | Interruptible spare -> Spot
Commitment discounts -> RI / Savings Plans
Dedicated physical server -> Dedicated Host
```
