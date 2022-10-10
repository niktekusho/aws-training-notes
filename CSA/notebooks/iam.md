# AWS Identity and Access Management Service

IAM = Identity and Access Management

IAM manages the authentication and authorization of AWS services:

- securely controls individual and group access to AWS resources;
- makes it easy to provide multiple users secure access to AWS resources;
- centralizes control of AWS account;
- enables shared access to a single AWS account via IAM users;
- not used for application-level authentication;
- is universal (global) and does not apply to regions;
- is eventually consistent (changes are not applied immediately);
- replicates data across multiple data centers around the world;

IAM DOES NOT REQUIRE A REGION!!! -> Every user in a AWS account is managed from a single service, regardless of region.

IAM manages:

- users;
- groups;
- access policies;
- roles;
- user credentials;
- user password policies;
- MFA (Multi-Factor Authentication);
- API keys for programmatic access.


IAM is called on interactions with AWS using the web console, the local AWS CLI, and the API.

IAM can assign temporary credentials to provide users with temporary access to services.

To sign-in to the web console you must provide:

- an account ID or account alias
- username
- password

Link url example: `https://{accountIdOrAlias}.signin.aws.amazon.com/console/`

AWS recommends to use the available SDKs to make API calls to IAM.
The IAM Query API allow a user to make direct calls to the IAM web service.

## Principals

Principals = person or application that can request an action or operation on an AWS resource.

Principals = User + Role + Federated User + Application

Federated User = User identity managed by a third-party service (example Okta, Active Directory, etc.)

Note: one AWS account usually has multiple principals.

The first principal of an AWS account is the root user.

Principal allows users and services to assume a role.

## Authentication

All "principals" must be authenticated to send requests to AWS (with a few exceptions).

### Interactive flow

Users can authenticate to AWS Management Console (web) using:

- username;
- password;
- MFA token (optional). 

It's possible to allow ALL users to change their passwords via the global password policy.

It's possible to allow SOME users to change their passwords:

1. disable the global "change password" option
2. use a managed IAM policy to grant permissions to change user for the selected users (using groups).

Default password policy = min 8 characters, min 3 types of characters (uppercase, lowercase, numbers, symobls), not equal to AWS account name or email

### MFA

MFA can be enforced for the root user and for individual users under the account.

MFA = something you know (password) + something you have (OTP with virtual or physical token) + [something you are (biometrics), not used in AWS]

Physical tokens come from third parties!

### Programmatic flows

Users can authenticate to AWS APIs or use the CLI using an access key ID and a secret key.
The secret access key is showed/returned only at creation time! 
If the secret access key is lost, a new pair of access keys needs to be generated.

A user can have 2 active access keys active at a time.

Access key ID and secret access key cannot be used to access the AWS Management Console!

Access key ID and secret access key shuold be used when calling an AWS API in:

- program code
- AWS CLI
- AWS PowerShell

It's possible to allow users to change their keys through IAM policy (not console?).

It's possible to disable a user's access key.

SSL/TLS certificates can authenticate a user with some services.

AWS recommends AWS Certificate Manager (ACM) to manage server certificates.
Users should use IAM server certificates only when HTTPS connections are required in a region not supported by ACM. 

## Authorization

Access to a resource is managed via permissions defined in policies.

Policies = JSON documents that specify the allowed and/or denied permissions. Policies define the permissions for the identities or resources they are associated with.

2 types of permission policies:

- Identity-based policies
- Resource-based policies

By default new users are created with NO access to any AWS services and can only login to the AWS web console.

Permissions must be explicitly granted to allow a user to access an AWS service.

The policies determine whether to authorize the request.
Every time we interact with a resource, we call an API action that needs to be authorized.
The user requesting the action must have the appropriate permissions (identity policy), and the appropriate action must be allowed by the resource policy.

Request context = info used by IAM to allow or deny a request to AWS.

1. AWS authenticates the principal that makes the request (IAM)
2. Request context is processed with the following attributes:
   - Actions: actions or operations the principal wants to perform
   - Resources: AWS resource object upon which actions are performed
   - Principal: user/role/federated user/application that sent the request
   - Environment data: IP address, user-agent, SSL status, time of day
   - Resource data: data related to the resource being requested
3. Resource-based policies are evaluated on the context
4. Identity-based policies are evaluated on the context

IAM checks each policy that matches the request context:

- if a single policy has a deny -> exit early with DENIED
- if a single policy has a deny and an allow -> DENIED
- if no explicit deny and an allow -> ALLOWED
- if no policy -> DENIED

The root user has access to all resources in the account by default.

## Actions and resources

Action = operations exposed by a service (view, create, edit, delete, etc.) and performed on a resource.

Resource = entity that exists in a service (example EC2 instances, DynamoDB tables, IAM users)

Actions must be allowed explicitly in the resource policy.

A principal can perform an action if an ALLOW policy exists which include:

- the affected resource 
- the action
- the principal

## Users

User = entity that represents a person or service.

Users can be assigned:

- a password to access the AWS Management Console
- an access key ID and a secret access key for programmatic use

IAM user components:

- username;
- password;
- permissions to access various resources.

Account root user = the user created during the signup process. HAS FULL PERMISSIONS! Use of this user should be avoided! This user should have MFA (Multi-Factor Authentication) enabled! It's impossible to remove permissions to this account!

Account root user credentials = email address used to create an AWS account and password

Individual users = Up to 5000 users per account. NO PERMISSIONS BY DEFAULT! Users have a friendly name and an ARN (Amazon Resource Name).
ARNs are assigned to every resource in AWS. They contain the unique ID (length 12), the type of resource, and its friendly name. Example: arn:aws:iam::012345678901:user/Pippo

IAM user unique ID is returned only when the user is created using API, AWS Powershell and AWS CLI.  

Individual users have a name and an alias. The alias is used to login to the web console.
Individual users can be created with access to the web console, SDKs/APIs or both.
Individual users can be assigned tags to organize, track or control access to them.

Service accounts = IAM users created to represent applications

Users gain permissions applied to the group through the policy.

Users can be members of up to 10 groups.

## Groups

Group = Collection of users used to organize them AND APPLY PERMISSIONS TO THEM

A group is NOT an identity and so cannot be identified as a principal in policies!

Groups cannot be nested!

A group can have up to 10 "custom"/managed policies.

Users are assigned to groups.
Policies should be assigned to groups.

## Roles

Role = identity with a specific set of permissions.

Roles are assumed by users, applications, and services. When a role is assumed, the identity gains the role's permissions.

There are no credentials associated with a role!

IAM users can temporarily assume a role to acquire permissions to perform a specific task.

A role can be assigned to a federated user (external identity provider).

Temporary credentials derived from roles automatically expire.

Roles can be assumed:

- via AWS Management Console
- via AWS CLI, AWS PowerShell or AWS API

Roles can be used for granting applications running on EC2 instances permissions to AWS APIs via EC2 instance profiles.

1 EC2 instance = max 1 role

Role is assigned at creation time or can be assigned at any later time. 

Roles have 2 types of policies:

- Permission policy: grants the user of the role the specified permissions on a resource. The user also NEEDS to have the policy (not only the role?) !?
- Trust policy: specifies the trusted principals allowed to assume the role.

The all wildcard * cannot be used for roles principals.

Trust Policy example:

```json
{
	"Effect": "Allow",
	"Principal": {
		"Service": "ec2.amazonaws.com"
	},
	"Action": "sts:AssumeRole
}
```

## Policies

Policies = documents that define permissions. Written in JSON.

Policies are applied to users, groups and roles.

The IAM policy simulator tool is great at helping test and validate the effects of a policy.

It's possible to conditionally apply a policy via the "Condition" element in the policy.

Policies can be Standalone or Inline.
Standalone policies have ARN that includes the policy name.

Bucket Policy = Policy assigned to a resource that manages access to itself.

### Identity-based policies

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

### Resource-based policies

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

### AWS managed policies

- Standalone policies created and administered by AWS
- Can be used on common job functions
- Can be attached to multiple users, groups or roles across ALL users in ALL AWS accounts
- Cannot change the permissions assigned

Job specific AWS managed policies:

- Administrator
- Billing
- Database Administrator
- Data Scientist
- Developer Power User (user access allows all permissions except the management of groups and users in IAM)
- Network Administrator
- Security Auditor
- Support User
- System Administrator
- View-Only User

### Customer managed policies

- Standalone policies created by the customer
- Can be attached to multiple users, groups or roles across ALL users in ONE AWS account
- Can be created by copying an AWS managed policy and customized
- Useful when AWS managed policies don't meet the needs

### Inline policies

- Policies embedded in the user, group or role
- 1:1 relationship with the entity (entity deletion will delete the policy)
- Can not be attached to other users, groups or roles
- Useful when the permissions in a policy should not be assignable to other entities.

### Permissions Boundaries

Permission boundaries sets the max permissions that the entity can have.
Permission boundaries can be applied to users and roles.
Permission boundaries are used to limit the scope of permissions granted by a policy and avoid privilege escalation.

A has only IAMFullAccess.
A can create a user X via iam:CreateUser with the AdministratorAccess permission.
A now can use all other AWS resources via the X user.

How to solve the problem? Introduce permission boundary: A can create users with the same or fewer permissions!


### Types of policy

- Identity-based policies - attached to IAM identity (users, groups or roles).
- Resource-based policies - attached to a resource, define permissions for a principal accessing it.
- IAM permissions boundaries - set the max permissions an identity-based policy can grant an IAM entity.
- AWS organizations Service Control Policies (SCP) - AWS organizations allow to organizations to manage multiple ACCOUNTS. SCP limits the max permissions for each ACCOUNT.
- Session policies - used with AssumeRole API actions


### Policy Evaluation Flow

If "normal" identity policy + "normal" resource policy = Permissions UNION

If "normal" identity policy + "bounded" resource policy = Permissions INTERSECTION

If "normal" identity policy + Org SCP = Permissions INTERSECTION

All requests are implicitly DENIED, except root user (full access).

Explicit ALLOWS in identity or resource policies override this default.

If a permission boundary, org SCP or session policy is active then it might override the previous ALLOWS.

AN EXPLICIT DENY IN ANY POLICY OVERRIDES ANY ALLOWS.

### IAM Policy Structure

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

## IAM Instance Profiles

IAM Instance Profile = container for IAM role used to pass data to EC2 instance when it (EC2 instance) starts. It contains max 1 role (a role can be included in multiple profiles).

AWS CLI commands:

- `aws iam create-instance-profile`
- `aws iam add-role-to-instance-profile`
- `aws iam list-instance-profiles`
- `aws iam list-instance-profiles-for-role`
- `aws iam get-instance-profile`
- `aws iam remove-role-from-instance-profile`
- `aws iam delete-instance-profile`


## AWS Security Token Service (STS)

AWS STS = provides short-lived credentials for IAM users or federated users

- STS global service by default
- STS region service if desired
- supports AWS CloudTrail (records AWS calls and put log files in S3 buckets)
- temporary credentials can last from minutes to hours
- STS can renew temporary credentials automatically before they expire if the user requesting them still has permissions to do so
- temporary credentials are not stored, but are generated on the fly

Advantages:

- don't publish applications with long-term credentials
- provide access to AWS resources to users without having to define an AWS identity for them
- temporary credentials -> no manual revoke
- credentials cannot be used when they expire

APIs:

- `AssumeRole` (IAM users only)
- `AssumeRoleWithSAML` (like `AssumeRole` but for federated users coming from a SAML authentication response)
- `AssumeRoleWithWebIdentity` (like `AssumeRole` but for users coming with a web identity token)
- `GetSessionToken` (IAM users and root users only)
- `GetFederationToken` (IAM users and root users only)

STS works using IAM Roles called "Instance Profiles": for an application "Pippo"" on EC2 that needs temporary access to an S3 bucket, "Pippo" requests STS to assume role "Can Access S3", then STS provides temporary credentials (AccessKeyId, Expiration, SecretAccessKey, SessionToken) to allow "Pippo" on EC2 to connect to S3.

Temporary credentials are used with identity federation, delegation, cross-account-access and IAM roles.

## Cross Account Access

- Lets users from one AWS account access resources in a different AWS account.
- Makes it easier to switch roles within AWS Management Console.
- The accessed resource must have a resource-based policy with the appropriate permissions or the user must assume a role (identity-based policy) with the required permissions.

Scenario:

1. Account A has role X
   1. Role X has permission policy "ALLOW role X access to bucket"
   2. Role X has trust policy "ALLOW user PIPPO from account B to assume role X"
2. User PIPPO from account B has identity policy "ALLOW PIPPO to assume role X in Account A"
3. PIPPO `sts:AssumeRole`s from B to A
4. PIPPO is allowed to access bucket


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
