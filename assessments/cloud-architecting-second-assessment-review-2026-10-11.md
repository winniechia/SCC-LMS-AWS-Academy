# AWS Academy Cloud Architecting — Second Assessment Review

Date: 2026-10-11

## Results

Second assessment: **220/250 (88%), PASS**. Module scores show Securing Access **0/20**, Adding a Database Layer **20/20**, and Building Serverless Architectures and Microservices **10/20**. All other modules received full marks. The first assessment also scored 220/250, but had a different distribution of missed questions.

## Q1 — IAM identity

Scenario: A new CTO needs permanent access to create, modify, and delete AWS resources.

Selected: **IAM group**. Likely expected: **IAM user**. An IAM group organizes users and permissions; it is not an individual sign-in identity. Modern workforce best practice is federation via IAM Identity Center and temporary credentials. The screenshot does not provide the official correct answer.

三年級比喻：IAM user 是個人識別證；group 是一群人共用的規則；role 是臨時職務證。

## Q2 — Project-specific permissions

Scenario: Developers should access only specified resources within their own projects.

Selected: **Service Control Policy (SCP)**. Likely expected: **identity-based policy**. Identity policies define actions and resources for IAM identities. SCPs set organization-level maximum permissions and do not grant access. Permissions boundaries also limit rather than grant access. The official correct answer is not shown.

三年級比喻：Identity policy 規定每位學生能進哪個教室；SCP 是全校都不能突破的規定。

## Q23 — Lambda runtime environments (select two)

Selected: **D (Lambda service VPC) + E (CloudFront edge cache)**. The AWS Academy review **explicitly marked this selection Incorrect**.

Revised hypothesis: **B (CloudFront regional edge cache) + D (Lambda service VPC)**, **not verified by an official answer key**. Standard Lambda execution is AWS-managed; customer VPC connectivity is different from running directly inside the customer VPC. AWS documentation describes Lambda@Edge using both edge location and regional edge cache terminology, so the B/E distinction deserves verification. ECS is not a Lambda runtime environment. Do not memorize B+D as confirmed.

三年級比喻：一般 Lambda 在 AWS 管理的工作室；Lambda@Edge 在靠近使用者的 CloudFront 網路執行。

## Evidence and study checkpoints

Confirmed: overall module score totals 220/250; two Securing Access questions missed; one Serverless question missed; Q23 D+E was marked Incorrect. Q1/Q2 expected answers are strong inferences from question wording and module scores. Q23 B+D remains a hypothesis. The previous assistant recommendation of D+E did not match the Academy grading.

SAA-C03: review IAM user/group/role; identity policies versus SCPs and permission boundaries; Lambda execution environments versus VPC connectivity and CloudFront Functions.

Personal AWS Rebuild checkpoint: Not required for these conceptual questions; optional IAM least-privilege practice may help.

## Final course assessment result — October 11, 2026

After refreshing the AWS Academy gradebook, **Academy Cloud Architecting Course Assessment** displayed **100/100**, timestamped **Oct 11 at 12:09am**. **The learner confirms this final result means all 25 questions were answered correctly (25/25).**

The earlier **220/250 (88%)** module-score screenshot and the Q23 **Incorrect** review are retained above as **historical review evidence from an earlier displayed result**, not the final outcome. The available screenshots do not independently explain the change in scoring; the **25/25 interpretation is the learner's explicit confirmation**. The Q1, Q2, and Q23 sections remain useful as study notes, not as claims of errors in the final 100/100 result.
