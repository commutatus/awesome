---
title: DNS management
nav_order: 16
parent: Efficient
grand_parent: Engineering
---

1. TOC
{:toc}

# DNS management

One of the most crucial and critical parts of any IT infrastructure is the Domain Name Server (DNS). In simple words, DNS is like a phone book for IP addresses that helps users to access any website using human-readable text that is known as a domain name. DNS orchestrate all the complex working [^1] between an IP address and a domain name in the background. All the services online depend on it, which makes its management important.

**Points to keep in mind while creating DNS records,**
- Always prefer a CNAME record to forwards one domain or subdomain to another domain.
- Never point a record to an IP address.
- If it is absolutely necessary to use an IP address then switch to an Elastic IP address.
- To redirect a root domain to any subdomain use services like Amazon S3 and/or CloudFront rather than handled in your application code level or in the Nginx or Apache servers.

In this document, we will walkthrough the step-by-step instruction on how to associating an Elastic IP address with a running instance and how site redirection using Amazon S3, CloudFront, and Route 53 works.

## Elastic IP address

When working with a cloud environment like AWS, IP address assigned to the instances are dynamic in nature. If an instance goes down or restarted in AWS it is likely that the IP address will be changed. To maintain the communication of your website with the AWS account you would require to change the IP address in all the places it has been used, which is too much work.

To counter the issue it is encouraged to switch to Elastic IP address. According to AWS, an Elastic IP address [^2] is a static and public IPv4 address designed for dynamic cloud computing which is reachable from the internet.

By using an Elastic IP address,
- You can mask the failure of an instance or software by rapidly remapping the address to another instance in your account.
- It is allocated to your AWS account and is yours until you release it.
- It does not change over time and can be maintained across.

It is strongly encouraged to use an Elastic IP address primarily for the ability to remap the address to another instance in the case of instance failure and to use **DNS hostnames** [^3] for all other inter-node communication.

### Allocating an Elastic IP address

Public IPv4 addresses are allocated from either of the following pool of address,
- Amazon's pool of IPv4 addresses
- A pool that you own and bring to your account
- A pool that you own and continue to advertise

You can also use global IP addresses from AWS Global Accelerator [^4].

**To allocate an Elastic IP address**
1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/).
2. In the navigation pane, choose **Elastic IPs**.
3. Choose **Allocate new address**.
4. Choose an option from the required public IPv4 pool or use global IP addresses from AWS Global Accelerator. For **IPv4 address pool**, choose **Amazon pool**.
5. Choose **Allocate**, and close the confirmation screen.

To quickly find a specific Elastic IP address you can assign them custom tags, for example, **Name**, which will be a key-value pair (adding value to it is optional).

- Currently, Elastic IP addresses are not supported for IPv6.
- By default, all AWS accounts are limited to five (5) Elastic IP addresses per region. The limit is not applicable when you allocate it from a pool of IP addresses that you have brought to your AWS account.
{: .fs-3.pl-6.info-bg}

### Associating an Elastic IP address with a running instance

Now that you have allocated an Elastic IP address, you can associate it with a running instance to enable communication with the internet, you must also ensure that your instance is in a public subnet.

**To associate an Elastic IP address with an instance**
1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/).
2. In the navigation pane, choose **Elastic IPs**.
3. Select an Elastic IP address and choose **Actions -> Associate Elastic IP address**.
4. Select the instance from **Instance** and then choose **Associate**.

## Site redirection using Amazon S3, CloudFront, and Route 53

Consider you visited `some-website.com` and the browser changes the endpoint to `www.some-website.com` with HTTP 301 status code, indicating `some-website.com` is permanently moved to `www.some-website.com`. This is known as site redirection. It is a process of forwarding/changing the requested endpoint to a different one.

It is generally done when a website is rebranded, moved to a different domain, and/or URL has been (or needs to be) changed. If you don't want to see the panicky **404 Not Found** response from the browser then site redirection is what you are looking for.

You may wonder how site redirection is different from creating a CNAME record at the DNS level in the server? When a CNAME record is pointing towards the value/route traffic, it will fetch the content from it, but the URL won't be changed. For instance, if you have a CNAME record `www.some-website.com` which is pointing to `some-website.com` then the browser will render all the content from `some-website.com` to `www.some-website.com` but the URL will not be redirected in the user's browser.

The site redirection can be handled at your application code level or in the Nginx or Apache servers, but you can achieve this with AWS S3 as well.

### Creation of Amazon S3 bucket

The first step is to create an Amazon S3 bucket that will have the same name as the DNS record and allow public access.

**To create an Amazon S3 bucket**
1. Open the [Amazon S3 console](https://console.aws.amazon.com/s3/).
2. Choose **Create Bucket**, it will open a wizard.
3. In **Bucket name**, enter a DNS-compliant name for your bucket.
4. In **Region**, choose the AWS Region where you want the bucket to reside.
5. In **Bucket settings for Block Public Access**, make sure you uncheck the Block Public Access settings for your bucket.
6. Choose **Create bucket**.

- After creation bucket name can't be changed.
{: .fs-3.pl-6.info-bg}

### Enable static website hosting

To redirect the request, you need to enable static website hosting for your bucket and add the target bucket website address or personal domain.

**To redirect requests for a bucket website**
1. Open the [Amazon S3 console](https://console.aws.amazon.com/s3/).
2. Under **Buckets**, choose the name of the bucket that you want to redirect requests from (for example, `some-website.com`).
3. Choose **Properties**.
4. Under **Static website hosting**, choose **Edit**.
5. Choose **Enable** for static website hosting.
6. Choose **Redirect requests for an object** for hosting type.
7. Enter the **host name** you want to redirect request to.
8. Choose the **Protocol** (optional).
9. Choose **Save changes**.

- Have a **CNAME record** created for the host name you want to redirect the request to.
{: .fs-3.pl-6.info-bg}

### Add bucket policy

Once you enabled the static website hosting, you need to grant public read access to your bucket by adding a bucket policy. It will allow anyone on the internet to access your bucket.

**To add bucket policy**
1. Open the [Amazon S3 console](https://console.aws.amazon.com/s3/).
2. Under **Buckets**, choose the name of your bucket.
3. Choose **Permissions**.
4. Under **Bucket Policy**, choose **Edit**.
5. Copy the following bucket policy, and paste it in the **Bucket policy editor**, this will grant public read access to your website.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::some-website.com/*"
            ]
        }
    ]
}
```
6. To use this bucket policy with your own bucket, you must update `some-website.com` in **Resource** to match your bucket name.
7. Choose **Save changes**.

- After adding the policy check the bucket endpoint once to verify if everything is working as intended. Inside your bucket, head to **Properties -> Static website hosting -> Bucket website endpoint**
{: .fs-3.pl-6.info-bg}

### Generate SSL Certificates using AWS Certificate Manager

Before we move ahead with creation of CloudFront distribution you would be requiring a custom SSL certificate as you will be using your own domain name in the URLs for your object as an alternate name. The list of certificates can include any of the following:

- Certificates provided by AWS Certificate Manager.
- Certificates that you purchased from a third-party certificate authority and uploaded to ACM.
- Certificates that you purchased from a third-party certificate authority and uploaded to the IAM certificate store.

For the scope of this article we will be using the certificate provided by ACM. It is generate free of cost.

**To generate a certificate using AWS Certificate Manager**
1. Open the [AWS Certificate Manager console](https://console.aws.amazon.com/acm/home).
2. Choose **Request a certificate**.
3. On the **Request a certificate** page, choose **Request a public certificate** followed by **Request a certificate** CTA to continue.
4. On the **Add domain names** page, type your domain name that you want to provide certificate for. You can either use a subdomain name, such as `www.some-website.com`, or a root domain name such as `some-website.com`.
    - You can also use an asterisk (`*`) as a wild card to protect several site names in the same domain. For example, `*.some-website.com` protects `www.some-website.com`, and `login.some-website.com`.
    - When you request a wild card certificate, the asterisk (`*`) must be in the leftmost position of the domain name and can protect only one subdomain level. For example, `*.some-website.com` can protect `www.some-website.com`, and `login.some-website.com`, but it cannot protect `www.login.some-website.com`. Also note that `*.some-website.com` protects only the subdomains of `some-website.com`, it does not protect the root domain (`some-website.com`).
    {: .fs-3.pl-6.info-bg}
5. To add another name, choose **Add another name to this certificate** and type the name in the text box. This is useful for protecting both a root domain (such as `some-website.com`) and its subdomains such as `*.some-website.com`).
6. Choose **Next** when you finish adding the domain names.
7. On the **Select validation method** page, choose either **DNS validation** or **Email validation**. We recommend that you use DNS validation[^5] rather than email validation if you have permission to modify the DNS configuration for the domains in your certificate request. For the scope of this article, we will use the DNS validation.
8. On the **Add tags** page, you can optionally tag your certificate.
9. **Review** the certificate and click **Confirm and request**.
10. You must complete the validation before ACM can issue your certificate, expand the panel with the domain name and add the **CNAME** record to the DNS configuration for your domain.
11. If you are using **Route 53** as your DNS management tool then choose **Create record in Route 53** and ACM will update your DNS configuration for you.
12. Click **Create** to create a record in Route 53.

Once the record is created, wait for **Validation status** to show **Success**. This completes generation of the certificate using **AWS Certificate Manager (ACM)**.

### Create a CloudFront distribution

If everything works as expected for you then you can move ahead and create a CloudFront[^6] distribution with an Amazon S3 origin.

**To create a distribution with an Amazon S3 origin**
1. Open the [CloudFront console](https://console.aws.amazon.com/cloudfront)
2. Choose **Create Distribution**.
3. On the **Select a delivery method for your content** page, under **Web**, choose **Get Started**.
4. On the **Create Distribution** page, in the **Origin Settings** section, for **Origin Domain Name**, enter the Amazon S3 website endpoint for your bucket. CloudFront fills in the **Origin ID** for you.
5. For **Default Cache Behavior Settings**, keep the values set to the defaults.
6. For **Distribution Settings**, keep the default values except the following fields,
- Set **Alternate Domain Names (CNAMEs)** to the root domain and subdomain, for example, `some-website.com` and `www.some-website.com`.
- For **SSL Certificate**, choose **Custom SSL Certificate (example.com)**, and choose the custom certificate that covers the domain and subdomain names. You must have created that in the previous step.
7. Choose **Create Distribution**.

- Once your distribution is deployed, you can reference your content with the new CloudFront domain name. It can be checked in the **Status** column.
- Verify that your CloudFront distribution is working, enter the domain name of the distribution in a web browser.
{: .fs-3.pl-6.info-bg}

### Add alias record in Route 53

Before adding a record, make sure you have registered your domain with Route 53. We will create an alias record that will map the domain in the hosted zone. In our case `some-website.com`.

**To add an alias record**
1. Open the [Route 53 console](https://console.aws.amazon.com/route53).
2. Choose **Hosted zones** and select the name of hosted zone from the list that matches your root domain.
3. Choose **Create record**.
4. Choose **Simple routing**, and choose **Next**.
5. Choose **Define simple record**.
6. In **Record name**, leave it as it is for the root domain. For subdomain, like `blog.some-website.com` input `blog`.
7. In **Value/Route traffic to**, choose **Alias to S3 website endpoint**.
8. Choose the **Region**, for example, Asia Pasific (Mumbai) [ap-south-1].
9. Choose the **S3 bucket**, for example, s3-website.ap-south-1.amazonaws.com (some-website.com)
10. In **Record type**, choose **A ‐ Routes traffic to an IPv4 address and some AWS resources**.
11. Disable the **Evaluate target health**.
12. Choose **Define simple record**.

- Verify that the redirection is working properly. If it is not, try clearing the browser cache and compare the name server for the hosted zone and your domain is correct.
- For the S3 bucket to appear in the list, your S3 bucket name and Route 53 record name should be the same.
{: .fs-3.pl-6.info-bg}

---
**References:**

[^1]: [How DNS works](https://howdns.works)
[^2]: Learn more about [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
[^3]: [DNS hostnames](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html#vpc-dns-hostnames)
[^4]: [AWS Global Accelerator](https://aws.amazon.com/global-accelerator/)
[^5]: [Benifits over email validation](https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-validate-dns.html)
[^6]: [What is Amazon CloudFront?](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
