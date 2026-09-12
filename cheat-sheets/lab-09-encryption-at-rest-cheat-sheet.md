# Lab 09 — Encryption at Rest Quick Review

## One-line memory

> **KMS manages/protects keys. S3 and EBS store encrypted data. CloudTrail records API activity.**
>
> **KMS 管鑰匙；S3/EBS 放加密資料；CloudTrail 記錄操作。**

## Mansion analogy / 大宅院

| AWS concept | 大宅院 |
| --- | --- |
| S3 | 大倉庫 |
| EBS | EC2 旁邊的私人儲藏室 |
| Data key | 真正鎖/解資料的工作鑰匙 |
| KMS key | 保護工作鑰匙的主鑰匙 |
| AWS KMS | 中央鑰匙管理室 |
| CloudTrail | 警衛操作紀錄簿 |

## Envelope encryption

```text
Data
  -> encrypted with data key
Data key
  -> protected by KMS key
```

**Memory:** KMS protects the key that protects the data.

## Lab proof

```text
Encrypted EBS + enabled KMS key
    -> attach works

Encrypted EBS + disabled KMS key
    -> fresh attach fails

Re-enable KMS key
    -> attach works again
```

The data is not erased when the KMS key is disabled. The required cryptographic operation is blocked.

## Why detach first?

An attached encrypted volume can already have the usable data key in memory. Detaching and performing a fresh attach forces the key-access flow to occur again.

## CloudTrail events worth recognizing

- `DisableKey`
- `AttachVolume`
- `CreateGrant`
- `Decrypt`
- `GenerateDataKeyWithoutPlaintext`
- `RetireGrant`

Think: **Who called what AWS API, and when?**

## SAA recognition

- Encryption at rest + key control -> think **KMS**
- EBS encrypted with customer controlled AWS key -> **customer managed KMS key**
- Need API audit history -> **CloudTrail**
- Key disabled -> cryptographic use blocked until enabled
- Key administrator and key user are different permission concepts
- Symmetric customer managed KMS key -> know automatic key rotation concept

## S3 reminder

The classroom lab reviewed default S3 server-side encryption with SSE-S3. On SAA questions, read the key-management/control requirement carefully before choosing an S3 encryption option.

## Future personal rebuild

**🟢 YES — Very High Learning Value**

> Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.
>
> 不是只重做課堂步驟，而是把 AWS Academy 事先準備好的架構，從零自己建立一次。

Use the newest AWS Console GUI available at rebuild time. Do not reuse Academy account IDs, ARNs, volume/instance IDs, or other temporary identifiers.
