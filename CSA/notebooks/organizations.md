# AWS Organizations and Control Tower

Service that allows to collect, organize and manage multiple AWS accounts under the same organization.

Organization's root account = Main account = Master account = Management account = account that created the organization

Organizational Units = containers used to group other AWS accounts (can be hierarchical)

Policies are applied to root accounts or Organizational Units.

Organizations can create AWS accounts programmatically via the Organizations API.

Organizations can also use AWS Single Sign On with on premise directories (ex Active Directory).

Management account can enable CloudTrail for accounts in the Organization.

## Feature Sets

?!

- Consolidated Billing = Single bill in the main account (Management account).
  Aggregates costs of each service across each account.
  Costs are shown by account and by service. Good when customers get volume pricing discounts. 
  Paying Account = Management Account = cannot access resources of other accounts
  Linked Accounts = Independent accounts (by default up to 20 max)
  Unused reserved EC2 instances are applied across the org

- All Features = Gives more features to the organization:
  - SCP (Service Control Policy): controls tagging and available API actions (must be enabled)
  - 



## Service Control Policy

SCP = mechanism to control the **maximum permissions available** in an Organization's accounts **using JSON**.

**SCPs can not be applied to the managament account's root user!**

**SCPs are applied to root users of non-management accounts!**

SCPs can be applied at different points in the hierarchy.

SCPs work via exclusions: to deny permissions to users customers need to create SCPs that deny explicitly.

By default the SCP rule `FullAWSAccess" is applied to every OU and account ("allow all"):

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "*",
			"Resource": "*"
		}
	]
}
```

Tag Policy enforces tag standardization ?

Example of a SCP (only t2.micro EC2 instances):

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"$id": "RequireMicroInstanceType",
			"Effect": "Deny",
			"Action": "ec2:RunInstances",
			"Resource": "arc:aws:ec2:*:*:instance/*",
			"Condition": {
				"StringNotEquals": {
					"ec2:InstanceType": "t2.micro"
				}
			}
		}
	]
}
```

SCPs are inherited by each organization units in the subtree they are applied.

Example:

```
Root -> Dev -> Prod ------------------> Cannot launch instances other than t2.micro 
         |
SCP:RequireMicroInstanceType
```


# AWS Control Tower

Control Tower = extension of Organizations that provides some additional control

Landing Zone = well-architected multi-account baseline based on AWS best-practices

Landing Zones use Guardrails for governance and compliance:

- Preventive guardrails: SCPs that disallow API actions
- Detective guardrails: AWS Config rules and Lambda functions that monitor activity in the Landing Zone

**Root user in the management account can perform actions disallowed by guardrails!**

Control Tower by default creates:

- Organization Root/Management account (shuold be there)
  - Security OU
    - Audit account (gathers audit info)
    - Log Archive account (consolidates logs from all accounts in the Organization)
  - Sandbox OU
    - Empty by default, should be the OU in which to put devs and tests accounts
  - Production OU
    - Empty by default, should be the OU in which to put production accounts

## Architecture Patterns

> Company needs a method of quickly creating AWS accounts (programmatically)

Use Organizations API or Console to create AWS accounts.

> Users in a member account in AWS Organizations should be restricted from making changes in IAM

Create SCP that deny changes in IAM.

> AWS account must be moved between Organizations

Use Organizations API or Console to migrate the account.

> Solutions architect created a new account and needs to launch resources

Architect should switch roles to access with the new account

> Multiple member accounts in AWS Organizations require the same permissions to be restricted using SCPs

Attach SCPs to OU and move accounts to restricted OU.

> Devs have each an AWS account for testing. Security team wish to enable central governance.

(Create org if not existing)
Invite AWS accounts into.
