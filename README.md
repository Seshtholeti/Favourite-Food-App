import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';
import csvParser from 'csv-parser';
import { ConnectClient, GetMetricDataV2Command, GetContactAttributesCommand, ListQueuesCommand } from '@aws-sdk/client-connect';
import { subDays, format } from 'date-fns';

const s3 = new S3Client();
const client = new ConnectClient({ region: 'us-east-1' });

// // Function to fetch all queue IDs
// const fetchAllQueueIds = async () => {
//   const queueIds = [];
//   try {
//     const listQueuesCommand = new ListQueuesCommand({
//       InstanceId: process.env.InstanceId,
//     });
//     const data = await client.send(listQueuesCommand);
//     console.log('Fetched Queues:', data.QueueSummaryList);  // Log queue details for debugging
//     if (data.QueueSummaryList && data.QueueSummaryList.length > 0) {
//       data.QueueSummaryList.forEach(queue => {
//         queueIds.push(queue.Id);
//       });
//     }
//   } catch (err) {
//     console.error('Error fetching queue IDs:', err);
//   }
//   console.log('Queue IDs:', queueIds);  // Log the final array of queue IDs
//   return queueIds;
// };

export const handler = async () => {
  const bucketName = 'customeroutbound-data';
  const fileName = 'CustomerOutboundNumber.csv';
  const instanceId = 'bd16d991-11c8-4d1e-9900-edd5ed4a9b21';
  
  const yesterdayStart = `${format(subDays(new Date(), 1), 'yyyy-MM-dd')}T00:00:00Z`;
  const yesterdayEnd = `${format(subDays(new Date(), 1), 'yyyy-MM-dd')}T23:59:59Z`;

  try {
    // Fetch CSV from S3
    const params = { Bucket: bucketName, Key: fileName };
    const command = new GetObjectCommand(params);
    const response = await s3.send(command);
    const stream = response.Body;

    if (!stream) {
      throw new Error('No stream data found in the S3 object.');
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
          }
        })
        .on('end', resolve)
        .on('error', reject);
    });

    if (phoneNumbers.length === 0) {
      throw new Error('No phone numbers found in the CSV file.');
    }

    // // Fetch metrics from Amazon Connect
    // const queueIds = await fetchAllQueueIds();
    // if (queueIds.length === 0) {
    //   throw new Error('No queues found');
    // }

    
    // // console.log('Queue IDs for Metric Command:',JSON.stringify(queueIds));

    // // Verify that queueIds is a valid array
    // if (!Array.isArray(queueIds) || queueIds.length === 0) {
    //   throw new Error('Invalid or empty Queue IDs');
    // }
    
    const queueId = "f8c742b9-b5ef-4948-8bbf-9a33c892023f";
    
    const RArn = "arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21"

    const metricDataInput = {
      ResourceArn: RArn,
      StartTime: new Date(yesterdayStart),
      EndTime: new Date(yesterdayEnd),
      Interval: { IntervalPeriod: 'DAY' },
      Filters: [{
        FilterKey: "QUEUE",
        FilterValues: [queueId]
      }],
      Groupings: ['QUEUE'],
      Metrics: [
        { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
        { Name: "CONTACTS_ABANDONED", Unit: "COUNT" },
      ],
    };

    // Log metric data input for debugging
    console.log('**** Metric Command ******', JSON.stringify(metricDataInput,null,));

    const metricCommand = new GetMetricDataV2Command(metricDataInput);
    const metricResponse = await client.send(metricCommand);

    const contactDetails = [];

    // Extract and format contact details
    for (const result of metricResponse.MetricResults) {
      console.log("res",metricResponse.MetricResults )
      const agentId = result.Dimensions?.AGENT || 'N/A';
      console.log("agent",agentId)
      const phoneNumber = result.Dimensions?.PhoneNumber || 'N/A';
      const disposition = result.Dimensions?.Disposition || 'Unknown';
      const contactId = "09f2c3c7-c424-4ad2-be1f-246be15b51a4";
      
      console.log('**cID:', contactId)

      for (const collection of result.Collections) {
        if (collection.Metric.Name === 'CONTACTS_HANDLED' && collection.Value > 0) {
          // Fetch contact attributes from Amazon Connect
          const attributesCommand = new GetContactAttributesCommand({
            InstanceId: instanceId,
            InitialContactId: contactId,
            
            
            
          });
          
          console.log('sending GetContactAttributesCommand with contact id', contactId)
          
          

          const attributesResponse = await client.send(attributesCommand);

          contactDetails.push({
            contactId,
            agentId,
            timestamp: new Date().toISOString(),
            outboundPhoneNumber: phoneNumber,
            disposition: disposition || 'Completed',
            attributes: attributesResponse.Attributes || {},
          });
        } else if (collection.Metric.Name === 'CONTACTS_ABANDONED' && collection.Value > 0) {
          contactDetails.push({
            contactId,
            agentId,
            timestamp: new Date().toISOString(),
            outboundPhoneNumber: phoneNumber,
            disposition: 'Abandoned',
            attributes: {},
          });
        }
      }
    }

    return {
      statusCode: 200,
      body: JSON.stringify({
        message: 'Contact details fetched successfully.',
        contactDetails,
      }),
    };
  } catch (error) {
    console.error('Error processing the Lambda function:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message, details: error.stack }),
    };
  }
};





