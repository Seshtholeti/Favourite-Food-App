{
    "eventVersion": "1.09",
    "userIdentity": {
        "type": "IAMUser",
        "principalId": "AIDA3F5TBEOPHDOWWS4HO",
        "arn": "arn:aws:iam::768637739934:user/Seshu",
        "accountId": "768637739934",
        "accessKeyId": "ASIA3F5TBEOPK4Y2Q55O",
        "userName": "Seshu",
        "sessionContext": {
            "attributes": {
                "creationDate": "2024-09-29T13:52:03Z",
                "mfaAuthenticated": "false"
            }
        },
        "invokedBy": "cloudformation.amazonaws.com"
    },
    "eventTime": "2024-09-29T14:24:33Z",
    "eventSource": "connect.amazonaws.com",
    "eventName": "CreateContactFlowModule",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "cloudformation.amazonaws.com",
    "userAgent": "cloudformation.amazonaws.com",
    "errorCode": "InvalidContactFlowModuleException",
    "requestParameters": {
        "InstanceId": "arn%3Aaws%3Aconnect%3Aus-east-1%3A768637739934%3Ainstance%2Fbd16d991-11c8-4d1e-9900-edd5ed4a9b21",
        "Content": "***",
        "ClientToken": "ded5a5a5-dece-4446-9fd3-7e6c9e3a6719",
        "Tags": {},
        "Name": "VoiceToChatFlowModule"
    },
    "responseElements": {
        "Problems": [
            {
                "message": "Invalid Action property value. Path: Actions[1].Transitions.NextAction"
            },
            {
                "message": "Invalid Action property value. Path: Actions[2].Transitions.NextAction"
            },
            {
                "message": "Invalid Action property value. Path: Actions[2].Transitions.Errors[0].NextAction"
            },
            {
                "message": "Invalid Action property value. Path: Actions[3].Transitions.NextAction"
            },
            {
                "message": "Invalid Action property value. Path: Actions[3].Transitions.Errors[0].NextAction"
            }
        ]
    },
    "requestID": "70f72e9e-4cd4-467f-9f59-2ff23bae7da4",
    "eventID": "cebcc045-82e0-4ba7-9f52-a480a7f349de",
    "readOnly": false,
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "768637739934",
    "eventCategory": "Management"
}
