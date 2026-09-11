# Lab 08 — Creating a VPC Peering Connection

[Study index](../README.md) | [Printable cheat sheet](../cheat-sheets/lab-08-vpc-peering-cheat-sheet.md) | [Future personal rebuild](../personal-labs/lab-08-vpc-peering-companion-plan.md)

## Purpose and lab record

This AWS Academy Guided Lab connected two VPCs privately with a **VPC peering connection**, configured the required routes in both directions, enabled **VPC Flow Logs** to CloudWatch Logs, tested an application-to-database connection across the peering link, and analyzed the resulting MySQL traffic.

The class document defines these core objectives: create a VPC peering connection, configure route tables to use it, enable VPC Flow Logs, test the peering connection, and analyze the logs.

Sensitive and temporary identifiers are intentionally omitted from these notes: account IDs, VPC IDs, route table IDs, peering IDs, ENI IDs, EC2 instance IDs, exact public IPs, and RDS endpoint values.

## 1. Architecture

```text
Internet
   |
   v
Lab VPC 10.0.0.0/16
+----------------------------------+
| Public subnet                    |
|   Application EC2                |
|                                  |
| Private subnet                   |
+----------------------------------+
                |
                | VPC Peering
                | Lab-Peer
                v
Shared VPC 10.5.0.0/16
+----------------------------------+
| Private subnet 1                 |
|   MySQL database                 |
|                                  |
| Private subnet 2                 |
+----------------------------------+
```

### Mansion analogy / 大宅院比喻

```text
Lab VPC      = 大宅院 A
Shared VPC   = 大宅院 B
VPC Peering  = 兩座大宅院之間的私人橋
Route Table  = 橋兩端的路牌
Flow Logs    = 警衛室的車流紀錄
```

> **Peering = bridge. Route tables = road signs at both ends.**
>
> **Peering = 橋；Route Tables = 橋兩端的路牌。**

## 2. Create and accept the peering connection

The lab created a peering connection named `Lab-Peer` with:

```text
Requester: Lab VPC
Accepter:  Shared VPC
```

The request was accepted and the peering connection reached **Active** state.

A VPC peering connection is a one-to-one private network connection between two VPCs. Creating and accepting the peering object does **not** automatically make traffic flow. Routing still must be configured.

## 3. Configure routing in both directions

The Lab VPC route table needed:

```text
Lab Public Route Table
10.5.0.0/16 -> Lab-Peer
```

The Shared VPC route table needed the reverse path:

```text
Shared-VPC Route Table
10.0.0.0/16 -> Lab-Peer
```

This is the central networking lesson of the lab:

```text
Peering connection = bridge exists
Route table         = traffic knows how to reach the bridge
```

Both sides need correct routing for a request/response conversation to work.

> **有橋不代表能通；去程與回程都必須有正確的路。**
>
> **Having a bridge does not guarantee connectivity; both forward and return routing must be correct.**

## 4. Enable VPC Flow Logs

A Flow Log was created on the Shared VPC with the lab-prescribed settings:

```text
Name: SharedVPCLogs
Aggregation interval: 1 minute
Destination: CloudWatch Logs
Log group: ShareVPCFlowLogs
Traffic type: All
IAM role: lab-provided VPC flow-logs role
```

The Flow Log reached **Active** state and began delivering events to the CloudWatch log group.

### Mental model

> **Flow Logs = network traffic metadata, not packet/application contents.**
>
> **Flow Logs = 車流紀錄，不是拆開包裹看內容。**

Flow Logs can show source/destination addresses, ports, protocol, ACCEPT/REJECT, and other metadata useful for network troubleshooting.

## 5. Test the application-to-database path

The Inventory application was configured with the supplied database endpoint and the lab database settings. The expected working traffic path was:

```text
Browser
   |
Application EC2 in Lab VPC
   |
   | MySQL TCP 3306
   v
Lab Public Route Table
   |
   v
Lab-Peer
   |
   v
Shared VPC Route Table
   |
   v
MySQL database in Shared VPC
```

The Shared VPC had no Internet Gateway for the database path, so successful application-to-database connectivity demonstrated that the traffic was using the VPC peering connection.

## 6. Real troubleshooting incident — Gateway Timeout

### Symptom

After configuring the Inventory application and clicking **Save**, the browser returned:

```text
Gateway Timeout
The gateway did not receive a timely response from the upstream server or application.
```

The browser could already reach the Application EC2, so the failing hop was likely behind the application:

```text
Browser -> Application EC2    working
Application EC2 -> database   investigate
```

### Evidence-based diagnosis

The Lab Public Route Table was correct:

```text
10.5.0.0/16 -> Lab-Peer
```

The Shared-VPC Route Table contained a mistyped return route:

```text
0.0.0.0/16 -> Lab-Peer   # incorrect
```

instead of:

```text
10.0.0.0/16 -> Lab-Peer  # correct Lab VPC CIDR
```

The forward road existed, but the correct return road did not.

### Academy permission constraint

Attempting to replace/delete the incorrect route failed because the temporary AWS Academy role was not authorized to perform `ec2:DeleteRoute`. The console reported that the edit was reverted.

Because the incorrect `0.0.0.0/16` route did not overlap the required `10.0.0.0/16` destination, the safe lab recovery was to **leave the harmless incorrect route in place and add the correct route separately**:

```text
10.0.0.0/16 -> Lab-Peer
```

After the correct return route was added, the Inventory application successfully displayed database records.

### Troubleshooting lesson

> **Gateway Timeout -> identify the failing hop -> inspect both route tables -> compare destination CIDRs exactly -> repair the return path.**

This incident reinforced that a one-digit/one-octet CIDR mistake can break a distributed application even when the peering connection itself is Active.

## 7. Analyze MySQL traffic in CloudWatch Flow Logs

The `ShareVPCFlowLogs` log group contained an ENI log stream. Filtering the events for `3306` revealed the database traffic.

Observed traffic pattern, sanitized:

```text
10.0.0.x  -> 10.5.1.x   ephemeral-port -> 3306   protocol 6   ACCEPT OK
10.5.1.x  -> 10.0.0.x   3306 -> ephemeral-port   protocol 6   ACCEPT OK
```

Interpretation:

| Field | Meaning |
| --- | --- |
| `10.0.0.x` | Application side in Lab VPC |
| `10.5.1.x` | Database side in Shared VPC |
| `3306` | MySQL port |
| Protocol `6` | TCP |
| `ACCEPT` | Traffic was allowed |
| `OK` | Flow Log record status was normal |

The reverse-direction records were especially useful evidence that both sides of the conversation were working.

> **Protocol 6 = TCP. Port 3306 = MySQL.**

## 8. 3rd-grade explanation / 三年級也能懂

Imagine two separate mansion estates:

```text
🏡 Mansion A = Lab VPC
🏡 Mansion B = Shared VPC
🌉 Private bridge = VPC Peering
🪧 Road signs = Route Tables
📖 Gate traffic notebook = VPC Flow Logs
🗄️ Database room = MySQL
```

Building the bridge is not enough. Both mansions need signs that say how to reach the other mansion.

When the return sign was accidentally written as `0.0.x.x` instead of `10.0.x.x`, the database response could not find the right road home. After the correct sign was added, the application worked.

Then CloudWatch was like checking the security guard's notebook and seeing:

> "Yes, cars traveled between Mansion A and Mansion B on the MySQL road, and the traffic was ACCEPTED."

## SAA takeaways

- VPC Peering is a **one-to-one** private connection between VPCs.
- The peering connection must be accepted before it becomes Active.
- Peering does not automatically update route tables.
- Both VPCs need correct routes for bidirectional application traffic.
- Exact CIDR matching matters.
- VPC Peering is not an Internet Gateway and does not require the database VPC to expose the database publicly.
- VPC Flow Logs provide network-flow metadata useful for troubleshooting.
- Protocol number `6` means TCP.
- MySQL commonly uses TCP port `3306`.
- `ACCEPT` and `REJECT` are key Flow Log fields.
- A Gateway Timeout can indicate an upstream network/application dependency problem; troubleshoot the failing hop instead of rebuilding everything.

## Lab Completion Checkpoint / Lab 結束檢查點

### Class Lab progress

**Core technical tasks completed and verified:**

```text
[✅] VPC peering created and accepted
[✅] Forward route configured
[✅] Reverse route corrected and verified
[✅] VPC Flow Logs enabled
[✅] Application -> database path tested successfully
[✅] Flow Logs analyzed for MySQL TCP/3306 traffic
```

Submission/grade status should be recorded separately after the Academy Submit step if needed.

### Personal AWS Rebuild Decision

**🟢 Yes — Very High Learning Value**

This lab is an excellent future personal-account rebuild because it forces us to construct and verify two VPCs, non-overlapping CIDRs, route tables, a private peering connection, application/database security, Flow Logs, CloudWatch logging, and failure diagnosis from first principles.

See the dedicated companion plan:

`personal-labs/lab-08-vpc-peering-companion-plan.md`

### Future rebuild rule

> **Rebuild from scratch using the AWS Console UI that is current on the day of the rebuild—not screenshots or button locations frozen in this 2026 classroom lab.**
>
> **未來從零重建時，以當天最新 AWS Console GUI 為準，不死背 2026 課堂文件的按鈕位置。**

The concepts and target architecture remain authoritative; console navigation must be re-verified against the current AWS UI at rebuild time.
