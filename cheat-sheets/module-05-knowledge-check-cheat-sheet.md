# Module 5 Knowledge Check — Cheat Sheet

[Study index](../README.md) | [Full study notes](../knowledge-checks/module-05-knowledge-check.md)

## EC2, images, and setup

- **EC2:** broad workload support and control of compute resources; **not serverless**.
- **AMI:** repeatable starting configuration, backup/image use, packaging/sharing software. It does not itself provide high availability or live shared files.
- **EBS-backed type change:** **Stop → Change type → Start**, subject to compatibility.
- **AMI:** “Who should I start like?” / 啟動範本。
- **Instance Metadata:** “Who am I right now?” / 目前身分與環境。
- **User Data:** bootstrap/setup instructions. Supplied AWS Academy guidance: **once at instance launch by default**, not automatically every reboot; configurable exceptions exist.

## Choose the resource, not just “fast”

| Hard-working resource | Instance family |
| --- | --- |
| CPU-heavy calculation | Compute Optimized |
| RAM-heavy working set | Memory Optimized |
| High local storage read/write | Storage Optimized |
| Specialized hardware acceleration | Accelerated Computing |

**Ask “WHAT resource is doing the hard work?” rather than “does it need to be fast?”** 問哪種資源最吃力。

## Storage and pricing

- **IOPS = operations/second:** many small, frequent I/Os.
- **Throughput = data/second:** large sequential transfers.
- **Provisioned IOPS SSD:** consistently high transactional I/O.
- **EFS:** managed shared files, automatic capacity scaling; Regional EFS offers multi-AZ resilience compared with a single EC2 NFS server. One Zone differs.
- **FSx for Windows File Server:** fully managed Windows file servers.
- **On-Demand:** usage-based, no long-term commitment.
- **Spot:** discounted spare capacity, interruptible.
- **Reserved Instances / Savings Plans:** commitment discounts, not dedicated physical servers.
- **Dedicated Host:** a physical server dedicated to the customer.

## Weak Spots Repaired / 已修正觀念

**AMI ≠ shared files or HA. Compute ≠ disk I/O. Accelerated ≠ faster storage.** Choose CPU, RAM, storage, or special hardware based on the workload's actual demand.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

**AMI = recipe; Metadata = name tag; User Data = first-day checklist.** CPU = calculator; RAM = desk; storage = drawers; accelerator = special tool. **IOPS = trips; throughput = boxes moved.** EFS = shared bookshelf.

**食譜不是共享書架；折扣不是整台實體主機。**

## Final memory map / SAA recall

```text
Start like -> AMI | Am now -> Metadata | Setup -> User Data
CPU / RAM / local disk / special hardware -> choose family
Operations/s -> IOPS | Data/s -> throughput
Shared files -> EFS | Windows files -> FSx for Windows
No commitment / spare / commitment / physical host
-> On-Demand / Spot / RI + Savings Plans / Dedicated Host
```
