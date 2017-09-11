
The Pros and Cons of AWS Autotagging and Config Rule: Required tags

|Pros |Cons  |
|--:  |--:   |
|     |      |

You can autotag EC2 Instances, volumes, snapshots, and Amazon Machine Images (AMIs), as well as RDS instances and S3 buckets

Autotagged resources: EC2 Instances, volumes, snapshots, and Amazon Machine Images (AMIs), as well as RDS instances and S3 buckets

You can send automatic email alerts to all users when an improperly tagged resource (EC2 instance, volume, snapshot, AMI, etc. shown above) is detected.

You can modify the lambda code whenever you want to fit your needs.

"There is no additional charge for AWS CloudFormation." - https://aws.amazon.com/cloudformation/pricing/

Amazon CloudWatch Logs*
$0.50 per GB ingested**
$0.03 per GB archived per month***
Data Transfer OUT from CloudWatch Logs is priced equivalent to the “Data Transfer OUT from Amazon EC2 To” and “Data Transfer OUT from Amazon EC2 to Internet” tables on the EC2 Pricing Page.
Amazon CloudWatch Events - Custom Events****
$1.00 per million custom events generated*****

https://aws.amazon.com/cloudwatch/pricing/

There is no charge from AWS CloudTrail for creating a trail. By creating a CloudTrail trail, you can deliver two types of events to your Amazon S3 bucket.

Management Events: Represent administrative type of account activity for AWS services. For example, CloudTrail delivers Management Events for API calls such as launching EC2 instances or creating S3 buckets. The first copy of Management Events within each region is delivered free of charge. Additional copies of Management Events are charged at $2.00 per 100,000 events.
Data Events: Represent data or object level account activity for AWS resources. For example, CloudTrail delivers Data Events for S3 object level API such as Get, Put, Delete and List actions. Data Events are recorded only for the buckets you specify and are charged at $0.10 per 100,000 events.

https://aws.amazon.com/cloudtrail/pricing/



Cons:

A higher level of understanding of how cloudformation, cloudwatch, cloudtrail, AMI roles, AMI groups, and lambda code is required.



You can see a list of untagged resources in the AWS Config dashboard in one place.

You can set up the required tags config rule very easily with the provided template.

You can set up SNS email alerts as well (but the format of the message is not modifiable).




