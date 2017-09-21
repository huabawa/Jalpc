# AWS Autottagging and AWS Config Rule: Required tags Tutorial

In this tutorial, I will show you how to set up AWS Autotagging for EC2, RDS, and S3. I will also show you how to set up Config Rule: Required tags, which does not autotag resources, but does detect improperly tagged resources.

In order to set up Autotagging in AWS, you need to create a lambda function. Before you jump into Lambda to write your Autotagging lambda function, you should learn Boto3 and SNS because it will make things much easier. You should also learn why Auotagging is important to understand the whole purpose of setting up Autotagging.

Since it's easier to learn things by doing, let's assume that you have a detailed billing report in an S3 bucket that you need to parse with Boto3.

## How to parse the AWS detailed billing report located in S3 with Boto3

If you have an AWS linked account and you get errors, like Access Denied, when you want to get access to cost and usage reports, what should you do?

Luckily, there's a solution.

AWS stores and updates detailed billing reports in your S3 bucket, which you (the user) can access, so you might be wondering, how can I access it using command prompt or terminal?

### Accessing S3 bucket with Boto3

Here is a link to the latest boto3 documentation. https://boto3.readthedocs.io/en/latest/.
I recommend looking at the S3 bucket examples. https://boto3.readthedocs.io/en/latest/guide/s3-examples.html
But before that, let's first understand why we are using boto3.

### What is Boto3?

Like the boto3 documentation says, Boto3 is the Amazon Web Services (AWS) SDK for Python. This means you can write your own python software for all kinds of AWS resources (eg. EC2 and S3). Isn't that amazing? The user doesn't have to open the AWS console at all to open the s3 bucket!

### How to Install Boto3.

Trust me it's simple. Just follow the steps below to set up Boto3 on your computer.

1. Install pip if you don't have pip installed already. Update pip to make sure it is the latest version.
Link to pip installation documentation: https://pip.pypa.io/en/stable/installing/
2. Install AWS CLI. Link to AWS CLI installation guide: http://docs.aws.amazon.com/cli/latest/userguide/installing.html
3. Install Boto3. Link to Boto3 Installation: http://boto3.readthedocs.io/en/latest/guide/quickstart.html

Congrats! You have successfully installed boto3. The next step will be to write some python code to print out the S3 bucket list, list the contents of an S3 bucket, and more.

### Setting up AWS CLI
**You need to set up AWS CLI in order to access your AWS resources**
When you have installed AWS CLI, check that you have successfully installed it by typing, aws --version, in your terminal.
Something like this, aws-cli/1.11.136 Python/2.7.13 Darwin/15.6.0 botocore/1.6.3, should appear.
Then, type into the terminal, aws configure, to configure AWS CLI. Below is what should show on your terminal screen.
**Configure your AWS CLI to enter your following information.
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: 
Default output format [None]:  
**

For more information, go to http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html

### How to print out the S3 bucket list

Type into terminal, python.
Then, type the following.

```python
import boto3
s3 = boto3.client('s3')
response = s3.list_buckets()
buckets = [bucket['Name'] for bucket in response['Buckets']]
print("Bucket List: %s" % buckets)
```

### Listing Contents of a S3 bucket

Type into terminal, python.
Then, type the following.

```python
import boto3
s3 = boto3.resource('s3')
my_bucket = s3.Bucket('bucket_name')
for object in my_bucket.objects.all():
  print(object)
```
  * Click Return twice.

### Downloading a file in S3 bucket

Type into terminal, python.
Then, type one of the following.

Option 1:
```python
import boto3
s3 = boto3.resource('s3')
bucket = s3.Bucket('bucket_name')
s3.meta.client.download_file('bucket_name', 'file.you.want.to.download', 'directory/name-of-output-file')
```

Option 2:
```python
import boto3
s3 = boto3.resource('s3')
bucket = s3.Bucket('bucket_name')
bucket.download_file(key, 'file-name')
```

### Uploading a file to S3 bucket

Type into terminal, python.
Then, type the following.

```python
import boto3
s3 = boto3.resource('s3')
bucket = s3.Bucket('509248752274-dlt-utilization')
s3.meta.client.upload_file('directory/file-name', 'bucket-name', 'file-name')
```

### Print and Save Parsed detailed billing report

Type into terminal, python.
Then, type the following.

```python
import pandas
raw = pandas.read_csv('Directory of saved csv')
raw.loc[:,'LinkedAccountId':'SubscriptionId'] # display only columns 'LinkedAccountId' to 'SubscriptionId'
filtered = raw.loc[:,'LinkedAccountId':'SubscriptionId'] # save parsed detailed billing report
filtered.to_csv('OUT_FILE.csv') # saves in the same directory
```
More suggestions on how to parse the detailed billing report: http://blog.backslasher.net/aws-billing.html

For more detailed instructions on selecting data, click this link. https://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-label

### Copying an object from one s3 bucket to another s3 bucket

Type into terminal, python.
Then, type the following.

```python
import boto3
# Get the service client
s3 = boto3.client('s3')

# Copies object located in mybucket at mykey
# to the location otherbucket at otherkey
copy_source = {
    'Bucket': 'mybucket',
    'Key': 'mykey'
}
s3.copy(copy_source, 'otherbucket', 'otherkey')
```
**Next Step: Learn how SNS works** 
## How to set up EC2 SNS notifications

Let's assume that you want SNS email notifications whenever an EC2 is created, being shut down, and terminated.

Follow the steps below to set up EC2 start,stop,terminate SNS notifications.

1. Go to SNS in AWS Services.
2. Go to Topics. Create new topic.
3. Choose a topic name, like EC2-Start-Stop
4. Select (tick) the topic. Under Actions, select Subscribe to topic.
5. Select a protocol. Email is preferred. For endpoint, type in your email.
You will receive an email asking you to confirm the subscription.
6. Go to CloudWatch in AWS Services.
7. Click Events and then click Create rule.
8. Under Service Name, choose EC2.
9. Under Event Type, choose EC2 Instance State-change Notification
10. Choose any state or specific states depending on what you want.
11. Click Add Target. Scroll down and select SNS topic.
12. Select the topic that you had just created.
13. Click Configure Details. Write a name and description. Then, click Create rule.

**Next Step: Learn Why Autotagging is Important**
Sometimes it can be really hard to track down who is using the correct tags on AWS, especially when you have many people launching instances in different regions across the United States.

So, what strategy could you use to solve a problem of this kind?

Before we delve into this problem, let's understand why tagging is so important.

Here is a list of reasons. Please don't skip this part. You are very close to becoming a cloud computing guru!

1. Tags can help you track cost
2. Tags can help you track AWS resource usage
3. Tags can help you find your own AWS resource easily
4. Tags can help you find whoever isn't tagging their resources

## How to track cost by tags.

There are two ways to track cost.

#### Cost Explorer

In AWS Services Billing, you can find Cost Explorer, which shows you month-date spendings and daily spendings. On the left side of Cost Explorer, you have the option of selecting which tags to filter, for example, Name: Sally. On top of the graph, you can even group the costs and usage by tag key.
This is very convenient because you can see how much is spent by each user.

Since each user could be running more than one instance at a time, we want the users to tag their resources with these tag keys.

* Name
* Project
* End_date

**The End_date is the end date of the project.

Another way to track cost by tags is to read the csv file that is saved twice a day to the S3 bucket, named AccountNumber-dlt-utilization. 
These files are zip files, so you will need to unzip them first. If you want to receive daily or weekly summaries of the costs, you can write some lambda code to parse the csv, add up the costs, and create an SNS mailing list to email all of the account users what the weekly spendings are. 
To do this, follow this link here. 

Before we get into some lambda code, let's first understand something even more important.

After reading about cost tracking, you might ask what if the users forget to tag their resources?

## Next Step: Set up Autagging 

A way to solve this problem is Autotagging.

**How the Autotagging solution we'll be using works is summarized below.**

When a resource is created, an API call is made and recorded by CloudWatch. Then, a CloudWatch event rule triggers a lambda function providing it with event details.
The lambda function extracts every resource ID and the user's identity and applies two tags, Owner and PrincipalId (current user’s aws:userid value), to the created resource.

A simpler one sentence summary:

When EC2 Instances, Amazon Elastic Block Store (EBS) volumes, EBS snapshots or Amazon Machine Images (AMIs) are created, cloudwatch invokes the autotag lambda function.

Follow the steps below to autotag your EC2 Instance, Amazon Elastic Block Store (EBS) volumes, EBS snapshots and Amazon Machine Images (AMIs).

1. Go to CloudFormation. You can either click [here](https://s3.amazonaws.com/awsiammedia/public/sample/autotagec2resources/AutoTag.template) to download the template and upload it to S3 OR specify this Amazon S3 template URL https://s3.amazonaws.com/awsiammedia/public/sample/autotagec2resources/AutoTag.template . You can click through with no effort, for example, you do not need to tag your stack. Deploy the cloudformation template in the region of your choosing to create an Autotag stack. 

    *Note: Make sure CloudTrail is enabled in this region because cloudwatch events will not work if it is not turned on.*

2. Once your CloudFormation stack has status CREATE_COMPLETE: Click on it's resources tab. There will be a Security Group with Logical ID ManageEC2InstancesGroup. The associated Physical ID is a link to this Security Group. Click on this link to manually add IAM Users that you want this stack applied to.

   *Note: You must add IAM users to the group manually. Also, if the added IAM user tries to stop an instance that someone else created, he or she will get an error message.*

Here is an article about autotagging written by an AWS blogger. https://aws.amazon.com/blogs/security/how-to-automatically-tag-amazon-ec2-resources-in-response-to-api-events/

### Next step: Auototag RDS instances and S3 buckets.

#### Create two CloudWatch Rules. One for S3 bucket and one for RDS.

When the autotagging template is deployed in cloudformation, a cloudwatch events rule is created. If you now go to cloudwatch and click on rules under events, you will see New-EC2Resource-Event. Open it and you will see the event pattern, which has four event names:
"CreateVolume", "RunInstances", "CreateImage", and "CreateSnapshot".

If we want to autotag rds instances, we would need to... Yes, you guessed it. We would need to create an event rule for rds.

Here are the steps:

1. Go to Rules Under Events in CloudWatch. Click on 'Create Rule'.
2. Choose RDS for service name and AWS API Call via CloudTrail for event type.
3. Select Specific operation(s) and then click on the plus button.
4. Add CreateDBCluster and CreateDBInstance.
5. On the right-hand side, click on add target.
6. Select Lambda Function and choose AutoTag-CFAutoTag-XXXXXXX (the name of the autotag lambda function created by the template).
7. Select Default under Configure version/alias.
8. Click on Configure details on the bottom right corner to create your rule. 

Now, create another cloudwatch rule for s3 bucket. Follow the steps below (some steps are the same as the steps for rds).

1. Same as above.
2. Choose s3 for service name and AWS API Call via CloudTrail for event type.
3. Same as above.
4. Add CreateBucket.

5., 6., 7., 8. Same as above.

Now that we have created the CloudWatch Rules, what do we do next?

1. Edit the Lambda code
2. Create a new IAM Role

Let's start with the IAM role. Here's what you have to do.

Create a new IAM Role with "rds:AddTagsToResource", "rds:Describe*" in Action. 

To make your work easier, delete what was in the json previously and just copy and paste the following json. 
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "cloudtrail:LookupEvents"
            ],
            "Resource": [
                "*"
            ],
            "Effect": "Allow",
            "Sid": "Stmt1458923097000"
        },
        {
            "Action": [
                "ec2:CreateTags",
                "ec2:Describe*",
                "rds:AddTagsToResource",
                "rds:Describe*",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "*"
            ],
            "Effect": "Allow",
            "Sid": "Stmt1458923121000"
        }
    ]
}
```

Then, go to your lambda function AutoTag-CFAutoTag-XXXXXXX and add the following lines in the following order.

To save you time, copy and paste the following lambda code to your lamda function. Remember to enter your own accountID, access key id, and secret access key in the code.

**WARNING: DELETE your account ID, access key id, and secret access key BEFORE you UPLOAD this code to the INTERNET!**

```python
from __future__ import print_function
import json
import boto3
import logging
import time
import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    #logger.info('Event: ' + str(event))
    #print('Received event: ' + json.dumps(event, indent=2))

    ids = []
    idc = ''
    ide = ''

    try:
        region = event['region']
        detail = event['detail']
        eventname = detail['eventName']
        arn = detail['userIdentity']['arn']
        principal = detail['userIdentity']['principalId']
        userType = detail['userIdentity']['type']
        accountID = 'change to your own accountID' # Change to your own accountID
        
        if userType == 'IAMUser':
            user = detail['userIdentity']['userName']

        else:
            user = principal.split(':')[1]


        logger.info('principalId: ' + str(principal))
        logger.info('region: ' + str(region))
        logger.info('eventName: ' + str(eventname))
        logger.info('detail: ' + str(detail))

        ec2 = boto3.resource('ec2')
        rds = boto3.client('rds')
        # s3 = boto3.resource('s3')
        client = boto3.client('s3')

        if eventname == 'CreateVolume':
            ids.append(detail['responseElements']['volumeId'])
            logger.info(ids)

        elif eventname == 'RunInstances':
            items = detail['responseElements']['instancesSet']['items']
            for item in items:
                ids.append(item['instanceId'])
            logger.info(ids)
            logger.info('number of instances: ' + str(len(ids)))

            base = ec2.instances.filter(InstanceIds=ids)

            #loop through the instances
            for instance in base:
                for vol in instance.volumes.all():
                    ids.append(vol.id)
                for eni in instance.network_interfaces:
                    ids.append(eni.id)

        elif eventname == 'CreateImage':
            ids.append(detail['responseElements']['imageId'])
            logger.info(ids)

        elif eventname == 'CreateSnapshot':
            ids.append(detail['responseElements']['snapshotId'])
            logger.info(ids)
        
        elif eventname == 'CreateDBInstance':
            idc = 'arn:aws:rds:' + region + ':' + accountID + ':db:' + detail['requestParameters']['dBInstanceIdentifier'].lower() # example: arn:aws:rds:us-east-1:509248752274:db:affafafafaa
            logger.info(idc)
            
        elif eventname == 'CreateBucket':
            ide = detail['requestParameters']['bucketName']
            logger.info(ide)
            
        else:
            logger.warning('Not supported action')

        if ids:
            for resourceid in ids:
                print('Tagging resource ' + resourceid)
            
            instances = []
            for status in ec2.meta.client.describe_instance_status()['InstanceStatuses']:
                instances.append(status['InstanceId'])

            def filterInstances(instances):
                filtertemplate = [{'Name': 'resource-id','Values': instances}]
                return filtertemplate

            for instance in instances:
                tags = ec2.meta.client.describe_tags(Filters=filterInstances(instances))
            
            print(tags)
            print(ids)
            # for index, item in enumerate(my_list):
            for index, tag in enumerate(tags['Tags']):
                if tag['Key'] != 'Project' or tag['Key'] != 'End_date' or tag['Key'] != 'Name':
                    print ('SNS ready')
                    sns = boto3.client('sns', aws_access_key_id='AAAAAAAAAA', aws_secret_access_key='AAAAAAAAAAAAA')
                    response = sns.publish(
                        TopicArn='arn:aws:sns:us-west-2:0000000000:Autotag',
                        Message= user + '(' + principal + ') did not include Name, Project and End_date tags tags in ' + ','.join(ids) + '. Please add these tags asap. Thanks!',
                        Subject='AutoTag Alert'
                    )
                if index == 1:
                    break
            ec2.create_tags(Resources=ids, Tags=[{'Key': 'Owner', 'Value': user}, {'Key': 'PrincipalId', 'Value': principal}])
        elif idc:
            response = rds.list_tags_for_resource(ResourceName=idc)
            for index, tag in enumerate(response['TagList']):
                if tag['Key'] != 'Project' or tag['Key'] != 'End_date' or tag['Key'] != 'Name':
                    print ('SNS ready')
                    sns = boto3.client('sns', aws_access_key_id='AAAAAAAAAAAAA', aws_secret_access_key='AAAAAAAAAAAAA')
                    response = sns.publish(
                        TopicArn='arn:aws:sns:us-west-2:0000000000000:Autotag',
                        Message= user + '(' + principal + ') did not include Name, Project and End_date tags in ' + idc + '. Please add these tags asap. Thanks!',
                        Subject='AutoTag Alert'
                    )
                if index == 1:
                    break
            print(response)
            rds.add_tags_to_resource(ResourceName=idc, Tags=[{'Key': 'Owner', 'Value': user}, {'Key': 'PrincipalId', 'Value': principal}])    
        elif ide:
            client.put_bucket_tagging(
                Bucket=ide,
                Tagging={
                    'TagSet': [
                        {
                            'Key': 'Owner',
                            'Value': user
                        },
                        {
                            'Key': 'PrincipalId',
                            'Value': principal
                        },
                    ]
                }
            )
            response = client.get_bucket_tagging(
                Bucket=ide
            )
            for index, tag in enumerate(response['TagSet']):
                if tag['Key'] != 'Project' or tag['Key'] != 'End_date' or tag['Key'] != 'Name':
                    print ('SNS ready')
                    sns = boto3.client('sns', aws_access_key_id='AAAAAAAAAA', aws_secret_access_key='AAAAAAAAAAAAA')
                    response = sns.publish(
                        TopicArn='arn:aws:sns:us-west-2:00000000000000:Autotag',
                        Message= user + '(' + principal + ') did not include Name, Project and End_date tags in ' + ide + '. Please add these tags asap. Thanks!',
                        Subject='AutoTag Alert'
                    )
                if index == 1:
                    break
        logger.info(' Remaining time (ms): ' + str(context.get_remaining_time_in_millis()) + '\n')
        return True
    except Exception as e:
        logger.error('Something went wrong: ' + str(e))
        return False
```
Wait! You're not done yet. Before you leave this page, go to SNS to create a topic and subscribe to it. Here are the steps.

1. Go to SNS.
2. Click on Create Topic in the SNS Dashboard.
3. Type a Topic Name and Display Name.
4. Click Create Topic.
5. Click Topics on the left.
6. Select the topic that you just created.
7. Click on Actions and select Subscribe to topic.
8. Select Email for Protocol. Type in your email and click Create subscription.
9. You will receive a subscription confirmation in a few minutes. Confirm it and then paste the topic arn into your lambda function.

The final step is to add all of the users in the account to the IAM group created by the template. It is called something like, Autotag-ManageEC2InstancesGroup. If you want autotagging to work in another region, you have to set up autotagging in that region by following the same steps as shown above.

If you want to see how the lambda code is working in cloudwatch, you can go to CloudWatch and click on Logs to see the latest lambda function execution. Each of of the function executions shows what happens when each method in the function is executed.

Congrats! You have now completed the Autotagging tutorial. 

Summary of what we have just done:

We created an autotagging service that automatically tags resources with the owner and principal ID and automatically sends you an email when it detects improperly tagged resources. Isn't that great?

How do we ensure that people are tagging all the time? The SNS code that we wrote solves this problem. Another way 
to detect users that aren't following the tagging rules is this solution, Required tags, that I will introduce to you in the following section.

## AWS Config: Required Tags Tutorial

You can use AWS config to quickly find all the users who are not tagging their resources with the required tags. In the steps below, I will show you how that's done.

1. Go to this website http://docs.aws.amazon.com/config/latest/developerguide/required-tags.html
2. Scroll to the bottom of the webpage.
3. Click launch stack.
4. Cick next.
5. Type the tag key and values that you'd like AWS config to check.
6. Click next until you reach create. When you do, click create.
   *Note: AWS config must be turned on before you launch required tags.*
7. When the stack, required-tags-stack, has been created, go to the AWS config dashboard.
8. On the left side, you will see Rules. Click on it.
9. Click on rule name, required-tags. Here, you will see all the noncompliant EC2, RDS, and S3 buckets.


# AWS Autotagging vs. AWS Config Rule: Required tags

Since I have introduced two ways to detect improperly tagged resources in this tutorial, I would like to help you distinguish between the two, so you know which solution works best for you and how to manage the cost of the option that you choose. 

NOTE: If you choose to use Autotagging, the autotagged resources are EC2 Instances, volumes, snapshots, and Amazon Machine Images (AMIs), as well as RDS instances and S3 buckets.

## Pros and Cons AWS Autotagging and AWS Config Rule: Required tags

|Pros                                                  |Cons      |
|--:                                                   |--:                                                                  |
|You can autotag resources.                            |You must have a higher level of understanding of how cloudformation,      cloudwatch, cloudtrail, AMI roles, AMI groups, and lambda code work together                                                |
|You can send automatic email alerts.                  |                                                                     |
|The lambda function is modifiable to fit your needs.  |                                                                     |


|Pros                                                            |Cons                                 |
|--:                                                             |--:                                  |
|You can see a list of untagged resourcesin the Config dashboard.|You cannot autotag resources.        |
|Setting up the required tags config rule is very easy.          |                                     |
|You can send SNS email alerts.                                  |                                     |

The biggest difference between AWS Config rule Required Tags and Autotagging that we care most about is the fact that you cannot autotag with Required Tags. Autotagging can only be done with lambda code.

Although functionality is an important factor for consideration, we must not forget another important factor, cost.

Let's compare the cost of using Required Tags and the Autotagging Lambda function.

## Autotagging Lambda function

Pricing information for CloudFormation, CloudTrail, CloudWatch, SNS and Lambda
(All of this information is obtained from AWS)

### CloudFormation 
"There is no additional charge for AWS CloudFormation." - https://aws.amazon.com/cloudformation/pricing/

### CloudWatch 
##### Amazon CloudWatch Logs

* $0.50 per GB ingested

* $0.03 per GB archived per month

Data Transfer OUT from CloudWatch Logs is priced equivalent to the “Data Transfer OUT from Amazon EC2 To” and “Data Transfer OUT from Amazon EC2 to Internet” tables on the EC2 Pricing Page.

##### Amazon CloudWatch Events - Custom Events
* $1.00 per million custom events generated
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

### How to calculate Lambda costs

Example:

Your lambda function has 128 MB of allocated memory. It is executed 20 million times per month and runs for 800 ms each time it is executed. What would you be charged?

The following information is obtained from AWS Lambda Pricing - https://aws.amazon.com/lambda/pricing/

##### Monthly compute charges

The monthly compute price is $0.00001667 per GB-s and the free tier provides 400,000 GB-s.

Total compute (seconds) = 20M * (0.8sec) = 16,000,000 seconds

Total compute (GB-s) = 16,000,000 * 128MB/1024 = 2,000,000 GB-s

Total Compute – Free tier compute = Monthly billable compute seconds

2,000,000 GB-s – 400,000 free tier GB-s = 1,600,000 GB-s

Monthly compute charges = 1,600,000 * $0.00001667 = $26.672 

##### Monthly request charges

The monthly request price is $0.20 per 1 million requests and the free tier provides 1M requests per month.

Total requests – Free tier request = Monthly billable requests

20M requests – 1M free tier requests = 19M Monthly billable requests

Monthly request charges = 19M * $0.2/M = $3.80 

##### Total compute charges

Total charges = Compute charges + Request charges = $26.672 + $3.80 = $30.472 per month

$30.472 dollars each month might seem a lot, but this is because we are hypothetically setting execution rate per month at 20 million, which does not happen unless you have many many IAM users launching instances, volumes, etc. everyday. The actual cost is actually much lower when both the compute time per month and the number of executions in each month are within the free tier range. Furthermore, the cost estimation for the other AWS services are overestimated as well to show you how costly it can be when the services are working in concert.

### Example Cost breakdown of Autotagging:

Charges for CloudFormation = $0 

Charges for 2 GB of CloudWatch Logs  = $0.50 per GB * 2 + $0.03 per GB * 2 = $1.06

Charges for 1 million generated custom CloudWatch Events = $1.00

Charges for first copy of 150,000 Management events from CloudTrail = $0 

Charges for second copy of 150,000 Management events from CloudTrail = $2 * 150,000/100,000 = $3

Charges for 1M data events from CloudTrail = $0.10 * 1M/100,000 = $1

Charges for 10,000 SNS tag alert emails = $2 * 10,000/100,000 = $0.2

Charges for lambda function with 128 MB allocated memory, 20 million executions per month, and 800 ms duration = $30.472/month

Total = $6.26 + $30.472/month 

## AWS Config Rule Costs:

The following information is obtained from AWS. - https://aws.amazon.com/config/pricing/

With AWS Config, you are charged a one-time fee based on the number of Configuration Items recorded. 
$0.003 per Configuration Item recorded

A rule is active if it has one or more evaluations in a month.
Config Rules costs: $2 per active rule per month

For every active rule, your account receives at no extra charge for that month: 20,000 evaluations.
Unused evaluations do not accumulate.
If you need more evaluations for your rules, additional evaluations will be charged at: $0.10 per thousand evaluations

### Example:

10 Configuration items ONE-TIME FEE = $0.003 * 10 = $0.03
3 Config Rules = $2 * 3 = $6.00

Total = $0.03 + $6.00/per month

## Summary of the costs AWS Autotagging:
According to the AWS Lambda pricing page, "The Lambda free tier includes 1M free requests per month and 400,000 GB-seconds of compute time per month." As we have demonstrated in the example calculation of the costs of Autotagging, the costs only become high when you have over 1M requests per month. 

## Conclusion
These cost estimations are merely for showing you how to calculate the costs for each option, not for convincing you to choose one over the other. Just as a reminder, AWS Config cannot automatically tag untagged resources! I hope you enjoyed reading about the pros and cons as well as how to calculate the costs of using AWS Autotagging and AWS Config.

#### Glossary

##### CloudWatch: 
An AWS service that deploys templates (like packages) that configure resources for you. 
Definition from AWS below.
http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html
AWS CloudFormation is a service that helps you model and set up your Amazon Web Services resources so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. You create a template that describes all the AWS resources that you want (like Amazon EC2 instances or Amazon RDS DB instances), and AWS CloudFormation takes care of provisioning and configuring those resources for you. You don't need to individually create and configure AWS resources and figure out what's dependent on what; AWS CloudFormation handles all of that. The following scenarios demonstrate how AWS CloudFormation can help.

##### CloudTrail: 
An AWS service that records every action performed by the user, role, or AWS service as events.
Definition from AWS below.
http://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
AWS CloudTrail is an AWS service that helps you enable governance, compliance, and operational and risk auditing of your AWS account. Actions taken by a user, role, or an AWS service are recorded as events in CloudTrail. Events include actions taken in the AWS Management Console, AWS Command Line Interface, and AWS SDKs and APIs.

##### AWS Config (AWS definition): 
http://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html
An AWS services that provides you with a detailed view of the configuration of AWS resources in your AWS account. This includes how the resources are related to one another and how they were configured in the past so that you can see how the configurations and relationships change over time.






