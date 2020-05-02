---
title: Cost Optimisation
nav_order: 10
parent: Efficient
grand_parent: Engineering

---

A cost optimisation exercise should be conducted on all projects that run on AWS every 3 months at least. If the project needs it to be done more frequently, it should be done!

---

##### Review - In brief

The general procedure that should be followed are

- Review the usage of all fixed resources in the project (EC2 instances, load balancers, EB environments)
- Review the family & type of instance based on it's usage
- Review the need of the fixed resource and if there are any alternatives
- Review the possibility for savings plan/reserved instances implementation

##### Review - In detail

1. Review usage of all fixed resource

     Login to the AWS console and go through each of the following resources and answer the following questions

   - EC2 Instances
   - Load Balancers
   - EB Environments

   

   1. Is the resource serving traffic and in active usage (it might have been accidentaly created, retried for usage of new resources etc)
   2. Is the resource being utilized effectivetly (atleast 50% of CPU & Memory usage on a weekly average unless it is the lowest size possible)
   3. Is there a chance of using a spot instance? (Is it critical and does it have a graceful failure - best candidates are load balances EB environments)

2. Review the family & type of instance based on it's usage

   - Is the instance of the optimal size based on it's usage?
   - Is the instance of latest family?

3. Review the need of the fixed resource and if there are any alternatives

   - Can we convert this fixed resouce to a serverless alternative or deploy to S3 + Cloudfront or Amplify? The answer is generally yes for UI servers.

4. Review the possibility for savings plan/reserved instances implementation

   1. Is the resource consistently used for 730 hours a month? 
   2. Do we forsee removing the resource in the next 1 year?


   If the answer to question 1 is yes and question 2 is no, then we move forward with reserving capacity in the form of reserved instances or a savings plan.

---

Plan of action

Construct a plan of action with the following details to submit to the project manager / client for each action item after the review

1. Title
2. Method of approach 
3. Expected cost savings in USD
4. Expected performance improvements (if applicable)
5. Downtime
6. Requirement from client (if applicable)