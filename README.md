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
  ContactFlowModuleS3Bucket:
    Type: String
  ContactFlowModuleS3Key:
    Type: String

Resources:
  ConnectContactFlowModule1:
    Type: AWS::Connect::ContactFlowModule
    Properties:
      InstanceArn: !Ref ConnectInstanceArn
      Name: VoiceToChatFlowModule1
      Content:
        Fn::Transform:
          Name: "AWS::Include"
          Parameters:
            Location: !Sub "s3://${ContactFlowModuleS3Bucket}/${ContactFlowModuleS3Key}"

  LambdaFunction1:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: Voice-to-chat-transfer-unique1
      Handler: index.handler
      Role: !Ref LambdaExecutionRole
      Code:
        S3Bucket: voice-to-chat-lambda-solution
        S3Key: Voice-to-chat-transfer-2b6ec221-f880-43a1-af57-544ebd835c7b.zip
      Runtime: python3.10
      Timeout: 15

  PinpointApp1:
    Type: AWS::Pinpoint::App
    Properties:
      Name: voice-to-chat1

  PinpointEmailChannel1:
    Type: AWS::Pinpoint::EmailChannel
    Properties:
      ApplicationId: !Ref PinpointApp1
      FromAddress: ati.pat85@outlook.com  # Using the same email address as before
      Identity: !Ref EmailIdentityArn
      RoleArn: !Ref LambdaExecutionRole

  S3Bucket1:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "my-unique-bucket-name-${AWS::AccountId}-${AWS::Region}-1"

  CloudFrontDistribution1:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Origins:
          - DomainName: !GetAtt S3Bucket1.RegionalDomainName
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
  ConnectContactFlowModuleId1:
    Description: "Connect contact flow module ID"
    Value: !Ref ConnectContactFlowModule1
  LambdaFunctionArn1:
    Description: "Lambda function ARN"
    Value: !GetAtt LambdaFunction1.Arn
  PinpointAppId1:
    Description: "Pinpoint app ID"
    Value: !Ref PinpointApp1
  S3BucketName1:
    Description: "S3 bucket name"
    Value: !Ref S3Bucket1
  CloudFrontDistributionId1:
    Description: "CloudFront distribution ID"
    Value: !Ref CloudFrontDistribution1

```