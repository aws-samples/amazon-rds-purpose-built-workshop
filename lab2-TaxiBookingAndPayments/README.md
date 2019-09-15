# LAB 2: Taxi Booking And Payments

* [Overview](#overview)  
* [Lab Overview](#lab-overview)
* [Setup AWS Cloud 9 Environment](#aetup-aws-cloud9-environment)  
    * [Install JQ](#install-jq)
    * [Update SAM CLI](#update-sam-cli)
    * [Install AWS SDK for Python](#install-aws-sdk-for-python)
* [Enable Amazon Dynamodb Streams](#enable-amazon-dynamodb-streams)
* [Deploying the AWS Lambda Function](#deploying-aws-lambda-function) 
    * [Packaging PG8000 binaries](#packaging-pg8000-binaries)
    * [Deploy AWS Lambda Function and AWS Lambda Layer using AWS SAM template](#deploy-aws-lambda-function-an-aws-lambda-layer-using-aws-sam-template)
* [Taxi Ride Workflow](#taxi-ride-workflow)
    *[Taxi Trip Booking Workflow](#taxi-trip-booking-workflow)
    
 
## Overview
![architecture.png](./assets/architecture.png)

## Lab Overview

## Setup the AWS Cloud 9 Environment

### Install JQ

1. Open the AWS Management Console for [AWS Cloud9](https://us-west-2.console.aws.amazon.com/cloud9/home?region=us-west-2#). You will will leverage AWS Cloud9 IDE for throughout this lab for running scripts, deploying AWS SAM (Serverless Application Model) templates, execute SQL queries etc.
2. Click on __Open IDE__ for the AWS Cloud9 IDE that was created as part of the Amazon Cloudformation teamplate that you deployed
3. Open a terminal window in the  AWS Cloud9 IDE by clicking on __Window__ from the menu bar on the top and select __New Terminal__
4. Copy and paste the command below in the terminal window to install [JQ](https://stedolan.github.io/jq/) 

```shell script
cd ~/environment
sudo yum -y install jq gettext
```

### Update SAM CLI
   
- To update the AWS SAM CLI to the latest version copy paste the following commands in the terminal window in the AWS Cloud9 IDE
   
 ```shell script

cd ~/environment
pip install --user --upgrade awscli aws-sam-cli
sam --version
```  
> Note: Ensure that the SAM CLI version is 0.21.0 or above.

### Install AWS SDK for Python

- To install [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html?id=docs_gateway) (AWS SDK for Python) copy paste the following commands in the terminal window in the AWS Cloud9 IDE
   
 ```shell script
cd ~/environment
curl -O https://bootstrap.pypa.io/get-pip.py # Get the install script. 
sudo python3 get-pip.py # Install pip.
pip3 install boto3 --user
 ```

###
1. Open the AWS Management Console for CloudFormation from [here](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2).  
2. In the upper-right corner of the AWS Management Console, confirm you are in the US West (Oregon) Region.  
3. Click on _Stacks_ in the navigation pane on the right.  
4. Under _Stacks_ copy and make a note of the name of the Amazon CloudFormation stack (e.g. _CF-AWSDBWorkshop2019_) that you deployed in the previous lab.
5. Substitute the string (_<substitue-name-of-copied-cf-stack-name>_) in the command below with the name of the Amazon CloudFormation stack. Copy and paste the following commands in the terminal window in the AWS Cloud9 IDE to set the environment variable _$AWSDBWORKSHOP_CFSTACK_NAME_ . 

```shell script
AWSDBWORKSHOP_CFSTACK_NAME="<substitue-name-of-copied-cf-stack-name>"
echo "export AWSDBWORKSHOP_CFSTACK_NAME=${AWSDBWORKSHOP_CFSTACK_NAME}" >> ~/.bash_profile
. ~/.bash_profile
echo $AWSDBWORKSHOP_CFSTACK_NAME
```   

> Note: Ensure that the name of the Amazon CloudFormation stack that you deployed in the previous lab printed as an output in the terminal (e.g. _CF-AWSDBWorkshop2019_)

## Enable Amazon Dynamodb Streams
In this section you will enable Amazon Dynamodb stream for the Amazon DynamoDB Tables named _'aws-db-workshop-trips'_ that was created as part of the Amazon CloudFormaiton teample.

5. Copy and paste the commands below in the terminal window to enable streams for the Amazon DynamoDB Tables named _'aws-db-workshop-trips'_
```shell script
STREAM_ID=$(aws dynamodb update-table --table-name aws-db-workshop-trips --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES | jq '.TableDescription.LatestStreamArn' | cut -d'/' -f4)
STREAM_NAME=stream/${STREAM_ID::-1}
echo "export AWSDBWORKSHOP_DDB_STREAM_NAME=${STREAM_NAME}" >> ~/.bash_profile
. ~/.bash_profile
echo $AWSDBWORKSHOP_DDB_STREAM_NAME
```

Now that you have enabled the Amazon DynamoDB stream the next step is to deploy the AWS Lambda function that will process the records from the stream.

## Deploying AWS Lambda Function
In this section we will be using AWS Serverless Applicaiton Model ([SAM]((https://aws.amazon.com/serverless/sam/))) CLI to deploy a Lambda Function within the same Amazon Virtual Private Network([VPC](https://aws.amazon.com/vpc/)). The SAM deployment will also include a python interface to the PostgreSQL database engine as an AWS Lambda Layer. This Lambda function will read the taxi trip information from the Amazon DynamoDB stream as they are inserted / updated in the Amazon DynamoDB table ('aws-db-workshop-trips'). Only when a trip is completed (denoted by the _STATUS_ attribute in the trip item/ record) the function inserts information into a relational table (_trips_) in Amazon Aurora database. 

### Packaging the PG8000 binaries 

In this section you will download and package the binaries for [PG8000](https://pypi.org/project/pg8000/) - a python interface to the PostgreSQL database engine. The package will deployed as an AWS Lambda Layer.

- Copy and paste the commands below in the terminal window in the Cloud9 IDE.

```shell script
cd ~/environment
mkdir pglayer
virtualenv -p python3 pglayer
cd pglayer
source bin/activate
mkdir -p pg8000-layer/python
pip install pg8000 -t pg8000-layer/python
cd pg8000-layer
zip -r pg8000-layer.zip python
mkdir ~/environment/amazon-rds-purpose-built-workshop/src/ddb-stream-processor/dependencies/
cp ~/environment/pglayer/pg8000-layer/pg8000-layer.zip ~/environment/amazon-rds-purpose-built-workshop/src/ddb-stream-processor/dependencies/
```

### Deploy AWS Lambda Function and AWS Lambda Layer using AWS SAM template
In this section you will validate the SAM template that contains the configuration for the AWS Lambda function and the AWS Lambda Layer. 


1. To validate the SAM template copy and paste the commands below in the terminal window

```shell script
cd ~/environment/amazon-rds-purpose-built-workshop/src/ddb-stream-processor
sam validate
```

> Note: Ensure that the SAM template is valid. Look for the following line in the terminal as the output
> ```
> /home/ec2-user/environment/amazon-rds-purpose-built-workshop/src/ddb-stream-processor/template.yaml is a valid SAM Template 
>```

2. To package the AWS SAM application, copy and paste the commands below in the terminal window. This will create a template-out.yaml file is the same folder and will upload the packaged binaries to the specified Amazon S3 bucket.

```
cd ~/environment/amazon-rds-purpose-built-workshop/src/ddb-stream-processor
S3_BUCKETNAME=$(aws cloudformation describe-stacks --stack-name $AWSDBWORKSHOP_CFSTACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="S3bucketName") | .OutputValue')
echo $S3_BUCKETNAME
sam package --output-template-file template-out.yaml --s3-bucket $S3_BUCKETNAME
```

3. [_Optional_]  Please take sometime to review the template-out.yaml file.

4. Copy and paste the command below to ensure that the packages have been uploaded to the Amazon S3 bucket.

 ```shell script
aws s3 ls s3://$S3_BUCKETNAME
```

> Sample Output: 
> ```
> 019-09-15 16:39:56      70451 14b63970e9437bf82ea16664d46a929e  
> 2019-09-15 16:39:56      71954 d3eec91527b02d78de30ae42198cd0c0
> ```

5. Substitute the string (_<substitue-with-the-password-of-aurora-database>_) with password string on the Aurora database. Copy and paste the commands to deploy the AWS Lambda Function along with the AWS Lambda Layer. The AWS Lambda function will read the taxi trip information from the Amazon DynamoDB stream as they are inserted / updated in the Amazon DynamoDB table ('aws-db-workshop-trips'). The AWS Lambda Layer include the [PG8000](https://pypi.org/project/pg8000/) - a python interface to the PostgreSQL database engine.

```shell script
cd ~/environment/amazon-rds-purpose-built-workshop/src/ddb-stream-processor

AURORADB_NAME=$(aws cloudformation describe-stacks --stack-name $AWSDBWORKSHOP_CFSTACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="AuroraDBName") | .OutputValue')
echo $AURORADB_NAME
AURORACLUSTERENDPOINT_NAME=$(aws cloudformation describe-stacks --stack-name $AWSDBWORKSHOP_CFSTACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="AuroraClusterEndpointName") | .OutputValue')
echo $AURORACLUSTERENDPOINT_NAME
AURORADBMASTERUSER_NAME=$(aws cloudformation describe-stacks --stack-name $AWSDBWORKSHOP_CFSTACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="AuroraDBMasterUser") | .OutputValue')
echo $AURORADBMASTERUSER_NAME
LAMBDASECURITYGROUP_ID=$(aws cloudformation describe-stacks --stack-name $AWSDBWORKSHOP_CFSTACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="LambdaSecurityGroupId") | .OutputValue')
echo $LAMBDASECURITYGROUP_ID
LAMBDASUBNET1_ID=$(aws cloudformation describe-stacks --stack-name $AWSDBWORKSHOP_CFSTACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="LambdaSubnet1") | .OutputValue')
LAMBDASUBNET2_ID=$(aws cloudformation describe-stacks --stack-name $AWSDBWORKSHOP_CFSTACK_NAME | jq -r '.Stacks[].Outputs[] | select(.OutputKey=="LambdaSubnet2") | .OutputValue')
echo $LAMBDASUBNET1_ID,$LAMBDASUBNET2_ID

sam deploy --template-file template-out.yaml --capabilities CAPABILITY_IAM --stack-name SAM-AWSDBWorkshop2019 --parameter-overrides DDBStreamName=$AWSDBWORKSHOP_DDB_STREAM_NAME SecurityGroupIds=$LAMBDASECURITYGROUP_ID VpcSubnetIds=$LAMBDASUBNET1_ID,$LAMBDASUBNET2_ID DatabaseName=$AURORADB_NAME DatabaseHostName=$AURORACLUSTERENDPOINT_NAME DatabaseUserName=$AURORADBMASTERUSER_NAME DatabasePassword=<substitue-with-the-password-of-aurora-database>
```

>Note: This may take few minutes. Ensure that the SAM teamplate was successfully deployed. Look for the following line in the terminal as output
>```
>Successfully created/updated stack - SAM-AWSDBWorkshop2019
>```

Now you have successfully deployed the Lambda function.
 
 
 ## Taxi Ride Workflow
 In this section you run the python scripts to simulate the booking of a taxi ride followed by acceptance of the drive and completion of the ride by the driver. After the trip is complete you will run backend SQL queries to process billing and driver payments.
 
 ### Taxi Trip Booking Workflow
 
1. Copy and paste the following commands as a rider to book a new trip.
 
```shell script
cd ~/environment/amazon-rds-purpose-built-workshop/src/ddb-python-script/
python3 rider-book-trip.py
```

> From the output of the script make a note of the tripinfo value. You will be entering this value when prompted by the subsequent scripts. It a randomly generated string that uniquely identifies a trip. It will similar to 
>```
> 0724662,2019-09-15T20:41:30.455031Z
>``` 

2. Copy and paste the following command as a driver to accept a trip. The script will prompt for the 'tripinfo' value. Enter the value from the output of the previous python script you just ran as a rider to book a new trip.

```shell script
python3 driver-accept-trip.py
```
 
3. Copy and paste the following command as a driver to accept a trip. The script will prompt for the 'tripinfo' value. Enter the same 'tripinfo' value that you provided as input to the previous python script you just ran as a driver to accept a trip.

```shell script
python3 driver-accept-trip.py
```
