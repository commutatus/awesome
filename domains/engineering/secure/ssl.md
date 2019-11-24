---
title: SSL
nav_order: 2
parent: Secure
grand_parent: Engineering
---
*Secure Sockets Layer* (**SSL**) is a protocol developed by Netscape for transmitting private documents via the Internet. SSL uses a cryptographic system that uses two keys to encrypt data − a public key known to everyone and a private or secret key known only to the recipient of the message.

Following are the ways we apply SSL in various frameworks.

<details markdown="1">
  <summary>Rails</summary>
  {:.pointer}
  SSL can be configured on a rails stack from cloud66 console using the following steps.
  {:.pl-4}
  1. On the stack page, add an `Add-In` from the Add-Ins menu.
  	[![add-ins page](/assets/images/add-ins-c66.png)](/assets/images/add-ins-c66.png)
  2. Find the `SSL` add-in in the networking menu and click `INSTALL NOW`.
  3. In the `New SSL Certificate Information` select LetsEncrypt.
  4. Enter the allowed domains (complete domain name) and click `Add LetsEncrypt SSL`.
  	[![certificate page](/assets/images/domain-name-ssl-c66.png)](/assets/images/domain-name-ssl-c66.png)
  {:.pl-7}
</details>
{:.pl-4}

<details markdown="1">
  <summary>Elastic Beanstalk - NodeJS</summary>
  {:.pointer}
  Configuring SSL to NodeJS application deployed on Application Load Balanced AWS Elastic Beanstalk application. This will be done in 2 parts. These are:
  {:.pl-4}
  1. Creating an SSL certificate using AWS Certificate Manager (ACM).
  2. Configuring SSL to your load balancers.
  {:.pl-7}
  You will need an AWS account for this. If you don’t have, create one.
  {:.pl-4}

  <details markdown="1">
  <summary>Creating a certificate</summary>
  {:.pointer}
  1. Login to **AWS console**.
  2. Click on the **Services** option on the top left corner, then under **Security, Identity, & Compliance** heading go to **Certificate Manager**.
  3. On **ACM** page select **Provision certificates**.
  4. Keep the default selection checked *(Request a public certificate)* and click on **Request a certificate** button.
      [![ssl-creation-request](/assets/images/elb-ssl-creation-request.png)](/assets/images/elb-ssl-creation-request.png)
  5. Add the full domain name you have purchased and click **Next**.
      [![add-domain](/assets/images/elb-ssl-creation-add-domain.png)](/assets/images/elb-ssl-creation-add-domain.png)
  6. In the **Select Validation method,** select **DNS Validation** and click on **Review**.
      [![Select validation method](/assets/images/elb-ssl-creation-validation-method.png)](/assets/images/elb-ssl-creation-validation-method.png)
  7. Review the selected values for **Domain name** and **Validation** and click **Confirm and request button**.
  8. This takes some time around 5–10 minutes to issued by Amazon. By the time you can Add the highlighted `CNAME` record values to the DNS configuration for your domain and click **Continue**.
      [![CNAME Records](/assets/images/elb-ssl-creation-cname-records.png)](/assets/images/elb-ssl-creation-cname-records.png)
      **NOTE: — Adding CNAME records, you have to add to the DNS provider platform. This might get automatically done if your domain is hosted on Route 53**
  {:.pl-7}
  </details>
  {:.pl-4}

  <details markdown="1">
  <summary>Adding SSL to Load Balancers</summary>
  {:.pointer}
  1.  Go to **Services** > Under **Compute** heading > Select **EC2 (Elastic Compute Cloud).**
  2. On the left panel of the **EC2** page, under **Load balancing,** select **Load Balancers.**
      [![LB selection](/assets/images/elb-goto-load-balancers-page.png)](/assets/images/elb-goto-load-balancers-page.png)
  3.  Select your desired environment *(if multiple),* and under **Listeners** tab click **Add Listener**.
      [![Add listener](/assets/images/elb-lb-add-listener.png)](/assets/images/elb-lb-add-listener.png)
  4. Change the protocol and port to **HTTPS** and **443** respectively. Then under the **Default action(s)** section, click **+ add action** and select **Forward to…** option and select your application name and click the **☑** button. After that select the **default SSL certificate** from the drop and click on the **Save** button at the top.
      [![HTTPS forwarding](/assets/images/elb-lb-https-forward.png)](/assets/images/elb-lb-https-forward.png)
      **By doing this, anybody who visits the `https://<your_domain>.<extension>` URL will forwards to our Angular application under HTTPS protocol. But what if someone comes to `http://` URL of our application? For this, we have to redirect our traffic to `https://` . We will see it in next steps below:**
  5. For the same environment, again under the **Listener’s** tab, select the **HTTP** option and click the **Edit** button.
      [![HTTP Edit config](/assets/images/elb-lb-http-edit.png)](/assets/images/elb-lb-http-edit.png)
  6. Here you don’t have to change the protocol, instead, you just have to add the default action(s) as **Redirect to…** and fill port number as **443** next to HTTPS dropdown > click on **☑** button > click on **Update.**
      [![Redirect http to https](/assets/images/elb-lb-https-redirect.png)](/assets/images/elb-lb-https-redirect.png)
  7. Stay on the same page and from left pane under the **Network & Security** heading select, **Security groups.**
  8. Select one instance of EC2 > Choose **Inbound** > Click **Edit** *(popup opens) >* Click **Add rule** > Select **HTTPS** from the dropdown > Click **Save**.
      [![Adding rules to inbound traffic](/assets/images/elb-lb-add-rules-security-grps.png)](/assets/images/elb-lb-add-rules-security-grps.png)
  9. Repeat **step 8** for other instances as well. Wait for a while around 10 minutes maybe and then check `your_domain.ext` it should work!
  {:.pl-7}
  </details>
  {:.pl-4}
</details>
{:.pl-4}
---

- ***DNS***



----


Supplementary resources -
