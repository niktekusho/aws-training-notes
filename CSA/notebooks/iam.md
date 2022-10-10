# AWS Identity and Access Management Service

IAM = Identity and Access Management

IAM manages the authentication and authorization of AWS services.

IAM is called on interactions with AWS using the web console, the local AWS CLI, and the API.

All "principals" must be authenticated to send requests to AWS (with a few exceptions).

Principals = person or application that can request an action or operation on an AWS resource.

Principals = User + Role + Federated User (?) + Application

Note: one AWS account usually has multiple principals.

Permissions can be assigned to users, and policies can control access to resources:

- Identity-based policies
- Resource-based policies

The policies determine whether to authorize the request.
Every time we interact with a resource, we call an API action that needs to be authorized.
The user requesting the action must have the appropriate permissions (identity policy), and the appropriate action must be allowed by the resource policy.

Group = Collection of users used to organize them AND APPLY PERMISSIONS TO THEM

Role = identity with a specific set of permissions.

Users are assigned to groups.
Policies are assigned to groups.

Roles are assumed by users, applications, and services. When a role is assumed, the identity gains the role's permissions.

Policies define the permissions for the identities or resources they are associated with.

Users gain permissions applied to the group through the policy.

Account root user = the user created during the signup process. HAS FULL PERMISSIONS! Use of this user should be avoided! This user should have MFA (Multi-Factor Authentication) enabled! It's difficult/impossible to remove permissions to this account!

Individual users = Up to 5000 users per account. NO PERMISSIONS BY DEFAULT! Users have a friendly name and an ARN (Amazon Resource Name).
ARNs are assigned to every resource in AWS. They contain the resource number (length 12), the type of resource, and its friendly name. Example: arn:aws:iam::012345678901:user/Pippo

Users can authenticate to AWS Web console using a username and password. 

Users can authenticate to AWS APIs or use the CLI using access keys.

Users can be members of up to 10 groups.

A group can have up to 10 "custom"/managed policies.

Policies = documents that define permissions. Written in JSON.

Bucket Policy = Policy assigned to a resource that manages access to itself.

Individual users have a name and an alias. The alias is used to login to the web console.
Individual users can be created with access to the web console, SDKs/APIs or both.
Individual users can be assigned tags to organize, track or control access to them.

MFA = something you know (password) + something you have (OTP with virtual or physical token) + [something you are (biometrics), not used in AWS]

Physical tokens come from third parties!

IAM DOES NOT REQUIRE A REGION!!! -> Every user in a AWS account is managed from a single service, regardless of region.

AWS STS = Security Token Service = provides short-lived credentials 

STS works using IAM Roles called "Instance Profiles": for an application "Pippo"" on EC2 that needs temporary access to an S3 bucket, "Pippo" requests STS to assume role "Can Access S3", then STS provides temporary credentials (AccessKeyId, Expiration, SecretAccessKey, SessionToken) to allow "Pippo" on EC2 to connect to S3.
STS can renew temporary credentials automatically before they expire.

Trust Policy controls who can assume the role. Example:

```json
{
	"Effect": "Allow",
	"Principal": {
		"Service": "ec2.amazonaws.com"
	},
	"Action": "sts:AssumeRole
}
```

Temporary credentials are used with identity federation, delegation, cross-account-access and IAM roles.

## Password policy

Default password policy = min 8 characters, min 3 types of characters (uppercase, lowercase, numbers, symobls), not equals to AWS account name or email

Users can change their own passwords if they have the required IAM permissions.
The password policy can enable ALL users to change their passwords.

If the password policy does not enable the permission, the same permission can be set for each groups of users via managed policies for finer control.


## Identity-based policies

Identity-based policies are JSON permissions policy documents that control:

- what actions an identity can perform,
- on which resources,
- under what conditions.

Identity-based policies can be applied to users, groups, and roles.

Identity-based policy example:

```json
{
	"Version": "2012-10-07",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "*",
			"Resource": "*"
		}
	]
}

```


Inline policies = 1-1 with the user, group or role. Not shareable with other users, groups or roles. Can not be attached to other users, gorups or roles.
Example: if you delete a user, inline policies get deleted along with it.

Managed policies = Standalone policies = Shared between users, groups or roles. Can be attached to other users, groups or roles.
AWS managed or customer managed. AWS managed policies can not be modified but they can be used.


## Resource-based policies

Resource-based policies are JSON permissions policy documents attached to a resource (example S3).
Resource-based policies grant the specified principal permissions to perform specific actions on the resource.

Trust Policy in IAM roles is a resource-based policy.
Permissions Policy in IAM roles is an identity-based policy.

Resource-based policy example:

```json
{
	"Version": "2012-10-07",
	"Id": "Policy123456789",
	"Statement": [
		{
			"$id": "Stmt123456789",
			"Effect": "Allow",
			"Principal": {
				"AWS": "arn:aws:iam::123456789012:user/Pippo"
			},
			"Action": "s3:*",
			"Resource": "arn:aws:s3::somecompany",
			"Condition": {
				"StringLike": {
					"s3:prefix": "Confidential/*"
				}
			}
		}
	]
}

```


## Permissions Boundaries

Permission boundaries sets the max permissions that the entity can have.
Permission boundaries can be applied to users and roles.
Permission boundaries are used to limit the scope of permissions granted by a policy and avoid privilege escalation.

A has only IAMFullAccess.
A can create a user X via iam:CreateUser with the AdministratorAccess permission.
A now can use all other AWS resources via the X user.

How to solve the problem? Introduce permission boundary: A can create users with the same or fewer permissions!

## Authorize requests to AWS

1. AWS authenticates the principal that makes the request (IAM)
2. Request context is processed with the following attributes:
   - Actions: actions or operations the principal wants to perform
   - Resources: AWS resource object upon which actions are performed
   - Principal: user/role/federated user/application that sent the request
   - Environment data: IP address, user-agent, SSL status, time of day
   - Resource data: data related to the resource being requested
3. Resource-based policies are evaluated on the context
4. Identity-based policies are evaluated on the context


## Types of policy

- Identity-based policies - attached to users, groups or roles.
- Resource-based policies - attached to a resource, deifne permissions for a principal accessing it.
- IAM permissions boundaries - set the max permissions an identity-based policy can grant an IAM entity.
- AWS organizations Service Control Policies (SCP) - AWS organizations allow to organizations to manage multiple ACCOUNTS. SCP limits the max permissions for each ACCOUNT.
- Session policies - used with AssumeRole API actions


## Policy Evaluation Flow

If "normal" identity policy + "normal" resource policy = Permissions UNION

If "normal" identity policy + "bounded" resource policy = Permissions INTERSECTION

If "normal" identity policy + Org SCP = Permissions INTERSECTION

All requests are implicitly DENIED, except root user (full access).

Explicit ALLOWS in identity or resource policies override this default.

If a permission boundary, org SCP or session policy is active then it might override the previous ALLOWS.

AN EXPLICIT DENY IN ANY POLICY OVERRIDES ANY ALLOWS.

## IAM Policy Structure

```json
{
	"Statement": [{
		"Effect": "Allow|Deny",
		"Action": "API action to allow/deny",
		"Resource": "ARN of the resource affected by action",
		"Condition": { 
			"condition": {
				"key": "value"
			}
		}
	}]
}

```

Condition element is OPTIONAL and controls WHEN the policy is in effect.

## Best Practice

- Lock the root user credentials (and use MFA)
- Create individual IAM users
- Assign permissions to groups and then assign groups to IAM users (don't assign permissions directly to users)
- Always grant the least privilege you can grant
- While getting started with permissions, use AWS managed policies. Go further to custom policies once you're accustomed to the process and understand implications. Watch out for mistakes!
- When you start using custom policies, use customer managed policies instead of inline policies (attached directly to users).
- Regularly audit the permissions assigned to users and principals of an AWS account.
- Enforce complex password policy for ALL the users in the AWS account.
- Use roles for application running on EC2 and delegate permissions in general.
- Do not share access keys!
- Rotate credentials regularly.
- Remove unnecessary credentials regularly.
- When defining custom permissions (using JSON), leverage the "Condition" field to enforce extra security in the permission (example limit IP-address to access a resource).
- Monitor the activity in the account regularly.


## Architecture Patterns

1. A select group of users should be allowed to change their passwords.

Create a group for the selected few users and apply a permission policy that grants iam:ChangePassword API permission.

2. An EC2 instance must be delegated with permissions to a DynamoDB table.

Create a role for the EC2 instance and assign a permission policy that grants access to the database.

3. A company has just created an AWS account. They need to assign permissions to users based on job function.

Enable MFA for the root user.
Create IAM users and a group for each job function. Assign each user to the appropriate group(s). Then assign the appropriate permission policies to each group.

4. A solutions architect needs to restrict access to an AWS service based on the source IP address of the requester.

Create a custom permission policy and use the Condition property to control access based on the IP address.

5. A dev needs to make API calls from their AWS CLI.

Instruct the dev on how to create the access keys and how to use those to securely access AWS.

6. A group of users require full access to all EC2 API actions.

Create a permissions policy that uses a wildcard for the Action element relating to EC2 (ec2:*)
