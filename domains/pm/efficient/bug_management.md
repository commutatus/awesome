---
title: Bug Management
nav_order: 1
parent: Efficient
grand_parent: PM
---
# Bug Management

This is a document which explains how Bug Management should work in the project. The contents of this document are:

<details markdown="1">
  <summary>Definition of a bug</summary>
  {:.pointer}

  A **bug** is an error, flaw, failure or fault in a program or system that causes it to produce an incorrect or unexpected result, or to behave in unintended ways.
  {:.pl-4}
</details>

<details markdown="1">
  <summary>Types of bugs</summary>
  {:.pointer}

  Bugs are classified on the basis of severity. While reporting a bug the reporter will be required to select the appropriate severity of the bug being reported. Each type of bug is explained below:
  {:.pl-4}

  <h3 class='pl-4 m-0 mb-2'>Severity 1 – Critical</h3>
  Complete failure of the system. The user is unable load or access the system. _Eg. One of the platforms is down._
  {:.pl-4}

  <h3 class='pl-4 m-0 mb-2'>Severity 2 – High</h3>
  The primary user flow is affected which prevents the user from performing actions on the system that are critical to business operations. _Eg. A user on YOP is unable to apply for an opportunity_.
  {:.pl-4}

  <h3 class='pl-4 m-0 mb-2'>Severity 3 – Moderate</h3>
  An essential part of the system which offers assistance to the user but is not part of the primary user flow is not working. _Eg. Analytics in EXPA is not working_.
  {:.pl-4}

  <h3 class='pl-4 m-0 mb-2'>Severity 4 – Low</h3>
  A functionality which does not have a high degree of impact on the primary actions a user can perform on the system is not working. _Eg. The gender filter in the CRM is not working._
  {:.pl-4}

  <h3 class='pl-4 m-0 mb-2'>Severity 5 – Cosmetic</h3>
  There is a problem with the look and feel of a part of the system which has no consequence on the functionality of that part of the system. _Eg. Alignment, Color, Text issues._
  {:.pl-4}
</details>

<details markdown="1">
  <summary>Bug Impact</summary>
  {:.pointer}

  - **Single user:** Only a single user is affected from the bug.
  - **Multiple users:** Number of users affected are more than 1.  
  - **All users:** All users are affected.  
  - **Not sure:** Not sure if more users are affected.
  {:.pl-7}
</details>

<details markdown="1">
  <summary>Priority guideline</summary>
  {:.pointer}
  - The following serves as a guideline to choose the priority of a bug.  
  - However if there is a Severity 5 or any other severity issue which needs to be done urgently then the priority field can be used to denote its urgency.
  {:.pl-7}

  Priority guideline matrix | Severity 1 – Critical | Severity 2 – High | Severity 3 – Moderate | Severity 4 – Low | Severity 5 – Cosmetic
  :--- | :--- | :--- | :--- | :--- | :---
  **All users** | Highest | High | Medium | Low | Lowest
  **Multiple users** | Highest | High | Medium | Low | Lowest
  **Not sure** | Highest | High | Medium | Low | Lowest
  **Single user** | Medium | Low | Low | Lowest | Lowest
  {:.priority-guideline-table}
</details>

<details markdown="1">
  <summary>Different statuses and their meaning</summary>
  {:.pointer}

  ![different-status-meaning](/assets/images/different-status-meaning.png)
  
  Status | Meaning
  :---: | :---
  `TO DO` | 1. Initial status of any ticket after creation.<br/> 2. The ticket has been created with all the mandatory fields filled out.
  `ACKNOWLEDGED` | The developer/product owner has seen the bug ticket and is aware of the severity, impact and priority it holds but has not started any investigation on it.
  `IN PROGRESS` | The assigned developer is either investigating or fixing the issue.
  `CODE REVIEW` | The developer has completed work on the ticket and has requested for his code to be reviewed by a peer.
  `QUALITY CHECK` | 1. The developer’s code has been approved and the fix has been deployed to the staging environment.<br/> 2. It is being reviewed by the assigned developer on the staging requirement.
  `STAGING` | 1. The developer has verified that the fix works as expected on the staging environment.<br/> 2. The product owner is in the process of verifying whether the fix works as expected or not on the staging environment.
  `STAGING VERIFIED` | 1. The product owner has verified that the fix works as expected on the staging environment.<br/> 2. The product owner has given the assigned developer the permission to deploy the code to the production environment.
  `PRODUCTION` | 1. The fix has been deployed to the production environment.<br/> 2. The AIESEC platform manager is in the process of verifying the fix on production.
  `COMPLETED` | The fix has been verified on production and the ticket has been resolved and closed.
  {:.different-status-table}

  Transition | Status change | Who does this | When do they do this?
  :--- | :--- | :--- | :---
  Acknowledge the bug | `TO DO` → `ACKNOWLEDGED` | Development team leader | The team leader has: <br> - Read the bug ticket.<br> - Understood the bug being reported.<br> - Confirms there is sufficient information for the team to proceed with investigation.
  Start work | `ACKNOWLEDGED` → `IN PROGRESS` | Assigned developer | The developer has initiated investigation on the bug.
  Submit for code review | `IN PROGRESS`  → `CODE REVIEW` | Assigned developer | The developer has:<br> - Fixed the reported issue.<br> - Requested their team leader to review the code before the code can be put on the staging environment.
  Submit for quality check | `CODE REVIEW` → `QUALITY CHECK` | Lead developer | The team leader has:<br> - Reviewed the code and approved it.<br> - Deployed the code to the staging environment.
  Changes required | `CODE REVIEW` → `IN PROGRESS` | Lead developer | The team leader:<br> - Is not satisfied with the code submitted for review.<br> - Has given specific feedback on what needs to be changed.
  Deploy to staging | `QUALITY CHECK` → `STAGING` | Assigned developer | The developer has:<br> - Verified that the fix works as expected on the staging environment.
  Verify on Staging | `STAGING` → `STAGING VERIFIED` | Product owner / Development team leader (for technical bugs) | The product owner has:<br> - Verified that the fix works as expected on the staging environment.
  Changes requested | `STAGING` → `IN PROGRESS` | Product owner / Development team leader (for technical bugs) | The product owner has:<br> - Found a mis-match between the current output and the expected output as per the description of the ticket.
  Deploy to beta | `STAGING VERIFIED` → `BETA` | Development team leader | The team leader has:<br> - Deployed the code to the beta environment.
  Verify on beta | `BETA` → `BETA VERIFIED`<br><br>This transition will only occur in a situation where the development team leader feels that the bug needs to be verified on beta before moving it to production. | AIESEC product manager | The AIESEC product manager has:<br> - Verified that the fix works as expected on the beta environment.
  Deploy to production (directly from beta) | `BETA` → `PRODUCTION` | Development team leader | The development team leader has:<br> - Deployed the code to the production environment.
  Verify on production | `PRODUCTION` → `COMPLETED` | AIESEC product manager | The AIESEC product manager has:<br> - Verified that the fix works as expected on the production environment.
  {:.different-status-table}

  The overall ownership of the ticket is on the developer assigned. They are responsible for ensuring the ticket moves to completed at the earliest.
  {:.info-bg.fs-3}

  The “Who does this“ field mentions the person who transitions the ticket in an ideal workflow. However, in the case where the current assignee of a ticket needs more information from someone or is blocked because of a dependency then the assignee can change to another person.
  {:.info-bg.fs-3}
</details>

<details markdown="1">
  <summary>Dependency management</summary>
  {:.pointer}

  Ideally all dependencies should be taken care of in the sprint planning meeting. However if there are dependencies realized after the sprint planning meeting please refer this [link](https://whimsical.com/VFhAvTGP4CKucTRv4xpJir) to learn how to manage it.
  {:.info-bg.fs-3}
</details>

<details markdown="1">
  <summary>Ticket fields specific to bugs</summary>
  {:.pointer}

  Name | Definition | Who fills? | When to fill?
  :--- | :--- | :--- | :---
  Severity | The degree of impact the bug has on the system. | Reporter | Creation of the ticket
  Impact | The number of users affected from the bug. | Reporter | Creation of the ticket
  Priority | The order in which a bug should be worked on. | Reporter | Creation of the ticket
  Bug source | What led to the bug on the production environment. | Developer assigned | `IN PROGRESS` → `CODE REVIEW`
  Bug report | A brief description of the core reason behind the issue  and what actions were taken that the bug does not recur. | Developer assigned | `IN PROGRESS` → `CODE REVIEW`
  Reporting source | How did the bug get exposed? | Reporter | Creation of the ticket
  {:.ticket-fields-specific-table}
</details>

<details markdown="1">
  <summary>Bug sources</summary>
  {:.pointer}

  A bug source is the reason which led the bug to make it to the production environment. Below all the sources/options are listed with their definition.
  {:.pl-4}

  Source | Definition
  :--- | :---
  Has already been resolved | The issue has already been resolved and the ticket being referred to doesn’t need to be worked on. This can happen in a case where between the time in which the user was affected by the bug and the time that a developer investigated the issue, a fix was deployed to the production system to fix the bug.
  Human error | Syntax errors that should not have passed tests. Non-adherence to business rules clearly defined in the issue. Feature does not work in the most general work flow as well.
  It is not a bug | After investigation we found out that the reported issue does not classify as a bug as per the definition of a bug. For example, a user could not perform an action because they didn’t have the permission to perform the action. In this case if the permissions are set-up correctly as per the permissions framework then this will not qualify as a bug.
  Legacy code | The issue was because of incompatibility with features delivered by other suppliers contracted previously or in case the feature has not seen any active development, usage in the last 6 months.
  System performance issues | The issue was because of additional load on the database/servers.
  Third party issues | The issue was because of compatibility with a third party service.
  Unable to reproduce | After multiple attempts the assigned developer was unable to reproduce the bug and hence is unable to investigate further.
  Was an unpredictable scenario | The issue occurred due to a scenario which could not have been predicted before the user actually experienced the bug.
  {:.bug-source-table}
</details>

<details markdown="1">
  <summary>Bug report</summary>
  {:.pointer}

  A bug report is a brief description of the core reason behind the issue and what actions were taken to ensure that the bug does not recur. A bug report should contain the following elements:
  {:.pl-4}

  <h3 class='pl-4 m-0 mb-2'>Bug source description</h3>
  - An elaboration on the bug source. For example if the bug source was “System performance issue“. then the explanation should involve what caused the number of requests to increase which increased the load on the database.
  {:.pl-7}

  <h3 class='pl-4 m-0 mb-2'>How was non-recurrence handled?</h3>
  - Explanation on what changes were made which not only fix the bug for the moment but also ensure that the bug will not recur in the near future.
  {:.pl-7}
</details>

<details markdown="1">
  <summary>Reporting source</summary>
  {:.pointer}

  Reporting source is the identification of the source from where the bug got exposed. It helps in understanding which bugs are being reported by users vs which are being caught by proactive measures. The table below explains each of the reporting sources and their definition.
  {:.pl-4}

  Reporting source | Definition
  :--- | :---
  AI Platforms team | The AI platforms team found this issue while manually testing the platform.
  Automated tests | The automated tests exposed the bug.
  Error tracking platform | One of the error tracking platforms like Sentry or Rollbar exposed the bug.
  Manual testing | The bug was exposed through manual testing done by the Commutatus team.
  User | The bug was reported by a user of the platform.

</details>

<details markdown="1">
  <summary>Description of the ticket</summary>
  {:.pointer}

  The description of every bug ticket should have the following elements:
  {:.pl-4.m-0.mb-2}

  <h3 class='pl-4 m-0 mb-2'>Overview</h3>
  - General explanation of the bug.   
  {:.pl-7}

  <h3 class='pl-4 m-0 mb-2'>Expected behavior</h3>
  - How should the feature / functionality being reported ideally behave.  
  {:.pl-7}

  <h3 class='pl-4 m-0 mb-2'>Actual behavior</h3>
  - How the feature / functionality being reported currently behaves.   
  {:.pl-7}

  <h3 class='pl-4 m-0 mb-2'>Instances of the bug</h3>
  In order of priority any of the following:
  {:.pl-4.mb-1}

  - Screen recording of the bug occurrence preferably with the console  
  - Screenshot of the bug occurrence preferably with the console  
  {:.pl-7}

  <h3 class='pl-4 m-0 mb-2'>Specific details</h3>
  - Details of the affected user(s) like email id, name, user id etc.  
  - Details of any other components affected like opportunity id, opportunity name etc  
  - Any other details which may be relevant to the bug
  {:.pl-7}

  <h3 class='pl-4 m-0 mb-2'>How to replicate</h3>

  If there is a screen recording of the entire flow then this section can be left empty. In case there is no screen recording available then the reporter is required to fill this in the format below:
  {:.pl-4.mb-1}
  - Go to the affected page
  - Perform the affected action
  - Check the wrong outcome
  {:.pl-7}

  Check out an example of a well reported bug below
  {:.ml-4.fs-3.success-bg}
  ![bug-description-example](/assets/images/bug-description-example.png)
  {:.ml-4}
</details>

<details markdown="1">
  <summary>How to report a bug?</summary>
  {:.pointer}

  Check out [here](https://whimsical.com/S3QudnTgygCUyEd33SkqoX) on how to report a bug.
  {:.pl-4}
</details>

<details markdown="1">
  <summary>Response and resolution timelines</summary>
  {:.pointer}

  &nbsp; | Response / Acknowledgement | Resolution
  :--- | :--- | :---
  Severity 1 | 30 minutes | 4 hours
  Severity 2 | 8 hours | 24 hours (unless communicated otherwise)
  Severity 3, 4, & 5 | 2 working days | As per complexity of the bug and the capacity available.

</details>

<details markdown="1">
  <summary>Verification timelines</summary>
  {:.pointer}

  For smooth bug management and accurate resolution of bugs, all bugs should be verified within 2 working days.
  {:.ml-4.fs-3.info-bg}
</details>
