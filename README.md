# AWS Multi-Account Management

This repository describes the high-level concepts that I use to manage my AWS accounts. The concepts are based on AWS guidelines and my own opinions about account separation and use.

While I use this repository mainly to organize my thoughts, it also contains links to other repositories that contain implementations I use to manage my accounts. Hopefully there's something here that's helpful to someone. Comments are always welcome.

There are a few design tenets I use to drive my AWS account management strategy.

* More accounts reduce blast radius, automation counters the account sprawl.
* Security matters, secure and monitor your resources
* Cost matters and can be managed if you put in some effort.
* Build small things and grow them, a few days or weeks of works can get you something of value now. You'll understand whats missing after using it for awhile.
* Pick a deployment method to the account infrastructure. I currently use CloudFormation although I've recently been looking at Terraform. Neither is a silver-bullet, both have their strengths and weaknesses. There are other good products as well.
* AWS is not the end-all-be-all. Use what is appropriate, I use GitHub for code management.
* AWS does keep getting better, expect your custom developed thing to turn into an AWS (or other provider) service. Move to the service when appropriate.

# Multi-Account Architecture
In the last few years a lot has been written and presented on the topic of AWS multi-account strategies. Based on that information I organize my AWS accounts as follows.

## Organizations Account
The account where I run AWS Organizations from. This account provides the following.

* Centralized Billing
* AWS Organizations API for account management
* AWS Organizations Service Control Policies (SCP)

While it's overkill for me, an organization may decide to have multiple billing accounts. This means the management of accounts is spread across multiple AWS Organizations hierarchies.

### Organizational Unit Structure
I use organizational units to make decisions about what account infrastructure is deployed to each account. The hierarchy is described below.

* root: AWS default root
  * mpb: I create my own root OU under the AWS root OU
    * prod: Accounts that are running production workloads
    * test: Accounts used to test workloads before going to production
    * dev: Accounts used for development

OU structure is highly subjective, I'm sure mine won't be right for you. I also know that my structure will evolve over time.

What is notably missing from my OU structure is sandbox accounts. These will likely be used by most organizations. I think it's a good idea to isolate (limit communication through peering, transit routing) these accounts from your other accounts so that a disaster does not impact your primary accounts (dev, test, prod). It may be possible to relax the IAM policies in sandbox accounts to allow more freedom for experimentation. Of course relaxed means different things to different organizations.

### Deployed Services
I try to keep the number of resources limited in this account. The only resource currently deployed in this account is an IAM role that my [Management Account](#management-account) can assume to use the AWS Organizations API. The implementation for that role can be found in the [AWS Roles]() repository.

## Logging Account
This account is used to collect CloudTrail and Config logs from all of the other accounts in the organization. I only perform central collection of the logs, using one bucket for CloudTrail and one bucket for Config. Currently, I don't have a log analysis system or perform any notifications based on log entries. I also don't centrally collect CloudWatch logs, but would do it here if I did.

An organization may want to perform both central collection and local log collection. This allows users in an account to see what is happening in their account without needing access to the central log  collection. The local log collection should be pruned more aggressively than the central collection. Personally, I would only collect centrally and provide a mechanism for review and analysis to users in other accounts.

If I added a log analysis service I would not field that solution in this account. I would deploy the analysis tool in another account and provide access to the logs via a bucket policy and an appropriate notification mechanism (SNS/SQS/Lambda). The notification mechanism will generally depend upon the tool.

### Deployed Services
Central CloudTrail and Config buckets are deployed in this account as well as central SNS topics for each service. The CloudTrail implementation can be found [here](), the Config implementation is [here]().

## Management Account
TBD

## Services Account
TBD

## Production Account
TBD

## Test Account
TBD

## Development Account
TBD

## Account Management Resources

I've done a lot of research into how to setup and manage multiple accounts. The following is a list of items that I've found helpful.

* [AWS Secure Initial Account Setup](https://aws.amazon.com/answers/security/aws-secure-account-setup/)
* [AWS Multiple Account Billing Strategy](https://aws.amazon.com/answers/account-management/aws-multi-account-billing-strategy/)
* [AWS Multiple Account Security Strategy](https://aws.amazon.com/answers/account-management/aws-multi-account-security-strategy/)
* [AWS Tagging Strategies](https://aws.amazon.com/answers/account-management/aws-tagging-strategies/)
* [How to Use AWS Organizations to Automate End-to-End Account Creation]( https://aws.amazon.com/blogs/security/how-to-use-aws-organizations-to-automate-end-to-end-account-creation)
* [Using AWS CloudFormation Custom Resources to Create Amazon Route 53 Resources in Another Account](https://aws.amazon.com/blogs/mt/multi-account-strategy-using-aws-cloudformation-custom-resources-to-create-amazon-route-53-resources-in-another-account)
* [CIS Benchmarks on AWS](https://aws.amazon.com/quickstart/architecture/compliance-cis-benchmark/)
* [Continuous Delivery of Nested AWS CloudFormation Stacks Using AWS CodePipeline](https://aws.amazon.com/blogs/devops/continuous-delivery-of-nested-aws-cloudformation-stacks-using-aws-codepipeline/)
* [Centralized Logging](https://aws.amazon.com/answers/logging/centralized-logging/)
* [AWS CloudFormation Validation Pipeline](https://aws.amazon.com/answers/devops/aws-cloudformation-validation-pipeline/)
* [Build Continuous Delivery Workflows for CloudFormation Stacks](https://aws.amazon.com/blogs/aws/codepipeline-update-build-continuous-delivery-workflows-for-cloudformation-stacks/)
* [AWS Instance Scheduler](https://aws.amazon.com/answers/infrastructure-management/instance-scheduler/)
* [AWS Ops Automator](https://aws.amazon.com/answers/infrastructure-management/ops-automator/)
* [AWS Landing Zone](https://aws.amazon.com/answers/aws-landing-zone/)
* [Real-Time Insights on AWS Account Activity](https://aws.amazon.com/answers/account-management/real-time-insights-account-activity/)
* [AWS Config Rules Update: Aggregate Compliance Data Across Accounts and Regions](https://aws.amazon.com/blogs/aws/aws-config-update-aggregate-compliance-data-across-accounts-regions/?nc1=b_rp)
* [How to develop custom AWS Config rules using the Rule Development Kit](https://aws.amazon.com/blogs/mt/how-to-develop-custom-aws-config-rules-using-the-rule-development-kit/?nc1=b_rp)
* [Announcing the Golden AMI Pipeline](https://aws.amazon.com/blogs/awsmarketplace/announcing-the-golden-ami-pipeline/?nc1=b_rp)
* [How to create custom AWS Config rules with AWS CodeStar](https://aws.amazon.com/blogs/mt/how-to-create-custom-aws-config-rules-with-aws-codestar/?nc1=b_rp)
* [How to Use AWS Config to Monitor for and Respond to Amazon S3 Buckets Allowing Public Access](https://aws.amazon.com/blogs/security/how-to-use-aws-config-to-monitor-for-and-respond-to-amazon-s3-buckets-allowing-public-access/?nc1=b_rp)
* [Integrating AWS CloudFormation with AWS Systems Manager Parameter Store](https://aws.amazon.com/blogs/mt/integrating-aws-cloudformation-with-aws-systems-manager-parameter-store/)
* [Query for the latest Amazon Linux AMI IDs using AWS Systems Manager Parameter Store](https://aws.amazon.com/blogs/compute/query-for-the-latest-amazon-linux-ami-ids-using-aws-systems-manager-parameter-store/?nc1=b_rp)
* [SAML Identity Federation: Follow-Up Questions, Materials, Guides, and Templates from an AWS re:Invent 2016 Workshop (SEC306)](https://aws.amazon.com/blogs/security/saml-identity-federation-follow-up-questions-materials-guides-and-templates-from-an-aws-reinvent-2016-workshop-sec306/)
* [SAML Workshop](http://federationworkshopreinvent2016.s3-website-us-east-1.amazonaws.com/)
* [How to rotate your Twitter API key and bearer token automatically with AWS Secrets Manager](https://aws.amazon.com/blogs/security/how-to-rotate-your-twitter-api-key-and-bearer-token-automatically-with-aws-secrets-manager/?nc1=b_rp)
* [How to Audit Cross-Account Roles Using AWS CloudTrail and Amazon CloudWatch Events](https://aws.amazon.com/blogs/security/how-to-audit-cross-account-roles-using-aws-cloudtrail-and-amazon-cloudwatch-events/)
* [Serverless Security Automation](https://www.youtube.com/watch?v=IxHtZNcpLZw)
* [AWS re:Invent 2016: How to Automate Policy Validation (SEC311)](https://www.youtube.com/watch?v=pm-4vSMSDyc&t=600s)
* [AWS Lambda-backed Custom Resources for Stack Outputs, Resources, and Parameters](https://stelligent.com/2016/02/16/aws-lambda-backed-custom-resources-for-stack-outputs-resources-and-parameters/)
* [Unit and Integration Testing for AWS Lambda](https://serverless.zone/unit-and-integration-testing-for-lambda-fc9510963003)
* [AWS IAM Generator
](https://github.com/awslabs/aws-iam-generator)
* [Monitor AWS Trusted Advisor Checks](https://pprakash.me/tech/2017/03/14/monitor-aws-trusted-advisor-checks/)
* [AWS Global Transit Network](https://aws.amazon.com/answers/networking/aws-global-transit-network/)
* [Splunk Enterprise on AWS](https://aws.amazon.com/quickstart/architecture/splunk-enterprise/)
