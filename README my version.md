# aws-unauthorized-iam-activity-investigation
## Overview

This lab focuses on detecting and investigating unauthorized or suspicious IAM activity in AWS.

In a cloud environment, an attacker does not necessarily need malware or a traditional exploit. If they obtain valid AWS credentials, they can potentially use legitimate AWS APIs to:

Enumerate IAM resources
Assume roles
Create users
Create access keys
Attach privileged policies
Modify roles
Modify trust policies
Establish persistent access
Access cloud resources
Attempt to disable or modify logging

This is why identity activity is an important cloud security signal.

The SOC analyst's job is not simply to find an unusual IAM event and call it malicious. The analyst needs to determine:

Who performed the action, what did they do, when did they do it, where did it come from, and does the sequence make sense?

AWS CloudTrail records AWS API activity.

For example, if someone performs an operation such as:

CreateUser
CreateAccessKey
CreateRole
AttachRolePolicy
AssumeRole
UpdateAssumeRolePolicy

CloudTrail can record information about that activity.

Pattern 1 — New Identity Creation
CreateUser
      ↓
CreateAccessKey
      ↓
AttachUserPolicy

An attacker could create a new identity and give it permissions for persistence.

Pattern 2 — Privilege Escalation
AssumeRole
      ↓
Modify Policy
      ↓
AttachRolePolicy
      ↓
Gain Higher Privileges

The important point is that one event alone may not be suspicious.

The sequence is more meaningful.

Pattern 3 — Role Trust Abuse

Look for:

UpdateAssumeRolePolicy

An attacker who can modify a role's trust policy may attempt to allow another identity or account to assume that role.

Pattern 4 — Credential Persistence

Look for:

CreateAccessKey

especially when it occurs shortly after suspicious identity or privilege activity.

Pattern 5 — CloudTrail Tampering

Look for:

StopLogging
DeleteTrail
UpdateTrail

These actions deserve attention because an attacker may try to reduce visibility.

---

## Investigation Objectives

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

## Environment

| Item | Value |
|---|---|
| Cloud Platform | Amazon Web Services (AWS) |
| Environment | AWS Academy Learner Lab |
| Primary Region | us-west-2 |
| Services | IAM, CloudTrail |
| Investigation Type | Cloud Security / IAM Threat Hunting |
| Account Access | Temporary assumed IAM role |

---

## Investigation Finding

The most significant finding was an explicit denial affecting CloudTrail:

`cloudtrail:LookupEvents`

The active AWS Labs role was denied access through a Service Control Policy.

The error indicated:

`AccessDeniedException`

and stated that the action was explicitly denied by an organization-level Service Control Policy.

This means that CloudTrail Event History could not be queried from the current identity.

---

## Important Investigation Principle

The following distinction is important:

    No events observed
    ≠
    Unable to retrieve events

Because the current identity could not perform `cloudtrail:LookupEvents`, the investigation could not use the Event History interface to establish whether suspicious API activity had occurred.

The correct SOC conclusion is therefore:

> CloudTrail visibility was restricted by an explicit Service Control Policy denial.

It would be incorrect to conclude that no suspicious IAM activity occurred simply because the Event History page displayed zero events.

---

## Investigation Workflow

The investigation followed this process:

    AWS Console
        ↓
    IAM Dashboard
        ↓
    IAM Users
        ↓
    IAM Roles
        ↓
    IAM Policies
        ↓
    CloudTrail Event History
        ↓
    AccessDeniedException
        ↓
    SCP Analysis
        ↓
    Investigation Limitation
        ↓
    Final Assessment

---

## Key Findings

### Finding 1 — No IAM Users

The IAM Users page displayed:

`IAM users: 0`

No permanent IAM user accounts were identified.

### Finding 2 — 26 IAM Roles

The account contained:

`26 IAM roles`

Several were associated with AWS services such as Lambda, EC2, CodeStar, and OpsWorks.

### Finding 3 — Large IAM Policy Library

The IAM Policies page displayed:

`1,571 policies`

The visible policies included AWS-managed administrative and service-specific policies.

### Finding 4 — CloudTrail Visibility Restricted

CloudTrail Event History could not be queried because the current AWS Labs role was explicitly denied:

`cloudtrail:LookupEvents`

### Finding 5 — SCP Explicit Deny

The CloudTrail error identified a Service Control Policy as the source of the denial.

This indicates that the restriction was enforced above the IAM permission layer.

---

## SOC Relevance

This investigation demonstrates how cloud SOC analysts must distinguish between:

- A genuine absence of suspicious activity.
- A lack of telemetry.
- A lack of authorization to access telemetry.

IAM configuration provides context about identities and privileges, while CloudTrail provides activity evidence.

When both are available, they can be correlated to investigate:

- Unauthorized IAM changes.
- Privilege escalation.
- Suspicious role assumption.
- Access key creation.
- Policy modification.
- Persistence mechanisms.
- CloudTrail tampering.

---

