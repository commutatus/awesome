---
title: AWS IAM Management
nav_order: 7
parent: Secure
grand_parent: Engineering
---

AWS Accounts are very sensitive and hence IAM accounts must be created for different parts of the application with specific naming conventions.

We follow prefixes in the name of the IAM users to identify the type of IAM accounts they are 

1. Developer - `dev_`
2. Deployment - `deployment_`
3. Application - `application_`

**Developers**

1. Never use the root account, but create an IAM account with just the amount of permissions that are required for you to use the platform. 
2. Your secret keys must only be used on your local system and must be organised using the standards defined in [AWS credential management](https://awesome.commutatus.com/domains/engineering/secure/aws-credentials.html).
3. Never use your keys in an application or share them with another developer.
4. Examples of naming convention - `dev_mani`, `dev_mkv`, `dev_aditya_tiwari`
5. All developer IAM accounts **must** have 2FA enforced.

**Deployment**

1. IAM accounts starting with `deployment_`are using by CI/CD platforms such as Github Actions, Travis or Cloud66.
2. These IAM accounts must not have console access and their permissions must be granular. An example of Cloud66's requirements can be found [here](https://help.cloud66.com/c66_aws_iam_policy.json).
3. Examples of naming convention - `deployment_cloud66`, `deployment_gh_actions`, `deployment_travis`

**Deployment**

1. IAM accounts starting with `application_` are keys that are for the application.
2. Each environment of the application must have a different IAM account.
3. These IAM accounts must not have console access.
4. Examples of naming convention - `application_development`, `application_staging`, `application_production`