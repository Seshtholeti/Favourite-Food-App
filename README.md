import { S3Client, GetObjectCommand } from '@aws-sdk/client-s3';
import csvParser from 'csv-parser';
import { ConnectClient, GetMetricDataV2Command } from '@aws-sdk/client-connect';
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
      console.log("agent", agentId);
      const phoneNumber = result.Dimensions?.PhoneNumber || 'N/A';
      const disposition = result.Dimensions?.Disposition || 'Unknown';
      const contactId = "09f2c3c7-c424-4ad2-be1f-246be15b51a4";

      console.log('**cID:', contactId);

      for (const collection of result.Collections) {
        if (collection.Metric.Name === 'CONTACTS_HANDLED' && collection.Value > 0) {
          contactDetails.push({
            contactId,
            agentId,
            timestamp: new Date().toISOString(),
            outboundPhoneNumber: phoneNumber,
            disposition: disposition || 'Completed',
            attributes: {},  // Removed contact attributes section
          });
        } else if (collection.Metric.Name === 'CONTACTS_ABANDONED' && collection.Value > 0) {
          contactDetails.push({
            contactId,
            agentId,
            timestamp: new Date().toISOString(),
            outboundPhoneNumber: phoneNumber,
            disposition: 'Abandoned',
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


Test Event Name
MyEventName

Response
{
  "statusCode": 200,
  "body": "{\"message\":\"Contact details fetched successfully.\",\"contactDetails\":[{\"contactId\":\"09f2c3c7-c424-4ad2-be1f-246be15b51a4\",\"agentId\":\"N/A\",\"timestamp\":\"2024-11-29T01:27:28.208Z\",\"outboundPhoneNumber\":\"N/A\",\"disposition\":\"Unknown\",\"attributes\":{}},{\"contactId\":\"09f2c3c7-c424-4ad2-be1f-246be15b51a4\",\"agentId\":\"N/A\",\"timestamp\":\"2024-11-29T01:27:28.208Z\",\"outboundPhoneNumber\":\"N/A\",\"disposition\":\"Abandoned\",\"attributes\":{}}]}"
}

Function Logs
START RequestId: fa1e31e8-d8c8-4934-bebe-22b49c820410 Version: $LATEST
2024-11-29T01:27:27.828Z	fa1e31e8-d8c8-4934-bebe-22b49c820410	INFO	**** Metric Command ****** {
  "ResourceArn": "arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21",
  "StartTime": "2024-11-28T00:00:00.000Z",
  "EndTime": "2024-11-28T23:59:59.000Z",
  "Interval": {
    "IntervalPeriod": "DAY"
  },
  "Filters": [
    {
      "FilterKey": "QUEUE",
      "FilterValues": [
        "f8c742b9-b5ef-4948-8bbf-9a33c892023f"
      ]
    }
  ],
  "Groupings": [
    "QUEUE"
  ],
  "Metrics": [
    {
      "Name": "CONTACTS_HANDLED",
      "Unit": "COUNT"
    },
    {
      "Name": "CONTACTS_ABANDONED",
      "Unit": "COUNT"
    }
  ]
}
2024-11-29T01:27:28.170Z	fa1e31e8-d8c8-4934-bebe-22b49c820410	INFO	res [
  {
    Collections: [ [Object], [Object] ],
    Dimensions: {
      QUEUE: 'f8c742b9-b5ef-4948-8bbf-9a33c892023f',
      QUEUE_ARN: 'arn:aws:connect:us-east-1:768637739934:instance/bd16d991-11c8-4d1e-9900-edd5ed4a9b21/queue/f8c742b9-b5ef-4948-8bbf-9a33c892023f'
    },
    MetricInterval: {
      EndTime: 2024-11-28T23:59:59.000Z,
      Interval: 'DAY',
      StartTime: 2024-11-28T00:00:00.000Z
    }
  }
]
2024-11-29T01:27:28.208Z	fa1e31e8-d8c8-4934-bebe-22b49c820410	INFO	agent N/A
2024-11-29T01:27:28.208Z	fa1e31e8-d8c8-4934-bebe-22b49c820410	INFO	**cID: 09f2c3c7-c424-4ad2-be1f-246be15b51a4
END RequestId: fa1e31e8-d8c8-4934-bebe-22b49c820410
REPORT RequestId: fa1e31e8-d8c8-4934-bebe-22b49c820410	Duration: 1848.35 ms	Billed Duration: 1849 ms	Memory Size: 128 MB	Max Memory Used: 105 MB	Init Duration: 730.01 ms

Request ID
fa1e31e8-d8c8-4934-bebe-22b49c820410
