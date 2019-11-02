---
title: Error Management
nav_order: 1
parent: Efficient
grand_parent: Engineering
---

Error Management

It's very important to have a standardised set of error codes and messages in a platform to ensure that the frontend and backend are aligned on how to handle different situations and react accordingly. 

The table below has the following components

- Code - A larger level grouping of the error similar to the status codes used in HTTP responses
- Sub Code - A sub set of the code identifying the specific error for a system.
- Message - A message that can be shown to the end user.



Some common errors are described here and are supported by the following application tempates 

- Commutatus Rails Templates
- Commutatus Angular Starter Pack
- Commutatus React Boilerplate



| Name                        | Code                  | Sub-code                  | Message                              | Comments                                 |
| --------------------------- | --------------------- | ------------------------- | ------------------------------------ | ---------------------------------------- |
| Failed Login                | unauthorized          | invalid_login_combination | Incorrect email/password combination |                                          |
| Invalid Access Token        | unauthorized          | invalid_access_token      |                                      |                                          |
| Permission Denied           | unauthorized          | permission_denied         |                                      |                                          |
| Internal Server Error       | internal_server_error | internal_server_error     |                                      | We normally also return the Rollbar UUID |
| ActiveRecord::RecordInvalid | unprocessable_entity  | record_invalid            | e.message from ActiveRecord          |                                          |

