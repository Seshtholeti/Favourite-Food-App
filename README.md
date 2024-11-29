{
  "statusCode": 200,
  "body": "{\"message\":\"Contact details fetched successfully.\",\"contactDetails\":[{\"contactId\":\"09f2c3c7-c424-4ad2-be1f-246be15b51a4\",\"agentId\":\"N/A\",\"timestamp\":\"2024-11-29T01:22:19.268Z\",\"outboundPhoneNumber\":\"N/A\",\"disposition\":\"Unknown\",\"attributes\":{}},{\"contactId\":\"09f2c3c7-c424-4ad2-be1f-246be15b51a4\",\"agentId\":\"N/A\",\"timestamp\":\"2024-11-29T01:22:19.268Z\",\"outboundPhoneNumber\":\"N/A\",\"disposition\":\"Abandoned\",\"attributes\":{}}]}"
}


here only the contact Id has been fetched, it has to fetch the outbound phone numbers as well, and for that phone number you have to fetch whether the outbound call has been initiated checking whether it is answered or not also the agent id 
