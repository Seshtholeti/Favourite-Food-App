this is the error I am getting for the template

2024-09-05 11:01:20 UTC+0530
voice-to-chat-model
ROLLBACK_COMPLETE
-
-
2024-09-05 11:01:20 UTC+0530
LambdaFunction
DELETE_COMPLETE
-
-
2024-09-05 11:01:16 UTC+0530
PinpointApp
DELETE_COMPLETE
-
-
2024-09-05 11:01:16 UTC+0530
ConnectContactFlow
DELETE_COMPLETE
-
-
2024-09-05 11:01:15 UTC+0530
PinpointApp
DELETE_IN_PROGRESS
-
-
2024-09-05 11:01:15 UTC+0530
S3Bucket
DELETE_COMPLETE
-
-
2024-09-05 11:01:14 UTC+0530
PinpointEmailChannel
DELETE_COMPLETE
-
-
2024-09-05 11:01:14 UTC+0530
ConnectContactFlow
DELETE_IN_PROGRESS
-
-
2024-09-05 11:01:14 UTC+0530
LambdaFunction
DELETE_IN_PROGRESS
-
-
2024-09-05 11:01:12 UTC+0530
voice-to-chat-model
ROLLBACK_IN_PROGRESS
-
The following resource(s) failed to create: [LambdaFunction, S3Bucket, PinpointEmailChannel]. Rollback requested by user.
2024-09-05 11:01:12 UTC+0530
LambdaFunction
CREATE_FAILED
-
Resource creation cancelled
2024-09-05 11:01:12 UTC+0530
PinpointEmailChannel
CREATE_FAILED
-
Resource creation cancelled
2024-09-05 11:01:12 UTC+0530
ConnectContactFlow
CREATE_COMPLETE
-
-
2024-09-05 11:01:12 UTC+0530
PinpointEmailChannel
CREATE_IN_PROGRESS
-
-
2024-09-05 11:01:11 UTC+0530
S3Bucket
CREATE_FAILED
-
Resource handler returned message: "voice-to-chat-widget-new already exists (Service: S3, Status Code: 0, Request ID: null)" (RequestToken: 75cd2621-e1d5-69e0-1b07-9eff5a9a6c28, HandlerErrorCode: AlreadyExists)
2024-09-05 11:01:11 UTC+0530
ConnectContactFlow
CREATE_IN_PROGRESS
-
Resource creation Initiated
2024-09-05 11:01:11 UTC+0530
PinpointApp
CREATE_COMPLETE
-
-
2024-09-05 11:01:11 UTC+0530
LambdaFunction
CREATE_IN_PROGRESS
-
Resource creation Initiated
2024-09-05 11:01:11 UTC+0530
PinpointApp
CREATE_IN_PROGRESS
-
Resource creation Initiated
2024-09-05 11:01:10 UTC+0530
LambdaFunction
CREATE_IN_PROGRESS
-
-
2024-09-05 11:01:10 UTC+0530
S3Bucket
CREATE_IN_PROGRESS
-
-
2024-09-05 11:01:10 UTC+0530
PinpointApp
CREATE_IN_PROGRESS
-
-
2024-09-05 11:01:10 UTC+0530
ConnectContactFlow
CREATE_IN_PROGRESS
-
-
2024-09-05 11:01:08 UTC+0530
voice-to-chat-model
CREATE_IN_PROGRESS
-
Transformation succeeded
2024-09-05 11:01:05 UTC+0530
voice-to-chat-model
CREATE_IN_PROGRESS
-
User Initiated

AWSTemplateFormatVersion: 2010-09-09
Description: Template for Voice-To-Chat Solution

Parameters:
  ConnectInstanceArn:
    Type: String
  LambdaExecutionRole:
    Type: String
  EmailIdentityArn:
    Type: String
  ContactFlowS3Bucket:
    Type: String
  ContactFlowS3Key:
    Type: String

Resources:
  ConnectContactFlow:
    Type: AWS::Connect::ContactFlow
    Properties:
      InstanceArn: !Ref ConnectInstanceArn
      Name: VoiceToChatFlow
      Type: CONTACT_FLOW
      Fn::Transform:
        Name: "AWS::Include"
        Parameters:
          Location: !Sub "s3://${ContactFlowS3Bucket}/${ContactFlowS3Key}"

  LambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: Voice-to-chat-transfer-unique
      Handler: index.handler
      Role: !Ref LambdaExecutionRole
      Code:
        S3Bucket: voice-to-chat-lambda-solution
        S3Key: Voice-to-chat-transfer-2b6ec221-f880-43a1-af57-544ebd835c7b.zip
      Runtime: python3.10
      Timeout: 15

  PinpointApp:
    Type: AWS::Pinpoint::App
    Properties:
      Name: voice-to-chat

  PinpointEmailChannel:
    Type: AWS::Pinpoint::EmailChannel
    Properties:
      ApplicationId: !Ref PinpointApp
      FromAddress: ati.pat85@outlook.com
      Identity: !Ref EmailIdentityArn
      RoleArn: !Ref LambdaExecutionRole

  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: voice-to-chat-widget-new

  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Origins:
          - DomainName: !GetAtt S3Bucket.RegionalDomainName
            Id: S3Origin
            S3OriginConfig: {}
        Enabled: true
        DefaultCacheBehavior:
          TargetOriginId: S3Origin
          ViewerProtocolPolicy: redirect-to-https
          ForwardedValues:
            QueryString: false
        DefaultRootObject: index.html

Outputs:
  ConnectContactFlowId:
    Description: "Connect contact flow ID"
    Value: !Ref ConnectContactFlow
  LambdaFunctionArn:
    Description: "Lambda function ARN"
    Value: !GetAtt LambdaFunction.Arn
  PinpointAppId:
    Description: "Pinpoint app ID"
    Value: !Ref PinpointApp
  S3BucketName:
    Description: "S3 bucket name"
    Value: !Ref S3Bucket
  CloudFrontDistributionId:
    Description: "CloudFront distribution ID"
    Value: !Ref CloudFrontDistribution


