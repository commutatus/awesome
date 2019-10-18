---
title: AWS credential management
nav_order: 2
parent: Secure
grand_parent: Engineering
---

Managing AWS credentials in an organized and secure fashion is very important specially when working on multiple client projects across multiple AWS accounts.

Here are the recommendations based on [AWS recommendations](https://aws.amazon.com/blogs/security/a-new-and-standardized-way-to-manage-credentials-in-the-aws-sdks/).

Store all AWS access keys & secrets in the following files based on your operating system - 

- ~/.aws/credentials (Linux/Mac)
- C:\Users\*USERNAME*\.aws\credentials (Windows)

AWS scripts will automatically look for credentials in these locations.

The format for the credentials is the same for all the SDKs and the AWS CLI:

```
[default]
aws_access_key_id = ACCESS_KEY
aws_secret_access_key = SECRET_KEY
aws_session_token = TOKEN
```

When managing credentials for multiple AWS accounts, use the following format - 

```
[default]
aws_access_key_id = ACCESS_KEY
aws_secret_access_key = SECRET_KEY
aws_session_token = TOKEN
 
[client-1]
aws_access_key_id = client_1_access_key_ID
aws_secret_access_key = client_1_secret_access_key
 
[client-2]
aws_access_key_id = client_2_access_key_ID
aws_secret_access_key = client_2_access_key_ID
```

Whe managing multiple credentials for the same AWS account, use the following format - 

```
[default]
aws_access_key_id = ACCESS_KEY
aws_secret_access_key = SECRET_KEY
aws_session_token = TOKEN
 
[client-1-fe-deployment]
aws_access_key_id = client_1_fe_deployment_access_key_ID
aws_secret_access_key = client_1_fe_deployment_secret_access_key
 
[client-1-s3]
aws_access_key_id = client_2_fe_deployment_access_key_ID
aws_secret_access_key = client_2_fe_deployment_secret_access_key
```

These two strategies can be combined to have very fine grained and easy to organize credentials.

Usage of AWS commands based on the profile is done by appending the `--profile` keyword.

```
aws s3 cp test.txt s3://mybucket/test2.txt --profile=client-1-s3
```

Some recommendations 

- Never make AWS Access Keys & Secrets for the root AWS account, always use the IAM account.
- Agree on the naming convention for the profile name with your team mates so that scripts can be built that automatically pick this up.



