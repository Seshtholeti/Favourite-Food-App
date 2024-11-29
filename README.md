import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';
import csvParser from 'csv-parser';
import { ConnectClient, SearchContactsCommand, GetContactAttributesCommand } from '@aws-sdk/client-connect';
const s3 = new S3Client();
const connect = new ConnectClient();
export const handler = async (event) => {
 const bucketName = 'customeroutbound-data';
 const fileName = 'CustomerOutboundNumber.csv';
 const instanceId = 'bd16d991-11c8-4d1e-9900-edd5ed4a9b21';
 try {
   // Step 1: Fetch phone numbers from CSV
   const phoneNumbers = await fetchPhoneNumbersFromCSV(bucketName, fileName);
   if (phoneNumbers.length === 0) {
     throw new Error('No phone numbers found in the CSV file.');
   }
   // Step 2: Get yesterday's date
   const yesterday = new Date();
   yesterday.setDate(yesterday.getDate() - 1);
   const startOfYesterday = new Date(yesterday.setHours(0, 0, 0, 0)).toISOString();
   const endOfYesterday = new Date(yesterday.setHours(23, 59, 59, 999)).toISOString();
   // Step 3: Check call records for yesterday
   const callDetails = await getCallDetails(startOfYesterday, endOfYesterday, instanceId, phoneNumbers);
   // Step 4: Log or return the call details
   console.log('Call Details:', callDetails);
   return {
     statusCode: 200,
     body: JSON.stringify(callDetails),
   };
 } catch (error) {
   console.error('Error processing the Lambda function:', error);
   return {
     statusCode: 500,
     body: JSON.stringify({ error: error.message, details: error.stack }),
   };
 }
};
const fetchPhoneNumbersFromCSV = async (bucketName, fileName) => {
 const params = { Bucket: bucketName, Key: fileName };
 const command = new GetObjectCommand(params);
 const response = await s3.send(command);
 const stream = response.Body;
 if (!stream) {
   throw new Error("No stream data found in the S3 object.");
 }
 const phoneNumbers = [];
 await new Promise((resolve, reject) => {
   stream
     .pipe(csvParser({ separator: ';' }))
     .on('data', (row) => {
       const phoneNumber = row.PhoneNumber || row['Name;PhoneNumber']?.split(';')[1]?.trim();
       if (phoneNumber) {
         let formattedNumber = phoneNumber.replace(/\D/g, '');
         if (formattedNumber.length === 10) {
           formattedNumber = `+91${formattedNumber}`;
         } else if (formattedNumber.length === 11) {
           formattedNumber = `+1${formattedNumber}`;
         }
         phoneNumbers.push(formattedNumber);
         console.log(phoneNumbers)
       }
     })
     .on('end', resolve)
     .on('error', reject);
 });
 return phoneNumbers;
};
const getCallDetails = async (startTime, endTime, instanceId, phoneNumbers) => {
 const callDetails = [];
 let nextToken = null;
 try {
   // Loop through each phone number and get its call details
   for (const number of phoneNumbers) {
     let shouldContinue = true;
     while (shouldContinue) {
       
       const searchContactsParams = {
         InstanceId: instanceId,
         TimeRange: {
           Type: 'INITIATION_TIMESTAMP',  
           StartTime: new Date(startTime),
           EndTime: new Date(endTime),
         },
         SearchCriteria: {
           Channels: ['VOICE'],
           InitiationMethods: ['OUTBOUND'],
         },
         MaxResults: 10, // Adjust the number of results as needed
         NextToken: nextToken,  // Include the token for pagination
       };
       const searchContactsCommand = new SearchContactsCommand(searchContactsParams);
       const contactsData = await connect.send(searchContactsCommand);
       console.log('Search Contacts Response:', contactsData); // Log the response for debugging
       // Step 2: Fetch detailed attributes for each contact
       for (const contact of contactsData.Contacts) {
         // Only consider contacts that match the phone number
         if (contact.ContactId) {
           const contactAttributesParams = {
             InstanceId: instanceId,
             InitialContactId: contact.ContactId,
           };
           const getAttributesCommand = new GetContactAttributesCommand(contactAttributesParams);
           const attributesData = await connect.send(getAttributesCommand);
           console.log('Contact Attributes:', attributesData); // Log the attributes for debugging
           // Step 3: Check if the customer phone number matches
           const customerPhoneNumber = attributesData.Attributes.CustomerPhoneNumber;
           if (customerPhoneNumber && customerPhoneNumber.includes(number)) {
             // Push the detailed call information
             callDetails.push({
               outboundNumber: number,
               contactId: contact.ContactId,
               agentId: attributesData.Attributes.AgentId || 'N/A',
               disposition: attributesData.Attributes.Disposition || 'N/A',
               timestamp: contact.InitiationTimestamp,
             });
           }
         }
       }
       // Step 3: Handle pagination
       nextToken = contactsData.NextToken;
       if (!nextToken) {
         shouldContinue = false;
       }
     }
   }
 } catch (error) {
   console.error("Error fetching call details:", error);
   throw new Error("Error fetching call details from Amazon Connect.");
 }
 return callDetails;
};
this is the response

MyEventName

Response
{
  "statusCode": 200,
  "body": "[]"
}

Function Logs
T10:27:07.510Z,
  QueueInfo: {}
} 17e1e61e-f825-4533-a729-b421cfe590fb
2024-11-29T05:32:12.631Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '7fbe22e8-aaf6-4e00-b8a8-3788fd1e92de',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}
2024-11-29T05:32:12.632Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:33:54.580Z,
  Id: 'f66b1647-5af0-4402-a0c1-d191a76da356',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:29:15.855Z,
  QueueInfo: {}
} f66b1647-5af0-4402-a0c1-d191a76da356
2024-11-29T05:32:12.771Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '17d1f790-dfbf-4fce-a05d-fd36b57e1efc',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}
2024-11-29T05:32:12.771Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {
    ConnectedToAgentTimestamp: 2024-11-28T05:27:49.471Z,
    Id: '9783145b-f6e4-4d42-9414-eb8943c0d9aa'
  },
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:29:05.786Z,
  Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:24:55.314Z,
  QueueInfo: {
    EnqueueTimestamp: 2024-11-28T05:27:33.212Z,
    Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f'
  }
} b139fffc-dc3f-41d1-9496-5a97f44b62fa
2024-11-29T05:32:12.862Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'e8bdc941-8ffc-486c-8ce8-8355cb8acae9',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}
2024-11-29T05:32:12.862Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:14:58.950Z,
  Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:09:44.135Z,
  QueueInfo: {
    EnqueueTimestamp: 2024-11-28T05:13:20.979Z,
    Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f'
  }
} bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9
2024-11-29T05:32:12.974Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'ef4c6534-523b-41a8-a42b-d1757f13f7ec',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}
2024-11-29T05:32:12.974Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:09:23.956Z,
  Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:09:12.948Z,
  QueueInfo: {}
} 88a1ac71-9b65-479f-b7b0-e610794fabb4
2024-11-29T05:32:13.076Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '4a4c883f-36c6-4694-8af2-61dca066d414',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}
2024-11-29T05:32:13.076Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Call Details: []

but call details array is empty.

this is cloud watch logs

2024-11-29T05:32:10.889Z
START RequestId: 02b80bde-b024-4c85-9a88-e779ae076452 Version: $LATEST
2024-11-29T05:32:11.026Z
2024-11-29T05:32:11.026Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sdsd 2024-11-28T00:00:00.000Z 2024-11-28T23:59:59.999Z

2024-11-29T05:32:11.026Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sdsd 2024-11-28T00:00:00.000Z 2024-11-28T23:59:59.999Z
2024-11-29T05:32:11.102Z
2024-11-29T05:32:11.102Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Search Contacts Response: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'aebf1a01-2c26-498e-b711-9a6ff78ca3cb',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Contacts: [
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T10:27:18.408Z,
      Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
      InitiationMethod: 'API',
      InitiationTimestamp: 2024-11-28T10:27:07.931Z,
      QueueInfo: {}
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T10:27:49.684Z,
      Id: '17e1e61e-f825-4533-a729-b421cfe590fb',
      InitiationMethod: 'API',
      InitiationTimestamp: 2024-11-28T10:27:07.510Z,
      QueueInfo: {}
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:33:54.580Z,
      Id: 'f66b1647-5af0-4402-a0c1-d191a76da356',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:29:15.855Z,
      QueueInfo: {}
    },
    {
      AgentInfo: [Object],
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:29:05.786Z,
      Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:24:55.314Z,
      QueueInfo: [Object]
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:14:58.950Z,
      Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:09:44.135Z,
      QueueInfo: [Object]
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:09:23.956Z,
      Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:09:12.948Z,
      QueueInfo: {}
    }
  ],
  TotalCount: 6
}

2024-11-29T05:32:11.102Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Search Contacts Response: { '$metadata': { httpStatusCode: 200, requestId: 'aebf1a01-2c26-498e-b711-9a6ff78ca3cb', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Contacts: [ { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:18.408Z, Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.931Z, QueueInfo: {} }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:49.684Z, Id: '17e1e61e-f825-4533-a729-b421cfe590fb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.510Z, QueueInfo: {} }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:33:54.580Z, Id: 'f66b1647-5af0-4402-a0c1-d191a76da356', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:29:15.855Z, QueueInfo: {} }, { AgentInfo: [Object], Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:29:05.786Z, Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:24:55.314Z, QueueInfo: [Object] }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:14:58.950Z, Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:44.135Z, QueueInfo: [Object] }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:09:23.956Z, Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:12.948Z, QueueInfo: {} } ], TotalCount: 6 }
2024-11-29T05:32:11.102Z
2024-11-29T05:32:11.102Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T10:27:18.408Z,
  Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
  InitiationMethod: 'API',
  InitiationTimestamp: 2024-11-28T10:27:07.931Z,
  QueueInfo: {}
} b9ad4fa6-b463-4d52-9825-3c9a372ebefb

2024-11-29T05:32:11.102Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:18.408Z, Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.931Z, QueueInfo: {} } b9ad4fa6-b463-4d52-9825-3c9a372ebefb
2024-11-29T05:32:11.205Z
2024-11-29T05:32:11.205Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'b4dab2b7-5313-448b-8d51-b0ba6ac74fab',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {
    agent: 'NotAnswered',
    phoneNumber: '+919949921498',
    CallAgent: 'NotConnected',
    customer: 'NotAnswered'
  }
}

2024-11-29T05:32:11.205Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: 'b4dab2b7-5313-448b-8d51-b0ba6ac74fab', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: { agent: 'NotAnswered', phoneNumber: '+919949921498', CallAgent: 'NotConnected', customer: 'NotAnswered' } }
2024-11-29T05:32:11.205Z
2024-11-29T05:32:11.205Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T10:27:49.684Z,
  Id: '17e1e61e-f825-4533-a729-b421cfe590fb',
  InitiationMethod: 'API',
  InitiationTimestamp: 2024-11-28T10:27:07.510Z,
  QueueInfo: {}
} 17e1e61e-f825-4533-a729-b421cfe590fb

2024-11-29T05:32:11.205Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:49.684Z, Id: '17e1e61e-f825-4533-a729-b421cfe590fb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.510Z, QueueInfo: {} } 17e1e61e-f825-4533-a729-b421cfe590fb
2024-11-29T05:32:11.341Z
2024-11-29T05:32:11.341Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '3ca0a702-a5be-474d-9ff5-2d9509977036',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:11.341Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: '3ca0a702-a5be-474d-9ff5-2d9509977036', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:11.341Z
2024-11-29T05:32:11.341Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:33:54.580Z,
  Id: 'f66b1647-5af0-4402-a0c1-d191a76da356',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:29:15.855Z,
  QueueInfo: {}
} f66b1647-5af0-4402-a0c1-d191a76da356

2024-11-29T05:32:11.341Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:33:54.580Z, Id: 'f66b1647-5af0-4402-a0c1-d191a76da356', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:29:15.855Z, QueueInfo: {} } f66b1647-5af0-4402-a0c1-d191a76da356
2024-11-29T05:32:11.483Z
2024-11-29T05:32:11.483Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'b320bf5d-4f5c-4826-b0f5-dc4725f3d148',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:11.483Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: 'b320bf5d-4f5c-4826-b0f5-dc4725f3d148', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:11.483Z
2024-11-29T05:32:11.483Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {
    ConnectedToAgentTimestamp: 2024-11-28T05:27:49.471Z,
    Id: '9783145b-f6e4-4d42-9414-eb8943c0d9aa'
  },
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:29:05.786Z,
  Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:24:55.314Z,
  QueueInfo: {
    EnqueueTimestamp: 2024-11-28T05:27:33.212Z,
    Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f'
  }
} b139fffc-dc3f-41d1-9496-5a97f44b62fa

2024-11-29T05:32:11.483Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: { ConnectedToAgentTimestamp: 2024-11-28T05:27:49.471Z, Id: '9783145b-f6e4-4d42-9414-eb8943c0d9aa' }, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:29:05.786Z, Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:24:55.314Z, QueueInfo: { EnqueueTimestamp: 2024-11-28T05:27:33.212Z, Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f' } } b139fffc-dc3f-41d1-9496-5a97f44b62fa
2024-11-29T05:32:11.598Z
2024-11-29T05:32:11.598Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '0e9d3809-8ecd-47b2-bb30-9c898d883d92',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:11.598Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: '0e9d3809-8ecd-47b2-bb30-9c898d883d92', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:11.598Z
2024-11-29T05:32:11.598Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:14:58.950Z,
  Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:09:44.135Z,
  QueueInfo: {
    EnqueueTimestamp: 2024-11-28T05:13:20.979Z,
    Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f'
  }
} bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9

2024-11-29T05:32:11.598Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:14:58.950Z, Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:44.135Z, QueueInfo: { EnqueueTimestamp: 2024-11-28T05:13:20.979Z, Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f' } } bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9
2024-11-29T05:32:11.736Z
2024-11-29T05:32:11.736Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'ea052038-49d4-4101-a5f5-7bd3acb7bce6',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:11.736Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: 'ea052038-49d4-4101-a5f5-7bd3acb7bce6', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:11.736Z
2024-11-29T05:32:11.736Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:09:23.956Z,
  Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:09:12.948Z,
  QueueInfo: {}
} 88a1ac71-9b65-479f-b7b0-e610794fabb4

2024-11-29T05:32:11.736Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:09:23.956Z, Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:12.948Z, QueueInfo: {} } 88a1ac71-9b65-479f-b7b0-e610794fabb4
2024-11-29T05:32:11.863Z
2024-11-29T05:32:11.863Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '6e57f567-2e6e-49d9-8845-1a8d477c29fa',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:11.863Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: '6e57f567-2e6e-49d9-8845-1a8d477c29fa', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:11.864Z
2024-11-29T05:32:11.864Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sdsd 2024-11-28T00:00:00.000Z 2024-11-28T23:59:59.999Z

2024-11-29T05:32:11.864Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sdsd 2024-11-28T00:00:00.000Z 2024-11-28T23:59:59.999Z
2024-11-29T05:32:12.416Z
2024-11-29T05:32:12.416Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Search Contacts Response: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'adeb39e2-8b86-45a9-a99b-88ec9f371f2b',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 2,
    totalRetryDelay: 470
  },
  Contacts: [
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T10:27:18.408Z,
      Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
      InitiationMethod: 'API',
      InitiationTimestamp: 2024-11-28T10:27:07.931Z,
      QueueInfo: {}
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T10:27:49.684Z,
      Id: '17e1e61e-f825-4533-a729-b421cfe590fb',
      InitiationMethod: 'API',
      InitiationTimestamp: 2024-11-28T10:27:07.510Z,
      QueueInfo: {}
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:33:54.580Z,
      Id: 'f66b1647-5af0-4402-a0c1-d191a76da356',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:29:15.855Z,
      QueueInfo: {}
    },
    {
      AgentInfo: [Object],
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:29:05.786Z,
      Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:24:55.314Z,
      QueueInfo: [Object]
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:14:58.950Z,
      Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:09:44.135Z,
      QueueInfo: [Object]
    },
    {
      AgentInfo: {},
      Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4',
      Channel: 'VOICE',
      DisconnectTimestamp: 2024-11-28T05:09:23.956Z,
      Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4',
      InitiationMethod: 'INBOUND',
      InitiationTimestamp: 2024-11-28T05:09:12.948Z,
      QueueInfo: {}
    }
  ],
  TotalCount: 6
}

2024-11-29T05:32:12.416Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Search Contacts Response: { '$metadata': { httpStatusCode: 200, requestId: 'adeb39e2-8b86-45a9-a99b-88ec9f371f2b', extendedRequestId: undefined, cfId: undefined, attempts: 2, totalRetryDelay: 470 }, Contacts: [ { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:18.408Z, Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.931Z, QueueInfo: {} }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:49.684Z, Id: '17e1e61e-f825-4533-a729-b421cfe590fb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.510Z, QueueInfo: {} }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:33:54.580Z, Id: 'f66b1647-5af0-4402-a0c1-d191a76da356', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:29:15.855Z, QueueInfo: {} }, { AgentInfo: [Object], Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:29:05.786Z, Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:24:55.314Z, QueueInfo: [Object] }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:14:58.950Z, Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:44.135Z, QueueInfo: [Object] }, { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:09:23.956Z, Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:12.948Z, QueueInfo: {} } ], TotalCount: 6 }
2024-11-29T05:32:12.416Z
2024-11-29T05:32:12.416Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T10:27:18.408Z,
  Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb',
  InitiationMethod: 'API',
  InitiationTimestamp: 2024-11-28T10:27:07.931Z,
  QueueInfo: {}
} b9ad4fa6-b463-4d52-9825-3c9a372ebefb

2024-11-29T05:32:12.416Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b9ad4fa6-b463-4d52-9825-3c9a372ebefb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:18.408Z, Id: 'b9ad4fa6-b463-4d52-9825-3c9a372ebefb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.931Z, QueueInfo: {} } b9ad4fa6-b463-4d52-9825-3c9a372ebefb
2024-11-29T05:32:12.522Z
2024-11-29T05:32:12.522Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'ce71b1f1-287f-4c61-95c5-851b768f8049',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {
    agent: 'NotAnswered',
    phoneNumber: '+919949921498',
    CallAgent: 'NotConnected',
    customer: 'NotAnswered'
  }
}

2024-11-29T05:32:12.522Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: 'ce71b1f1-287f-4c61-95c5-851b768f8049', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: { agent: 'NotAnswered', phoneNumber: '+919949921498', CallAgent: 'NotConnected', customer: 'NotAnswered' } }
2024-11-29T05:32:12.523Z
2024-11-29T05:32:12.523Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T10:27:49.684Z,
  Id: '17e1e61e-f825-4533-a729-b421cfe590fb',
  InitiationMethod: 'API',
  InitiationTimestamp: 2024-11-28T10:27:07.510Z,
  QueueInfo: {}
} 17e1e61e-f825-4533-a729-b421cfe590fb

2024-11-29T05:32:12.523Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/17e1e61e-f825-4533-a729-b421cfe590fb', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T10:27:49.684Z, Id: '17e1e61e-f825-4533-a729-b421cfe590fb', InitiationMethod: 'API', InitiationTimestamp: 2024-11-28T10:27:07.510Z, QueueInfo: {} } 17e1e61e-f825-4533-a729-b421cfe590fb
2024-11-29T05:32:12.631Z
2024-11-29T05:32:12.631Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '7fbe22e8-aaf6-4e00-b8a8-3788fd1e92de',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:12.631Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: '7fbe22e8-aaf6-4e00-b8a8-3788fd1e92de', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:12.632Z
2024-11-29T05:32:12.632Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:33:54.580Z,
  Id: 'f66b1647-5af0-4402-a0c1-d191a76da356',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:29:15.855Z,
  QueueInfo: {}
} f66b1647-5af0-4402-a0c1-d191a76da356

2024-11-29T05:32:12.632Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/f66b1647-5af0-4402-a0c1-d191a76da356', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:33:54.580Z, Id: 'f66b1647-5af0-4402-a0c1-d191a76da356', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:29:15.855Z, QueueInfo: {} } f66b1647-5af0-4402-a0c1-d191a76da356
2024-11-29T05:32:12.771Z
2024-11-29T05:32:12.771Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '17d1f790-dfbf-4fce-a05d-fd36b57e1efc',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:12.771Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: '17d1f790-dfbf-4fce-a05d-fd36b57e1efc', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:12.771Z
2024-11-29T05:32:12.771Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {
    ConnectedToAgentTimestamp: 2024-11-28T05:27:49.471Z,
    Id: '9783145b-f6e4-4d42-9414-eb8943c0d9aa'
  },
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:29:05.786Z,
  Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:24:55.314Z,
  QueueInfo: {
    EnqueueTimestamp: 2024-11-28T05:27:33.212Z,
    Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f'
  }
} b139fffc-dc3f-41d1-9496-5a97f44b62fa

2024-11-29T05:32:12.771Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: { ConnectedToAgentTimestamp: 2024-11-28T05:27:49.471Z, Id: '9783145b-f6e4-4d42-9414-eb8943c0d9aa' }, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/b139fffc-dc3f-41d1-9496-5a97f44b62fa', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:29:05.786Z, Id: 'b139fffc-dc3f-41d1-9496-5a97f44b62fa', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:24:55.314Z, QueueInfo: { EnqueueTimestamp: 2024-11-28T05:27:33.212Z, Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f' } } b139fffc-dc3f-41d1-9496-5a97f44b62fa
2024-11-29T05:32:12.862Z
2024-11-29T05:32:12.862Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'e8bdc941-8ffc-486c-8ce8-8355cb8acae9',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:12.862Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: 'e8bdc941-8ffc-486c-8ce8-8355cb8acae9', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:12.862Z
2024-11-29T05:32:12.862Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:14:58.950Z,
  Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:09:44.135Z,
  QueueInfo: {
    EnqueueTimestamp: 2024-11-28T05:13:20.979Z,
    Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f'
  }
} bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9

2024-11-29T05:32:12.862Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:14:58.950Z, Id: 'bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:44.135Z, QueueInfo: { EnqueueTimestamp: 2024-11-28T05:13:20.979Z, Id: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f' } } bf2ee891-89ee-4a5c-887b-ed7fbaa76fc9
2024-11-29T05:32:12.974Z
2024-11-29T05:32:12.974Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'ef4c6534-523b-41a8-a42b-d1757f13f7ec',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:12.974Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: 'ef4c6534-523b-41a8-a42b-d1757f13f7ec', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:12.974Z
2024-11-29T05:32:12.974Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	sd {
  AgentInfo: {},
  Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4',
  Channel: 'VOICE',
  DisconnectTimestamp: 2024-11-28T05:09:23.956Z,
  Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4',
  InitiationMethod: 'INBOUND',
  InitiationTimestamp: 2024-11-28T05:09:12.948Z,
  QueueInfo: {}
} 88a1ac71-9b65-479f-b7b0-e610794fabb4

2024-11-29T05:32:12.974Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO sd { AgentInfo: {}, Arn: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/contact/88a1ac71-9b65-479f-b7b0-e610794fabb4', Channel: 'VOICE', DisconnectTimestamp: 2024-11-28T05:09:23.956Z, Id: '88a1ac71-9b65-479f-b7b0-e610794fabb4', InitiationMethod: 'INBOUND', InitiationTimestamp: 2024-11-28T05:09:12.948Z, QueueInfo: {} } 88a1ac71-9b65-479f-b7b0-e610794fabb4
2024-11-29T05:32:13.076Z
2024-11-29T05:32:13.076Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Contact Attributes: {
  '$metadata': {
    httpStatusCode: 200,
    requestId: '4a4c883f-36c6-4694-8af2-61dca066d414',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Attributes: {}
}

2024-11-29T05:32:13.076Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Contact Attributes: { '$metadata': { httpStatusCode: 200, requestId: '4a4c883f-36c6-4694-8af2-61dca066d414', extendedRequestId: undefined, cfId: undefined, attempts: 1, totalRetryDelay: 0 }, Attributes: {} }
2024-11-29T05:32:13.076Z
2024-11-29T05:32:13.076Z	02b80bde-b024-4c85-9a88-e779ae076452	INFO	Call Details: []

2024-11-29T05:32:13.076Z 02b80bde-b024-4c85-9a88-e779ae076452 INFO Call Details: []
2024-11-29T05:32:13.078Z
END RequestId: 02b80bde-b024-4c85-9a88-e779ae076452

END RequestId: 02b80bde-b024-4c85-9a88-e779ae076452
2024-11-29T05:32:13.078Z
REPORT RequestId: 02b80bde-b024-4c85-9a88-e779ae076452	Duration: 2188.07 ms	Billed Duration: 2189 ms	Memory Size: 128 MB	Max Memory Used: 94 MB	
