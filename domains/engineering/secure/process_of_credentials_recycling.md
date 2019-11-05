---
title: Process of credentials recycling
nav_order: 6
parent: Secure
grand_parent: Engineering
---

Recycling the credentials is an important process in web development. It should be done at regular intervals or due to the following reasons.

## When should we recycle credentials

### The credentials were exposed

- 	The credentials should be recycled when they are exposed. 
-	The exposing credentials can be done by mistakenly by commiting the 	particular credentials in the commit(github) or some other version control.

Example:


-	If the developer working in any public repository or any other Open Source project, If he unknowingly exposed the credentials in any of his commit, the code which he committed can be viewed by anyone, and he/she can misuse the credentials from which we exposed.

-	At this case the credentials should be recycled.

### The credentials are lost

-	The Credentials should be recycled when the developer to lose the credentials

Example:

-	When the developer uses rails encrypted credentials feature, It needs master key to encrypt the credentials.
-	When we loose the master key there is no way to recover our credentials back, if we don't know what are the credentials we used to encrypt.

### Unauthorized Usage

-	The Credentials should be recycled when we suspect any unauthorized person uses our credentials.

Example: 

-	If the monthly bill of our postmark is increased suddenly, we should run a diagnosis if some unwanted emails were gone, if it is confirmed then our credentials got exposed in any form.
-	At this stage process of recycling the credentials is the safe way.
After following the above mentioned step, the better way to check how the keys got exposed, if it is not found then recycling of credentials won't help.

### Developer Quits

-	The Process of recycling the credentials is important when the developer quits the job, the one who works in that project, he may know the credentials which he worked.
-	He might can misuse the credentials or accidentally he may use our project credentials in some other Open Source project.
-	In this case its safe to recycle the credentialls at the time when the developer quits.

### Recycle credentials in interval basis

-	To Reduce chaos the Recycling Credentials should be done in a regular intervals.
-	In this case we can make sure our credentials or not compromised.

## What is the process

### Notify other users

-	In certain cases we can only use one credentials at a time, In this case if we generate new secret id or secret key we need to inform the other users who were using the old one because the old one became invalid if we generate the new credentials.
-	Then the old credentials should be replaced by the new one all over projects where we used.

Example: 

-	RazorPay has only one valid id and key at a time.
-	Secret keys are visible only at the time of key generation. If you have already generated your keys and do not remember your secret key, you will need to re-generate the keys from the Dashboard and replace it in your integration code.

### Generate new key

-	To recycle the credentials just generate a new key and replace everywhere in the project instead of the old one.

###	Test the newly generated key

-	After replacing the new credentials over the old one just test in the code whether the newly generated key works.

### Remove the old key

-	The major procedure to follow in the process of recycling the credentials is to removing the old key from the projects and from the third party apps.
-	Deleting the old key from the third party apps is the important one after generating the new one because the old keys still have access to the app.

Example:

-	Postmarkapp will allow multiple keys to be in active, if someone exposes some old key in any other open source, the intruder can use the postmarkapp by our old key, It is safe to remove old keys from the apps and from the code base.

### Test after deleting the old keys

-	In some other cases when we are testing the code base before deleting the old credentials after generating the new credentials, the function may work due to the old credentials, that's why its safe to test after deleting the old credentials too.

### Log

-	Its a good process to log the time and date of when we recycled the credentials in which project, so we know when to recycle the credentials next time.