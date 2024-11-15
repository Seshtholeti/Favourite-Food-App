import { ConnectClient, GetCurrentMetricDataCommand } from "@aws-sdk/client-connect";
// Initialize the Connect client
const client = new ConnectClient({ region: 'us-east-1' });
// Helper function to get an array of dates between two dates
function getDateRange(startDate, endDate) {
 const dates = [];
 let currentDate = new Date(startDate);
 while (currentDate <= new Date(endDate)) {
   dates.push(new Date(currentDate).toISOString().split("T")[0]); // Get only the date part (YYYY-MM-DD)
   currentDate.setDate(currentDate.getDate() + 1); // Move to the next day
 }
 return dates;
}
export const handler = async (event) => {
 const { resourceType, selectedResources, startDate, endDate } = event;
 // Validate that selectedResources is not empty
 if (!selectedResources || selectedResources.length === 0) {
   return { error: "You must select at least one resource." };
 }
 const instanceId = process.env.InstanceId;
 // Prepare response structure
 const response = {
   resourceType,
   startDate,
   endDate,
   selectedResources,
   metricsByDate: []
 };
 if (resourceType === "Queues") {
   const queueIds = selectedResources;
   const dateRange = getDateRange(startDate, endDate);
   // Loop through each date in the range
   for (const date of dateRange) {
     const input = {
       InstanceId: instanceId,
       Filters: {
         Queues: queueIds,
         Channels: ["VOICE"],
       },
       Groupings: ["QUEUE"],
       CurrentMetrics: [
         { Name: "AGENTS_ONLINE", Unit: "COUNT" },
         { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
         { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
         { Name: "AGENTS_ERROR", Unit: "COUNT" },
         { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
         { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
       ],
       StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
       EndTime: new Date(`${date}T23:59:59Z`).toISOString(),
     };
     try {
       const command = new GetCurrentMetricDataCommand(input);
       const data = await client.send(command);
       // Process the metrics for each queue on the current date
       const dailyMetrics = {
         date,
         metrics: [],
       };
       if (data.MetricResults && data.MetricResults.length > 0) {
         data.MetricResults.forEach((result) => {
           const queueMetrics = result.Collections.map((collection) => ({
             metricName: collection.Metric.Name,
             metricValue: collection.Value,
           }));
           dailyMetrics.metrics.push({
             queueId: result.Dimensions.Queue.Id,
             queueName: result.Dimensions.Queue.Name || "Unknown Queue",
             metrics: queueMetrics,
           });
         });
       }
       // Append the daily metrics to the response
       response.metricsByDate.push(dailyMetrics);
     } catch (err) {
       console.error(`Error fetching metrics for date ${date}:`, err);
       response.metricsByDate.push({
         date,
         error: err.message,
       });
     }
   }
   // Return the collected response
   return response;
 } else {
   return { error: "Unsupported resource type" };
 }
};
