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

Identity-based policies can be applied to users, groups, and roles.

Account root user = the user created during the signup process. HAS FULL PERMISSIONS! Use of this user should be avoided! This user should have MFA (Multi-Factor Authentication) enabled!

Individual users = Up to 5000 users per account. NO PERMISSIONS BY DEFAULT! Users have a friendly name and an ARN (Amazon Resource Name).
ARNs are assigned to every resource in AWS. They contain the resource number (length 12), the type of resource, and its friendly name. Example: arn:aws:iam::012345678901:user/Pippo

Users can authenticate to AWS Web console using a username and password. 

Users can authenticate to AWS APIs or use the CLI using access keys.

Users can be members of up to 10 groups.

Policies = documents that define permissions. Written in JSON.
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




Bucket Policy = Policy assigned to a resource that manages access to itself.

