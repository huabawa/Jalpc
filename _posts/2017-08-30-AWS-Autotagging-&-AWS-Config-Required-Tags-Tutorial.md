# AWS EC2, RDS, and S3 Autottagging Tutorial

Sometimes it can be really hard to track down who is using the correct tags on AWS, especially when you have many people launching instances in different regions across the United States.

So, what strategy could you use to solve a problem of this kind?

Before we delve into this problem, let's understand why tagging is so important.

Here is a list of reasons. Please don't skip this part. 

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

## Autagging 

A way to solve this problem is Autotagging.

The autotagging solution we'll be using is summarized below.

When a resource is created, an API call is made and recorded by CloudWatch. Then, a CloudWatch event rule triggers a lambda function providing it with event details.
The lambda function extracts every resource ID and the user's identity and applies two tags, Owner and PrincipalId (current user’s aws:userid value), to the created resource.

A simpler one sentence summary:

When EC2 Instances, Amazon Elastic Block Store (EBS) volumes, EBS snapshots or Amazon Machine Images (AMIs) are created, cloudwatch invokes the autotag lambda function.

Follow the steps below to autotag your EC2 Instance, Amazon Elastic Block Store (EBS) volumes, EBS snapshots and Amazon Machine Images (AMIs).

1. Deploy the cloudformation template in the region of your choosing to create an Autotag stack. Copy and paste the following Amazon S3 template URL.
https://s3.amazonaws.com/awsiammedia/public/sample/autotagec2resources/AutoTag.template

    *Note: Make sure CloudTrail is enabled in this region because cloudwatch events will not work if it is not turned on.*

2. When the autotag stack is created, you will see CREATE_COMPLETE in the status.
Now you can assign IAM users to the created IAM group ManageEC2InstancesGroup under Resources.

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

Now, create another cloud watch rule for s3 bucket. Follow the steps below (some steps are the same as the steps for rds).

1. Same as above.
2. Choose s3 for service name and AWS API Call via CloudTrail for event type.
3. Same as above.
4. Add CreateBucket.

5., 6., 7., 8. Same as above.

Now that we have created the CloudWatch Rules, what do we do next?

1. Edit the Lambda code
2. Create a new IAM Role

Let's start with the IAM role. Here's what you have to do.

Create a new IAM Role with "rds:AddTagsToResource", "rds:Describe*" in Action, as shown below.
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

The final step is to add all of the users in the account to the IAM group created by the template. It is called something like, Autotag-ManageEC2InstancesGroup. 

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






