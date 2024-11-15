
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { ConnectClient, ListUsersCommand, GetCurrentMetricDataCommand } from "@aws-sdk/client-connect";
// Create an S3 client
const s3Client = new S3Client({ region: 'us-east-1' });
// The Connect client
const client = new ConnectClient({ region: 'us-east-1' });
// Function to convert data to CSV format
function convertToCSV(data) {
 const header = Object.keys(data[0]).join(',') + '\n';
 const rows = data.map(row => Object.values(row).join(',')).join('\n');
 return header + rows;
}
// Function to convert data to JSON format
function convertToJSON(data) {
 return JSON.stringify(data, null, 2);
}
// Function to upload file to S3
async function uploadToS3(fileContent, bucketName, fileName) {
 const command = new PutObjectCommand({
   Bucket: bucketName,
   Key: fileName,
   Body: fileContent,
   ContentType: fileName.endsWith('.csv') ? 'text/csv' : 'application/json', // Adjust content type based on the format
 });
 await s3Client.send(command);
}
// Function to get date range
function getDateRange(startDate, endDate) {
 const dates = [];
 let currentDate = new Date(startDate);
 while (currentDate <= new Date(endDate)) {
   dates.push(new Date(currentDate).toISOString().split("T")[0]);
   currentDate.setDate(currentDate.getDate() + 1);
 }
 return dates;
}
// Function to aggregate metrics
function aggregateMetrics(dailyMetrics) {
 const aggregated = {};
 dailyMetrics.forEach((day) => {
   day.metrics.forEach((metric) => {
     metric.metrics.forEach(({ metricName, metricValue }) => {
       if (!aggregated[metricName]) {
         aggregated[metricName] = 0;
       }
       aggregated[metricName] += metricValue; // Sum up the values
     });
   });
 });
 return Object.entries(aggregated).map(([metricName, metricValue]) => ({
   metricName,
   metricValue,
 }));
}
// Function to fetch metrics
const fetchMetrics = async (filters, groupings, metrics, type, dateRange) => {
 const dailyMetrics = [];
 for (const date of dateRange) {
   const input = {
     InstanceId: process.env.InstanceId,
     Filters: filters,
     Groupings: groupings,
     CurrentMetrics: metrics,
     StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
     EndTime: new Date(`${date}T23:59:59Z`).toISOString(),
   };
   try {
     const command = new GetCurrentMetricDataCommand(input);
     const data = await client.send(command);
     const metricsForDate = {
       date,
       type,
       metrics: [],
     };
     if (data.MetricResults && data.MetricResults.length > 0) {
       data.MetricResults.forEach((result) => {
         const resultMetrics = result.Collections.map((collection) => ({
           metricName: collection.Metric.Name,
           metricValue: collection.Value,
         }));
         metricsForDate.metrics.push({
           queueId: result.Dimensions.Queue.Id || "Unknown Queue",
           queueName: result.Dimensions.Queue.Name || "Unknown Queue",
           metrics: resultMetrics,
         });
       });
     }
     dailyMetrics.push(metricsForDate);
   } catch (err) {
     console.error(`Error fetching metrics for date ${date}:`, err);
     dailyMetrics.push({
       date,
       errorMessage: err.message,
     });
   }
 }
 return dailyMetrics;
};
// Main handler function
export const handler = async (event) => {
 const { resourceType, selectedResources, startDate, endDate, isCumulativeReport, format } = event;
 const instanceId = process.env.InstanceId;
 const bucketName = process.env.S3_BUCKET_NAME; // Fetch the bucket name from environment variables
 if (!bucketName) {
   return { statusCode: 500, body: JSON.stringify({ error: "S3 bucket name is not set in environment variables." }) };
 }
 const response = {
   resourceType,
   startDate,
   endDate,
   selectedResources,
   metricsByDate: [],
   cumulativeMetrics: {},
 };
 // Validate that selectedResources is not empty
 if (!selectedResources || selectedResources.length === 0) {
   return { error: "You must select at least one resource." };
 }
 const dateRange = getDateRange(startDate, endDate);
 // Handle Cumulative Report logic
 if (isCumulativeReport) {
   // Cumulative report for Agents
   if (resourceType === "Agents") {
     try {
       const listUsersCommand = new ListUsersCommand({ InstanceId: instanceId });
       const usersResponse = await client.send(listUsersCommand);
       const agents = usersResponse.UserSummaryList;
       const filteredAgents = agents.filter((agent) => selectedResources.includes(agent.UserId));
       const filters = {
         ROUTING_PROFILE: 
         Queues: selectedResources,
         Channels: ["VOICE"],
       };
       const groupings = ["QUEUE", "ROUTING_PROFILE"];
       const metrics = [
         { Name: "AGENTS_ONLINE", Unit: "COUNT" },
         { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
         { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
       ];
       const dailyMetrics = await fetchMetrics(filters, groupings, metrics, "Agent", dateRange);
       response.cumulativeMetrics = aggregateMetrics(dailyMetrics);
       // Convert the report to the chosen format (CSV or JSON)
       let fileContent;
       if (format === 'csv') {
         fileContent = convertToCSV(response.cumulativeMetrics);
       } else {
         fileContent = convertToJSON(response.cumulativeMetrics);
       }
       // Generate a unique file name
       const fileName = `cumulative-report-${Date.now()}.${format}`;
       // Upload the report to S3
       await uploadToS3(fileContent, bucketName, fileName);
       // Generate the S3 URL for accessing the file
       const fileUrl = `https://s3.amazonaws.com/${bucketName}/${fileName}`;
       // Log the result to the console
       console.log('File uploaded successfully.');
       console.log(`File URL: ${fileUrl}`);
       // Return the S3 URL in the response
       return {
         statusCode: 200,
         body: JSON.stringify({
           message: "File uploaded successfully.",
           fileUrl: fileUrl,
         }),
       };
     } catch (error) {
       return { statusCode: 500, body: JSON.stringify({ error: error.message }) };
     }
   }
   // Cumulative report for Queues
   else if (resourceType === "Queues") {
     const filters = {
       Queues: selectedResources,
       Channels: ["VOICE"],
     };
     const groupings = ["QUEUE"];
     const metrics = [
       { Name: "AGENTS_ONLINE", Unit: "COUNT" },
       { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
       { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
       { Name: "AGENTS_ERROR", Unit: "COUNT" },
       { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
       { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
     ];
     const dailyMetrics = await fetchMetrics(filters, groupings, metrics, "Queue", dateRange);
     response.cumulativeMetrics = aggregateMetrics(dailyMetrics);
     // Convert the report to the chosen format (CSV or JSON)
     let fileContent;
     if (format === 'csv') {
       fileContent = convertToCSV(response.cumulativeMetrics);
     } else {
       fileContent = convertToJSON(response.cumulativeMetrics);
     }
     // Generate a unique file name
     const fileName = `cumulative-report-${Date.now()}.${format}`;
     // Upload the report to S3
     await uploadToS3(fileContent, bucketName, fileName);
     // Generate the S3 URL for accessing the file
     const fileUrl = `https://s3.amazonaws.com/${bucketName}/${fileName}`;
     // Log the result to the console
     console.log('File uploaded successfully.');
     console.log(`File URL: ${fileUrl}`);
     // Return the S3 URL in the response
     return {
       statusCode: 200,
       body: JSON.stringify({
         message: "File uploaded successfully.",
         fileUrl: fileUrl,
       }),
     };
   }
 } else {
   // **Daily report logic**
   if (resourceType === "Agents") {
     try {
       const listUsersCommand = new ListUsersCommand({ InstanceId: instanceId });
       const usersResponse = await client.send(listUsersCommand);
       const agents = usersResponse.UserSummaryList;
       const filteredAgents = agents.filter((agent) => selectedResources.includes(agent.UserId));
       const filters = {
         Agents: filteredAgents.map((agent) => agent.Arn),
         Queues: selectedResources,
         Channels: ["VOICE"],
       };
       const groupings = ["QUEUE", "AGENTS"];
       const metrics = [
         { Name: "AGENTS_ONLINE", Unit: "COUNT" },
         { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
         { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
       ];
       const dailyMetrics = await fetchMetrics(filters, groupings, metrics, "Agent", dateRange);
       response.metricsByDate = dailyMetrics;
       // Convert the report to the chosen format (CSV or JSON)
       let fileContent;
       if (format === 'csv') {
         fileContent = convertToCSV(dailyMetrics);
       } else {
         fileContent = convertToJSON(dailyMetrics);
       }
       // Generate a unique file name
       const fileName = `daily-report-${Date.now()}.${format}`;
       // Upload the report to S3
       await uploadToS3(fileContent, bucketName, fileName);
       // Generate the S3 URL for accessing the file
       const fileUrl = `https://s3.amazonaws.com/${bucketName}/${fileName}`;
       // Log the result to the console
       console.log('File uploaded successfully.');
       console.log(`File URL: ${fileUrl}`);
       // Return the S3 URL in the response
       return {
         statusCode: 200,
         body: JSON.stringify({
           message: "File uploaded successfully.",
           fileUrl: fileUrl,
         }),
       };
     } catch (error) {
       return { statusCode: 500, body: JSON.stringify({ error: error.message }) };
     }
   } else if (resourceType === "Queues") {
     const filters = {
       Queues: selectedResources,
       Channels: ["VOICE"],
     };
     const groupings = ["QUEUE"];
     const metrics = [
       { Name: "AGENTS_ONLINE", Unit: "COUNT" },
       { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
       { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
       { Name: "AGENTS_ERROR", Unit: "COUNT" },
       { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
       { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
     ];
     const dailyMetrics = await fetchMetrics(filters, groupings, metrics, "Queue", dateRange);
     response.metricsByDate = dailyMetrics;
     // Convert the report to the chosen format (CSV or JSON)
     let fileContent;
     if (format === 'csv') {
       fileContent = convertToCSV(dailyMetrics);
     } else {
       fileContent = convertToJSON(dailyMetrics);
     }
     // Generate a unique file name
     const fileName = `daily-report-${Date.now()}.${format}`;
     // Upload the report to S3
     await uploadToS3(fileContent, bucketName, fileName);
     // Generate the S3 URL for accessing the file
     const fileUrl = `https://s3.amazonaws.com/${bucketName}/${fileName}`;
     // Log the result to the console
     console.log('File uploaded successfully.');
     console.log(`File URL: ${fileUrl}`);
     // Return the S3 URL in the response
     return {
       statusCode: 200,
       body: JSON.stringify({
         message: "File uploaded successfully.",
         fileUrl: fileUrl,
       }),
     };
   }
 }
};

use list routingprofiles using this
import { ConnectClient, ListRoutingProfilesCommand } from "@aws-sdk/client-connect"; // ES Modules import
// const { ConnectClient, ListRoutingProfilesCommand } = require("@aws-sdk/client-connect"); // CommonJS import
const client = new ConnectClient(config);
const input = { // ListRoutingProfilesRequest
  InstanceId: "STRING_VALUE", // required
  NextToken: "STRING_VALUE",
  MaxResults: Number("int"),
};
const command = new ListRoutingProfilesCommand(input);
const response = await client.send(command);
// { // ListRoutingProfilesResponse
//   RoutingProfileSummaryList: [ // RoutingProfileSummaryList
//     { // RoutingProfileSummary
//       Id: "STRING_VALUE",
//       Arn: "STRING_VALUE",
//       Name: "STRING_VALUE",
//       LastModifiedTime: new Date("TIMESTAMP"),
//       LastModifiedRegion: "STRING_VALUE",
//     },
//   ],
//   NextToken: "STRING_VALUE",
// };

store the routing profile ids in an array same like queues and assign it to the Routing profiles in agent metrics
