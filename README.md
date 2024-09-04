```yaml
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

```