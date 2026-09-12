# Lab 09 — Amazon Cognito Personal AWS Companion Rebuild Plan

## Decision

**🟢 YES — High Learning Value**

This is a future independent rebuild plan. It is **not** a record of resources already built in a personal AWS account.

## Objective

> **Do not merely replay the classroom lab. Rebuild the architecture that the classroom had prepared for me.**
>
> **不是只重做課堂步驟，而是把課堂事先準備好的架構，從零自己建立一次。**

The rebuild must be performed in a personal AWS account without AWS Academy pre-created lab infrastructure. ChatGPT should guide the rebuild concept-first and step-by-step.

Use the **newest AWS Console GUI available at the future rebuild date**. Do not depend on today's menu positions, screenshots, or button names when AWS has changed the console.

---

## Architecture to rebuild deliberately

```text
Browser / sample web app
        |
        v
Application delivery / hosting
        |
        v
Cognito User Pool
        |
        v
Authentication token
        |
        v
Cognito Identity Pool
        |
        v
Temporary AWS credentials
        |
        v
Authenticated IAM Role
        |
        v
Least-privilege IAM Policy
        |
        v
DynamoDB test table
```

The personal rebuild should reproduce the **learning architecture**, not necessarily the exact Birds application or classroom implementation.

---

## Build-from-zero checklist

### Phase 1 — Plan and cost guardrails

- Choose a Region deliberately.
- Review current Cognito, CloudFront, DynamoDB, S3, compute, and data-transfer pricing/free-tier behavior at rebuild time.
- Define a short-lived lab naming convention.
- Decide whether a static front end plus a small backend is sufficient instead of reproducing Cloud9 exactly.
- Create a cleanup checklist before creating resources.

### Phase 2 — Application foundation

Create the pieces that AWS Academy had already prepared or automated:

- A small test web application with public and protected pages
- Application hosting/delivery
- S3 and CloudFront if they remain appropriate for the architecture at rebuild time
- Backend/application server only if required by the chosen implementation
- DynamoDB test table

The goal is to understand why every resource exists.

### Phase 3 — Cognito User Pool

Create from scratch:

- User Pool
- Sign-in configuration
- Password policy
- Required attributes as appropriate
- App client
- Cognito domain / managed login configuration if appropriate
- Test users
- Optional admin group if it supports the learning goal

Validate:

```text
Protected page
 -> login
 -> Cognito User Pool
 -> successful authentication
 -> token returned to app
```

### Phase 4 — Cognito Identity Pool

Create an Identity Pool and connect it to the User Pool/app client.

Understand rather than copy:

- Why Identity Pool is separate from User Pool
- Authenticated vs unauthenticated identities
- How an authenticated User Pool token becomes evidence used in the Identity Pool flow
- How temporary AWS credentials are obtained

### Phase 5 — IAM from scratch

Create the authenticated IAM role and least-privilege policy deliberately.

Start narrow. The application should receive only the DynamoDB actions/resources needed for the test.

Verify the role trust relationship for Cognito identities and understand why Cognito can obtain credentials for the role.

### Phase 6 — Application integration

Configure the application with generated identifiers without hard-coding reusable documentation to one account.

Recognize ID formats:

```text
User Pool ID     -> region_xxxxx
Identity Pool ID -> region:UUID
```

Configure the login-provider mapping conceptually as:

```text
cognito-idp.<region>.amazonaws.com/<user-pool-id> -> authenticated token
```

### Phase 7 — End-to-end validation

Prove each boundary separately:

1. Public page works without authentication.
2. Protected page requires login.
3. User Pool authentication succeeds.
4. Application receives/uses the authentication token correctly.
5. Identity Pool obtains temporary AWS credentials.
6. IAM role/policy allows the intended DynamoDB action.
7. An action outside the allowed policy is denied.
8. No long-term AWS access key is embedded in the application.

### Phase 8 — Failure drills

Intentionally reproduce safe configuration mistakes and diagnose them:

- Swap User Pool ID and Identity Pool ID
- Remove/incorrectly configure the login-provider mapping
- Remove one required DynamoDB permission
- Point at the wrong DynamoDB resource

The goal is to recognize symptoms and trace the flow:

```text
Authentication problem?
 -> User Pool / app login flow

Temporary credential problem?
 -> Identity Pool / Logins provider mapping / trust

AWS AccessDenied after credentials work?
 -> IAM role/policy/resource scope
```

### Phase 9 — Cleanup

Before ending the personal lab:

- Remove temporary application resources
- Remove CloudFront/S3 resources if created solely for the lab
- Remove DynamoDB test table
- Remove Cognito Identity Pool
- Remove Cognito User Pool/app client/domain as appropriate
- Remove lab-only IAM roles and policies after dependencies are removed
- Remove any compute/backend resources
- Check billing/cost tools for unexpected remaining resources

---

## 3rd-grade architecture test

I should be able to explain the rebuild without AWS jargon:

> The membership desk proves who I am. It gives me proof of login. The temporary-pass desk accepts that proof and gives me a short-lived AWS pass. The job hat and rulebook decide which records room I can use and what I can do there.

> 會員中心先確認我是誰，給我登入證明；臨時通行證櫃台接受這個證明，給我短期 AWS 通行證；工作帽和規章再決定我可以進哪個資料房、可以做哪些事情。

If I cannot explain each arrow, the rebuild is not finished.

---

## Completion definition

The personal rebuild is complete only when I can:

- Build the prerequisites without Academy setup scripts
- Explain User Pool vs Identity Pool without memorization
- Trace token -> Identity Pool -> temporary credentials -> IAM role -> AWS resource
- Diagnose a broken identity/credential flow
- Demonstrate least privilege
- Clean up the resources safely
