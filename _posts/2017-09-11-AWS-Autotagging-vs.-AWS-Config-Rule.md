
# AWS Autotagging vs. AWS Config Rule: Required tags

NOTE: If you choose to use Autotagging, the autotagged resources are EC2 Instances, volumes, snapshots, and Amazon Machine Images (AMIs), as well as RDS instances and S3 buckets>

|Pros                                                  |Cons      |
|--:                                                   |--:                                                                  |
|You can autotag resources.                            |You must have a higher level of understanding of how cloudformation,      cloudwatch, cloudtrail, AMI roles, AMI groups, and lambda code work together                                                 |
|You can send automatic email alerts.                  |                                                                     |
|The lambda function is modifiable to fit your needs.  |                                                                     |


|Pros                                                            |Cons                                 |
|--:                                                             |--:                                  |
|You can see a list of untagged resourcesin the Config dashboard.|You cannot autotag resources.        |
|Setting up the required tags config rule is very easy.          |                                     |
|You can send SNS email alerts.                                  |                                     |

The biggest difference between AWS Config rule Required Tags and Autotagging that we most care about is the fact that you cannot autotag with Required Tags. Autotagging is only possible when you write lambda code.

Although functionality is an important factor of consideration, we must not forget another important factor, cost.

Let's compare the cost of using Required Tags and the Autotagging Lambda function.

## Autotagging Lambda function

Pricing information for CloudFormation, CloudTrail, CloudWatch, SNS and Lambda
(All of this information is obtained from AWS)

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

### SNS
|Endpoint Type     |Free Tier     |Price             |
|--:               |--:           |--:               |
|email/email-JSON  |1,000	        |$2.00 per 100,000 |
- https://aws.amazon.com/sns/pricing/

### Lambda Pricing Details
The Lambda free tier includes 1M free requests per month and 400,000 GB-seconds of compute time per month.
##### Requests
You are charged for the total number of requests across all your functions. Lambda counts a request each time it starts executing in response to an event notification or invoke call, including test invokes from the console.

First 1 million requests per month are free
$0.20 per 1 million requests thereafter ($0.0000002 per request)

##### Duration
Duration is calculated from the time your code begins executing until it returns or otherwise terminates, rounded up to the nearest 100ms. The price depends on the amount of memory you allocate to your function. You are charged $0.00001667 for every GB-second used.

You can find Duration and Memory Size at the bottom your Autotag CloudWatch log. Below is what it looks like.
***
REPORT RequestId: 0a28cd44-9475-11e7-a8f8-11f3df1b4e23	Duration: 871.20 ms	Billed Duration: 900 ms Memory Size: 128 MB	Max Memory Used: 54 MB	
***

#### How to calculate Lambda costs

Example:

Your lambda function has 128 MB of allocated memory. It is executed 20 million times per month and runs for 800 ms each time it is executed. What would you be charged?

The following information is obtained from AWS Lambda Pricing - https://aws.amazon.com/lambda/pricing/

### Monthly compute charges

The monthly compute price is $0.00001667 per GB-s and the free tier provides 400,000 GB-s.

Total compute (seconds) = 20M * (0.8sec) = 16,000,000 seconds

Total compute (GB-s) = 16,000,000 * 128MB/1024 = 2,000,000 GB-s

Total Compute – Free tier compute = Monthly billable compute seconds

2,000,000 GB-s – 400,000 free tier GB-s = 1,600,000 GB-s

Monthly compute charges = 1,600,000 * $0.00001667 = $26.672 

### Monthly request charges

The monthly request price is $0.20 per 1 million requests and the free tier provides 1M requests per month.

Total requests – Free tier request = Monthly billable requests

20M requests – 1M free tier requests = 19M Monthly billable requests

Monthly request charges = 19M * $0.2/M = $3.80 

### Total compute charges

Total charges = Compute charges + Request charges = $26.672 + $3.80 = $30.472 per month

$30.472 dollars each month might seem a lot, but this is because we are hypothetically setting execution rate per month at 20 million, which does not happen unless you have many many IAM users launching instances, volumes, etc. everyday. The actual cost is actually much lower when both the compute time per month and the number of executions in each month are within the free tier range. Furthermore, the cost estimation for the other AWS services are overestimated as well to show you how costly it can be when the services are working in concert.

### Example Cost breakdown:

Charges for CloudFormation = $0 

Charges for 2 GB of CloudWatch Logs  = $0.50 per GB * 2 + $0.03 per GB * 2 = $1.06

Charges for 1 million generated custom CloudWatch Events = $1.00

Charges for first copy of 150,000 Management events from CloudTrail = $0 

Charges for second copy of 150,000 Management events from CloudTrail = $2 * 150,000/100,000 = $3

Charges for 1M data events from CloudTrail = $0.10 * 1M/100,000 = $1

Charges for 10,000 SNS tag alert emails = $2 * 10,000/100,000 = $0.2

Charges for lambda function with 128 MB allocated memory, 20 million executions per month, and 800 ms duration = $30.472/month

Total = $6.26 + $30.472/month 

## AWS Config Rule:

The following information is obtained from AWS. - https://aws.amazon.com/config/pricing/

With AWS Config, you are charged a one-time fee based on the number of Configuration Items recorded. 
$0.003 per Configuration Item recorded

A rule is active if it has one or more evaluations in a month.
Config Rules costs: $2 per active rule per month

For every active rule, your account receives at no extra charge for that month: 20,000 evaluations.
Unused evaluations do not accumulate.
If you need more evaluations for your rules, additional evaluations will be charged at: $0.10 per thousand evaluations

Example:

10 Configuration items ONE-TIME FEE = $0.003 * 10 = $0.03
3 Config Rules = $2 * 3 = $6.00

Total = $0.03 + $6.00/per month

## Conclusion
These cost estimations are merely for showing you how to calculate the costs for each option, not for convincing you to choose one over the other. Just as a reminder, AWS Config cannot automatically tag untagged resources! I hope you enjoyed reading this article on the pros and cons as well as how to calculate the costs of using AWS Autotagging and AWS Config.















