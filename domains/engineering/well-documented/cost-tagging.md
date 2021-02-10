---
title: Cost Tagging
nav_order: 1
parent: Well documented
grand_parent: Engineering
---

When operating in an AWS environment, it is important that costs of the account are tagged to specific components, projects and environments of the account so that it is possible to analyse later and understand where costs can be optimized.

Cost Tagging conventions followed at Commutatus are -

1. **project -** default to the project name
2. **component** - api, user-interface
3. **sub-component -** database, api-server, load-balancer, ui-server, elasticsearch, assets, user-generated-files
4. **environment -** staging, production, beta, development
5. **managed-by** - commutatus, customer-name
