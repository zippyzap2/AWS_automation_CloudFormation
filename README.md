# AWS CloudFormation End-to-End Project

## Introduction

This project provides a comprehensive **Infrastructure as Code (IaC)** solution using **AWS CloudFormation** to automate the provisioning and management of a robust AWS environment. The template integrates key AWS services such as **EC2, RDS, S3, IAM, WAF, Shield, CloudWatch, AWS Config, AWS Systems Manager (SSM), and AWS CodePipeline** to ensure a highly available, secure, and scalable cloud architecture.

By implementing this project, users can efficiently deploy cloud resources, automate security compliance checks, monitor infrastructure health, enable CI/CD for seamless application deployments, and manage operational tasks with minimal manual intervention. This solution is ideal for organizations looking to enhance their DevOps practices by leveraging automation, security, and operational insights provided by AWS.



## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Prerequisites](#prerequisites)
4. [CloudFormation Template Overview](#cloudformation-template-overview)
5. [Step-by-Step Implementation](#step-by-step-implementation)
   - [Step 1: Set Up CloudFormation Stack](#step-1-set-up-cloudformation-stack)
   - [Step 2: Deploy Compute Resources](#step-2-deploy-compute-resources)
   - [Step 3: Configure Storage](#step-3-configure-storage)
   - [Step 4: Set Up Database](#step-4-set-up-database)
   - [Step 5: Implement Monitoring](#step-5-implement-monitoring)
   - [Step 6: Secure the Infrastructure](#step-6-secure-the-infrastructure)
   - [Step 7: Deploy and Test the Application](#step-7-deploy-and-test-the-application)
6. [Managing the Infrastructure](#managing-the-infrastructure)
7. [Cost Optimization Tips](#cost-optimization-tips)
8. [Infrastructure as Code (IaC) Implementation](#infrastructure-as-code-iac-implementation)
9. [Integrate CI/CD with AWS CodePipeline](#integrate-cicd-with-aws-codepipeline)
10. [CloudWatch Monitoring Configuration](#cloudwatch-monitoring-configuration)

---


![diagram-export-2-12-2025-5_34_18-PM](https://github.com/user-attachments/assets/30e67e9d-c640-4510-a599-cb09f51a47a5)



## CloudFormation Template Overview

AWS CloudFormation enables Infrastructure as Code (IaC) by defining AWS resources in a YAML template.

### Updated CloudFormation YAML Template with Security Features

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyIAMRole:
    Type: 'AWS::IAM::Role'
    Properties:
      RoleName: MyWebAppRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Policies:
        - PolicyName: MyWebAppPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 's3:*'
                  - 'cloudwatch:*'
                  - 'logs:*'
                Resource: '*'

  MyEC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c55b159cbfafe1f0 # Change based on region
      KeyName: my-key-pair # Change to your key pair
      IamInstanceProfile: !Ref MyIAMRole

  MyS3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: my-webapp-static-files

  MyRDSInstance:
    Type: 'AWS::RDS::DBInstance'
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: MySQL
      MasterUsername: admin
      MasterUserPassword: MySecurePassword123
      AllocatedStorage: 20
      PubliclyAccessible: false

  MyWAF:
    Type: 'AWS::WAFv2::WebACL'
    Properties:
      Name: MyWebAppWAF
      Scope: REGIONAL
      DefaultAction:
        Allow: {}
      Rules:
        - Name: BlockBadBots
          Priority: 1
          Action:
            Block: {}
          Statement:
            RateBasedStatement:
              Limit: 1000
              AggregateKeyType: IP
      VisibilityConfig:
        SampledRequestsEnabled: true
        CloudWatchMetricsEnabled: true
        MetricName: MyWAFMetric

  MyShield:
    Type: 'AWS::Shield::Protection'
    Properties:
      Name: MyWebAppShield
      ResourceArn: !Ref MyEC2Instance
```

### Deployment Steps

1. Save this YAML as `cloudformation-template.yaml`.
2. Deploy the stack using AWS CLI:
   ```sh
   aws cloudformation create-stack --stack-name MyWebAppStack --template-body file://cloudformation-template.yaml --capabilities CAPABILITY_NAMED_IAM
   ```
3. Monitor stack creation:
   ```sh
   aws cloudformation describe-stacks --stack-name MyWebAppStack
   ```
4. Verify EC2, RDS, and S3 resources, IAM roles, AWS WAF, and AWS Shield in AWS Console.

With these security features, the infrastructure is better protected from threats while maintaining secure access controls.

---

## Integrate CI/CD with AWS CodePipeline

AWS CodePipeline automates the software release process, allowing developers to build, test, and deploy code automatically.

### Steps to Set Up CodePipeline

1. **Create an S3 Bucket for Artifacts**:
   ```sh
   aws s3 mb s3://my-codepipeline-artifacts
   ```
2. **Create a CodeCommit Repository** (Optional):
   ```sh
   aws codecommit create-repository --repository-name MyWebAppRepo
   ```
3. **Configure AWS CodeBuild**:
   - Create `buildspec.yml`:
     ```yaml
     version: 0.2
     phases:
       install:
         commands:
           - echo Installing dependencies...
       build:
         commands:
           - echo Building application...
     ```
4. **Define Pipeline JSON** (`pipeline.json`):
   ```json
   {
     "pipeline": {
       "name": "MyWebAppPipeline",
       "roleArn": "arn:aws:iam::123456789012:role/AWSCodePipelineServiceRole",
       "artifactStore": {
         "type": "S3",
         "location": "my-codepipeline-artifacts"
       },
       "stages": [
         {
           "name": "Source",
           "actions": [
             {
               "name": "SourceAction",
               "actionTypeId": {
                 "category": "Source",
                 "owner": "AWS",
                 "provider": "S3",
                 "version": "1"
               },
               "outputArtifacts": [{ "name": "SourceArtifact" }],
               "configuration": {
                 "S3Bucket": "my-codepipeline-artifacts",
                 "S3ObjectKey": "source.zip"
               }
             }
           ]
         }
       ]
     }
   }
   ```
5. **Create Pipeline**:
   ```sh
   aws codepipeline create-pipeline --cli-input-json file://pipeline.json
   ```
6. **Monitor Pipeline Execution**:
   ```sh
   aws codepipeline list-pipeline-executions --pipeline-name MyWebAppPipeline
   ```
7. **Deploy New Changes**:
   - Commit and push code to trigger a pipeline run.

By integrating AWS CodePipeline, every code change is built, tested, and deployed automatically, ensuring seamless and efficient software delivery.

## AWS Config for Compliance and Security Monitoring

AWS Config continuously monitors and records AWS resource configurations and helps automate compliance auditing.

### AWS Config Setup Using CloudFormation

#### CloudFormation Template for AWS Config

```yaml
Resources:
  AWSConfigRecorder:
    Type: 'AWS::Config::ConfigurationRecorder'
    Properties:
      Name: default
      RoleARN: !GetAtt AWSConfigRole.Arn
      RecordingGroup:
        AllSupported: true
        IncludeGlobalResourceTypes: true

  AWSConfigRole:
    Type: 'AWS::IAM::Role'
    Properties:
      RoleName: AWSConfigRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: config.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Policies:
        - PolicyName: AWSConfigPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 'config:*'
                  - 's3:PutObject'
                Resource: '*'

  AWSConfigBucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: my-aws-config-logs

  AWSConfigDeliveryChannel:
    Type: 'AWS::Config::DeliveryChannel'
    Properties:
      Name: default
      S3BucketName: !Ref AWSConfigBucket
```

### Deploy AWS Config Stack

1. Save this YAML as `aws-config-template.yaml`.
2. Deploy using AWS CLI:
   ```sh
   aws cloudformation create-stack --stack-name AWSConfigStack --template-body file://aws-config-template.yaml --capabilities CAPABILITY_NAMED_IAM
   ```
3. Verify configuration in **AWS Config Console**.
4. Enable AWS Config rules for **compliance checks**.

With AWS Config, you can continuously monitor compliance and maintain security best practices.

## AWS Systems Manager (SSM) for Automation and Management

AWS Systems Manager (SSM) provides operational insights, patch management, and secure access to AWS resources.

### AWS SSM Setup Using CloudFormation

#### CloudFormation Template for AWS Systems Manager
```yaml
Resources:
  SSMIAMRole:
    Type: 'AWS::IAM::Role'
    Properties:
      RoleName: SSMIAMRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Policies:
        - PolicyName: SSMPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 'ssm:*'
                  - 'ec2messages:*'
                  - 'cloudwatch:*'
                  - 'logs:*'
                Resource: '*'

  SSMParameter:
    Type: 'AWS::SSM::Parameter'
    Properties:
      Name: MyAppConfig
      Type: String
      Value: "AppConfigurationValue"

  SSMAssociation:
    Type: 'AWS::SSM::Association'
    Properties:
      Name: AWS-UpdateSSMAgent
      Targets:
        - Key: InstanceIds
          Values:
            - !Ref MyEC2Instance

  SSMManagedInstance:
    Type: 'AWS::SSM::ManagedInstance'
    Properties:
      InstanceId: !Ref MyEC2Instance
```

### Deploy AWS SSM Stack
1. Save this YAML as `aws-ssm-template.yaml`.
2. Deploy using AWS CLI:
   ```sh
   aws cloudformation create-stack --stack-name AWSSSMStack --template-body file://aws-ssm-template.yaml --capabilities CAPABILITY_NAMED_IAM
   ```
3. Verify SSM configurations in **AWS Systems Manager Console**.
4. Use **Session Manager** for secure remote access to EC2 instances without SSH.
5. Enable **Patch Manager** for automatic OS updates.

With AWS Systems Manager, administrators can automate maintenance tasks and improve security with remote access management.

## CloudWatch Monitoring Configuration

AWS CloudWatch enables monitoring, logging, and alerting for various AWS services, ensuring high availability and performance.

### CloudWatch Logs

1. **Enable CloudWatch Logs for EC2**:

   - Install CloudWatch Agent:
     ```sh
     sudo yum install -y amazon-cloudwatch-agent
     ```
   - Configure logs in `/opt/aws/amazon-cloudwatch-agent/bin/config.json`:
     ```json
     {
       "logs": {
         "logs_collected": {
           "files": {
             "collect_list": [
               {
                 "file_path": "/var/log/messages",
                 "log_group_name": "EC2-System-Logs",
                 "log_stream_name": "{instance_id}"
               }
             ]
           }
         }
       }
     }
     ```
   - Start CloudWatch Agent:
     ```sh
     sudo systemctl start amazon-cloudwatch-agent
     ```

2. **Enable RDS Enhanced Monitoring**:

   - Navigate to **RDS Console** → Select DB Instance → Modify → Enable Enhanced Monitoring.
   - Choose monitoring interval and IAM role.

3. **Enable S3 Server Access Logging**:

   - Navigate to **S3 Console** → Select bucket → Properties → Server Access Logging → Enable and set a target bucket.

### CloudWatch Metrics and Alarms

1. **Monitor EC2 Metrics** (CPU, Memory, Disk, Network):
   ```sh
   aws cloudwatch put-metric-alarm --alarm-name "High-CPU-Utilization" \
     --metric-name CPUUtilization --namespace AWS/EC2 \
     --statistic Average --period 300 --threshold 80 \
     --comparison-operator GreaterThanThreshold --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
     --evaluation-periods 2 --alarm-actions arn:aws:sns:us-east-1:123456789012:MySNSTopic
   ```
2. **Monitor RDS Metrics** (CPU, Free Storage, Connections):
   ```sh
   aws cloudwatch put-metric-alarm --alarm-name "RDS-CPU-Usage" \
     --metric-name CPUUtilization --namespace AWS/RDS \
     --statistic Average --period 300 --threshold 75 \
     --comparison-operator GreaterThanThreshold --dimensions Name=DBInstanceIdentifier,Value=mydbinstance \
     --evaluation-periods 2 --alarm-actions arn:aws:sns:us-east-1:123456789012:MySNSTopic
   ```
3. **Monitor S3 Bucket Size & Requests**:
   - Go to **CloudWatch Console** → Select Metrics → S3 → View Bucket Metrics.

### How to View CloudWatch Metrics & Logs

1. **Go to AWS CloudWatch Console**
2. **Navigate to Metrics** → Filter by **EC2, RDS, or S3**
3. **Check Log Groups** under **CloudWatch Logs**
4. **View Alarm Status** under **Alarms**

With these CloudWatch configurations, administrators can effectively monitor infrastructure health and receive alerts for potential issues.

## Conclusion

This CloudFormation template **automates deployment** of a scalable, secure, and cost-effective web application infrastructure. You can modify parameters to suit your needs and deploy applications efficiently.

---

## Next Steps

- **Enhance Security:** Use AWS Config and Security Hub for continNo ous security monitoring.
- **Use Route 53:** Set up a custom domain name.
- **Integrate CI/CD:** Use AWS CodePipeline for automation to improve deployment efficiency.

---

### Author: DevOps Engineer

#### License: MIT

