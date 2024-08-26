AWSTemplateFormatVersion: 2010-09-09
Description: Template for Voice-To-Chat Solution

Parameters:
  LambdaExecutionRole:
    Type: String
    Description: ARN of the IAM role for Lambda execution
  EmailIdentityArn:
    Type: String
    Description: ARN of the SES email identity for Pinpoint email channel

Resources:
  ConnectInstance:
    Type: AWS::Connect::Instance
    Properties:
      IdentityManagementType: CONNECT_MANAGED
      InstanceAlias: VoiceToChatInstance

  ConnectContactFlow:
    Type: AWS::Connect::ContactFlow
    Properties:
      InstanceArn: !GetAtt ConnectInstance.Arn
      Name: VoiceToChatFlow
      Content: >
        {
          "Version": "2019-10-30",
          "StartAction": "1ff34355-4c6a-42fb-8e71-627d4ffcde6a",
          "Metadata": {
            "entryPointPosition": {
              "x": 1114.4,
              "y": 41.6
            },
            "ActionMetadata": {
              "c3d3116b-4833-414d-85c7-54d7ba28ce0a": {
                "position": {
                  "x": 3432.8,
                  "y": 5.6
                }
              },
              "51925f2b-42d6-4172-8dcc-c794be502eff": {
                "position": {
                  "x": 3142.4,
                  "y": -166.4
                }
              },
              "a4893b51-4ae1-44ba-8127-0ad84b24d220": {
                "position": {
                  "x": 4043.2,
                  "y": 12
                }
              },
              "053786fc-1a9d-49bb-9f3b-0615313e7475": {
                "position": {
                  "x": 2664.8,
                  "y": -164
                },
                "parameters": {
                  "LambdaFunctionARN": {
                    "displayName": "Voice-to-chat-transfer"
                  }
                },
                "dynamicMetadata": {
                  "check": false
                }
              },
              "a403434c-d7b9-4cd6-80c3-ce76d77112ea": {
                "position": {
                  "x": 1862.4,
                  "y": 4.8
                }
              },
              "1ff34355-4c6a-42fb-8e71-627d4ffcde6a": {
                "position": {
                  "x": 1432.8,
                  "y": 13.6
                }
              },
              "fbd09b5a-c04e-46c9-900f-3bcbfb693913": {
                "position": {
                  "x": 2665.6,
                  "y": 330.4
                }
              },
              "4750120e-10b0-4cd8-92af-664d52233b80": {
                "position": {
                  "x": 3153.6,
                  "y": 368.8
                }
              },
              "24d5690d-cdfc-4e17-a84f-d018629c7cf8": {
                "position": {
                  "x": 3149.6,
                  "y": 103.2
                }
              },
              "3b2ac413-3ab7-4702-8545-8d4e416da148": {
                "position": {
                  "x": 2269.6,
                  "y": -42.4
                },
                "conditionMetadata": [
                  {
                    "id": "e1066e78-0c96-4241-9280-1a0bc90c68d5",
                    "value": "1"
                  },
                  {
                    "id": "9ee4d5d3-ede3-451a-8797-6974299d1592",
                    "value": "2"
                  }
                ]
              },
              "3de54805-ed88-465a-b9d7-ced52cd08303": {
                "position": {
                  "x": 2676,
                  "y": 100.8
                },
                "parameters": {
                  "LambdaFunctionARN": {
                    "displayName": "Voice-to-chat-transfer"
                  }
                },
                "dynamicMetadata": {
                  "check": false
                }
              }
            }
          },
          "Actions": [
            {
              "Parameters": {
                "Text": "lambda error"
              },
              "Identifier": "c3d3116b-4833-414d-85c7-54d7ba28ce0a",
              "Type": "MessageParticipant",
              "Transitions": {
                "NextAction": "a4893b51-4ae1-44ba-8127-0ad84b24d220",
                "Errors": [
                  {
                    "NextAction": "a4893b51-4ae1-44ba-8127-0ad84b24d220",
                    "ErrorType": "NoMatchingError"
                  }
                ]
              }
            },
            {
              "Parameters": {
                "Text": "You will receive a chat bot link for the chat channel to your registered Email. Please attempt to click the link so that you can use the chatbot.\nThank you for calling have a nice day."
              },
              "Identifier": "51925f2b-42d6-4172-8dcc-c794be502eff",
              "Type": "MessageParticipant",
              "Transitions": {
                "NextAction": "a4893b51-4ae1-44ba-8127-0ad84b24d220",
                "Errors": [
                  {
                    "NextAction": "c3d3116b-4833-414d-85c7-54d7ba28ce0a",
                    "ErrorType": "NoMatchingError"
                  }
                ]
              }
            },
            {
              "Parameters": {},
              "Identifier": "a4893b51-4ae1-44ba-8127-0ad84b24d220",
              "Type": "DisconnectParticipant",
              "Transitions": {}
            },
            {
              "Parameters": {
                "LambdaFunctionARN": "arn:aws:lambda:us-east-1:768637739934:function:Voice-to-chat-transfer",
                "InvocationTimeLimitSeconds": "3",
                "LambdaInvocationAttributes": {
                  "check": "email"
                },
                "ResponseValidation": {
                  "ResponseType": "STRING_MAP"
                }
              },
              "Identifier": "053786fc-1a9d-49bb-9f3b-0615313e7475",
              "Type": "InvokeLambdaFunction",
              "Transitions": {
                "NextAction": "51925f2b-42d6-4172-8dcc-c794be502eff",
                "Errors": [
                  {
                    "NextAction": "a4893b51-4ae1-44ba-8127-0ad84b24d220",
                    "ErrorType": "NoMatchingError"
                  }
                ]
              }
            },
            {
              "Parameters": {
                "LambdaFunctionARN": "arn:aws:lambda:us-east-1:768637739934:function:Voice-to-chat-transfer",
                "InvocationTimeLimitSeconds": "3",
                "LambdaInvocationAttributes": {
                  "check": "email"
                },
                "ResponseValidation": {
                  "ResponseType": "STRING_MAP"
                }
              },
              "Identifier": "3de54805-ed88-465a-b9d7-ced52cd08303",
              "Type": "InvokeLambdaFunction",
              "Transitions": {
                "NextAction": "3b2ac413-3ab7-4702-8545-8d4e416da148",
                "Errors": [
                  {
                    "NextAction": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
                    "ErrorType": "NoMatchingError"
                  }
                ]
              }
            },
            {
              "Parameters": {
                "AttributeName": "FirstName",
                "Condition": "Equals",
                "Value": "Email"
              },
              "Identifier": "3b2ac413-3ab7-4702-8545-8d4e416da148",
              "Type": "CheckContactAttributes",
              "Transitions": {
                "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
                "Conditions": [
                  {
                    "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
                    "Condition": "Equal",
                    "Value": "1"
                  },
                  {
                    "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
                    "Condition": "Equal",
                    "Value": "2"
                  }
                ],
                "Default": "fbd09b5a-c04e-46c9-900f-3bcbfb693913"
              },
              "Errors": [
                {
                  "NextAction": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
                  "ErrorType": "NoMatchingError"
                }
              ]
            },
            {
              "Parameters": {
                "LambdaFunctionARN": "arn:aws:lambda:us-east-1:768637739934:function:Voice-to-chat-transfer",
                "InvocationTimeLimitSeconds": "3",
                "LambdaInvocationAttributes": {
                  "check": "email"
                },
                "ResponseValidation": {
                  "ResponseType": "STRING_MAP"
                }
              },
              "Identifier": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
              "Type": "InvokeLambdaFunction",
              "Transitions": {
                "NextAction": "51925f2b-42d6-4172-8dcc-c794be502eff",
                "Errors": [
                  {
                    "NextAction": "4750120e-10b0-4cd8-92af-664d52233b80",
                    "ErrorType": "NoMatchingError"
                  }
                ]
              }
            },
            {
              "Parameters": {
                "LambdaFunctionARN": "arn:aws:lambda:us-east-1:768637739934:function:Voice-to-chat-transfer",
                "InvocationTimeLimitSeconds": "3",
                "LambdaInvocationAttributes": {
                  "check": "email"
                },
                "ResponseValidation": {
                  "ResponseType": "STRING_MAP"
                }
              },
              "Identifier": "4750120e-10b0-4cd8-92af-664d52233b80",
              "Type": "InvokeLambdaFunction",
              "Transitions": {
                "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
                "Errors": [
                  {
                    "NextAction": "51925f2b-42d6-4172-8dcc-c794be502eff",
                    "ErrorType": "NoMatchingError"
                  }
                ]
              }
            },
            {
              "Parameters": {
                "AttributeName": "FirstName",
                "Condition": "Equals",
                "Value": "Email"
              },
              "Identifier": "24d5690d-cdfc-4e17-a84f-d018629c7cf8",
              "Type": "CheckContactAttributes",
              "Transitions": {
                "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
                "Conditions": [
                  {
                    "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
                    "Condition": "Equal",
                    "Value": "1"
                  },
                  {
                    "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
                    "Condition": "Equal",
                    "Value": "2"
                  }
                ],
                "Default": "fbd09b5a-c04e-46c9-900f-3bcbfb693913"
              },
              "Errors": [
                {
                  "NextAction": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
                  "ErrorType": "NoMatchingError"
                }
              ]
            },
            {
              "Parameters": {
                "LambdaFunctionARN": "arn:aws:lambda:us-east-1:768637739934:function:Voice-to-chat-transfer",
                "InvocationTimeLimitSeconds": "3",
                "LambdaInvocationAttributes": {
                  "check": "email"
                },
                "ResponseValidation": {
                  "ResponseType": "STRING_MAP"
                }
              },
              "Identifier": "4750120e-10b0-4cd8-92af-664d52233b80",
              "Type": "InvokeLambdaFunction",
              "Transitions": {
                "NextAction": "51925f2b-42d6-4172-8dcc-c794be502eff",
                "Errors": [
                  {
                    "NextAction": "c3d3116b-4833-414d-85c7-54d7ba28ce0a",
                    "ErrorType": "NoMatchingError"
                  }
                ]
              }
            }
          ]
        }

  LambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: Voice-to-chat-transfer
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
          - DomainName: !GetAtt S3Bucket.DomainName
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
  ConnectInstanceId:
    Description: "Connect instance ID"
    Value: !Ref ConnectInstance
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


The following resource(s) failed to create: [PinpointApp, LambdaFunction, S3Bucket, ConnectInstance]. Rollback requested by user.
2024-08-22 12:51:23 UTC+0530
PinpointApp
CREATE_FAILED
-
Resource creation cancelled
2024-08-22 12:51:23 UTC+0530
LambdaFunction
CREATE_FAILED
-
Resource creation cancelled
2024-08-22 12:51:23 UTC+0530
S3Bucket
CREATE_FAILED
-
Resource creation cancelled
2024-08-22 12:51:23 UTC+0530
ConnectInstance
CREATE_FAILED
-
Properties validation failed for resource ConnectInstance with message: #: required key [Attributes] not found
this is the error I am getting
