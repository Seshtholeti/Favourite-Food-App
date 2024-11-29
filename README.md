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
    // Fetch the CSV file from S3
    const params = { Bucket: bucketName, Key: fileName };
    const command = new GetObjectCommand(params);
    const response = await s3.send(command);
    const stream = response.Body;
    if (!stream) {
      throw new Error("No stream data found in the S3 object.");
    }

    // Parse phone numbers from the CSV
    const phoneNumbers = [];
    await new Promise((resolve, reject) => {
      stream
        .pipe(csvParser({ separator: ';' }))
        .on('data', (row) => {
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
          }
        })
        .on('end', resolve)
        .on('error', reject);
    });

    if (phoneNumbers.length === 0) {
      throw new Error('No phone numbers found in the CSV file.');
    }

    // Get yesterday's date to filter records from Amazon Connect
    const yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1); // Subtract one day to get yesterday's date
    const startDate = new Date(yesterday.setHours(0, 0, 0, 0)).toISOString(); // Start of yesterday
    const endDate = new Date(yesterday.setHours(23, 59, 59, 999)).toISOString(); // End of yesterday

    // Log the details for each phone number
    const callDetails = [];

    // Loop through the phone numbers and check Amazon Connect records
    for (const phoneNumber of phoneNumbers) {
      try {
        // Search contacts from Amazon Connect for yesterday's date
        const searchContactsParams = {
          InstanceId: instanceId,
          StartTime: startDate,
          EndTime: endDate,
          MaxResults: 100, // Adjust as necessary
        };
        const contactResponse = await connect.send(new SearchContactsCommand(searchContactsParams));

        // Filter to find the specific record that matches the phone number
        const contactRecord = contactResponse.Contacts.find(contact => contact.CustomerEndpoint.Address === phoneNumber);

        if (contactRecord) {
          // Get additional attributes like agent ID, disposition
          const contactAttributesParams = {
            InstanceId: instanceId,
            ContactId: contactRecord.ContactId,
          };
          const attributesResponse = await connect.send(new GetContactAttributesCommand(contactAttributesParams));
          
          // Extract agent ID and disposition
          const agentId = attributesResponse.Attributes['AgentId'];
          const disposition = attributesResponse.Attributes['Disposition'] || 'Not Answered';

          // Log the result
          callDetails.push({
            phoneNumber,
            agentId,
            disposition,
          });

        } else {
          callDetails.push({
            phoneNumber,
            agentId: 'N/A',
            disposition: 'No Call Record Found',
          });
        }
      } catch (error) {
        console.error(`Error processing phone number ${phoneNumber}:`, error);
        callDetails.push({
          phoneNumber,
          agentId: 'Error',
          disposition: 'Error Fetching Record',
        });
      }
    }

    console.log('Call Details:', callDetails);

    // Return a success response
    return {
      statusCode: 200,
      body: JSON.stringify({ message: 'Process completed successfully', details: callDetails }),
    };
  } catch (error) {
    console.error('Error processing the Lambda function:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message, details: error.stack }),
    };
  }
};