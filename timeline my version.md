# Investigation Timeline

## Timeline

### T1 — AWS Console Access

**Activity:**  
The AWS Academy Learner Lab environment was opened.

**Observation:**  
The AWS Management Console was accessible and the investigation environment was confirmed.

**Assessment:**  
The investigation was started from the AWS Console.

---

### T2 — IAM Dashboard Review

**Activity:**  
The IAM Dashboard was opened.

**Observation:**

- User groups: 0
- IAM users: 0
- IAM roles: 26
- Policies: 5
- Identity providers: 0

**Assessment:**  
The initial IAM baseline showed that the environment primarily relied on IAM roles rather than traditional IAM users.

---

### T3 — IAM Users Investigation

**Activity:**  
IAM > Users was opened.

**Observation:**

- IAM users: 0
- No resources to display

**Assessment:**  
No permanent IAM users were identified in the AWS account.

This does not mean that the account had no active identities because AWS role-based access can use temporary credentials.

---

### T4 — IAM Roles Investigation

**Activity:**  
IAM > Roles was opened.

**Observation:**

- Total roles: 26
- Several AWS service roles were visible.
- Examples included:
  - `aws-codestar-service-role`
  - `aws-opsworks-cm-service-role`
  - `aws-opsworks-ec2-role`
  - `AWSCloudFormationStackSetExecution...`
  - `AWSLabs-LabFunction-LabAdmin-v1-...`
  - `AWSLabs-LabFunction-LabAdmin-v2-...`

**Assessment:**  
The environment contained multiple service and lab-related roles.

The presence of these roles alone was not considered evidence of malicious activity.

---

### T5 — IAM Policy Investigation

**Activity:**  
IAM > Policies was opened.

**Observation:**

The Policies page displayed:

`1,571 policies`

The visible policies included AWS-managed policies such as:

- `AccessAnalyzerServiceRolePolicy`
- `AccountAccessManager...`
- `AccountManagement...`
- `AdministratorAccess`

**Assessment:**  
The policy page represented a broad AWS policy library.

The presence of a highly privileged policy such as `AdministratorAccess` does not by itself prove that the account was compromised.

---

### T6 — AdministratorAccess Observation

**Activity:**  
The `AdministratorAccess` policy was observed in the IAM policy list.

**Observation:**  
The policy provides broad administrative access to AWS services.

**Assessment:**  
The policy was treated as a high-privilege capability requiring identity and attachment context before any security conclusion could be made.

No direct evidence of abuse was established from the policy's presence alone.

---

### T7 — CloudTrail Investigation Started

**Activity:**  
AWS CloudTrail was opened.

Navigation:

`CloudTrail > Event history`

**Observation:**  
The Event History page was displayed for the Oregon region (`us-west-2`).

**Assessment:**  
CloudTrail Event History was selected as the primary source for investigating AWS API activity.

---

### T8 — CloudTrail Event History Access Attempted

**Activity:**  
CloudTrail Event History was accessed to review recent AWS activity.

**Observation:**  
The page displayed an `AccessDeniedException`.

The error indicated that the current AWS identity was not authorized to perform:

`cloudtrail:LookupEvents`

**Assessment:**  
The current investigation identity could not retrieve CloudTrail Event History.

---

### T9 — Explicit Deny Identified

**Activity:**  
The CloudTrail error message was examined.

**Observation:**  
The error stated that the action was denied with an explicit deny in a Service Control Policy.

**Denied Action:**

`cloudtrail:LookupEvents`

**Assessment:**  
The restriction was enforced through an organization-level Service Control Policy rather than simply being a missing IAM permission.

---

### T10 — Current Identity Identified

**Activity:**  
The identity contained in the CloudTrail error was reviewed.

**Observation:**  
The identity was an assumed AWS Labs role using the following general ARN structure:

`arn:aws:sts::<ACCOUNT_ID>:assumed-role/<ROLE_NAME>/<SESSION_ID>`

**Assessment:**  
The investigation was being performed through a temporary role session rather than a permanent IAM user.

---

### T11 — CloudTrail Event Count Reviewed

**Activity:**  
The CloudTrail Event History page was reviewed after the access-denied error appeared.

**Observation:**

`Event history (0)`

**Assessment:**  
The zero-event display could not be interpreted as proof that no CloudTrail activity existed.

Because `cloudtrail:LookupEvents` was denied, the investigation lacked authorization to retrieve the relevant events.

---

### T12 — Investigation Visibility Limitation Recorded

**Activity:**  
The CloudTrail access restriction was documented.

**Observation:**  
The following investigation activities could not be directly performed:

- Reviewing recent IAM API calls
- Reviewing role assumptions
- Investigating access key creation
- Reviewing IAM policy changes
- Reviewing role modifications
- Correlating IAM actions with source IP addresses
- Establishing an accurate CloudTrail event timeline

**Assessment:**  
CloudTrail visibility was restricted.

This became the primary limitation of the investigation.

---

### T13 — IAM and CloudTrail Correlation

**Activity:**  
IAM findings were compared with the CloudTrail investigation results.

**Observation:**

IAM provided the following baseline:

- 0 IAM users
- 26 IAM roles
- AWS-managed policies
- Highly privileged policies available in the environment

CloudTrail activity could not be retrieved because of the SCP restriction.

**Assessment:**  
The IAM configuration could be documented, but recent IAM API activity could not be conclusively correlated with the identities and roles observed.

---

### T14 — Threat Hunting Assessment

**Activity:**  
Potential IAM abuse patterns were identified for future investigation if CloudTrail access becomes available.

**Relevant API activities include:**

- `CreateUser`
- `CreateAccessKey`
- `CreateRole`
- `AttachUserPolicy`
- `AttachRolePolicy`
- `PutUserPolicy`
- `PutRolePolicy`
- `UpdateAssumeRolePolicy`
- `AssumeRole`
- `StopLogging`
- `DeleteTrail`
- `UpdateTrail`

**Assessment:**  
These events would be valuable for detecting persistence, privilege escalation, role abuse, and attempts to reduce cloud logging visibility.

---

### T15 — Evidence Assessment

**Activity:**  
All available evidence was reviewed.

**Observed evidence:**

- IAM users: 0
- IAM roles: 26
- IAM policies displayed: 1,571
- AdministratorAccess policy visible
- CloudTrail Event History inaccessible
- `cloudtrail:LookupEvents` explicitly denied
- Service Control Policy identified as the denial source

**Assessment:**  
The available evidence did not establish direct proof of unauthorized IAM activity.

However, the CloudTrail access restriction prevented the investigation from conclusively ruling out such activity.

---

### T16 — Final Investigation Status

**Activity:**  
The investigation was concluded based on the available evidence.

**Final Status:**

`No compromise confirmed`

**Investigation Limitation:**

`CloudTrail Event History could not be queried because the current AWS identity was explicitly denied cloudtrail:LookupEvents by a Service Control Policy.`

**Final Assessment:**  
The investigation successfully established the IAM configuration baseline and identified a significant cloud-logging visibility restriction. The available evidence was insufficient to confirm or rule out unauthorized IAM activity because the required CloudTrail API data could not be retrieved.

---

