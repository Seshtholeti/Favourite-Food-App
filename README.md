import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';
import csvParser from 'csv-parser';
import { ConnectClient, StartOutboundVoiceContactCommand } from '@aws-sdk/client-connect';
const s3 = new S3Client();
const connect = new ConnectClient();
export const handler = async (event) => {
 const bucketName = 'customeroutbound-data';
 const fileName = 'CustomerOutboundNumber.csv';
 const contactFlowId = '09f2c3c7-c424-4ad2-be1f-246be15b51a4';
 const instanceId = 'bd16d991-11c8-4d1e-9900-edd5ed4a9b21';
 const queueId = 'f8c742b9-b5ef-4948-8bbf-9a33c892023f';
 try {
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
         console.log('Row parsed:', row);
         const phoneNumber = row.PhoneNumber || row['Name;PhoneNumber']?.split(';')[1]?.trim();
         if (phoneNumber) {
           // Format phone number to E.164 format
           let formattedNumber = phoneNumber.replace(/\D/g, ''); 
           if (formattedNumber.length === 10) {
             formattedNumber = `+91${formattedNumber}`; 
           } else if (formattedNumber.length === 11) {
             formattedNumber = `+1${formattedNumber}`; 
           }
           phoneNumbers.push(formattedNumber);
           console.log('PhoneNumber formatted:', formattedNumber);
         } else {
           console.log('PhoneNumber not found in row:', row);
         }
       })
       .on('end', resolve)
       .on('error', reject);
   });
   if (phoneNumbers.length === 0) {
     throw new Error('No phone numbers found in the CSV file.');
   }
   console.log('Phone numbers parsed and formatted:', phoneNumbers);
   // Initiate outbound calls for each phone number
   for (const destinationPhoneNumber of phoneNumbers) {
     const input = {
       DestinationPhoneNumber: destinationPhoneNumber,
       ContactFlowId: contactFlowId,
       InstanceId: instanceId,
       QueueId: queueId,
     };
     const voiceCommand = new StartOutboundVoiceContactCommand(input);
     const callResponse = await connect.send(voiceCommand);
     console.log(`Call initiated for ${destinationPhoneNumber}:`, callResponse);
   }
   return {
     statusCode: 200,
     body: JSON.stringify({ message: 'Outbound calls initiated successfully' }),
   };
 } catch (error) {
   console.error('Error processing the Lambda function:', error);
   return {
     statusCode: 500,
     body: JSON.stringify({ error: error.message, details: error.stack }),
   };
 }
};


this the lambda code which initiated a outbound call to a phone number.

now we have to develope a lambda which will trigger everyday morning from the csv file it will fetch the customer phone numbers , it will take the yesterday records from the amazon connectnd check whether the call  has been answered or not.  after everything print the details like, to which number outbound call has been initiated, the agent id who has handled it and disposition whether answered or nit
