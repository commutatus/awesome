---
title: Elastic IP address
nav_order: 16
parent: Efficient
grand_parent: Engineering
---


# Elastic IP address

When working with a cloud environment like AWS, IP address assigned to the instances are dynamic in nature. If an instance goes down or restarted in AWS it is likely that the IP address will be changed. To maintain the communication of your website with the AWS account you would require to change the IP address in all the places it has been used, which is too much work.

To counter the issue it is encouraged to switch to Elastic IP address. According to AWS, an Elastic IP address [^1] is a static and public IPv4 address designed for dynamic cloud computing which is reachable from the internet.

By using an Elastic IP address,
- You can mask the failure of an instance or software by rapidly remapping the address to another instance in your account.
- It is allocated to your AWS account and is yours until you release it.
- It does not change over time and can be maintained across.

It is strongly encouraged to use an Elastic IP address primarily for the ability to remap the address to another instance in the case of instance failure and to use **DNS hostnames** [^2] for all other inter-node communication.

### Allocating an Elastic IP address

Public IPv4 addresses are allocated from either of the following pool of address,
- Amazon's pool of IPv4 addresses
- A pool that you own and bring to your account
- A pool that you own and continue to advertise

You can also use global IP addresses from AWS Global Accelerator [^3].

**To allocate an Elastic IP address**
1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/).
2. In the navigation pane, choose **Elastic IPs**.
3. Choose **Allocate new address**.
4. Choose an option from the required public IPv4 pool or use global IP addresses from AWS Global Accelerator. For **IPv4 address pool**, choose **Amazon pool**.
5. Choose **Allocate**, and close the confirmation screen.

To quickly find a specific Elastic IP address you can assign them custom tags, for example, **Name**, which will be a key-value pair (adding value to it is optional).

**Note**
- Currently, Elastic IP addresses are not supported for IPv6.
- By default, all AWS accounts are limited to five (5) Elastic IP addresses per region. The limit is not applicable when you allocate it from a pool of IP addresses that you have brought to your AWS account.

### Associating an Elastic IP address with a running instance

Now that you have allocated an Elastic IP address, you can associate it with a running instance to enable communication with the internet, you must also ensure that your instance is in a public subnet.

**To associate an Elastic IP address with an instance**
1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/).
2. In the navigation pane, choose **Elastic IPs**.
3. Select an Elastic IP address and choose **Actions -> Associate Elastic IP address**.
4. Select the instance from **Instance** and then choose **Associate**.

---
**References:**

[^1]: Learn more about [Elastic IP address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
[^2]: [DNS hostnames](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-dns.html#vpc-dns-hostnames)
[^3]: [AWS Global Accelerator](https://aws.amazon.com/global-accelerator/)
