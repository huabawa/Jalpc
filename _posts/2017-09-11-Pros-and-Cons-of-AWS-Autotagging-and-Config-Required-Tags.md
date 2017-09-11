
The Pros and Cons of AWS Autotagging and Config Rule: Required tags

|Pros |Cons  |
|--:  |--:   |
|     |      |

You can autotag EC2 Instances, volumes, snapshots, and Amazon Machine Images (AMIs), as well as RDS instances and S3 buckets

Autotagged resources: EC2 Instances, volumes, snapshots, and Amazon Machine Images (AMIs), as well as RDS instances and S3 buckets

You can send automatic email alerts to all users when an improperly tagged resource (EC2 instance, volume, snapshot, AMI, etc. shown above) is detected.

You can modify the lambda code whenever you want to fit your needs.

You can autotag and send SNS alerts at a very low cost.

Example:

## Cost breakdown

### CloudFormation
"There is no additional charge for AWS CloudFormation." - https://aws.amazon.com/cloudformation/pricing/

### CloudWatch
##### Amazon CloudWatch Logs*
$0.50 per GB ingested**
$0.03 per GB archived per month***
Data Transfer OUT from CloudWatch Logs is priced equivalent to the “Data Transfer OUT from Amazon EC2 To” and “Data Transfer OUT from Amazon EC2 to Internet” tables on the EC2 Pricing Page.
##### Amazon CloudWatch Events - Custom Events****
$1.00 per million custom events generated*****
- https://aws.amazon.com/cloudwatch/pricing/

### CloudTrail
There is no charge from AWS CloudTrail for creating a trail. By creating a CloudTrail trail, you can deliver two types of events to your Amazon S3 bucket.

Management Events: Represent administrative type of account activity for AWS services. For example, CloudTrail delivers Management Events for API calls such as launching EC2 instances or creating S3 buckets. The first copy of Management Events within each region is delivered free of charge. Additional copies of Management Events are charged at $2.00 per 100,000 events.
Data Events: Represent data or object level account activity for AWS resources. For example, CloudTrail delivers Data Events for S3 object level API such as Get, Put, Delete and List actions. Data Events are recorded only for the buckets you specify and are charged at $0.10 per 100,000 events.
- https://aws.amazon.com/cloudtrail/pricing/

Example:

Charges for CloudFormation = $0 
Charges for 2 GB of CloudWatch Logs  = $0.50 per GB * 2 + $0.03 per GB * 2 = $1.06
Charges for 1 million generated custom CloudWatch Events = $1.00
Charges for first copy of 150,000 Management events from CloudTrail = 
Charges for second copy of 150,000 Management events from CloudTrail = 
Charges for 1M data events from CloudTrail = 

Notification Deliveries
Endpoint Type	Free Tier	Price
Mobile Push Notifications	1 million	$0.50 per million
Worldwide SMS	100	Learn more
email/email-JSON	1,000	$2.00 per 100,000
HTTP/s	100,000	$0.60 per million
Simple Queue Service (SQS)	No charge for deliveries to SQS Queues
Lambda functions	No charge for deliveries to Lambda
https://aws.amazon.com/sns/pricing/

Lambda Pricing Details
The Lambda free tier includes 1M free requests per month and 400,000 GB-seconds of compute time per month.
Requests
You are charged for the total number of requests across all your functions. Lambda counts a request each time it starts executing in response to an event notification or invoke call, including test invokes from the console.

First 1 million requests per month are free
$0.20 per 1 million requests thereafter ($0.0000002 per request)
REPORT RequestId: 0a28cd44-9475-11e7-a8f8-11f3df1b4e23	Duration: 871.20 ms	Billed Duration: 900 ms Memory Size: 128 MB	Max Memory Used: 54 MB	

How to calculate Lambda costs

Example:
Your lambda function has 128 MB of allocated memory. It is executed 20 million times per month and runs for 800 ms each time it executed. What would you be charged?

Cost breakdown

Monthly compute charges
The monthly compute price is $0.00001667 per GB-s and the free tier provides 400,000 GB-s.

Total compute (seconds) = 30M * (0.2sec) = 6,000,000 seconds

Total compute (GB-s) = 6,000,000 * 128MB/1024 = 750,000 GB-s

Total Compute – Free tier compute = Monthly billable compute seconds

750,000 GB-s – 400,000 free tier GB-s = 350,000 GB-s

Monthly compute charges = 350,000 * $0.00001667 = $5.83

 

Monthly request charges

The monthly request price is $0.20 per 1 million requests and the free tier provides 1M requests per month.

Total requests – Free tier request = Monthly billable requests

30M requests – 1M free tier requests = 29M Monthly billable requests

Monthly request charges = 29M * $0.2/M = $5.80

 

Total compute charges

Total charges = Compute charges + Request charges = $5.83 + $5.80 = $11.63 per month




Cons:

A higher level of understanding of how cloudformation, cloudwatch, cloudtrail, AMI roles, AMI groups, and lambda code is required.



You can see a list of untagged resources in the AWS Config dashboard in one place.

You can set up the required tags config rule very easily with the provided template.

You can set up SNS email alerts as well (but the format of the message is not modifiable).




