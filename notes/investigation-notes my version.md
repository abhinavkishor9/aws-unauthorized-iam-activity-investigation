# Investigation Notes

# Investigation Objective

- Establish the IAM identity baseline of the AWS account.
- Examine the roles currently configured in the environment and understand their purpose.
- Review available IAM policies and identify permissions that could provide elevated access.
- Determine whether the account relies on permanent IAM users or temporary role-based access.
- Attempt to retrieve recent AWS API activity through CloudTrail Event History.
- Analyze the AccessDeniedException encountered during the CloudTrail investigation.
- Determine the specific AWS API action that was blocked.
- Identify the policy mechanism responsible for restricting the CloudTrail request.
- Assess how the restricted CloudTrail visibility affects the investigation.
- Distinguish between a genuine absence of suspicious activity and insufficient visibility into AWS activity.
- Identify IAM-related API activities that would require further investigation if CloudTrail access were available.
- Document the available evidence and investigation limitations without making unsupported claims of compromise.

---

# IAM Dashboard Review

The IAM Dashboard was opened to establish the initial identity and access baseline.

The dashboard displayed:

- User groups: 0
- Users: 0
- Roles: 26
- Policies: 5
- Identity providers: 0

The dashboard therefore indicated that the environment was primarily using role-based access rather than traditional IAM users.

---

# IAM Users Investigation

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

# IAM Roles Investigation

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

# IAM Policies Investigation

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

# AdministratorAccess Review

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

# AccessAnalyzerServiceRolePolicy Review

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

# CloudTrail Investigation

The investigation then moved to:

`CloudTrail → Event history`

The active region was:

`United States (Oregon) / us-west-2`

The CloudTrail page displayed:

`Event history (0)`

However, an `AccessDeniedException` appeared.

---

# CloudTrail Access Denial

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

# Identity Analysis

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

# Service Control Policy Analysis

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

