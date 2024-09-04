From the error message “Invalid request”, it seems that the issue is with the “Content” in the ‘ConnectContactFlow’ resource. I understand that you are trying to fetch the content file from S3 and hence for that you have mentioned the S3 bucket and S3 key in the template as below:
====================
      Content: !Sub |
        {
          "S3Bucket": "${ContactFlowS3Bucket}",
          "S3Key": "${ContactFlowS3Key}"
        }
====================

But in this way we will not be able to specify the content in the ‘AWS::Connect::ContactFlow’ resource. You can see in the CloudFormation document [2], the property ‘Content’ is of type: ‘String’ and you have to provide the content in the template itself.


==WORKAROUND==

However, as a workaround we can retrieve the ‘Content’ ContactFlow Json file from S3 by using ‘AWS::Include’ transform [3]. AWS::Include transform, which is a macro hosted by AWS CloudFormation, to insert boilerplate content into your templates. If the templates reference AWS::Include, CloudFormation inserts the contents of the specified file at the location of the transform in the template. Please see [3] for more information.

I have also replicated it and I was able to create the ContactFlow resource. You’ll need to add the “AWS::Include” transform in the ‘ContactFlow’ resource definition in the template as shown below:
———————————————————————
Resources:
  ConnectContactFlow:
    Type: AWS::Connect::ContactFlow
    Properties:
      InstanceArn: arn:aws:connect:us-east-1:93168*******:instance/85be09a1-cce0-************
      Name: VoiceToChatFlow
      Type: CONTACT_FLOW
      Fn::Transform:             <————— (Add this part)
        Name: 'AWS::Include’ 
        Parameters:  
         Location: 's3://test-bucket/Content.json' 
———————————————————————

The Content file will be retrieved from the S3 bucket. 

Property ‘Content’ will be declared in the ‘Content.json’ file and this file will be uploaded on the S3. The json content inside the ‘Content.json’ file should be in string in the following way, below is just a sample snippet, please change it with your content. 
—————————
"Content": "{\"Version\":\"2019-10-30\",\"StartAction\":\"ending-block\",\"Metadata\":{\"entryPointPosition\":{\"x\":40,\"y\":40},\"ActionMetadata\":{\"ending-block\":{\"position\":{\"x\":299.2,\"y\":33.6},\"isFriendlyName\":true}},\"Annotations\":[],\"name\”:\”Test-flow\",\"description\":\"\",\"type\":\"contactFlow\",\"status\":\"published\",\"hash\":{}},\"Actions\":[{\"Parameters\":{},\"Identifier\":\"ending-block\",\"Type\":\"EndFlowExecution\",\"Transitions\":{}}]}"



below is the json of the contact flow

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
            "id": "7d2aec52-ce09-449e-a8a3-3331b0700e6b",
            "value": "1"
          },
          {
            "id": "5c425df0-c855-4688-b086-9a5bc76b5eca",
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
    },
    "Annotations": [],
    "name": "voice to chat-Module",
    "description": "",
    "status": "published",
    "hash": {}
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
        "RecordingBehavior": {
          "RecordedParticipants": [
            "Agent",
            "Customer"
          ]
        },
        "AnalyticsBehavior": {
          "Enabled": "True",
          "AnalyticsLanguage": "en-US",
          "AnalyticsRedactionBehavior": "Disabled",
          "AnalyticsRedactionResults": "RedactedAndOriginal",
          "ChannelConfiguration": {
            "Chat": {
              "AnalyticsModes": []
            },
            "Voice": {
              "AnalyticsModes": [
                "PostContact"
              ]
            }
          }
        }
      },
      "Identifier": "a403434c-d7b9-4cd6-80c3-ce76d77112ea",
      "Type": "UpdateContactRecordingBehavior",
      "Transitions": {
        "NextAction": "3b2ac413-3ab7-4702-8545-8d4e416da148"
      }
    },
    {
      "Parameters": {
        "FlowLoggingBehavior": "Enabled"
      },
      "Identifier": "1ff34355-4c6a-42fb-8e71-627d4ffcde6a",
      "Type": "UpdateFlowLoggingBehavior",
      "Transitions": {
        "NextAction": "a403434c-d7b9-4cd6-80c3-ce76d77112ea"
      }
    },
    {
      "Parameters": {
        "Text": "error"
      },
      "Identifier": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
      "Type": "MessageParticipant",
      "Transitions": {
        "NextAction": "4750120e-10b0-4cd8-92af-664d52233b80",
        "Errors": [
          {
            "NextAction": "4750120e-10b0-4cd8-92af-664d52233b80",
            "ErrorType": "NoMatchingError"
          }
        ]
      }
    },
    {
      "Parameters": {},
      "Identifier": "4750120e-10b0-4cd8-92af-664d52233b80",
      "Type": "DisconnectParticipant",
      "Transitions": {}
    },
    {
      "Parameters": {
        "Text": "You will receive a chat bot link for the chat channel on your mobile device through SMS. Please attempt to click the link so that you can use the chatbot.Thank you for calling have a nice day."
      },
      "Identifier": "24d5690d-cdfc-4e17-a84f-d018629c7cf8",
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
      "Parameters": {
        "Text": "You can choose to receive Email Or SMS Texts please select your preference to send the Chat Link to an Email please Press 1 and to send it to a Mobile device Press 2 .",
        "StoreInput": "False",
        "InputTimeLimitSeconds": "5"
      },
      "Identifier": "3b2ac413-3ab7-4702-8545-8d4e416da148",
      "Type": "GetParticipantInput",
      "Transitions": {
        "NextAction": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
        "Conditions": [
          {
            "NextAction": "053786fc-1a9d-49bb-9f3b-0615313e7475",
            "Condition": {
              "Operator": "Equals",
              "Operands": [
                "1"
              ]
            }
          },
          {
            "NextAction": "3de54805-ed88-465a-b9d7-ced52cd08303",
            "Condition": {
              "Operator": "Equals",
              "Operands": [
                "2"
              ]
            }
          }
        ],
        "Errors": [
          {
            "NextAction": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
            "ErrorType": "InputTimeLimitExceeded"
          },
          {
            "NextAction": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
            "ErrorType": "NoMatchingCondition"
          },
          {
            "NextAction": "fbd09b5a-c04e-46c9-900f-3bcbfb693913",
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
          "check": "mobile"
        },
        "ResponseValidation": {
          "ResponseType": "STRING_MAP"
        }
      },
      "Identifier": "3de54805-ed88-465a-b9d7-ced52cd08303",
      "Type": "InvokeLambdaFunction",
      "Transitions": {
        "NextAction": "24d5690d-cdfc-4e17-a84f-d018629c7cf8",
        "Errors": [
          {
            "NextAction": "4750120e-10b0-4cd8-92af-664d52233b80",
            "ErrorType": "NoMatchingError"
          }
        ]
      }
    }
  ],
  "Settings": {
    "InputParameters": [],
    "OutputParameters": [],
    "Transitions": [
      {
        "DisplayName": "Success",
        "ReferenceName": "Success",
        "Description": ""
      },
      {
        "DisplayName": "Error",
        "ReferenceName": "Error",
        "Description": ""
      }
    ]
  }
}

