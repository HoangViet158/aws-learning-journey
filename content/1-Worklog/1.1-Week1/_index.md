---
title: "Week 1: AWS Foundations and FoodieRecipe Design"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## 1. Objectives

- Understand cloud computing and core AWS concepts.
- Secure the AWS account with IAM and MFA.
- Configure an AWS Budget Alert for cost control.
- Analyze requirements and design the complete FoodieRecipe architecture.

## 2. Work plan

> **Week 1 schedule:** Monday, 22/06/2026 – Friday, 26/06/2026.

| Day | Task | Expected result |
| :--: | ---- | --------------- |
| Monday | Study Cloud Computing, Regions, Availability Zones, and core AWS services | Understand the AWS foundation |
| Tuesday | Create the account, enable MFA, and study the Shared Responsibility Model | Establish basic account protection |
| Wednesday | Create IAM users/groups, apply least privilege, and configure the AWS CLI | Access AWS without using the root user |
| Thursday | Create a Budget Alert and enable cost notifications | Establish budget monitoring |
| Friday | Analyze FoodieRecipe features, data, and deployment architecture | Define the initial scope and architecture diagram |

## 3. Work completed

### 3.1. Account setup and IAM security

- Enabled MFA for the root user and restricted root account usage.
- Created a development IAM group and attached policies following **least privilege**.
- Created an IAM user for Console and AWS CLI operations.
- Verified the active identity with `aws sts get-caller-identity`.

**AWS account created successfully:**

![AWS account created successfully](/images/1-Worklog/1.1-Week1/aws-account-created.png)

**Verify the account and security configuration on the IAM Dashboard:**

![IAM Dashboard and account security status](/images/1-Worklog/1.1-Week1/iam-dashboard.png)

### 3.2. Budget management

- Created a monthly AWS cost budget.
- Configured thresholds for actual and forecasted spending.
- Registered an email recipient and verified the Budget status.

**Monitor costs and budget status in Billing and Cost Management:**

![AWS cost and Budget status monitoring](/images/1-Worklog/1.1-Week1/budget-dashboard.png)

### 3.3. FoodieRecipe analysis

Defined account, recipe, ingredient, category, search, like, comment, and AI image-workflow requirements. Designed an architecture using Next.js, NestJS, EC2, Docker, Nginx, RDS, S3, Rekognition, Bedrock, CloudFront, IAM, Secrets Manager, and CloudWatch, including deployment, observability, and cost-control flows.

## 4. Knowledge and skills gained

- Distinguished IaaS, PaaS, SaaS, and the roles of AWS services in the project.
- Understood IAM users, groups, roles, and policies.
- Applied MFA and least-privilege access.
- Learned budget alerting, cost awareness, and project scoping.

## 5. Challenges and solutions

| Challenge | Solution |
| --------- | -------- |
| Confusing users, roles, and policies | Drew their relationships and tested read-only permissions first |
| Uncertain cost estimates | Used AWS Pricing Calculator and set a low initial budget |
| An overly broad initial scope | Prioritized core features as an MVP |

## 6. Deliverables

- An AWS account protected by MFA with a development IAM user.
- A configured AWS Budget Alert.
- FoodieRecipe requirements, data model, application/AWS architecture, and deployment plan.

## 7. Next-week plan

Design Amazon S3, Amazon RDS, the data model, IAM, and Secrets Manager for FoodieRecipe.

## 8. References

- [Security best practices in AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Multi-factor authentication in AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)
- [Managing costs with AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [Best practices for AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-best-practices.html)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
