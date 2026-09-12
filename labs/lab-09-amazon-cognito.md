# Lab 09 — Securing Applications with Amazon Cognito

## Lab status

**Class Lab: Complete**

This lab connected a web application to Amazon Cognito for authentication and temporary AWS credentials, then validated access to an Amazon DynamoDB table.

> **User Pool = prove who you are. Identity Pool = obtain temporary AWS credentials. IAM role/policy = decide what those credentials can do.**
>
> **User Pool = 驗證你是誰；Identity Pool = 取得臨時 AWS 憑證；IAM Role/Policy = 決定這些憑證可以做什麼。**

---

## What I built / 我建立了什麼

The classroom environment provided a Birds web application and supporting AWS resources. I configured the application to use Amazon Cognito and validated the end-to-end identity flow.

```text
Browser / Birds App
        |
        v
Cognito User Pool
Authentication: Who are you?
        |
        v
User Pool token
        |
        v
Cognito Identity Pool
        |
        v
Temporary AWS credentials
        |
        v
Authenticated IAM Role + Policy
        |
        v
DynamoDB BirdSightings table
```

The final validation successfully connected to the `BirdSightings` DynamoDB table. The table contained 0 rows; the important result was that the authenticated application could successfully query it.

---

## Core concepts / 核心觀念

### Cognito User Pool — authentication

A User Pool is the application user directory and authentication layer. In this lab, protected pages redirected users to Cognito login. After successful authentication, Cognito returned a token to the application.

**大宅院比喻：** User Pool 是會員中心／門口警衛，負責確認：「你真的是這位會員嗎？」

### Cognito Identity Pool — temporary AWS credentials

Authentication alone does not automatically give an application user permission to call AWS services. The Identity Pool accepts the authenticated identity and enables the application to obtain temporary AWS credentials.

**大宅院比喻：** User Pool 驗完身分後，Identity Pool 像臨時通行證櫃台，根據可信的身分證明發 AWS 臨時通行證。

### IAM role and policy — authorization

The authenticated role used a customer-managed policy that allowed limited DynamoDB read/write access to the lab's `BirdSightings` table.

The important separation is:

- Cognito User Pool: authentication
- Cognito Identity Pool: temporary AWS credentials
- IAM role/policy: authorization to AWS resources

Identity Pool credentials do **not** by themselves define DynamoDB permissions. The assumed IAM role and its policies define the allowed AWS actions and resources.

---

## Authentication vs authorization / 驗證與授權

**Authentication:** Who are you? / 你是誰？

**Authorization:** What are you allowed to do? / 你被允許做什麼？

A successful login proves identity. It does not mean the user automatically has unlimited AWS permissions.

---

## Important configuration patterns

### User Pool ID vs Identity Pool ID

These identifiers are easy to confuse.

```text
User Pool ID     -> region_xxxxx
Identity Pool ID -> region:UUID
```

Do not copy classroom-specific IDs into reusable documentation or source code.

The application configuration must place the correct ID in the correct field.

### Login provider mapping

When requesting Cognito Identity credentials, the `Logins` mapping must identify the Cognito User Pool login provider and provide the authenticated user's token.

Conceptually:

```javascript
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
  IdentityPoolId: CONFIG.COGNITO_IDENTITY_POOL_ID_STR,
  Logins: {
    "cognito-idp.<region>.amazonaws.com/<user-pool-id>": token
  }
});
```

The login-provider key contains the **User Pool ID**, not the Identity Pool ID.

---

## Troubleshooting lessons / 除錯紀錄

### 1. Login loop after Cognito sign-in

**Symptom:** Cognito recognized the admin as signed in, but returning to the Birds application caused the protected page to request login again.

**Root cause:** The application configuration had the User Pool ID and Identity Pool ID in the wrong places / an incorrect pool ID configuration.

**Fix:** Correct the User Pool and Identity Pool configuration, save the application files, redeploy the website, and test again.

**Lesson:** A successful Cognito-hosted login does not prove that the application is correctly processing the returned identity/token.

### 2. `There was a problem with your credentials.`

**Symptom:** Authentication succeeded, but validating temporary AWS credentials failed.

**Root cause:** The `Logins` object used to create `AWS.CognitoIdentityCredentials` was empty, so the Identity Pool was not receiving the User Pool login provider/token mapping.

**Fix:** Add the Cognito User Pool provider key and authenticated token to `Logins`, save, redeploy, and validate again.

**Lesson:**

```text
User Pool login success
        !=
Identity Pool credential success
```

The token must be correctly passed into the Identity Pool credential flow.

### 3. `npm start` failed with ENOENT

**Symptom:** npm could not find `package.json`.

**Cause:** `npm start` was executed from the wrong directory.

**Lesson:** When npm reports `ENOENT` for `package.json`, first verify the current working directory rather than assuming npm itself is broken.

---

## SAA-C03 takeaways

Recognize these patterns quickly:

| Scenario | Think first |
| --- | --- |
| Application customers need sign-up/sign-in | Cognito User Pool |
| Authenticated app users need temporary AWS credentials | Cognito Identity Pool |
| Temporary credentials need access to DynamoDB/S3/etc. | IAM role + least-privilege policy |
| EC2/Lambda needs AWS service access without long-term keys | IAM role for the workload |
| Existing corporate identity system | Federation |
| Workforce needs centralized access to many AWS accounts | IAM Identity Center |

### Key exam distinction

**User Pool is not Identity Pool.**

```text
User Pool
   -> authentication
   -> token

Identity Pool
   -> temporary AWS credentials

IAM Role/Policy
   -> AWS authorization
```

---

## 3rd-grade understanding check / 三年級理解

Imagine a large mansion.

- **User Pool** = the membership desk checks your identity.
- **Token** = proof that the membership desk checked you.
- **Identity Pool** = the temporary-pass desk accepts that proof.
- **Temporary AWS credentials** = the temporary pass.
- **IAM Role** = the job hat attached to the pass.
- **IAM Policy** = the rulebook describing which rooms/actions the job hat allows.
- **DynamoDB** = the records room.

> Having a valid membership card does not mean you can enter every room.
>
> 有會員身分，不代表可以進入大宅院裡的每一個房間。

---

## What AWS Academy prepared for me / 課堂事先準備了什麼

The classroom lab reduced setup time by providing or preparing major parts of the environment, including the Birds application, Cloud9-based development environment, website/deployment scaffolding, supporting infrastructure, DynamoDB data layer, and IAM resources used by the exercise.

Therefore, completing the classroom steps is **not the same as building the architecture independently from zero**.

This distinction is important for future personal practice.

---

## Personal AWS rebuild decision

### 🟢 YES — High Learning Value

Rebuild this architecture later in a personal AWS account **from scratch**, without relying on AWS Academy pre-created lab resources.

Future objective:

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。**

The future rebuild should use the **newest AWS Console GUI available on the rebuild date**, rather than depending on screenshots or button locations from this classroom session.

The independent rebuild should include creating or deliberately replacing the classroom prerequisites: application hosting/deployment, CloudFront/S3 delivery as appropriate, application server/backend requirements, DynamoDB table, Cognito User Pool, app client/domain, users/groups as needed, Cognito Identity Pool, authenticated/unauthenticated IAM roles, least-privilege policies, application configuration, and end-to-end validation.

Do not copy classroom account IDs, ARNs, pool IDs, endpoints, temporary credentials, or other ephemeral identifiers into the rebuild.

---

## Cleanup checkpoint

For the AWS Academy classroom lab, use the lab platform's normal **End Lab** procedure after all required validation/submission steps are complete. Do not manually delete classroom resources unless the lab instructions explicitly require it.

For the future personal rebuild, create a separate cleanup checklist and verify chargeable resources before ending the session.
