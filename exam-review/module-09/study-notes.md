# Module 09 — Securing User, Application, and Data Access
AWS SAA-C03 | 中英雙語考前筆記 | 2026-10-24

> Learning sequence / 學習方式：三年級比喻 → 技術與用途 → Exam keywords 與取捨 → 排除方法 → 一次一題。
> Source: uploaded Module 9 Student Guide screen recording, `Student guide - Google Chrome 2026-10-08 15-52-32.mp4`, and the learner's Module 9 handoff. The recording opened successfully (1920 × 1032; approximately 5 minutes 46 seconds). Sampled readable frames confirmed the title and topics below; scrolling, blank/loading frames, and partially visible pages prevented exhaustive reading. This is a record of actual tutoring, not a complete transcription.
> Tutoring completed; no AWS resources were created, and no hands-on labs were executed or verified. 教學完成不代表實作 Lab 完成。
> GitHub destination: `winniechia/SCC-LMS-AWS-Academy/exam-review/module-09/study-notes.md`. Module 8 notes were read to follow the established format. Fetching the Module 9 path returned 404 before creation. No PDF was requested or created.

## 0. Learning Results / 學習成果

| Category / 類別 | Correct / Total | Result |
|---|---:|---:|
| Original scored questions / 原始計分題, including final review | 20/24 | 83.3% |
| Scored targeted reviews / 計分重測 | 3/4 | 75% |
| **Cumulative guided practice / 累計引導練習** | **23/28** | **82.1%** |
| Final four questions / 最後四題, subset of original items | 4/4 | 100% |

- Guided practice included immediately preceding explanations, analogies, repeated concepts, and unscored checks. It was not an independent mock exam and does not establish independent mastery or predict the certification result.
- Q4, Q6, Q9, and Q17 are scored targeted reviews. Q26–Q29 are the four original final-review questions; the final-review row is a subset, not four additional questions.
- Q25 repeated a previously disclosed check. Its D response was correct, but it was excluded from scoring as an unscored demonstration; the attempted retest was voided. This was an assistant question-design issue, not a learner error.
- Eight additional unnumbered checks were unscored (seven correct, one incorrect). None was added to the denominator.
- Final review ended at Q29. No extra questions were added. After the pacing update, Q19–Q29 comprised ten scored questions and one unscored demonstration, within the stated maximum of twelve further scored questions.
- Planning time estimates were not measured study duration.
- Review priorities: trust versus permissions policies; workforce versus customer identity; encrypted versus plaintext data keys and their order of use. Correct responses after explanation do not prove independent mastery.

### Coverage / 已教學範圍

| Checkpoint | Topics |
|---|---|
| 1 | IAM users/groups/policies, explicit Deny, ABAC and tag-change control |
| 2 | Federation, IdP, Identity Center, Cognito user/identity pools, STS, roles, EC2 credentials |
| 3 | Organizations, SCP, OU inheritance, Control Tower |
| 4 | Transit/rest encryption, symmetric/asymmetric encryption, KMS, envelope encryption, client/server encryption, SSE-S3/SSE-KMS |
| 5 | WAF, Shield, Macie, Inspector, Detective, Security Hub, Trusted Advisor |
| 6 | Exactly four final-review items; documentation |

## 1. IAM Groups, Policies, and Deny / 群組、政策與拒絕

**Third-grade analogy / 三年級比喻:** IAM user 是老師；IAM group 是科學老師組；policy 是「可以進實驗室」的規則單。把共同政策附加到群組，新老師加入群組就受到共同政策管理。

An IAM group collects IAM users; group policies grant permissions to members. A group is not a role that an application assumes. Cognito users are not IAM users and cannot simply be added to IAM groups.

| Policy situation | Result / 結果 |
|---|---|
| No applicable Allow | Implicit deny / 隱含拒絕 |
| Applicable Allow, no applicable Deny, all other required controls permit | Allowed / 允許 |
| Applicable explicit Deny | Denied despite another Allow / 明確拒絕蓋過允許 |

Neither user-versus-group attachment nor policy attachment date establishes precedence. For example, a user-level Deny on `s3:DeleteObject` blocks the action even if a group allows it.

**Exam keywords:** same duties, many IAM users, reduce administration, multiple policies, explicit deny.
**Elimination:** Identify the action and resource, inspect all applicable controls, and do not stop at the first Allow.

## 2. ABAC and Tag Governance / 屬性授權與標籤管理

**ABAC = Attribute-Based Access Control / 以屬性為基礎的存取控制。**
**Principal / 主體:** the requesting identity, such as an IAM user or role.

**Analogy:** 老師的識別證寫著 `Project=Garden`，教室也有專題標籤。警衛遵守「標籤相符才放行」的規則。識別證提供資料，規則才授權。

A policy can compare principal and resource attributes for supported actions/resources. Matching tags alone grant no access.

| Principal tag | Resource tag | Matching-tag Allow condition |
|---|---|---|
| Garden | Garden | Satisfied; other applicable controls still matter |
| River | Forest | Not satisfied; no Allow from this statement |

If no other policy grants access, a mismatch produces implicit deny, not automatically explicit Deny.

**Security trade-off:** A shared policy reduces project-by-project maintenance, but authorization-related tag changes must be controlled. Someone able to freely change their own Project tag might gain another project's access. Not every service/action supports every tagging condition; verify support.

**Learner evidence:** Q3 D confused tags with authorization; Q4 C correctly applied differing values. Q26 B identified self-retagging risk.
**Exam keywords:** matching tags, many projects, shared policy, least ongoing maintenance.
**Elimination:** Separate tag data from the policy that uses it; inspect who may change the attributes.

## 3. Federation, Workforce Identity, and Customer Identity / 身分聯合與登入對象

**IdP = Identity Provider / 身分提供者。**
**SSO = Single Sign-On / 單一登入。**

**Train analogy:** 學校驗證學生身分，車站信任學校證明，再按規則發限時車票。每個車站不必重新建立永久學生帳號。

- Authentication / 身分驗證: Who are you?
- Authorization / 授權: What may you do?
- Federation / 身分聯合: Trust authentication from an external identity provider and map it to authorized access.

| Requirement | Service / 服務 |
|---|---|
| Employees use corporate identities across AWS accounts | IAM Identity Center |
| Customers register/sign in to an application | Cognito user pools |
| App receives temporary AWS credentials using supported identity tokens | Cognito identity pools |

**LFM analogy:** 你與工作夥伴登入 AWS Console 管理帳號，考慮 Identity Center；讀者登入私人故事網站，考慮 Cognito user pools。This is a hypothetical learning design, not a change to the production site.

**Cognito distinction:**
- User pool = identity desk / 驗證乘客身分，發出登入 tokens.
- Identity pool = temporary pass desk / 交換取得受 IAM role 權限限制的暫時 AWS credentials.
- JWT = JSON Web Token. It is not an AWS access key, secret access key, or STS session token.
- Both pools can work together, but an app using its own backend may not need client-side AWS credentials. Supported external providers can also work with identity pools.

**Exam approach:** Identify who signs in and where: employee to AWS account, customer to app, or app to an AWS API. Expiration changes do not convert a JWT into AWS credentials.

## 4. Roles, Trust, STS, and EC2 / 角色、信任與暫時憑證

**STS = Security Token Service / 安全權杖服務。**

**Analogy:** Trust policy 是朋友允許你進家門；permissions policy 是進屋後可以看書、不能開保險箱。增加「可以看書」不會解決尚未獲准進門的問題。

| Policy | Question / 問題 |
|---|---|
| Trust policy / 信任政策 | Who may assume the role? / 誰可以取得角色？ |
| Permissions policy / 權限政策 | What may the session do afterward? / 取得角色後可以做什麼？ |

Cross-account example:
1. Caller has authorization for `sts:AssumeRole` on the target.
2. Target role's trust policy permits the caller.
3. STS issues temporary credentials.
4. Resource operations require the session's applicable permissions and other required controls.

Successful role assumption does not guarantee S3 access. A role lacking `s3:GetObject` needs the relevant permissions authorization, not another invitation to assume the role.

**Temporary credentials:** access key ID + secret access key + session token, with expiration.
For EC2 applications, attach a role through an instance profile. Standard AWS SDK credential providers can retrieve/refresh instance-role credentials. Avoid stored long-term keys; apply least privilege.

**Exam keywords:** AssumeRole fails, trust relationship, successful assumption but resource denied, EC2 application, no long-term keys.
**Elimination:** Locate the failing step before changing a policy.

## 5. Organizations, OUs, SCPs, and Control Tower / 多帳號治理

**OU = Organizational Unit / 組織單位。**
**SCP = Service Control Policy / 服務控制政策。**

**Train analogy:** Organizations 是鐵路總公司；accounts 是車站；OU 把訓練站分組；SCP 是總公司限制「不得拆除監視器」；IAM policy 是主管的工作許可。

- SCP sets permission boundaries; it does not grant permissions.
- Applicable OU policies are inherited.
- A lower-level Allow cannot override an inherited explicit Deny.
- SCP allow-list designs also require the action to be allowed through the relevant hierarchy; merely avoiding an explicit Deny is not always enough.
- Ordinary IAM administrators in governed member accounts remain constrained.
- Scope clarification from official documentation: SCPs do not constrain management-account users/roles or service-linked roles. Do not generalize the classroom example to all identities everywhere.

**Control Tower analogy:** 新車站開站套件。It helps set up/govern a multi-account landing zone using Organizations and other services.
- Landing zone: standardized foundation.
- Account Factory: repeatable account provisioning.
- Controls: preventive, detective, and proactive governance mechanisms; not all controls are SCPs.

**Trade-off:** Central governance reduces repeated account setup; configuration and hierarchy still require planning.
**Exam approach:** Separate granting access, limiting access, authenticating people, and provisioning governed accounts.

## 6. Encryption Location and Type / 加密位置與類型

**Diary analogy / 日記比喻:** 日記運送途中需要保護；存進倉庫後也需要保護。

| Distinction | Meaning |
|---|---|
| In transit / 傳輸中 | HTTPS/TLS protects network transmission |
| At rest / 靜態 | Encryption protects stored content |
| Client-side / 用戶端 | App encrypts before upload; S3 receives encrypted content |
| Server-side / 伺服器端 | S3 encrypts before storage after receiving content |

TLS = Transport Layer Security. Both client-side and server-side encryption can use TLS. Storage encryption does not replace HTTPS. HTTPS alone does not meet a requirement that document content be encrypted within the app before departure.

| Cryptography type | Analogy | Trade-off |
|---|---|---|
| Symmetric / 對稱式 | 同一把密鑰鎖上與打開 | Efficient for bulk data |
| Asymmetric / 非對稱式 | 公開鎖上、私人鑰匙打開 | Public/private separation for suitable encryption workflows |

For asymmetric encryption, public key encrypts and private key decrypts. Signing has a different direction/use and was not assessed here. Base64 is encoding, not encryption.

**Exam approach:** Locate the encryption boundary and identify the required key relationship; do not assume one type is universally better.

## 7. KMS and Envelope Encryption / 金鑰管理與信封加密

**KMS = Key Management Service / 金鑰管理服務。**

**Central analogy / 核心比喻:**
- Locked diary box = encrypted data / 加密資料.
- Usable small key = plaintext data key / 明文資料金鑰.
- Locked small box containing that key = encrypted data key / 加密的資料金鑰.
- KMS key protects the small key / KMS 金鑰保護資料金鑰.

Envelope encryption encrypts data with a data key, then encrypts that key under another key.

### Encryption / 加密

1. For a typical symmetric `GenerateDataKey` request, KMS returns plaintext and encrypted versions of a data key.
2. App uses the plaintext data key to encrypt the document.
3. Remove the plaintext data key from memory when no longer needed.
4. Persist encrypted document + encrypted data key.

**不要把可直接開箱的鑰匙貼在箱子外。Do not store the plaintext data key beside the encrypted document.**

### Decryption / 解密

1. Send encrypted data key to KMS for authorized decryption.
2. KMS returns the plaintext data key.
3. App uses that plaintext data key to decrypt the document.
4. Remove plaintext key material from memory when finished.

**先打開鑰匙盒，再拿鑰匙開日記箱。Open the key box first, then use the recovered key on the diary box.**
The encrypted data key cannot directly decrypt the document. A newly generated unrelated data key cannot decrypt old ciphertext.

### Key availability / 金鑰可用性

An alias is a label, not cryptographic key material. Q28 explicitly assumed the only capable KMS key was permanently deleted and no other recovery copies existed. A new key with the same alias cannot restore the old data keys. Real designs must account for key retention and any actual replicas/recovery mechanisms. Do not equate scheduled deletion with completed deletion.

Standard symmetric KMS encryption keys are not downloaded in plaintext in this workflow. This statement is scoped to these encryption keys, rather than every possible KMS feature/key type.

**Learner evidence:** Q16 A stored plaintext keys; Q17 A tried using an encrypted key directly. Several unscored analogies helped. Q25 was a disclosed repeat, excluded. Q28 showed correct key-loss reasoning, but does not independently prove the full decryption sequence is mastered.

**Exam approach:** Name each object before tracing it: document, plaintext data key, encrypted data key, KMS key. Separate what is stored from what is used during computation.

## 8. SSE-S3, SSE-KMS, and Two Access Checks / S3 加密與兩道授權

SSE = Server-Side Encryption.

| Option | Key management |
|---|---|
| SSE-S3 | S3-managed keys |
| SSE-KMS | KMS keys; choose a customer managed key when control of its key policy is required |

**Warehouse analogy:** 倉庫用自己的鎖，或使用受 KMS 管理、有獨立使用規則的鎖。Both are server-side encryption.

Read an SSE-KMS object:
- `s3:GetObject` permits object retrieval.
- `kms:Decrypt` authorization permits use of the relevant KMS key for decryption.
- Key policy and other applicable controls must permit the operation; an S3 Allow alone is insufficient.
- Upload/envelope operations can require `kms:GenerateDataKey`; multipart flows can need additional KMS permissions. Verify the exact operation before implementation.
- KMS keys/requests can add cost and operational dependencies; no price estimate was established.
- S3 default encryption does not mean a chosen customer managed KMS key is automatically used.

**Analogy:** 能拿到日記箱，不代表能使用解鎖的金鑰。
**Exam approach:** Check object authorization and key authorization separately; more S3 Allow statements do not fix missing KMS access.

## 9. Security Service Selection / 安全服務辨識

**Train-station analogy / 車站比喻:** 檢查入口內容、阻擋大量人潮、找敏感文件、查設備弱點、調查事件，與集中看板是不同工作。

| Service | Purpose / 用途 | Keywords |
|---|---|---|
| AWS WAF | Inspect/filter web requests / 檢查網頁請求 | HTTP/HTTPS, SQL injection, web ACL |
| AWS Shield | DDoS protection / 分散式阻斷服務防護 | Overwhelming attack traffic |
| Amazon Macie | Discover/classify S3 sensitive data / 發現敏感資料 | PII, identification/payment data, S3 |
| Amazon Inspector | Vulnerability assessment / 軟體弱點評估 | Known vulnerable packages, supported workloads |
| Amazon Detective | Investigate suspicious activity/relationships / 調查事件 | Understand what happened, related activity |
| AWS Security Hub | Unified security findings/risk experience / 集中安全發現 | Central security review |
| AWS Trusted Advisor | Broader AWS best-practice advice / 最佳實務建議 | Cost, security, performance, fault tolerance, quotas |

PII = Personally Identifiable Information / 個人可識別資訊.
DDoS = Distributed Denial of Service / 分散式阻斷服務.
SQL injection = malicious database instructions embedded in application input.

- WAF and Shield can complement each other; they are not universal substitutes.
- Macie findings do not automatically encrypt data or revoke every access path.
- Inspector identifies vulnerabilities; Detective investigates activity. Neither is a generic replacement for the other.
- Trusted Advisor check availability depends on current support/service offerings; no exhaustive current category list or support-tier entitlement was taught.
- **Current-documentation clarification:** The older course uses “Security Hub” broadly. Current AWS docs distinguish Security Hub's unified prioritization/response experience and **Security Hub CSPM (Cloud Security Posture Management)** for posture/compliance checks and related findings/integrations. Q24's Security Hub answer remains the course-level fit; the terminology is not an implementation guide.
- No service lifecycle/new feature was used to infer SAA-C03 inclusion or exclusion.

**Exam approach:** Identify the verb: block web content, resist DDoS, discover sensitive data, assess vulnerability, investigate, centralize, or optimize.

## 10. Wrong-Answer Review / 錯答複習

| Item | Response → Correct | Correction | Follow-up evidence |
|---|---|---|---|
| Q3 ABAC | D → A | Tags alone do not grant access / 需要政策 | Unscored No; Q4 and Q26 correct |
| Q5 workforce identity | A → B | Identity Center for workforce AWS access / 員工 AWS 存取 | Unscored Cognito customer check and Q6 correct |
| Q8 trust | A → B | Missing role trust is not fixed by S3 permissions / 信任與操作分開 | First unscored permissions check missed; later check/Q9 correct |
| Q16 persisted keys | A → D | Persist encrypted data key, not plaintext / 保存加密金鑰副本 | Unscored storage check correct |
| Q17 envelope sequence | A → C | Decrypt key first; plaintext key decrypts data / 先解金鑰再解資料 | Several scaffolded checks; Q25 excluded repeat; Q28 key-loss reasoning correct |

Future practice should use fresh, independently answered scenarios. The session ended at the agreed final review; these are later priorities, not more questions added now.

## 11. Complete Answer Log / 完整作答紀錄

Plain-text displayed option order was fixed. No screenshot order correction or invalid input occurred. Q19 was answered B after a pacing question and counted once.

| Q | Topic / 主題 | Given | Correct | Classification | Result |
|---|---|---|---|---|---|
| 1 | IAM group for shared developer permissions | B | B | Original | Correct |
| 2 | Explicit Deny overrides Allow | D | D | Original | Correct |
| 3 | ABAC needs policy, not tags alone | D | A | Original | Incorrect |
| 4 | River principal vs Forest resource tag | C | C | Targeted | Correct |
| 5 | Corporate SSO across AWS accounts | A | B | Original | Incorrect |
| 6 | Engineers use corporate identities for Console | D | D | Targeted | Correct |
| 7 | Direct S3 access needs identity-pool credentials | A | A | Original | Correct |
| 8 | Cross-account AssumeRole missing trust | A | B | Original | Incorrect |
| 9 | Which configuration belongs in trust policy | C | C | Targeted | Correct |
| 10 | EC2 role avoids stored long-term keys | B | B | Original | Correct |
| 11 | SCP prevents disabling CloudTrail | D | D | Original | Correct |
| 12 | OU explicit Deny remains effective | B | B | Original | Correct |
| 13 | Control Tower standardized landing zone | A | A | Original | Correct |
| 14 | TLS protects transmission | C | C | Original | Correct |
| 15 | Symmetric encryption for bulk data | B | B | Original | Correct |
| 16 | Persist encrypted document and encrypted data key | A | D | Original | Incorrect |
| 17 | KMS decrypts data key before backup decryption | A | C | Targeted | Incorrect |
| 18 | Encrypt before leaving application | C | C | Original | Correct |
| 19 | Customer managed KMS key for key-policy control | B | B | Original | Correct |
| 20 | SSE-KMS read lacks kms:Decrypt | C | C | Original | Correct |
| 21 | WAF blocks SQL injection requests | B | B | Original | Correct |
| 22 | Macie discovers S3 sensitive data | D | D | Original | Correct |
| 23 | Inspector finds software vulnerabilities | A | A | Original | Correct |
| 24 | Security Hub centralizes findings | C | C | Original | Correct |
| 25 | Plaintext data key decrypts document; disclosed repeat | D | D | Unscored demonstration | Excluded |
| 26 | ABAC self-retagging escalation risk | B | B | Original final review | Correct |
| 27 | JWT is not AWS credentials; identity-pool exchange | C | C | Original final review | Correct |
| 28 | Same KMS alias cannot recreate deleted key material | B | B | Original final review | Correct |
| 29 | Detective investigates related suspicious activity | D | D | Original final review | Correct |

### Unnumbered checks / 不計分確認

| After item | Check | Response | Result |
|---|---|---|---|
| Q3 | Matching tags alone without policy allow access? | No | Correct |
| Q5 | LFM reader email/password sign-in: Identity Center or user pool? | B: user pool | Correct |
| Q8 | Successful AssumeRole, missing GetObject: which policy? | A: trust | Incorrect; B permissions explained |
| Q8 follow-up | “Role may delete S3 objects”: which policy? | B: permissions | Correct |
| Q16 | Which data key is persisted beside ciphertext? | B: encrypted | Correct |
| Q17 | Open diary box or key box first? | B: key box | Correct |
| Q17 follow-up | Unopened key box represents which key? | A: encrypted data key | Correct |
| Q17 follow-up | KMS returned plaintext data key; which key decrypts data? | B: plaintext | Correct |

These eight checks are not eight scored questions. Q25 is separately logged as an unscored numbered demonstration; there are nine unscored responses total.

## 12. Personal AWS Rebuild — Completion Checkpoint

**Worth rebuilding? / 值得重建嗎？** Yes, selectively. Start with IAM policy reasoning and a small S3/KMS authorization exercise. A full Control Tower landing zone or fleet of security services is unnecessary for the first rebuild.

**Objectives / 學習目標:**
1. Explain trust vs permissions without relying on option letters.
2. Verify S3 access and KMS decryption as separate checks.
3. Trace data-key generation, storage, and decryption correctly.
4. Design ABAC with controlled tag modification.
5. Distinguish customer login, workforce login, and AWS API credentials.

**Non-production LFM example / 虛構範例:** Store a fictitious diary document in an isolated test bucket with a test customer managed KMS key. Compare a test role with S3 access only against one with the required KMS authorization. Use mock stories, no production keys, no real caregiving records, and no changes to the live LFM site.

| Proposed exercise | Scope |
|---|---|
| Paper role-policy exercise | Identify trust/caller permissions/resource permissions; no resources |
| S3 + KMS exercise | Tiny isolated object, least-privilege test role, authorized/denied read comparisons |
| Optional envelope exercise | Trace keys using a supported SDK; never print/persist plaintext keys in logs |
| Optional Cognito design | Draw user-pool login vs identity-pool AWS credentials before provisioning |
| Governance/security services | Start with paper selection cases; avoid organization-wide enablement for this rehearsal |

**Potential costs / 可能費用:** S3 storage/requests, KMS key and API usage, optional EC2/EBS/public IPv4, CloudTrail/log storage, Cognito usage, and any enabled WAF/security-service assessments. Control Tower can configure underlying billable services. Check current regional pricing and exercise duration before provisioning; no dollar quote was established.

**Cleanup / 清理:**
- Remove test objects, including versions/delete markers if enabled, then delete the test bucket when appropriate.
- Remove test instance profiles, roles/policies and optional Cognito resources after dependencies are removed.
- Terminate optional EC2; inspect retained EBS, snapshots, public IPv4, and logs.
- Disable/delete optional test WAF and assessment resources; check retained data and recurring charges.
- For KMS, first decide whether any retained ciphertext must remain recoverable. Do not schedule key deletion while needed data depends on it. Schedule deletion only for disposable test keys after reviewing dependencies; verify pending and final status.
- Never delete production LFM resources or organizational accounts as part of test cleanup.

**Authorization boundary:** Planning only. No rebuild, resource creation, identity changes, or lab verification occurred. Begin provisioning only when the learner explicitly asks.

## 13. Coverage Limits and Official Checks / 範圍限制與官方查核

The recording listed guided labs for securing applications with Cognito and encrypting data at rest. Neither was run here. Full SAML/OIDC message flows, production key rotation, live tag enforcement, actual account provisioning, and security-service deployment were not independently exercised. SAML/OIDC appeared in the guide but were not given a dedicated scored lesson; do not claim exhaustive guide coverage.

Official references checked during final documentation (2026-10-24) clarify the taught distinctions; they are paraphrased, not copied course pages:

- [Organizations SCP scope and permission limits](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [Cognito user-pool login with identity-pool AWS credentials](https://docs.aws.amazon.com/cognito/latest/developerguide/amazon-cognito-integrating-user-pools-with-identity-pools.html)
- [KMS cryptography and envelope encryption](https://docs.aws.amazon.com/kms/latest/developerguide/kms-cryptography.html)
- [Client-side envelope decryption sequence](https://docs.aws.amazon.com/kms/latest/cryptographic-details/client-side-encryption.html)
- [S3 SSE-KMS operation and permissions](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)
- [KMS deletion and recovery limitations](https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html)
- [Security Hub and Security Hub CSPM distinction](https://docs.aws.amazon.com/securityhub/latest/userguide/what-are-securityhub-services.html)
- [Macie findings and Security Hub CSPM integration](https://docs.aws.amazon.com/macie/latest/user/findings.html)

**Next module / 下一模組:** Read the actual Module 10 guide before choosing its title/topics. Carry forward the three review priorities without restarting completed modules. This session created no diagrams requested by the learner; the key-box and role-policy analogy tables above preserve the actual explanatory aids.

