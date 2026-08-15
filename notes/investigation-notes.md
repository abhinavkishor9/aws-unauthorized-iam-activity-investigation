# Investigation Notes

## Case Information

| Field | Details |
|---|---|
| Investigation | AWS Unauthorized IAM Activity Investigation |
| Platform | AWS |
| Environment | AWS Academy Learner Lab |
| Region | us-west-2 |
| Primary Services | IAM, CloudTrail |
| Investigation Type | Cloud IAM / Threat Hunting |
| Analyst | SOC Lab Investigation |

---

# 1. Investigation Objective

The objective of this investigation was to examine the IAM configuration of the AWS environment and determine whether CloudTrail could provide API-level evidence for investigating potentially unauthorized IAM activity.

The investigation focused on:

- IAM users
- IAM roles
- IAM policies
- Privileged permissions
- CloudTrail Event History
- Access-denied conditions
- Service Control Policy restrictions

---

# 2. IAM Dashboard Review

The IAM Dashboard was opened to establish the initial identity and access baseline.

The dashboard displayed:

- User groups: 0
- Users: 0
- Roles: 26
- Policies: 5
- Identity providers: 0

The dashboard therefore indicated that the environment was primarily using role-based access rather than traditional IAM users.

---

# 3. IAM Users Investigation

The IAM Users section was opened.

Observed result:

`IAM users: 0`

The page displayed:

`No resources to display`

## Interpretation

No permanent IAM users were identified in the account.

This is consistent with an AWS Academy environment where access is commonly provided through temporary role sessions.

## Security Relevance

An absence of IAM users does not mean that no identities are accessing the account.

Temporary role sessions can still perform AWS API operations depending on their assigned permissions.

Therefore, role investigation remained necessary.

---

# 4. IAM Roles Investigation

The IAM Roles page displayed:

`26 roles`

Several visible roles included:

- `aws-codestar-service-role`
- `aws-opsworks-cm-service-role`
- `aws-opsworks-ec2-role`
- `AWSCloudFormationStackSetExecution...`
- `AWSLabs-LabFunction-LabAdmin-v1-...`
- `AWSLabs-LabFunction-LabAdmin-v2-...`

The visible roles were associated with trusted entities such as:

- AWS services
- EC2
- Lambda
- Other AWS accounts

The Last Activity field was displayed as `-` for the visible entries.

## Interpretation

The observed roles appeared to represent AWS service and lab infrastructure functions.

The presence of a role alone is not evidence of malicious activity.

Further investigation would require examining:

- Trust relationships
- Attached policies
- Last accessed information
- CloudTrail role activity

---

# 5. IAM Policies Investigation

The IAM Policies page displayed:

`Policies (1,571)`

The filter was set to:

`All types`

Visible policies included:

- `AccessAnalyzerServiceRolePolicy`
- `AccountAccessManager...`
- `AccountManagement...`
- `AdministratorAccess`
- Other AWS-managed policies

## Important Observation

The policy count represents the available policy library and should not be interpreted as 1,571 locally created policies.

The IAM Dashboard separately displayed:

`Policies: 5`

This distinction is important when interpreting AWS console information.

---

# 6. AdministratorAccess Review

One of the visible policies was:

`AdministratorAccess`

This policy is highly privileged because it provides broad access to AWS services.

## SOC Perspective

A highly privileged policy should be investigated in the context of:

- Who can use it?
- Which role has it attached?
- Is the role expected to have administrative permissions?
- Was the role recently created?
- Was its trust policy modified?
- Was it recently assumed?

The existence of `AdministratorAccess` alone does not establish abuse.

---

# 7. AccessAnalyzerServiceRolePolicy Review

A visible AWS-managed policy was:

`AccessAnalyzerServiceRolePolicy`

The policy permissions included limited read/list operations for services such as:

- DynamoDB
- EC2
- EFS
- Elastic Container Registry

These permissions are associated with AWS service functionality and should not automatically be considered suspicious.

The important distinction is between:

`Policy exists`

and:

`Policy is attached to a suspicious identity and actively being abused`

---

# 8. CloudTrail Investigation

The investigation then moved to:

`CloudTrail → Event history`

The active region was:

`United States (Oregon) / us-west-2`

The CloudTrail page displayed:

`Event history (0)`

However, an `AccessDeniedException` appeared.

---

# 9. CloudTrail Access Denial

The error indicated that the active AWS role was not authorized to perform:

`cloudtrail:LookupEvents`

The denial was explicitly associated with a Service Control Policy.

The important portion of the error was:

`explicit deny in a service control policy`

## Interpretation

The current identity could not retrieve CloudTrail Event History.

Therefore, the zero-event display cannot be interpreted as:

`No suspicious activity occurred.`

Instead:

`CloudTrail activity could not be queried using the current identity.`

---

# 10. Identity Analysis

The CloudTrail error identified the current identity as an assumed role.

The ARN followed this general structure:

`arn:aws:sts::<ACCOUNT_ID>:assumed-role/<ROLE_NAME>/<SESSION_ID>`

This indicates that the investigation was being performed through a temporary STS role session rather than a permanent IAM user.

## SOC Relevance

Temporary role sessions are important during cloud investigations because attackers can also abuse legitimate role credentials if they obtain them.

An investigation should therefore correlate:

- Role name
- Session name
- Source IP
- User agent
- API calls
- Time
- Permissions

---

# 11. Service Control Policy Analysis

The denial identified an organization-level Service Control Policy.

Conceptually:

    IAM Permission
          +
    SCP Restriction
          ↓
    Final Effective Permission

An explicit deny takes precedence over an allow.

Therefore, even if the role had an IAM policy that allowed CloudTrail lookup, the SCP could still prevent the action.

---

# 12. Investigation Limitation

The inability to execute:

`cloudtrail:LookupEvents`

created a significant visibility limitation.

Without CloudTrail Event History, the investigation could not directly determine:

- Who performed recent IAM API calls
- Whether an IAM user was created
- Whether an access key was created
- Whether a role was modified
- Whether a policy was attached
- Whether a suspicious role was assumed
- Whether CloudTrail configuration was changed

This limitation must be explicitly documented in a SOC investigation.

---

# 13. Evidence Assessment

| Evidence | Observation | Assessment |
|---|---|---|
| IAM users | 0 | No permanent IAM users identified |
| IAM roles | 26 | Role-based AWS access present |
| IAM policies | 1,571 available policies | Large AWS policy library |
| Local IAM policy count | 5 | Console dashboard count |
| AdministratorAccess | Present in policy library | Highly privileged policy exists |
| CloudTrail Event History | 0 displayed | Cannot be treated as no activity |
| CloudTrail API | `cloudtrail:LookupEvents` | Explicitly denied |
| Denial source | Service Control Policy | Organization-level restriction |
| Compromise evidence | Not established | Insufficient CloudTrail visibility |

---

# 14. Threat Hunting Opportunities

If CloudTrail access were available, the following events would be investigated:

## Identity Creation

`CreateUser`

`CreateAccessKey`

## Privilege Escalation

`AttachUserPolicy`

`AttachRolePolicy`

`PutUserPolicy`

`PutRolePolicy`

`PutRolePermissionsBoundary`

## Role Abuse

`AssumeRole`

`UpdateAssumeRolePolicy`

`CreateRole`

## Persistence

`CreateAccessKey`

`CreateLoginProfile`

## Logging Tampering

`StopLogging`

`DeleteTrail`

`UpdateTrail`

---

# 15. Recommended Correlation

A suspicious IAM investigation should not rely on one event.

For example:

    CreateUser
         ↓
    CreateAccessKey
         ↓
    AttachUserPolicy
         ↓
    AssumeRole
         ↓
    Resource Access

A sequence like this would be significantly more interesting than an isolated IAM event.

The analyst should correlate:

- Identity
- Time
- API action
- Source IP
- User agent
- Region
- Resource
- Result

---

# 16. Final Assessment

No direct evidence of AWS account compromise was established from the available evidence.

The strongest finding was a visibility restriction preventing the current AWS Labs role from querying CloudTrail Event History.

The IAM environment showed:

- 0 IAM users
- 26 roles
- 1,571 available policies
- AWS-managed administrative and service policies

The investigation therefore established the IAM baseline and identified a CloudTrail visibility limitation, but did not have sufficient telemetry to confirm or rule out unauthorized IAM activity.
