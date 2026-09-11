# Lab 08 — VPC Peering Quick Review

[Full Lab 08 notes](../labs/lab-08-vpc-peering.md) | [Future personal rebuild](../personal-labs/lab-08-vpc-peering-companion-plan.md)

## Architecture in one picture

```text
Lab VPC 10.0.0.0/16
Application EC2
      |
      | route: 10.5.0.0/16 -> Lab-Peer
      v
   Lab-Peer
      ^
      | route: 10.0.0.0/16 -> Lab-Peer
      |
Shared VPC 10.5.0.0/16
MySQL database
```

> **Peering = bridge. Route tables = road signs at both ends.**
>
> **Peering = 橋；Route Tables = 橋兩端的路牌。**

## Must-know concepts

| Concept | Fast recall |
| --- | --- |
| VPC Peering | One-to-one private VPC connection |
| Active peering | Bridge exists, but routes still required |
| Lab -> Shared route | `10.5.0.0/16 -> peering` |
| Shared -> Lab route | `10.0.0.0/16 -> peering` |
| Flow Logs | Network-flow metadata |
| Protocol `6` | TCP |
| Port `3306` | MySQL |
| `ACCEPT` | Flow allowed |
| `REJECT` | Flow rejected |

## Real troubleshooting lesson

Symptom:

```text
Gateway Timeout
```

Diagnosis:

```text
Lab -> Shared route      correct
Shared -> Lab route      mistyped as 0.0.0.0/16
Required return route    10.0.0.0/16 -> Lab-Peer
```

AWS Academy did not permit `DeleteRoute`, so the incorrect non-overlapping route was left in place and the correct return route was added separately. The Inventory application then loaded successfully.

> **Active peering + wrong route = still broken.**
>
> **Peering 顯示 Active，不代表網路一定通。**

## Flow Log evidence

Sanitized pattern:

```text
10.0.0.x -> 10.5.1.x   ephemeral -> 3306   6   ACCEPT OK
10.5.1.x -> 10.0.0.x   3306 -> ephemeral   6   ACCEPT OK
```

This demonstrates request and response traffic between the application and database.

## 3rd-grade memory / 三年級記法

```text
VPC A/B      = 兩座大宅院
Peering      = 私人橋
Route Table  = 路牌
Flow Logs    = 警衛交通紀錄
3306         = MySQL 的門
6            = TCP
```

**有橋 ≠ 有路。兩邊都要有正確路牌。**

## Future personal rebuild

**🟢 Yes — Very High Learning Value**

Future rebuild must start from an empty personal AWS environment and recreate the architecture that AWS Academy prepared for us.

> **Use the newest AWS Console GUI available on the rebuild date. Do not memorize old screenshots or menu locations.**
>
> **未來重做時使用當天最新 AWS GUI，不死背現在的畫面位置。**
