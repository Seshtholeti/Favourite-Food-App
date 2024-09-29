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
      "Parameters": {},
      "Identifier": "a4893b51-4ae1-44ba-8127-0ad84b24d220",
      "Type": "DisconnectParticipant",
      "Transitions": {}
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
        "LambdaFunctionARN": "arn:aws:lambda:us-east-1:768637739934:function:Voice-to-chat-transfer",
        "InvocationTimeLimitSeconds": "8",
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
