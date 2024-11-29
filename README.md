import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';
import csvParser from 'csv-parser';
import { ConnectClient, GetMetricDataV2Command, GetContactAttributesCommand } from '@aws-sdk/client-connect';
import { subDays, format } from 'date-fns';

const s3 = new S3Client();
const client = new ConnectClient({ region: 'us-east-1' });

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

    const queueId = "f8c742b9-b5ef-4948-8bbf-9a33c892023f";
    const RArn = "arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21";

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
    console.log('**** Metric Command ******', JSON.stringify(metricDataInput, null, 2));

    const metricCommand = new GetMetricDataV2Command(metricDataInput);
    const metricResponse = await client.send(metricCommand);

    const contactDetails = [];

    // Extract and format contact details
    for (const result of metricResponse.MetricResults) {
      console.log("res", metricResponse.MetricResults);
      const agentId = result.Dimensions?.AGENT || 'N/A';
      const phoneNumber = result.Dimensions?.PhoneNumber || 'N/A';
      const disposition = result.Dimensions?.Disposition || 'Unknown';
      const contactId = result.Dimensions?.ContactId || 'N/A'; // Extract the contact ID from the metric results

      console.log('**cID:', contactId);

      for (const collection of result.Collections) {
        if (collection.Metric.Name === 'CONTACTS_HANDLED' && collection.Value > 0) {
          // Fetch contact attributes using GetContactAttributesCommand
          const attributesCommand = new GetContactAttributesCommand({
            InstanceId: instanceId,
            InitialContactId: contactId,
          });

          const attributesResponse = await client.send(attributesCommand);

          // Check if the call was answered, and extract the outbound phone number if available
          const outboundPhoneNumber = attributesResponse.Attributes?.['OutboundPhoneNumber'] || 'N/A';
          const callAnswered = attributesResponse.Attributes?.['CallAnswered'] === 'true' ? 'Answered' : 'Not Answered';

          contactDetails.push({
            contactId,
            agentId,
            timestamp: new Date().toISOString(),
            outboundPhoneNumber,
            disposition: disposition || 'Completed',
            callStatus: callAnswered, // New field to indicate call status
            attributes: attributesResponse.Attributes || {},
          });
        } else if (collection.Metric.Name === 'CONTACTS_ABANDONED' && collection.Value > 0) {
          contactDetails.push({
            contactId,
            agentId,
            timestamp: new Date().toISOString(),
            outboundPhoneNumber: phoneNumber,
            disposition: 'Abandoned',
            callStatus: 'Not Answered', // Since it's abandoned, the call wasn't answered
            attributes: {},  // Removed contact attributes section
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