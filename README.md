# Valtix Cloud Assessment Risk Report
Explain what this document does and how valtix can help provide a risk report for your cloud applications and infrastructure

# Requirements
For Valtix to generate a risk report, you need to do the following:
1. Enable VPC Flow Logs for all the VPCs running your applications and forward the logs to Valtix S3 Bucket.
1. Create an IAM Role that can be used by the Valtix Controller to get inventory details of your cloud account. This is optional and can be done after you get the initial report. Inventory helps in identifying the traffic that's destined to the applications/load balancers/ec2-instances. 

# Enable VPC Flow Logs
1. Create Flow Logs for your selected VPC with Filter **All** and Destination **Send to an S3 Bucket** with S3 Bucket ARN
as **aws:arn:s3:::valtix-vpc-flow-logs**
![VPC Selection](screenshots/vpc-flow-logs-01.png "Select VPC to Enable Flow Logs")

![VPC Selection](screenshots/vpc-flow-logs-02.png "Send Logs to Valtix S3 Bucket")
1. Once the traffic reaches your selected VPC, the logs are forwarded to the Valtix S3 Bucket that's provided.
1. Valtix needs a minimum of 7 days worth of logs to generate report

# Enable Inventory support on the Valtix Controller
VPC Flow logs provide the IP Addresses and Port numbers of the traffic that reach the VPC. To match those details
with the applications (Load Balancers, EC2 Instances) Valtix needs to get an inventory of your cloud. Inventory
collection also helps to show the dynamism of your cloud infrastructure like how instances, load balancers and
other resourced are getting created/destroyed. To enable inventory support, Valtix controller needs a 
cross-account IAM role that it can assume and collect the data.
## Create Cross-Account IAM Role
1. Create a new IAM Role with trusted entity as **Another AWS Account**, Account ID as **425355469185** and add an external id as **ValtixController**. (External ID can be anything that you wish to use)
![Role1](screenshots/role00.png "Create new role with trusted party as another aws account")

1. Do not assign any permissions, a JSON policy is provided later in the document and the role will be edited to add this policy

1. Keep clicking next and provide a role name as **ValtixController** (or any other name) and finish creating the role.

1. Select the role created above and in the Permissions tab click **Add inline policy** and select **JSON** in the policy editor

![Role2](screenshots/role-01.png "Edit role to add inline policy")

![Role3](screenshots/role-02.png "Open JSON editor")

1. Copy the following policy in the editor

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "ec2:*",
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "elasticloadbalancing:*",
            "Resource": "*"
        }
    ]
}
```

1. On the Valtix Controller, login with the username and password assigned to you, add AWS account with the IAM Role created above.
1. Once the account is added, click the 3 bars next to the account name and then **Invetory**, select all the regions and save. This enables Valtix controller to start listening to the inventory changes in your account.
1. On your cloud account, enable Cloud Trail and log the events to any of your S3 buckets.
1. Go to **CloudWatch** events and add a rule to send events to an EventBus in another account
1. Select **Event Pattern** and choose **Build Custom Event Pattern**
1. Copy the following content
```
{
  "source": [
    "aws.ec2",
    "aws.elasticloadbalancing",
    "aws.apigateway"
  ],
  "detail-type": [
    "AWS API Call via CloudTrail"
  ]
}
```
1. **Add Target** and choose **Event bus in another AWS account** and add account id **425355469185**
1. You can either create a new role or use an existing role that has permissions to publish events to the account specified above.
1. In the next page, add a rule name and click **Create Rule**
1. This configuration of the CloudWatch Event Rules enables inventory changes to be read by Valtix Controller and build inventory details.
