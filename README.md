import { S3Client, GetObjectCommand } from "@aws-sdk/client-s3";
import csvParser from "csv-parser";
import { ConnectClient, GetMetricDataV2Command, GetContactAttributesCommand } from "@aws-sdk/client-connect";
import { subDays, format } from "date-fns";

const s3 = new S3Client();
const client = new ConnectClient({ region: "us-east-1" });

export const handler = async () => {
  const bucketName = "customeroutbound-data"; // Your S3 bucket name
  const fileName = "CustomerOutboundNumber.csv"; // CSV file name
  const instanceId = "bd16d991-11c8-4d1e-9900-edd5ed4a9b21"; // Amazon Connect instance ID
  
  // Date range for the last day
  const yesterdayStart = `${format(subDays(new Date(), 1), 'yyyy-MM-dd')}T00:00:00Z`;
  const yesterdayEnd = `${format(subDays(new Date(), 1), 'yyyy-MM-dd')}T23:59:59Z`;

  try {
    // Fetch phone numbers from S3 CSV file
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
            let formattedNumber = phoneNumber.replace(/\D/g, ''); // Remove non-digit characters
            if (formattedNumber.length === 10) {
              formattedNumber = `+91${formattedNumber}`; // Add country code for India
            } else if (formattedNumber.length === 11) {
              formattedNumber = `+1${formattedNumber}`; // Add country code for USA
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

    // Define the Amazon Connect instance ARN and queue ID
    const RArn = "arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21";
    const queueId = "f8c742b9-b5ef-4948-8bbf-9a33c892023f"; // Replace with actual queue ID

    // Prepare metric data request
    const metricDataInput = {
      ResourceArn: RArn,
      StartTime: new Date(yesterdayStart),
      EndTime: new Date(yesterdayEnd),
      Interval: { IntervalPeriod: 'DAY' },
      Filters: [{
        FilterKey: "QUEUE",
        FilterValues: [queueId],
      }],
      Groupings: ['QUEUE'],
      Metrics: [
        { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
        { Name: "CONTACTS_ABANDONED", Unit: "COUNT" },
      ],
    };

    // Fetch metric data from Amazon Connect
    const metricCommand = new GetMetricDataV2Command(metricDataInput);
    const metricResponse = await client.send(metricCommand);

    const contactDetails = [];

    // Extract and format contact details from metric results
    for (const result of metricResponse.MetricResults) {
      const agentId = result.Dimensions?.AGENT || 'N/A';
      const phoneNumber = result.Dimensions?.PhoneNumber || 'N/A';
      const disposition = result.Dimensions?.Disposition || 'Unknown';
      const contactId = "09f2c3c7-c424-4ad2-be1f-246be15b51a4"; // Replace with actual contact ID if available

      for (const collection of result.Collections) {
        if (collection.Metric.Name === 'CONTACTS_HANDLED' && collection.Value > 0) {
          // Fetch contact attributes for handled contacts
          const attributesCommand = new GetContactAttributesCommand({
            InstanceId: instanceId,
            InitialContactId: contactId,
          });

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
          // Handle abandoned contacts
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

this is the error.

Response
{
  "statusCode": 500,
  "body": "{\"error\":\"Resource not found\",\"details\":\"ResourceNotFoundException: Resource not found\\n    at de_ResourceNotFoundExceptionRes (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:10072:21)\\n    at de_CommandError (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:9740:19)\\n    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)\\n    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-serde/dist-cjs/index.js:35:20\\n    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/core/dist-cjs/index.js:165:18\\n    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-retry/dist-cjs/index.js:320:38\\n    at async /var/runtime/node_modules/@aws-sdk/middleware-logger/dist-cjs/index.js:34:22\\n    at async Runtime.handler (file:///var/task/index.mjs:278:38)\"}"
}

Function Logs
START RequestId: a8b41ef8-6d7f-427f-be82-32c23ccc7bce Version: $LATEST
2024-11-29T01:09:20.645Z	a8b41ef8-6d7f-427f-be82-32c23ccc7bce	ERROR	Error processing the Lambda function: ResourceNotFoundException: Resource not found
    at de_ResourceNotFoundExceptionRes (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:10072:21)
    at de_CommandError (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:9740:19)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-serde/dist-cjs/index.js:35:20
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/core/dist-cjs/index.js:165:18
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-retry/dist-cjs/index.js:320:38
    at async /var/runtime/node_modules/@aws-sdk/middleware-logger/dist-cjs/index.js:34:22
    at async Runtime.handler (file:///var/task/index.mjs:278:38) {
  '$fault': 'client',
  '$metadata': {
    httpStatusCode: 404,
    requestId: '1c4dc4c8-df27-4759-9ecd-4fb1cf038205',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  }
}
