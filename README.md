import { ConnectClient, ListUsersCommand, GetCurrentMetricDataCommand } from "@aws-sdk/client-connect";
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
// Main handler function
export const handler = async (event) => {
 const { resourceType, selectedResources, startDate, endDate } = event;
 // Validate that selectedResources is not empty
 if (!selectedResources || selectedResources.length === 0) {
   return { error: "You must select at least one resource." };
 }
 const instanceId = process.env.InstanceId; // Your Amazon Connect instance ID
 // Prepare response structure
 const response = {
   resourceType,
   startDate,
   endDate,
   selectedResources,
   metricsByDate: []
 };
 // Handle metrics for Agents
 if (resourceType === "Agents") {
   try {
     // Step 1: List all agents
     const listUsersCommand = new ListUsersCommand({ InstanceId: instanceId });
     const usersResponse = await client.send(listUsersCommand);
     const agents = usersResponse.UserSummaryList;
     // Filter agents based on selectedResources if needed
     const filteredAgents = agents.filter(agent => selectedResources.includes(agent.UserId));
     console.log(filteredAgents)
     const dateRange = getDateRange(startDate, endDate);
     // Loop through each date in the range for Agents
     for (const date of dateRange) {
      const queue1 = selectedResources
       const input = {
        
         InstanceId: instanceId,
         Filters: {
           Agents: filteredAgents.map(agent => agent.Arn),
           Queues:queue1,// Use the ARNs of filtered agents
           Channels: ["VOICE"], // Specify channels as needed
         },
         Groupings: ["QUEUE","AGENTS"], 
         CurrentMetrics: [
           { Name: "AGENTS_ONLINE", Unit: "COUNT" },
           { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
           { Name: "CONTACTS_HANDLED", Unit: "COUNT" },
         ],
         StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
         EndTime: new Date(`${date}T23:59:59Z`).toISOString(),
       };
       try {
         const command = new GetCurrentMetricDataCommand(input);
         const data = await client.send(command);
         // Process the metrics for each agent on the current date
         const dailyMetrics = {
           date,
           type: 'Agent',
           metrics: [],
         };
         if (data.MetricResults && data.MetricResults.length > 0) {
           data.MetricResults.forEach((result) => {
             const agentMetrics = result.Collections.map((collection) => ({
               metricName: collection.Metric.Name,
               metricValue: collection.Value,
             }));
             dailyMetrics.metrics.push({
               routingProfileId: result.Dimensions.RoutingProfile.Id || "Unknown Profile",
               queueId: result.Dimensions.Queue.Id || "Unknown Queue",
               metrics: agentMetrics,
             });
           });
         }
         // Append the daily metrics to the response
         response.metricsByDate.push(dailyMetrics);
       } catch (err) {
         console.error(`Error fetching metrics for date ${date}:`, err);
         response.metricsByDate.push({
           date,
           errorMessage: err.message,
         });
       }
     }
   } catch (error) {
     console.error("Error listing users:", error);
     return {
       statusCode: 500,
       body: JSON.stringify({ error: error.message }),
     };
   }
 // Handle metrics for Queues
 } else if (resourceType === "Queues") {
   const queueIds = selectedResources; // Assuming selectedResources contains queue IDs
   const dateRange = getDateRange(startDate, endDate);
   // Loop through each date in the range for Queues
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
         type: 'Queue',
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
         errorMessage : err.message,
       });
     }
   }
 } else {
   return { error : "Unsupported resource type" };
 }
 return response;
}
