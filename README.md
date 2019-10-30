# Valtix Risk Report
Explain what this document does and how valtix can help provide a risk report for your cloud applications and infrastructure

# Requirements
For Valtix to generate a risk report, you need to do the following:
1. Enable VPC Flow Logs for all the VPCs running your applications and forward the logs to Valtix S3 Bucket.
1. Create an IAM Role that can be used by the Valtix Controller to get inventory details of your cloud account

# Enabling VPC Flow Logs
1. Create Flow Logs for your selected VPC with Filter **All** and Destination **Send to an S3 Bucket** with S3 Bucket ARN
as **aws:arn:s3:::valtix-vpc-flow-logs**
![VPC Selection]("screenshots/vpc-flow-logs-01.png" "Select VPC to Enable Flow Logs")
![VPC Selection]("screenshots/vpc-flow-logs-02.png" "Send Logs to Valtix S3 Bucket")
1. Once the traffic reaches your selected VPC, the logs are forwarded to the Valtix S3 Bucket that's provided.
1. Valtix needs a minimum of 7 days worth of logs to generate report

# Enabling Inventory support on the Valtix Controller
VPC Flow logs provide the IP Addresses and Port numbers of the traffic that reach the VPC. To match those details
with the applications (Load Balancers, EC2 Instances) Valtix needs to get an inventory of your cloud. Inventory
collection also helps to show the dynamism of your cloud infrastructure like how instances, load balancers and
other resourced are getting created/destroyed. To enable inventory support, Valtix controller needs a 
cross-account IAM role that it can assume and collect the data.
## Create Cross-Account IAM Role
1. Create a new IAM Role *Valtix-Controller-Role* with the following policy:
```
Show the policy here
```
1. Login to [Valtix Controller](https://www.google.com) using the username and password that was provided to you
by Valtix
1. Click on *Manage* > *Manage Accounts*
1. Add a new Account of type AWS and use the name of the role created above.
1. Once the account is added, click on the 3 bars on the account and *Enable Inventory*
