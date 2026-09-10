# Lab 07 — Creating a VPC

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-07-creating-a-vpc-cheat-sheet.md) | [Personal companion plan](../personal-labs/lab-07-vpc-companion-plan.md)

## Purpose and completed lab record

AWS Academy Guided Lab: **Creating a VPC**. The lab builds a custom VPC manually, creates public and private subnets, attaches an Internet Gateway, configures route tables, creates an HTTP security group, launches an EC2 application server, and verifies that the Inventory application is reachable from the internet.

These notes preserve the supplied lab settings and the troubleshooting actually observed during the session. Temporary account IDs, VPC/subnet/route-table/security-group/instance IDs, public IP addresses, and public DNS names are intentionally omitted.

## 1. Architecture built in the class lab

```text
Internet
   |
Internet Gateway (Lab IGW)
   |
Lab VPC 10.0.0.0/16
   |
   +-- Public Subnet 10.0.0.0/24
   |      |
   |      +-- Public Route Table
   |      |      10.0.0.0/16 -> local
   |      |      0.0.0.0/0   -> Lab IGW
   |      |
   |      +-- App Server EC2
   |             App-SG: HTTP TCP 80 from 0.0.0.0/0
   |             Inventory-App-Role
   |             User Data installs the inventory application
   |
   +-- Private Subnet 10.0.2.0/23
          |
          +-- Private/Main Route Table
                 10.0.0.0/16 -> local
```

The private subnet is created in this guided lab but no NAT Gateway is added. Do not import the NAT/Bastion design from a different challenge lab into this class record.

## 2. VPC and CIDR

The custom VPC uses `10.0.0.0/16`.

- IPv4 addresses contain 32 bits.
- `/16` fixes the first 16 bits and leaves 16 bits for addresses inside the network.
- `2^16 = 65,536` total addresses in the CIDR block.
- In simple notation for this lab, the VPC covers `10.0.x.x`.

The AWS-provided default VPC seen in the classroom uses a different CIDR (`172.31.0.0/16`). It is a separate VPC, not the parent of the Lab VPC. A VPC is not a subnet of another VPC.

> **VPC = the whole mansion estate. Subnet = one area carved out inside that estate.**
>
> **VPC 是整座大宅院；Subnet 是從宅院土地裡切出的一個區域。**

### CIDR size intuition

| CIDR | Total addresses | Simple memory |
| --- | ---: | --- |
| `/16` | 65,536 | Large network / 整座大宅院 |
| `/23` | 512 | Two `/24` blocks |
| `/24` | 256 | One smaller subnet |
| `/32` | 1 | One exact IPv4 address |

Every time the prefix length decreases by 1, the address count doubles. Every time it increases by 1, the address count halves.

## 3. Public and private subnets

### Public Subnet

- CIDR: `10.0.0.0/24`
- Auto-assign public IPv4: enabled
- Intended for internet-facing resources

### Private Subnet

- CIDR: `10.0.2.0/23`
- Covers `10.0.2.x` and `10.0.3.x`
- Larger than the public subnet in this lab

The fact that the private subnet is larger is a design choice for this lab, not an AWS requirement.

A subnet is **not** public just because it is named `Public Subnet`, and a public IPv4 address by itself does not create a route to the internet.

> **A subnet becomes public when its associated route table has a route to an Internet Gateway.**
>
> **Subnet 叫 Public 沒有用；真正讓它成為 Public Subnet 的，是它所使用的 Route Table 有通往 IGW 的路。**

## 4. Internet Gateway (IGW)

`Lab IGW` is created and attached to the **Lab VPC**, not directly to a subnet.

Mansion analogy:

- VPC = mansion estate
- IGW = the front gate connecting the estate to the outside world
- Route table = road signs telling traffic which gate/road to use

Attaching the IGW alone does not make the public subnet internet-connected. The subnet's route table must direct internet-bound traffic to the IGW.

## 5. Route tables

### Local route

A route table created for the VPC contains the local route:

```text
Destination     Target
10.0.0.0/16  -> local
```

This provides routing among addresses inside the VPC. Security controls can still permit or block specific traffic; **having a route is not the same as having permission**.

### Public route table

The critical internet route is:

```text
Destination     Target
0.0.0.0/0    -> Lab IGW
```

`0.0.0.0/0` means the IPv4 default route: traffic not matched by a more specific route can follow this path.

To create the public subnet behavior:

1. Create and attach an Internet Gateway.
2. Create a Public Route Table.
3. Add `0.0.0.0/0 -> Lab IGW`.
4. Explicitly associate the Public Route Table with the Public Subnet.

> **IGW = gate. Route table = road sign. Subnet association = deciding which neighborhood follows that sign.**
>
> **IGW 是大門；Route Table 是路標；Subnet association 是指定哪個院子看這塊路標。**

## 6. Security Group for the application

`App-SG` is attached to the application server and allows:

```text
Type:     HTTP
Protocol: TCP
Port:     80
Source:   0.0.0.0/0
```

A Security Group works at the resource/ENI level, not the subnet level. It is different from routing: the route determines where traffic can go; the security group determines whether the traffic is allowed at the resource boundary.

## 7. Application Server configuration

Recorded class-lab settings:

| Setting | Value |
| --- | --- |
| Name | `App Server` |
| AMI | Amazon Linux 2023 |
| Instance type | `t2.micro` |
| Key pair | `vockey` |
| VPC | Lab VPC |
| Subnet | Public Subnet |
| Public IPv4 | auto-assigned |
| Security group | `App-SG` |
| IAM instance profile | `Inventory-App-Role` |
| Storage | lab/default selection |

The supplied User Data installs Apache/PHP dependencies, downloads the inventory application and AWS SDK for PHP, then enables and starts `httpd`.

`Inventory-App-Role` was provided by the classroom environment. This record does not claim that the IAM role was created during the lab.

## 8. User Data: boot-time automation

User Data acts like a first-boot setup checklist:

```text
New EC2 starts
   -> install packages
   -> download application files
   -> place files under /var/www/html
   -> enable httpd
   -> start httpd
```

A healthy EC2 `2/2 checks passed` result means the instance and underlying AWS infrastructure passed status checks; it does not by itself prove that the application installed successfully. Application/bootstrap logs remain useful when the web page does not load.

## 9. Actual troubleshooting record

### Incident A — VPC wizard attempted extra resources

The first VPC creation attempt used a workflow that tried to create an S3 VPC endpoint. The AWS Academy role denied `CreateVpcEndpoint`, and rollback also encountered restricted cleanup permissions.

Correction: use **VPC only** and create the lab components manually, matching the purpose of the guided lab. Do not broaden classroom IAM permissions to solve an unintended resource choice.

### Incident B — main route table appeared with delay

Immediately after VPC creation, the Lab VPC's Main Route Table initially appeared absent in the console. A manual route table was temporarily created while troubleshooting, but the expected Main Route Table later appeared. Academy permissions prevented some cleanup and main-route replacement operations.

Lesson: after creating foundational VPC resources, **refresh and allow time for console/backend state to converge before creating replacement resources**. Verify the VPC details and Main Route Table rather than assuming a temporarily missing console row means the resource does not exist.

### Incident C — application timed out because the public route was missing

The application server had a public IPv4 address, the correct Public Subnet, `App-SG` HTTP/80 access, and a healthy EC2 status. The Inventory website still timed out.

Troubleshooting by layer found:

```text
IGW attached to Lab VPC                 YES
Public subnet associated to route table YES
App-SG HTTP TCP 80                      YES
NACL inbound/outbound                   ALLOW
Public route 0.0.0.0/0 -> IGW           MISSING
```

The Public Route Table contained only the local route. Adding:

```text
0.0.0.0/0 -> Lab IGW
```

completed the intended public-subnet routing.

> **A public IP is an address; it is not a road. / Public IP 是地址，不是道路。**

### Incident D — HTTP versus copied DNS name

The copied Public IPv4 DNS name did not include a URL scheme. Explicitly opening the successful application server with:

```text
http://<PUBLIC_DNS_NAME>
```

loaded the Inventory application. The class security group allowed HTTP TCP 80; HTTPS was not the tested path.

The successful page displayed the expected message that Settings must be configured to connect to the database.

**Classroom HTTP success is not a production recommendation.** A real public application should use an appropriate HTTPS/TLS design.

### Incident E — two application servers existed during troubleshooting

A second application server was launched while isolating whether the first server's first-boot setup had failed. The second instance's system log showed successful cloud-init/application extraction and `httpd` setup, and its Inventory page was successfully reached with explicit `http://`.

The duplicate server was a troubleshooting artifact, not part of the intended final architecture. A cleanup attempt/result for the extra instance was not supplied in this record.

## 10. Troubleshooting method learned

When a public EC2 website times out, check the path in layers instead of randomly changing settings:

```text
1. EC2 state and 2/2 status checks
2. Public IPv4 / Public DNS
3. Correct VPC and Public Subnet
4. Internet Gateway attached to the VPC
5. Public Route Table has 0.0.0.0/0 -> IGW
6. Public Subnet is associated with that route table
7. Security Group allows the intended protocol/port
8. NACL allows inbound and return traffic
9. Application bootstrap / cloud-init / service status
10. URL scheme: http:// vs https://
```

> **Road -> gate -> neighborhood -> guard -> server -> application -> protocol.**
>
> **先查路，再查大門、院子、警衛、主機、應用程式、協定。**

## 🧒 Explain It Like I'm in 3rd Grade / 三年級也能懂

| AWS concept | Mansion analogy |
| --- | --- |
| VPC | The whole mansion estate / 整座大宅院 |
| CIDR | How much address-land the mansion owns / 土地地址範圍 |
| Public Subnet | Front yard intended to face outside / 前院 |
| Private Subnet | Back house/private yard / 內院、後院 |
| Internet Gateway | Main gate to the outside world / 對外大門 |
| Route Table | Road signs / 路標 |
| `0.0.0.0/0 -> IGW` | “For outside destinations, use the main gate” / 去外面就走大門 |
| Security Group | Maid/guard beside the server / 主人身邊的丫鬟或守衛 |
| Network ACL | Guard at the subnet boundary / 院子大門警衛 |
| Public IPv4 | Public mailing address / 對外門牌 |
| DNS hostname | Easier-to-use address name / 容易使用的地址名稱 |
| User Data | First-day setup checklist / 新主機開工清單 |

The key story:

> The App Server can have a public address, but visitors still need a road from the front yard to the Internet Gateway. The guard must allow HTTP, and the shop inside the mansion must actually be open.
>
> App Server 就算有公開門牌，訪客還是需要「前院 -> 路標 -> IGW」這條路；警衛要放行 HTTP，裡面的店也必須真的開門。

## SAA Review

- A VPC is a logically isolated virtual network; subnets divide its CIDR space.
- `/16` is larger than `/23`, and `/23` is larger than `/24`.
- A public subnet is defined by routing to an Internet Gateway, not by its name alone.
- An IGW attaches to the VPC; a route table points traffic to it.
- `0.0.0.0/0` is the IPv4 default route.
- The VPC local route provides routing inside the VPC.
- Route tables answer **where traffic goes**; security groups answer **what traffic is allowed at a resource**.
- A public IPv4 address does not replace the need for an IGW route.
- Security Groups are stateful resource-level controls; NACLs are subnet-level stateless controls. The class lab used permissive NACL rules while troubleshooting.
- HTTP normally uses TCP 80; HTTPS normally uses TCP 443. A copied hostname without `http://` or `https://` does not itself state the desired application protocol.
- EC2 status checks and application health are different layers.
- User Data is first-boot automation; troubleshoot `cloud-init`/system logs when bootstrap behavior is uncertain.

## Lab Completion Checkpoint / Lab 結束檢查點

### 1. Class Lab Complete

| Check | Record |
| --- | --- |
| Lab architecture built | Yes |
| Public subnet internet route corrected | Yes — `0.0.0.0/0 -> IGW` added |
| Inventory application reached | Yes — explicit HTTP to the successful App Server |
| Expected “configure Settings to connect to database” page observed | Yes |
| Troubleshooting captured | Yes |
| Duplicate test App Server cleanup | Not supplied / must still be verified if the lab session remains active |

### 2. SAA Takeaways

Be able to explain why **Public IP ≠ public subnet**, why **IGW attachment ≠ route**, and why **route ≠ security permission**. Reconstruct the path from browser to EC2 without looking at notes.

### 3. 🧒 3rd-Grade Understanding Check

Can I explain this sentence without AWS vocabulary?

> “The house has a public address, but visitors still need a road to the mansion gate, and the guard must let HTTP traffic through.”

If yes, map each part back to Public IPv4, route table, IGW, and Security Group.

### 4. What AWS Academy Prepared for Me

The classroom supplied controlled AWS access and pre-created items including `vockey` and `Inventory-App-Role`. The course supplied the application User Data and downloadable lab assets. The exact internal policies and every supporting resource prepared by Academy were not enumerated in the supplied lab record.

A personal AWS account must deliberately create or replace these prerequisites rather than assuming classroom resources exist.

### 5. Personal AWS Rebuild Decision

**🟢 Yes — Very High Priority / Very High Learning Value**

A from-scratch rebuild would test whether the VPC concepts are truly understood without relying on Academy defaults. The first version should reproduce the core concepts safely and cheaply; later versions can compare multi-AZ design, private administration with SSM, NAT/endpoint choices, Flow Logs, and production-grade HTTPS.

### 6. Follow-up Companion Lab

See [Lab 07 VPC personal companion plan](../personal-labs/lab-07-vpc-companion-plan.md). It is a plan only; no personal cloud deployment is claimed by this documentation.

### 7. Cleanup Check

Before ending any personal rebuild, inventory and remove or intentionally retain EC2 instances, EBS volumes, public IPv4 allocations, NAT gateways if added, endpoints, Internet Gateways, route tables, subnets, security groups, logs, IAM resources, and the VPC. Check all Regions used and document anything intentionally retained.

Class Lab -> Documentation -> SAA Review -> 3rd-Grade Check -> Personal Rebuild Decision -> Cleanup Review

上課 Lab -> 文件化 -> SAA 複習 -> 三年級理解檢查 -> Personal Rebuild 判斷 -> Cleanup 檢查
