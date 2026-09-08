# Challenge Lab 05 — Creating a Dynamic Website for the Café

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/challenge-lab-05-dynamic-cafe-website-cheat-sheet.md)

## Completed work and scope

We investigated the development server, fixed missing menu data by attaching `CafeRole`, tested orders and Order History, created and copied an AMI, launched an Oregon production instance, and edited the application configuration from the original N. Virginia IDE.

These notes use the supplied completion record. They do not claim an unreported script execution, final Oregon website test, or exact installation/login command. Temporary public addresses are replaced with placeholders. Secret values, passwords, account IDs, tokens, and temporary lab credentials are excluded.

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| AWS concept | Simple analogy | Remember |
| --- | --- | --- |
| EC2 | Café building/computer | Runs the website and its software |
| Security Group | Security guard controlling doors | Decides which network traffic may enter or leave |
| Port 22 | Administrator/maintenance door | SSH administration |
| Port 8000 | Café customer website door | The website port used in this lab |
| IAM Role | AWS employee permission badge | Allows the application to perform authorized AWS actions |
| Secrets Manager | Locked safe | Holds protected application configuration and credentials |
| AMI | EC2 blueprint/recipe | A reusable machine image for launching another server |
| AWS Region | Another city | A separate geographic deployment area |
| Development | Practice café | Try changes and investigate problems |
| Production | Real café | The deployment intended to serve customers |

> **Security Group = network access. IAM Role = AWS permissions.**
>
> **安全群組管網路的門；IAM 角色管使用 AWS 服務的權限。**

Customers can enter the café even when the employee has no badge to open the locked safe. Likewise, a website can load while the application cannot retrieve its AWS-backed configuration.

## 1. Investigate the Lab IDE EC2 instance

We inspected the **Lab IDE** instance in **N. Virginia (`us-east-1`)**, its public subnet, public IPv4 address, initial inbound **TCP 80** rule, and IAM role configuration.

A public subnet has a route to an internet gateway. A public IPv4 address alone does not guarantee access: routing, security groups, network ACLs, and a listening application must also permit the connection. TCP 80 is the default HTTP port; the café application in this lab used TCP 8000, so its rule needed to match that port.

The server used **Apache, PHP, and MariaDB**:

- Apache serves web requests.
- PHP runs server-side application code（伺服器端程式）.
- MariaDB stores the application's database data.
- `/var/www/html` was the web document directory; `index.html` was part of the initial web-content work.

We added the **TCP 8000 security-group rule** for café website access. The exact initial IAM role name and package installation commands were not supplied, so they are not reconstructed here.

## 2. Secrets Manager and MariaDB access

We worked with **seven Secrets Manager secrets under `/cafe/*`**. Here, `*` describes the name prefix; it is not a single literal secret name. Only `/cafe/dbPassword` was named in the supplied record, so the other six names are not invented.

We retrieved `/cafe/dbPassword` and used its value for MariaDB login. The secret name is safe to retain for learning; its value is deliberately omitted. No literal `dbPassword` assignment from the script is reproduced.

Retrieving a secret requires AWS authorization, such as permission for `secretsmanager:GetSecretValue`. Using that value to log in to MariaDB is a separate database authentication step. See [AWS Secrets Manager permissions](https://docs.aws.amazon.com/secretsmanager/latest/userguide/auth-and-access_iam-policies.html).

## 3. Troubleshooting: the page loaded, but the menu was missing

| Observation or action | What it taught us |
| --- | --- |
| Café page loaded | The browser could reach the web server for that request |
| Menu items were initially missing | Page delivery alone did not prove the application could access AWS-backed data |
| Investigation found missing AWS permissions | The application needed the appropriate IAM role |
| Attached `CafeRole` to EC2 | The menu worked after the role was attached |
| Placed/tested orders and checked Order History | Tested application behavior beyond simply loading a page |

> **A webpage loading does not prove the application has all required AWS permissions. If the page loads but AWS-backed data does not, check IAM permissions/roles instead of assuming networking is broken.**
>
> 網頁能開，不代表 AWS 權限完整。選單沒有資料時，也要檢查 IAM 角色。

The observed fix was **attaching `CafeRole`**, not opening more network ports. In another incident, missing data could have other causes; here, the successful role attachment supported the permissions diagnosis. No error log text or exact policy document was supplied.

EC2 uses an **IAM instance profile** to associate a role with an instance. Applications can obtain temporary role credentials through the supported AWS credential mechanisms rather than storing long-lived access keys. See [IAM roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html).

## 4. AMI questions 5–7

These are the questions and selected answers from the completed lab.

| Question | Selected answer |
| --- | --- |
| **5. When you create an AMI from an instance, will the instance be rebooted?** | You have the option not to reboot, but by default it will be rebooted. |
| **6. In what ways can you modify the root volume properties when you create an AMI from an instance?** | You can edit the size and volume type, but not the 'delete on termination' setting. |
| **7. Can you add more volumes to an AMI that you create from an instance that only has one volume?** | Yes. |

**記憶：預設重新啟動，但可選擇不重啟；本題根磁碟可改大小與類型；可加入更多磁碟。**

## 5. Create the AMI and launch in Oregon

We created the **`CafeServer` AMI** and copied it from **`us-east-1`** to **`us-west-2`**.

**AMIs are Regional.** To launch from that image in Oregon, the image must exist in Oregon. Copying an AMI provides a machine image in the destination Region; it does not automatically reproduce all external service configuration. See [AWS AMI copy documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/CopyingAMIs.html).

Recorded destination launch settings:

| Setting | Completed selection |
| --- | --- |
| Instance name | `ProdCafeServer` |
| Region | Oregon — `us-west-2` |
| Image | Oregon copy of `CafeServer` |
| Instance type | `t2.small` |
| VPC | `Lab VPC Region 2` |
| Subnet | `Public Subnet` |
| Auto-assign public IP | Enabled |
| Security group | `cafeSG` |
| Inbound rules | TCP 22 and TCP 8000 from Anywhere (`0.0.0.0/0`) |
| IAM instance profile | `CafeRole` |

> **Training-lab requirement:** `0.0.0.0/0` on ports **22 and 8000** was required by this lab. It is **not a general production-security recommendation**. `0.0.0.0/0` means all IPv4 source addresses.
>
> 這是訓練環境要求，不代表正式環境應將管理與網站連接埠全面開放。

We copied the **Public IPv4 DNS** of `ProdCafeServer`. In these notes it is represented by `<OREGON_PROD_PUBLIC_DNS>`; no temporary hostname or public IP is retained.

## 6. Configure the destination from the original IDE

We edited **`set-app-parameters.sh` from the original N. Virginia VS Code IDE**, while targeting the Oregon production server.

The recorded Region change was:

```bash
# Before: derive the Region from the local Availability Zone
region=${az%?}

# After: explicitly target Oregon
region="us-west-2"
```

`${az%?}` is shell parameter expansion that removes the final character. For a standard Availability Zone name, this derives its Region. Because the script was being edited in the original N. Virginia environment, automatically using that environment's metadata would identify the source environment, not the Oregon destination.

We also replaced the **automatic metadata lookup for `publicDNS`** with the Oregon `ProdCafeServer` public DNS. The original lookup command was not supplied. The resulting assignment is shown with a placeholder only:

```bash
publicDNS="<OREGON_PROD_PUBLIC_DNS>"
```

This is a study excerpt, not a complete runnable script. No password assignment is included.

### Why Region-specific configuration was necessary

- **AMI location:** the launch image had to exist in `us-west-2`.
- **Service configuration:** Secrets Manager secrets are Regional resources. An AMI copy does not automatically create or replicate the external secrets in another Region. See [Secrets Manager replication](https://docs.aws.amazon.com/secretsmanager/latest/userguide/replicate-secrets.html).
- **Script target:** setting `region="us-west-2"` directs operations that use that variable toward Oregon instead of the original IDE's Region.
- **Website address:** setting `publicDNS` to the destination server prevents the configuration from retaining the development server's address.

**記憶：複製主機映像後，還要確認 Region、設定與網站地址。** The record confirms these edits; it does not provide a subsequent script run or Oregon application test result.

## 7. SAA exam takeaways

- **Security groups control network traffic; IAM roles authorize AWS API actions.** They solve different access problems.
- A public subnet and public address do not override firewall rules or start a web server.
- Port **22** is SSH; **80** is default HTTP; **8000** was this lab's café website port.
- Secrets Manager protects secrets; applications still need permission to retrieve them.
- EC2 receives a role through an instance profile. Do not embed access keys or secret values in study notes or source code.
- AMIs are Regional machine images. Copy the AMI before launching from it in another Region.
- A machine image does not replace destination networking, IAM attachment, or Region-specific external configuration.
- Test **menu data, orders, and Order History**, not just whether a page opens.
- “Production” is the lab instance's purpose; its lab-required open ports are not a general production design.
