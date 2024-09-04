AWSTemplateFormatVersion: 2010-09-09
Description: Template for Voice-To-Chat Solution

Parameters:
  ConnectInstanceArn:
    Type: String
    Description: ARN of the existing Connect instance
  LambdaExecutionRole:
    Type: String
    Description: ARN of the IAM role for Lambda execution
  EmailIdentityArn:
    Type: String
    Description: ARN of the SES email identity for Pinpoint email channel
  ContactFlowS3Bucket:
    Type: String
    Description: Name of the S3 bucket where the contact flow JSON is stored
  ContactFlowS3Key:
    Type: String
    Description: S3 key of the contact flow JSON file

Resources:
  ConnectContactFlow:
    Type: AWS::Connect::ContactFlow
    Properties:
      InstanceArn: !Ref ConnectInstanceArn
      Name: VoiceToChatFlow
      Type: CONTACT_FLOW
      Fn::Transform:
        Name: 'AWS::Include'
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
      FromAddress: noreply@yourdomain.com
      Identity: !Ref EmailIdentityArn


"{\"Version\": \"2019-10-30\", \"StartAction\": \"1ff34355-4c6a-42fb-8e71-627d4ffcde6a\", \"Metadata\": {\"entryPointPosition\": {\"x\": 1114.4, \"y\": 41.6}, \"ActionMetadata\": {\"c3d3116b-4833-414d-85c7-54d7ba28ce0a\": {\"position\": {\"x\": 3432.8, \"y\": 5.6}}, \"51925f2b-42d6-4172-8dcc-c794be502eff\": {\"position\": {\"x\": 3142.4, \"y\": -166.4}}, \"a4893b51-4ae1-44ba-8127-0ad84b24d220\": {\"position\": {\"x\": 4043.2, \"y\": 12}}, \"053786fc-1a9d-49bb-9f3b-0615313e7475\": {\"position\": {\"x\": 2664.8, \"y\": -164}, \"parameters\": {\"LambdaFunctionARN\": {\"displayName\": \"Voice-to-chat-transfer\"}}, \"dynamicMetadata\": {\"check\": false}}, \"a403434c-d7b9-4cd6-80c3-ce76d77112ea\": {\"position\": {\"x\": 1862.4, \"y\": 4.8}}, \"1ff34355-4c6a-42fb-8e71-627d4ffcde6a\": {\"position\": {\"x\": 1432.8, \"y\": 13.6}}, \"fbd09b5a-c04e-46c9-900f-3bcbfb693913\": {\"position\": {\"x\": 2665.6, \"y\": 330.4}}, \"4750120e-10b0-4cd8-92af-664d52233b80\": {\"position\": {\"x\": 3153.6, \"y\": 368.8}}, \"24d5690d-cdfc-4e17-a84f-d018629c7cf8\": {\"position\": {\"x\": 3149.6, \"y\": 103.2}}, \"3b2ac413-3ab7-4702-8545-8d4e416da148\": {\"position\": {\"x\": 2269.6, \"y\": -42.4}, \"conditionMetadata\": [{\"id\": \"7d2aec52-ce09-449e-a8a3-3331b0700e6b\", \"value\": \"1\"}, {\"id\": \"5c425df0-c855-4688-b086-9a5bc76b5eca\", \"value\": \"2\"}]}, \"3de54805-ed88-465a-b9d7-ced52cd08303\": {\"position\": {\"x\": 2676, \"y\": 100.8}, \"parameters\": {\"LambdaFunctionARN\": {\"displayName\": \"Voice-to-chat-transfer\"}}, \"dynamicMetadata\": {\"check\": false}}}, \"Annotations\": [], \"name\": \"voice to chat-Module\", \"description\": \"\", \"status\": \"published\", \"hash\": {}} , \"Actions\": [{\"Parameters\": {\"Text\": \"lambda error\"}, \"Identifier\": \"c3d3116b-4833-414d-85c7-54d7ba28ce0a\", \"Type\": \"MessageParticipant\", \"Transitions\": {\"NextAction\": \"a4893b51-4ae1-44ba-8127-0ad84b24d220\", \"Errors\": [{\"NextAction\": \"a4893b51-4ae1-44ba-8127-0ad84b24d220\", \"ErrorType\": \"NoMatchingError\"}]}}, {\"Parameters\": {\"Text\": \"You will receive a chat bot link for the chat channel to your registered Email. Please attempt to click the link so that you can use the chatbot.\\nThank you for calling have a nice day.\"}, \"Identifier\": \"51925f2b-42d6-4172-8dcc-c794be502eff\", \"Type\": \"MessageParticipant\", \"Transitions\": {\"NextAction\": \"a4893b51-4ae1-44ba-8127-0ad84b24d220\", \"Errors\": [{\"NextAction\": \"c3d3116b-4833
