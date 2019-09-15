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
   
- To update the AWS SAM CLI to the latest version please copy paste the following commands in the terminal window in the AWS Cloud9 IDE
   
 ```shell script

cd ~/environment
pip install --user --upgrade awscli aws-sam-cli
sam --version
```  
> Note: Please ensure that the SAM CLI version is 0.21.0 or above.

### Install AWS SDK for Python

- To install [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html?id=docs_gateway) (AWS SDK for Python) please copy paste the following commands in the terminal window in the AWS Cloud9 IDE
   
 ```shell script
cd ~/environment
curl -O https://bootstrap.pypa.io/get-pip.py # Get the install script. 
sudo python3 get-pip.py # Install pip.
pip3 install boto3 --user
 ```

## Enable Amazon Dynamodb Streams
In this section you will enable Amazon Dynamodb stream for the Amazon DynamoDB Tables named _'aws-db-workshop-trips'_ that was created as part of the Amazon CloudFormaiton teample.

5. Copy and paste the commands below in the terminal window to enable streams for the Amazon DynamoDB Tables named _'aws-db-workshop-trips'_
```shell script
STREAM_ARN=$(aws dynamodb update-table --table-name aws-db-workshop-trips --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES | jq '.TableDescription.LatestStreamArn' | cut -d'/' -f4)
echo stream/$STREAM_ARN
```

> Note: Please make a note of the output containing the Amazon Dynamodb Stream name _(e.g. stream/2019-09-14T05:39:36.199)_.

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


1. To validate the SAM template please copy and paste the commands below in the terminal window

```shell script
cd ~/environment/amazon-rds-purpose-built-workshop/src/ddb-stream-processor
sam validate
```

> Note: 

1. Open the AWS Management Console for CloudFormation from [here](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2).  
2. In the upper-right corner of the AWS Management Console, confirm you are in the US West (Oregon) Region.  
3. Click on _Stacks_ in the navigation pane on the right.  
4. Under _Stacks_ click on the name of the Amazon CloudFormaiton stack (e.g. _CF-AWSDBWorkshop2019_) that you deployed in the previous lab.  
5. Click the _Outputs_ tab in the    


 