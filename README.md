# AWS Unauthorized IAM Activity Investigation

## Overview

This project documents a hands-on AWS SOC investigation focused on IAM identities, roles, policies, and CloudTrail visibility. The investigation was performed in an AWS Academy environment to understand how a SOC analyst can identify potentially risky IAM configurations and determine whether AWS API activity can be investigated through CloudTrail.

The investigation also encountered an important visibility limitation: the active AWS Labs role was explicitly denied permission to perform `cloudtrail:LookupEvents` by a Service Control Policy (SCP). Therefore, the absence of CloudTrail events in the console was treated as an access limitation rather than evidence that no suspicious activity occurred.

---

## Investigation Objectives

- Understand the IAM identity structure of the AWS environment.
- Review IAM users, roles, and policies.
- Identify potentially privileged IAM permissions.
- Understand the difference between IAM users and temporary role-based access.
- Investigate CloudTrail Event History.
- Analyze the `AccessDeniedException` returned by CloudTrail.
- Determine why CloudTrail events could not be retrieved.
- Correlate IAM configuration with CloudTrail visibility.
- Document investigation limitations and findings.
- Build a timeline of the investigation.

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

## Key Concepts

### AWS IAM

AWS Identity and Access Management (IAM) controls authentication and authorization within AWS.

IAM determines:

- Who can access AWS resources.
- What actions an identity can perform.
- Which resources those actions apply to.
- Which AWS services or identities can assume roles.

---

### IAM Users

IAM users are identities with long-term credentials.

Typical examples include:

- Username
- Password
- Access keys
- MFA configuration
- Attached permissions

The investigated AWS environment contained:

`0 IAM users`

This indicates that no traditional permanent IAM users were present in the account at the time of investigation.

---

### IAM Roles

IAM roles provide temporary permissions that can be assumed by trusted entities.

The investigation environment contained:

`26 IAM roles`

Examples observed included:

- `aws-codestar-service-role`
- `aws-opsworks-cm-service-role`
- `aws-opsworks-ec2-role`
- `AWSCloudFormationStackSetExecution...`
- `AWSLabs-LabFunction-LabAdmin-v1-...`
- `AWSLabs-LabFunction-LabAdmin-v2-...`

The presence of service roles is normal in AWS environments. Their trust relationships and permissions must nevertheless be reviewed when investigating suspicious activity.

---

### IAM Policies

IAM policies define what actions an identity is allowed or denied to perform.

The IAM Policies page displayed:

`1,571 policies`

These included AWS-managed policies such as:

`AdministratorAccess`

An administrator-level policy provides broad access across AWS services and therefore requires careful control.

The policy library count should not be confused with the number of policies actively created or attached within the account.

---

### AWS CloudTrail

AWS CloudTrail records API activity performed within AWS.

CloudTrail events can provide information such as:

- Event name
- Event time
- Identity
- Source IP address
- Region
- User agent
- AWS service
- Resource
- Request parameters
- Result

CloudTrail is therefore an important source of evidence during cloud incident investigations.

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

## Limitations

The investigation was performed in an AWS Academy environment with restricted permissions.

The main limitation was the inability to execute:

`cloudtrail:LookupEvents`

because of an explicit Service Control Policy denial.

Therefore, this investigation does not claim that the environment was compromised.

Instead, it documents the available IAM configuration and the inability to retrieve CloudTrail activity using the current identity.

---

## Skills Demonstrated

- AWS IAM investigation
- IAM role analysis
- IAM policy analysis
- CloudTrail investigation
- AWS permission analysis
- Service Control Policy understanding
- Cloud security monitoring
- Evidence-based SOC investigation
- Investigation limitation documentation
- Cloud incident timeline construction

---

## Conclusion

This project demonstrates how a SOC analyst can investigate AWS IAM configuration and CloudTrail visibility while maintaining evidence-based conclusions. The environment contained no IAM users, 26 roles, and a large library of AWS-managed policies, while CloudTrail investigation was restricted by an explicit SCP denial on `cloudtrail:LookupEvents`. The key lesson is that restricted visibility must be documented as an investigation limitation rather than interpreted as proof that no suspicious activity occurred.
