# Module 11 — Automating Your Architecture
AWS SAA-C03 | 中英雙語考前筆記 | Completed 2026-10-25 (America/Los_Angeles)

> Learning method / 學習方式：三年級比喻 → 原理 → keywords and trade-offs → fixed-order questions → feedback.
> Sources / 教材：Uploaded Module 11 handoff, read fully, and Student guide - Google Chrome 2026-10-08 18-05-22.mp4, 302.975933 seconds. Both opened. Video frames were sampled every five seconds; blank, moving, and partial pages prevented complete page-by-page reading. These are tutoring notes, not a full guide transcription.
> No AWS resources were created. No hands-on labs were executed or verified here. 教學完成不等於 Lab 實作完成。
> Module 10 notes were read for format. The Module 11 destination returned 404 before creation. No PDF was requested or created.

## 0. Results and scoring audit / 成績與計分核對

| Category / 類別 | Correct / Total | Percentage |
|---|---:|---:|
| Original questions Q1–Q20 | 15/20 | 75.0% |
| Valid final review Q22–Q24 | 2/3 | 66.7% |
| Separate scored targeted retests | 0/0 | Not administered |
| **Cumulative guided practice** | **17/23** | **73.9%** |
| Unnumbered scaffolded checks | 6/7 | Excluded |
| Q21 duplicate review | 1/1 | Excluded; scored classification voided |

Exactly four final-review slots were completed: Q21–Q24. Q21 repeated Q11 too closely and was excluded; no replacement was added. Q22 combined detection with proposed-update review. Q23 combined attribute retrieval with URL construction. Q24 applied retention to post-deletion costs and cleanup. These three count once each. All eight unscored responses together were seven correct and one incorrect.

This was guided practice with explanations immediately before many questions, not an independent mock exam. Correct scaffolded checks do not establish independent mastery. No invalid inputs or unanswered questions were counted.

Initial planning estimate: 75–95 minutes, about twenty original questions plus four review slots, excluding hands-on labs. This is not measured study time.

## 1. Infrastructure as Code, templates, and stacks / 基礎設施即程式碼

**Third-grade analogy / 三年級比喻：**學校要建三間相同教室。Template 是設計清單，CloudFormation 是施工管理團隊，Stack 是依清單建立且一起管理的一組設備。

Infrastructure as Code (IaC) describes infrastructure in code rather than relying on repeated manual setup. A CloudFormation template uses YAML or JSON. One template can create separate development, test, and production stacks; each has its own resources.

**Why / 為什麼：**Repeatable deployments, consistent environments, reviewable changes, and version history reduce manual inconsistency. A reusable design still needs suitable parameters and Region-specific values.

**Keywords:** repeatable, consistent, version-controlled infrastructure.
**Elimination:** CloudWatch monitors; CloudTrail audits AWS API activity; Auto Scaling manages capacity; CloudFormation provisions infrastructure from a template.

## 2. Parameters, Resources, Outputs, and Mappings / 範本區段

**Desk-order analogy / 訂書桌比喻：**下單選藍色 → 工廠製作書桌 → 完工交付編號。

| Section | Role / 工作 | S3 example |
|---|---|---|
| Parameters | 接收部署輸入 / accepts inputs | Enter a bucket name |
| Resources | 定義建立的東西 / defines resources | Create the bucket |
| Outputs | 提供結果 / exposes results | Display the bucket name |
| Mappings | 固定對照表 / predefined lookup table | Environment → approved setting |

Resources is required in a CloudFormation template. Parameters, Outputs, and Mappings are optional. Outputs does not create resources. Parameters does not by itself declare an EC2 instance.

**Uniform analogy / 制服比喻：**Parameters 是選尺寸；Resources 是製作指示，使用所選尺寸。
**Classroom analogy:** Resources 蓋教室；Outputs 在交付單寫教室地址。交付單不會蓋出教室。

Illustrative fragment only, not deployable as-is:

```yaml
Parameters:
  ChosenSize:
    Type: String
    Default: t3.micro
Resources:
  TrainingServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref ChosenSize
      ImageId: ami-example
Outputs:
  ServerId:
    Value: !Ref TrainingServer
```

The AMI is a placeholder. Region, networking, permissions, and application readiness are not verified.

**Mappings analogy:** 年級對應固定制服顏色。Parameters receives Production; Mappings can look up the approved instance type. Use Fn::FindInMap / !FindInMap for lookup.

**Exam strategy:** Identify the verb before the AWS noun:
- Choose/input → Parameters.
- Create/define → Resources.
- Display/result → Outputs.
- Predefined lookup → Mappings.

## 3. Conditions / 條件

**Analogy:** 大型學校活動才加開第二個報到窗口。

Conditions can control whether a resource is created, such as an extra EC2 only in production. A parameter supplies the environment; a condition evaluates the decision.

**English:** Mappings answers “which predefined value?” Conditions answers “should this resource exist under this condition?”
**Keywords:** only in production, conditionally create.
Parameters alone does not implement the decision.

## 4. Ref, GetAtt, ImportValue, and Sub / 引用函數

| Function | Purpose / 用途 | Example |
|---|---|---|
| Ref / !Ref | 參數值，或資源類型指定的回傳值 | EC2 Ref returns instance ID |
| Fn::GetAtt / !GetAtt | 指定資源屬性 / named attribute | !GetAtt TrainingServer.PublicIp |
| Fn::ImportValue / !ImportValue | 另一個 Stack 匯出的值 | !ImportValue Training-Subnet-ID |
| Fn::Sub / !Sub | 將變數填入字串 / string substitution | training-${Environment} |
| Fn::FindInMap / !FindInMap | 查固定對照表 / mapping lookup | Region → AMI ID |

Do not memorize “Ref always returns an ID.” Its return value depends on the referenced object. Ref on an EC2 instance returns its instance ID; Ref on a parameter returns its supplied value.

**Student card analogy / 學生資料卡：**Ref 取得此例的識別值；GetAtt 取得指定欄位，例如 public IP。
**Office analogy / 辦公室：**同一份範本的邏輯名稱不是跨 Stack 的公共名稱；另一個辦公室以 Export 公布值，使用 ImportValue 領取。

Network stack fragment:

```yaml
Outputs:
  SharedSubnet:
    Value: !Ref PublicSubnet
    Export:
      Name: Training-Subnet-ID
```

Application stack resource property:

```yaml
SubnetId: !ImportValue Training-Subnet-ID
```

Export/ImportValue uses the same AWS account and Region. Export names must be unique within that scope. While a value is imported, the exporter cannot delete the exporting stack or change that exported value until imports are removed. Therefore Q21's “currently approved subnet” does not mean arbitrary automatic hot-swapping: resource migration and export changes require planning.

Current documentation also describes Fn::GetStackOutput for other reference needs. It was not taught or assessed here. Do not generalize ImportValue restrictions to every possible cross-stack mechanism.

**Sub analogy / 填空通知：**「歡迎來到＿＿班」把空格替換成班級名稱。

```yaml
Value: !Sub 'Training environment: ${Environment}'
```

For an EC2 instance receiving a public IPv4 address:

```yaml
Value: !Sub 'http://${WebServer.PublicIp}'
```

Sub's resource.attribute form resolves an attribute and inserts it in the string. Producing a URL does not configure HTTP software, routes, security groups, or reachability.

## 5. Safe updates: change sets / 安全更新

**Contractor analogy / 整修比喻：**施工前先看新增、修改、替換設備的清單，決定是否施工。

A change set previews proposed stack changes. Creating it does not apply the changes; execution does. Review replacements and deletions carefully.

**Keywords:** before applying, preview, add/modify/replace.
**Trade-off:** Preview improves review but does not guarantee successful execution. Current documentation describes pre-deployment validation for common failures; runtime constraints can still cause failure. Do not treat a change set as a guaranteed successful deployment or a backup.

## 6. Drift detection versus repair / 偏移偵測與修復

**Classroom analogy:** 設計圖是藍色門，現場變成紅色門。比對找出差異就是 drift detection。
**Inspector analogy:** 檢查員指出門鎖壞了，不代表門鎖已修好。

Drift detection compares supported resources' actual settings with expected stack configuration. Support varies by resource/property; it is not a universal audit of application data.

| Question / 問題 | Feature |
|---|---|
| 這次更新將改什麼？ / proposed future changes | Change set |
| 目前已哪裡不同？ / existing differences | Drift detection |
| 如何修回正確設定？ / remediation | A deliberate corrective action |

Detection does not automatically repair differences. Do not assume submitting an unchanged template repairs all drift. Inspect the difference, choose a supported corrective approach, and review any proposed update.

Q22 chose automatic repair after drift detection and remains a review priority.

## 7. Retain, backups, replacement, and costs / 保留、備份與費用

**Notebook analogy / 紀念冊比喻：**
- Retain = 拆書櫃時留下原本紀念冊。
- Backup = 另外保存副本。
Keeping the notebook cannot restore a page someone tore out.

```yaml
DeletionPolicy: Retain
```

Retain preserves a resource when the stack is deleted or an update removes its definition. It does not protect database records from application-level deletion and does not create a separate backup.

UpdateReplacePolicy is the separate attribute governing the old resource when an update replaces it. This distinction was introduced briefly, not independently assessed.

Retained resources can keep incurring charges. After stack deletion, review retained resources explicitly. Retain does not schedule automatic deletion.

For supported resources, DeletionPolicy: Snapshot creates a snapshot before deletion; this was a documentation clarification, not a taught or scored topic. Default deletion behavior has exceptions, including certain RDS resources defaulting to Snapshot. S3 bucket deletion also requires dealing with its contents, including versions where applicable.

**Exam strategy:** Distinguish resource survival, data recovery, and cost cleanup. They require different controls.

## 8. Nested stacks and cross-stack sharing / 巢狀堆疊與分享

**School blueprint analogy:** 教學樓總設計引用標準教室小設計。Parent stack 管理 nested child stacks，各有範本。

Nested stacks enables reusable components within a parent-managed architecture. Usually update through the parent so CloudFormation coordinates children.

| Requirement | Approach |
|---|---|
| 可重用元件作為父 Stack 的子堆疊 | Nested stacks |
| 獨立管理的 Stacks 分享既有資源 ID | Outputs Export + ImportValue in assessed scope |

Importing an ID does not copy the resource. Nested stacks and independently managed cross-stack references are not interchangeable lifecycle choices.

## 9. Regions and dependencies / Region 與相依性

**Different-city analogy:** 同一份教室設計可在不同城市使用，但材料編號可能不同。

AMI = Amazon Machine Image. AMI IDs are Region-specific; do not assume a hard-coded ID works elsewhere. Mappings can associate Regions with approved AMIs.

AWS::Region is a pseudo parameter supplied by CloudFormation. AWS::AccountId supplies the account ID, not the Region.

**Wall-and-whiteboard analogy:** 先蓋牆，再裝白板。
References such as Ref/GetAtt can create implicit dependencies. If EC2 references a subnet, CloudFormation determines the required order. Use DependsOn for necessary ordering not expressed by references. Template text order is not a creation-order guarantee.

```yaml
DependsOn: ResourceA
```

Placed on ResourceB, this makes B depend on A. It does not read attributes or retrieve cross-stack values.

## 10. Rollback / 回復

**Classroom construction analogy:** 桌椅送到但投影機裝設失敗，撤回未完成的施工。

For new stack creation with default failure-handling settings, failure triggers rollback and attempts to remove resources created during that operation. Retention policies or cleanup errors can leave resources. Failure-handling options can preserve successfully provisioned resources for troubleshooting; no such option was configured here.

This lesson's question concerned new stack creation, not every possible update rollback state.

## 11. Reference deployments, AI assistance, and change management

**Quick Starts course concept:** 專業團隊準備的圖書館參考設計加施工指南。The video presents reference architectures, CloudFormation templates, and deployment guidance. Review architecture, security, cost, and current support before actual use. Downloadable templates do not make provisioned resources free. Current catalog/lifecycle status was not verified.

**Amazon Q Developer course concept:** 幫忙畫教室設計的助手。AI can help draft and explain templates; generated code needs human review. Syntax validation is not a security assessment. In the assessed example, SSH from 0.0.0.0/0 violated the company-network-only requirement. Restrict to approved CIDR and review remaining settings. Current model/features/quotas were not surveyed.

**Version control analogy:** 保存每次修改的設計圖並在施工前審查。
Store templates in Git, review changes, and deploy approved versions. Restoring an old Git file does not itself change deployed AWS resources; an appropriate deployment/update is still necessary.

Well-Architected ideas illustrated here: repeatable operations, reviewable changes, security automation, controlled updates, and resource-cost cleanup. No full six-pillar assessment was administered.

## 12. Complete answer log / 完整作答紀錄

Given/correct letters refer to the actual displayed fixed option order.

| Q | Topic / 題目主題 | Given | Correct | Classification | Result |
|---|---|---|---|---|---|
| 1 | Repeatable version-controlled training environments | B | B | Original | Correct |
| 2 | Deployment-time instance type choice | B | C | Original | Incorrect |
| 3 | Expose VPC ID in Outputs tab | C | D | Original | Incorrect |
| 4 | Required section with fixed bucket settings | C | C | Original | Correct |
| 5 | Store fixed environment→instance-type table | A | A | Original | Correct |
| 6 | Extra EC2 only in production | D | D | Original | Correct |
| 7 | Preview additions/modifications/replacements | B | B | Original | Correct |
| 8 | Console-modified security group differences | C | C | Original | Correct |
| 9 | Preserve bucket when deleting stack | A | A | Original | Correct |
| 10 | Reusable networking child template | D | D | Original | Correct |
| 11 | Independent stack reads exported subnet ID | B | A | Original | Incorrect |
| 12 | EC2 PublicIp instead of instance ID | B | C | Original | Incorrect |
| 13 | Substitute environment into training label | D | D | Original | Correct |
| 14 | Current Region as AMI lookup key | B | B | Original | Correct |
| 15 | Explicit order without property reference | C | C | Original | Correct |
| 16 | Default failed new-stack creation | A | A | Original | Correct |
| 17 | Review reference deployment before use | D | D | Original | Correct |
| 18 | AI template opens SSH to all IPv4 | B | B | Original | Correct |
| 19 | Traceable, reviewed infrastructure workflow | C | C | Original | Correct |
| 20 | Retain versus deleted database-record recovery | A | B | Original | Incorrect |
| 21 | Exported subnet between independent stacks | C | C | Final review, duplicate; excluded | Correct, unscored |
| 22 | Detect drift, then review proposed update | D | B | Valid integrated final review | Incorrect |
| 23 | PublicIp attribute plus HTTP URL construction | D | D | Valid integrated final review | Correct |
| 24 | Retained bucket cost and explicit cleanup | A | A | Valid integrated final review | Correct |

### Option audit / 選項核對

Concise summaries of all displayed choices, not a verbatim transcript.

| Q | A | B | C | D |
|---|---|---|---|---|
| 1 | CloudWatch dashboard | CloudFormation template/stacks: meets need | CloudTrail replay | ASG creates networking |
| 2 | Outputs | Resources | Parameters: input choice | Mappings |
| 3 | Parameters | Mappings | Resources | Outputs: expose ID |
| 4 | Outputs | Parameters | Resources: required | Mappings |
| 5 | Mappings: table | Outputs | Resources | Parameters |
| 6 | Outputs | Mappings | Parameters alone | Conditions: decision |
| 7 | Drift detection | Change set: preview | Output IDs | Immediate update |
| 8 | Change set | Outputs | Drift: actual versus expected | Mappings |
| 9 | Retain: keep bucket | Output name | Mapping name | Prevent creation |
| 10 | Outputs | Drift | More parameters | Nested stacks: child template |
| 11 | ImportValue: exported ID | Ref other stack logical ID | Another subnet | Drift retrieval |
| 12 | Ref EC2: instance ID | ImportValue without export | GetAtt PublicIp: attribute | Invalid FindInMap form |
| 13 | ImportValue | FindInMap | GetAtt | Sub: substitution |
| 14 | Public IP | AWS::Region: Region key | Subnet ID | AWS::AccountId |
| 15 | Output A | Text order | DependsOn A on B: ordering | Mapping names |
| 16 | Rollback: default | Mark success | Convert to parameters | Drift automatic repair |
| 17 | Deploy without review | Assume free | Replace with manual setup | Review/adapt: appropriate |
| 18 | Syntax implies safe | Restrict SSH/review: meets need | Output ID only | Drift automatic fix |
| 19 | Manual edits/monthly notes | Overwrite laptop file | Git/review/approved deploy: traceable | Dashboard as definition |
| 20 | Retain restores records | Backup/PITR: recovery capability | Drift reconstructs data | Template recreates past data |
| 21 | Cross-stack direct Ref | Copied Mapping | Export/Import: correct but duplicate | Cross-stack logical DependsOn |
| 22 | Change set then drift execution | Drift then reviewed change set: sequence | Outputs/Mapping removes rule | Drift auto repair |
| 23 | Ref as URL | Import without export | DependsOn converts ID | GetAtt IP plus URL: meets need |
| 24 | Review preservation needs/clean up: appropriate | Automatic next-day deletion | Delete Git file stops fees | Drift deleted stack erases contents |

### Unnumbered scaffolded checks / 不計分確認

| After | Task | Given | Correct | Result |
|---|---|---|---|---|
| Q2 | Deployment environment input: Parameters vs Outputs | A | A | Correct |
| Q3 | Bucket definition: Parameters/Resources/Outputs | C | B | Incorrect |
| Previous check | Creating bucket: create vs display | A | A | Correct |
| Previous check | Create bucket: Resources vs Outputs | A | A | Correct |
| Q11 | Exported VPC ID: Ref vs ImportValue | B | B | Correct |
| Q12 | Named attribute: GetAtt vs ImportValue | A | A | Correct |
| Q20 | Deleted records: backup history vs Retain alone | A | A | Correct |

## 13. Review priorities / 複習重點

| Evidence | Correction / 修正 | Remaining limit |
|---|---|---|
| Q2 B; Q3 C; scaffolded bucket check C | Input → Parameters; create → Resources; report → Outputs | Subsequent simple checks correct, not independent mastery |
| Q11 B | Ref cannot directly resolve separate-stack logical ID; use exported value with ImportValue | Q21 repeated scenario, excluded |
| Q12 B | Named resource attribute → GetAtt | Q23 integrated IP and URL task correct during guided review |
| Q20 A | Retain is resource preservation, not record recovery | Simple recovery check correct after disclosure |
| Q22 D | Drift detection reports; remediation requires separate action | No further retest appended |

Future reviews should use meaningfully new applications, not shuffled disclosed questions. The promised review ended at Q24.

## 14. Personal AWS Rebuild — Completion Checkpoint

**Worth rebuilding? / 值得重建嗎？** Yes, selectively. Begin with a paper template or small isolated S3 exercise to reinforce sections, references, preview, drift, and retention. A full multi-AZ database is unnecessary for these immediate learning goals.

**Fictitious non-production LFM example / 虛構 LFM 範例：**A disposable story-assets bucket with fabricated text only. No real family records, no production domain, and no existing production infrastructure changes.

| Stage | Proposed objective / 目標 |
|---|---|
| Paper first | Label Parameters/Resources/Outputs; predict Ref/GetAtt/Sub values |
| Small optional lab | Deploy one private S3 bucket, inspect Outputs, change a safe tag via a change set |
| Optional drift exercise | Change a supported explicitly declared property outside CloudFormation; detect and deliberately remediate |
| Optional Retain exercise | Retain a disposable bucket, delete stack, observe remaining resource and clean it explicitly |
| Cross-stack/nested extensions | Compare lifecycle ownership on paper before creating additional stacks |

All steps are proposals, not performed work. Verify current console steps, property support, and cost before provisioning.

**Potential costs / 費用：**S3 storage, requests, versions and transfer; optional EC2/EBS/public IPv4; optional snapshots, logs or databases if the design expands. No current dollar quote was established. Retained resources continue applicable billing.

**Cleanup / 清理：**
- Review which data must be preserved before deletion.
- Inspect change sets for replacement/deletion effects.
- Delete disposable stacks in dependency order: importing application before exporting network.
- Check retained resources separately after stack deletion.
- For disposable S3 buckets, remove objects, versions/delete markers where applicable, and incomplete uploads as needed before deleting the bucket.
- Review retained volumes, snapshots, public IPs and optional resources if created.
- Remove only lab-owned permissions and resources; do not touch production LFM.
- Verify cleanup in AWS and inspect remaining charges.

No AWS resources may be provisioned until the learner explicitly asks to perform the rebuild.

## 15. Coverage limits and official checks / 教學限制與官方查核

Video-observed topics included IaC, template customization, conditions, change sets, drift, nested stacks, Quick Starts, Amazon Q Developer, Well-Architected automation, guided CloudFormation lab and café deployment lab. Neither lab was performed or verified. Earlier lab history is not fresh verification.

Not independently assessed: complete YAML/JSON authoring, IAM execution roles/capabilities, custom resources, deployment pipelines, nested-stack update behavior, replacement policies, snapshot restoration, or multi-Region deployment. Full current service limits and tool lifecycle status were not surveyed. No diagrams were requested or delivered; tables and analogies preserve the teaching aids used.

Official documentation checked during final documentation:
- [ImportValue scope, restrictions and export dependencies](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/intrinsic-function-reference-importvalue.html)
- [Drift detection and corrective action](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
- [DeletionPolicy behavior and retained-resource charges](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html)
- [Change sets, execution and validation limits](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-changesets.html)

Documentation checks refine the course concepts; no new feature was inferred to be included in or excluded from the certification exam.

**Next / 下一步：**Module 11 guided tutoring is complete. Read the actual Module 12 material before choosing its title and topics. Carry forward the review priorities without restarting prior modules.

