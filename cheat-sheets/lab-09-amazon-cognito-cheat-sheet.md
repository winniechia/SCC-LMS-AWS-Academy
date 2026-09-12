# Lab 09 — Amazon Cognito Quick Review

## One-line memory

> **User Pool = authentication. Identity Pool = temporary AWS credentials. IAM Role/Policy = AWS authorization.**
>
> **User Pool = 驗證身分；Identity Pool = 臨時 AWS 憑證；IAM Role/Policy = AWS 資源授權。**

## Core flow

```text
App user
  -> Cognito User Pool
  -> authentication token
  -> Cognito Identity Pool
  -> temporary AWS credentials
  -> authenticated IAM role
  -> least-privilege IAM policy
  -> AWS resource (lab: DynamoDB)
```

## Mansion analogy / 大宅院

| AWS | 大宅院 |
| --- | --- |
| Cognito User Pool | 會員中心／驗身分的警衛 |
| User Pool token | 已驗證身分的證明 |
| Cognito Identity Pool | 臨時通行證櫃台 |
| Temporary credentials | 有期限的 AWS 通行證 |
| IAM Role | 工作帽／臨時職位 |
| IAM Policy | 權限規章 |
| DynamoDB | 資料帳房 |

## ID recognition

```text
User Pool ID     -> region_xxxxx
Identity Pool ID -> region:UUID
```

Do not swap them.

## Troubleshooting memory

**Login loop after successful Cognito sign-in**
- Check User Pool configuration and returned-token handling.
- Verify User Pool ID and Identity Pool ID are in the correct application settings.

**Authentication works but temporary credentials fail**
- Check `AWS.CognitoIdentityCredentials`.
- `IdentityPoolId` must reference the Identity Pool.
- `Logins` must contain the Cognito User Pool provider + authenticated token.

Conceptual provider:

```text
cognito-idp.<region>.amazonaws.com/<user-pool-id>
```

**npm `ENOENT package.json`**
- Verify the current working directory before troubleshooting npm itself.

## SAA recognition

- Customer sign-up/sign-in -> **Cognito User Pool**
- App user needs temporary AWS credentials -> **Cognito Identity Pool**
- What may those credentials do? -> **IAM Role + Policy**
- Corporate existing identity -> **Federation**
- Workforce + many AWS accounts -> **IAM Identity Center**

## Future personal rebuild

**🟢 YES — High Learning Value**

> Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.
>
> 不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。

Future rebuild requirements:
- Personal AWS account, not AWS Academy pre-created resources
- Create the required application/infrastructure prerequisites from scratch
- Create Cognito User Pool and Identity Pool deliberately
- Create least-privilege IAM roles/policies deliberately
- Validate token -> temporary credentials -> AWS resource access end to end
- Use the newest AWS Console GUI available at rebuild time
- Do not reuse classroom IDs, ARNs, credentials, endpoints, or screenshots as fixed instructions
- Finish with a cost/resource cleanup check
