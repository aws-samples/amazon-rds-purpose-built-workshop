# LAB 2: Taxi Booking And Payments

* [Overview](#overview)  
* [Lab Overview](#lab-overview)
* [Enable Amazon Dynamodb Streams](#enable-amazon-dynamodb-streams)
 
## Overview
![architecture.png](./assets/architecture.png)

## Lab Overview

## Enable Amazon Dynamodb Streams
In this section you will enable Amazon Dynamodb stream for the Amazon DynamoDB Tables named _'aws-db-workshop-trips'_ that was created as part of the Amazon CloudFormaiton teample.

1. Open the AWS Management Console for [AWS Cloud9](https://us-west-2.console.aws.amazon.com/cloud9/home?region=us-west-2#).You will will leverage AWS Cloud9 IDE for throughout this lab for running scripts, deploying AWS SAM ([Serverless Application Model](https://aws.amazon.com/serverless/sam/)) templates, execute SQL queries etc.
2. Click on __Open IDE__ for the AWS Cloud9 IDE that was created as part of the Amazon Cloudformation teamplate that you deployed
3. Open a terminal window in the  AWS Cloud9 IDE by clicking on __Window__ from the menu bar on the top and select __New Terminal__
4. Copy and paste the command below in the terminal window to install [JQ](https://stedolan.github.io/jq/) 

```shell script
sudo yum -y install jq gettext
```
5. Copy and paste the command below in the terminal window to enable streams for the Amazon DynamoDB Tables named _'aws-db-workshop-trips'_
```shell script
STREAM_ARN=$(aws dynamodb update-table --table-name aws-db-workshop-trips --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES | jq '.TableDescription.LatestStreamArn' | cut -d'/' -f4)

echo stream/$STREAM_ARN
```

> Note: Please make a note of the output containing the Amazon Dynamodb Stream name _(e.g. stream/2019-09-14T05:39:36.199)_.

## SAM

1. Open the AWS Management Console for CloudFormation from [here](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2).  
2. In the upper-right corner of the AWS Management Console, confirm you are in the US West (Oregon) Region.  
3. Click on _Stacks_ in the navigation pane on the right.  
4. Under _Stacks_ click on the name of the Amazon CloudFormaiton stack (e.g. _CF-AWSDBWorkshop2019_) that you deployed in the previous lab.  
5. Click the _Outputs_ tab in the    


 