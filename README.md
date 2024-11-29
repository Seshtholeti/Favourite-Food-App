import AWS from 'aws-sdk';
import csv from 'csv-parser';
import { Readable } from 'stream';

const bucketName = 'customeroutbound-data';
const fileName = 'CustomerOutboundNumber.csv';
const instanceId = 'bd16d991-11c8-4d1e-9900-edd5ed4a9b21';
const contactFlowId = '09f2c3c7-c424-4ad2-be1f-246be15b51a4';

const s3 = new AWS.S3();
const connect = new AWS.Connect();

export const handler = async (event) => {
    try {
        // Fetch CSV from S3
        const csvData = await s3.getObject({ Bucket: bucketName, Key: fileName }).promise();
        const records = [];

        // Parse CSV data
        const stream = Readable.from(csvData.Body);
        await new Promise((resolve, reject) => {
            stream.pipe(csv())
                .on('data', (data) => records.push(data))
                .on('end', resolve)
                .on('error', reject);
        });

        // Get yesterday's date
        const yesterday = new Date();
        yesterday.setDate(yesterday.getDate() - 1);
        const startTime = new Date(yesterday.setHours(0, 0, 0)).toISOString();
        const endTime = new Date(yesterday.setHours(23, 59, 59)).toISOString();

        // Process each phone number
        for (const record of records) {
            const phoneNumber = record.phone; // Adjust based on your CSV structure

            // Fetch call details from Amazon Connect
            const params = {
                InstanceId: instanceId,
                Filters: {
                    StartTime: startTime,
                    EndTime: endTime,
                    // Add any other necessary filters here
                },
            };

            const response = await connect.getMetricData(params).promise();

            // Print call details for each phone number
            console.log(`Phone: ${phoneNumber}`);
            console.log('Call Details:', response);
            
            // You can further process the response to extract specific details if needed
            response.MetricResults.forEach(metric => {
                console.log(`Agent ID: ${metric.AgentId}, Contact Answered: ${metric.ContactAnswered}`);
            });
        }
    } catch (error) {
        console.error('Error processing the Lambda function:', error);
    }
};
