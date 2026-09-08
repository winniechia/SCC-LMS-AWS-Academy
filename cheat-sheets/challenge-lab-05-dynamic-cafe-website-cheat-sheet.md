# Challenge Lab 05 — Dynamic Café Website Cheat Sheet

[Study index](../README.md) | [Full notes](../labs/challenge-lab-05-dynamic-cafe-website.md)

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| Concept | Memory aid |
| --- | --- |
| EC2 | Café building/computer |
| Security Group | Security guard controlling doors |
| Port 22 / Port 8000 | Administrator/maintenance door / café customer website door |
| IAM Role / Secrets Manager | AWS employee permission badge / locked safe |
| AMI / AWS Region | EC2 blueprint/recipe / another city |
| Development / Production | Practice café / real café |

> **Security Group = network access. IAM Role = AWS permissions.**
>
> 安全群組管網路；IAM 角色管 AWS 權限。

## Actual work: investigate → fix → test

- Inspected **Lab IDE EC2**, public subnet, public IPv4, initial inbound **TCP 80**, and IAM role configuration.
- Café stack: **Apache + PHP + MariaDB**; web directory `/var/www/html`, initial `index.html`; added **TCP 8000** access.
- **Seven `/cafe/*` secrets**; retrieved `/cafe/dbPassword` for MariaDB login. Never copy the value into notes.
- Page loaded but menu items were missing → investigated AWS permissions → attached **`CafeRole`** → menu worked → tested orders and **Order History**.

> **A webpage loading does not prove the application has all required AWS permissions. If the page loads but AWS-backed data does not, check IAM permissions/roles instead of assuming networking is broken.**

## Copy the café to another Region

**`CafeServer` AMI: `us-east-1` → copy → `us-west-2` (Oregon).** AMIs are Regional; external secrets/configuration are not automatically copied with the image.

| Oregon launch setting | Value |
| --- | --- |
| Name / type | `ProdCafeServer` / `t2.small` |
| VPC / subnet | `Lab VPC Region 2` / `Public Subnet` |
| Auto-assign public IP | Enabled |
| Security group | `cafeSG`: TCP **22**, **8000** from `0.0.0.0/0` |
| IAM instance profile | `CafeRole` |

**Ports 22 and 8000 open to Anywhere were required by this training lab, not a general production-security recommendation.** 正式環境不可直接照搬訓練規則。

## Configuration: original IDE, destination values

From the **N. Virginia VS Code IDE**, edited `set-app-parameters.sh`:

```bash
# Replaced region=${az%?} with:
region="us-west-2"
# Replaced automatic publicDNS metadata lookup with:
publicDNS="<OREGON_PROD_PUBLIC_DNS>"
```

Copied the Oregon server's Public IPv4 DNS into the actual lab script; only a placeholder is retained here. Local metadata describes the source machine. Target Region and DNS must describe the destination. Script execution and a final Oregon website test were not supplied in the completion record.

## Exam memory

- **Network door ≠ AWS permission badge.** 網頁能開，不等於資料讀得到。
- EC2 role attachment uses an **instance profile**; secret retrieval needs IAM authorization.
- **AMI = Regional image**, not automatic replication of external AWS services.
- Test application data and orders, not just page loading.

## AMI questions 5–7 — selected lab answers

- **Q5 — Reboot when creating an AMI?** Yes by default; you can choose not to reboot.
- **Q6 — Root volume changes during AMI creation?** Size and volume type can be edited, but not the 'delete on termination' setting.
- **Q7 — Add volumes when the source instance has only one?** Yes.

**記憶：預設重啟；根磁碟可改大小與類型；可加磁碟。** See the full notes for the exact questions and selected answers.
