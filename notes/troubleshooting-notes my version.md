# Troubleshooting Notes


# 1. CloudTrail Event History Access Denied

## Error

The CloudTrail Event History page displayed:

`AccessDeniedException`

The error indicated that the current identity was not authorized to perform:

`cloudtrail:LookupEvents`

The error further identified an explicit deny in a Service Control Policy.

---

## Cause

The AWS Academy environment uses restricted permissions.

The current assumed role was subject to an organization-level Service Control Policy that explicitly denied the CloudTrail lookup operation.

The relevant permission was:

`cloudtrail:LookupEvents`

---

## Impact

Because Event History could not be queried:

- Recent CloudTrail events could not be retrieved.
- IAM API activity could not be directly reviewed.
- Recent role assumptions could not be confirmed.
- IAM policy changes could not be correlated with timestamps.
- Suspicious activity could not be conclusively confirmed or ruled out.

---

## Correct Interpretation

Do not document:

`No suspicious activity was found.`

Instead document:

`CloudTrail Event History could not be queried because the current AWS identity was explicitly denied cloudtrail:LookupEvents by a Service Control Policy.`

This distinction is critical for accurate SOC reporting.

---

# 2. IAM Users Showing Zero

## Observation

The IAM Users page displayed:

`IAM users (0)`

and:

`No resources to display`

---

## Is This an Error?

No.

The account can operate without traditional IAM users by using IAM roles and temporary credentials.

---

## Investigation Impact

The absence of IAM users means that investigation should focus on:

- IAM roles
- Role trust relationships
- STS sessions
- CloudTrail role activity
- Attached permissions

---

# 3. IAM Roles Showing Last Activity as "-"

## Observation

Several roles displayed:

`Last activity: -`

---

## Interpretation

This does not automatically mean that the role has never been used.

The value may indicate that IAM does not currently have activity information available for the displayed tracking period.

Therefore, the field should not be treated as definitive forensic evidence.

---

## Recommended Approach

Use CloudTrail to establish actual role usage when available.

Investigate:

`AssumeRole`

and correlate the event with:

- Event time
- Principal
- Source IP
- User agent
- Region

---

# 4. IAM Policies Count Appears Different

## Observation

The IAM Policies page displayed:

`1,571 policies`

while the IAM Dashboard displayed:

`Policies: 5`

---

## Explanation

These numbers represent different views of IAM policy information.

The Policies page can display the broader policy library available within AWS, including AWS-managed policies.

The Dashboard count reflects the account's locally relevant IAM policy resources.

---

## SOC Lesson

Do not interpret the two numbers as contradictory evidence.

Always check the context of the console page before using a count in an investigation.

---

# 5. AdministratorAccess Policy Visible

## Observation

The IAM Policies page displayed:

`AdministratorAccess`

---

## Interpretation

AdministratorAccess is a highly privileged AWS-managed policy.

Its presence does not prove that an attacker has administrative access.

The analyst must determine:

- Whether it is attached.
- Which identity has it.
- Whether that identity is expected to have it.
- Whether it was recently attached.
- Whether the identity was recently assumed.

---

# 6. AccessAnalyzerServiceRolePolicy

## Observation

The policy:

`AccessAnalyzerServiceRolePolicy`

was visible.

The policy contained permissions involving AWS services including:

- DynamoDB
- EC2
- EFS
- Elastic Container Registry

---

## Interpretation

This is an AWS-managed service policy.

Its presence should not automatically be classified as malicious.

The investigation should focus on whether the policy is attached to an unexpected identity or whether its associated role is behaving abnormally.

---

# 7. CloudTrail Event History Displays Zero Events

## Observation

CloudTrail displayed:

`Event history (0)`

while also displaying an access-denied warning.

---

## Correct Interpretation

The event count cannot be treated as a reliable statement that the account has no CloudTrail activity.

The console could not retrieve the events because:

`cloudtrail:LookupEvents`

was explicitly denied.

---

# 8. Service Control Policy Restriction

## Observation

The error referenced an organization-level Service Control Policy.

---

## Explanation

Service Control Policies establish permission boundaries for AWS accounts or organizational units.

They do not directly grant permissions.

For example:

    IAM Policy
       ↓
    Allows Action
       ↓
    SCP
       ↓
    Explicitly Denies Action
       ↓
    Final Result = Denied

An explicit deny takes precedence.

---

orm the requested operation.

For a SOC analyst, this distinction prevents false conclusions and ensures that investigation limitations are clearly documented.
