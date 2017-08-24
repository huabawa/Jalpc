How to parse the AWS detailed billing report located in S3 with Boto3

If you have an AWS linked account and you get errors like Access Denied, as shown below, when you want to get access to cost and usage reports, what should you do?

Luckily, there's a solution.

AWS stores and updates detailed billing reports in your S3 bucket, which you (the user) can access, so you might be wondering, how can I access it using command prompt or terminal?

Accessing S3 bucket with Boto3

Here is a link to the latest boto3 documentation. https://boto3.readthedocs.io/en/latest/.
I recommend looking at the S3 bucket examples. https://boto3.readthedocs.io/en/latest/guide/s3-examples.html
But before that, let's first understand why we are using boto3.

What is Boto3?

Like the boto3 documentation says, Boto is the Amazon Web Services (AWS) SDK for Python. This means you can write your own python software for EC2 and S3. Isn't that amazing? The user doesn't have to open the AWS console at all to open the s3 bucket!

How to Install Boto3.

Trust me it's simple. Just follow the steps below to set up Boto3 on your computer. When you get stuck, don't panic!

1. Install pip if you don't have pip installed already. Update pip to make sure it is the latest version.
Link to pip installation documentation: https://pip.pypa.io/en/stable/installing/
2. Install AWS CLI. Link to AWS CLI installation guide: http://docs.aws.amazon.com/cli/latest/userguide/installing.html
3. Install Boto3. Link to Boto3 Installation: http://boto3.readthedocs.io/en/latest/guide/quickstart.html

Congrats! You have successfully installed boto3. The next step will be to write some python code to print out the S3 bucket list, list the contents of an S3 bucket, and more.

Setting up AWS CLI

When you have installed AWS CLI, check that you have successfully installed it by typing, aws --version, in your terminal.
Something like this, aws-cli/1.11.136 Python/2.7.13 Darwin/15.6.0 botocore/1.6.3, should appear.
Type into the terminal, aws configure, to configure AWS CLI. For precise instructions, go to http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html

How to print out the S3 bucket list

Type into terminal, python.
Then, type the following.

import boto3
s3 = boto3.client('s3')
response = s3.list_buckets()
buckets = [bucket['Name'] for bucket in response['Buckets']]
print("Bucket List: %s" % buckets)

Listing Contents of a S3 bucket

import boto3
s3 = boto3.resource('s3')
my_bucket = s3.Bucket('bucket_name')
for object in my_bucket.objects.all():
  print(object)
  
  * Click Return twice.

Downloading a file in S3 bucket

import boto3
s3 = boto3.resource('s3')
bucket = s3.Bucket('bucket_name')
s3.meta.client.download_file('bucket_name', 'file.you.want.to.download', 'directory/name-of-output-file')

Option2:
import boto3
s3 = boto3.resource('s3')
bucket = s3.Bucket('bucket_name')
bucket.download_file(key, 'file-name')

Uploading a file to S3 bucket

import boto3
s3 = boto3.resource('s3')
bucket = s3.Bucket('509248752274-dlt-utilization')
s3.meta.client.upload_file('directory/file-name', 'bucket-name', 'file-name')

Print and Save Parsed detailed billing report

import pandas
raw = pandas.read_csv('Directory of saved csv')
raw.loc[:,'LinkedAccountId':'SubscriptionId'] # display only columns 'LinkedAccountId' to 'SubscriptionId'
filtered = raw.loc[:,'LinkedAccountId':'SubscriptionId'] # save parsed detailed billing report
filtered.to_csv('OUT_FILE.csv') # saves in the same directory

For detailed instructions on selecting data, click this link. https://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-label

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










