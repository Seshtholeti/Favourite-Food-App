import { ConnectClient, ListQueuesCommand, ListUsersCommand, GetCurrentMetricDataCommand, GetMetricDataV2Command } from "@aws-sdk/client-connect";
import { DynamoDBClient, PutItemCommand } from "@aws-sdk/client-dynamodb";
const connectClient = new ConnectClient({ region: 'us-east-1' });
const dynamoDbClient = new DynamoDBClient({ region: 'us-east-1' });
const TABLE_NAME = 'MetricData';
// Function to fetch queues dynamically using list queues
async function getQueues() {
   const input = {
       InstanceId: process.env.InstanceId,
   };
   try {
       const command = new ListQueuesCommand(input);
       const data = await connectClient.send(command);
       const queueIds = data.QueueSummaryList.map(queue => queue.Id);
       console.log("Fetched Queues:", queueIds);
       return queueIds;
   } catch (err) {
       console.error("Error fetching queues:", err);
       throw err;
   }
}
// Function to fetch agents dynamically using ListUsers
async function getAgents() {
   const input = { InstanceId: process.env.InstanceId };
   try {
       const command = new ListUsersCommand(input);
       const data = await connectClient.send(command);
       const agentIds = data.UserSummaryList.map(user => user.Id);
       console.log("Fetched Agents:", agentIds);
       return agentIds.slice(0, 10); // Limiting to a maximum of 10 agents
   } catch (err) {
       console.error("Error fetching agents:", err);
       throw err;
   }
}
// Fetch real-time metrics for each day over one month
async function getCurrentMetrics() {
   const currentTime = new Date();
   const endDate = new Date(currentTime);
   const startDate = new Date();
   startDate.setMonth(startDate.getMonth() - 1);
   console.log("Fetching daily real-time metrics from:", startDate.toISOString(), "to", endDate.toISOString());
   const queues = await getQueues();
   const agents = await getAgents();
   const dailyRealTimeMetrics = [];
   // Loop through each day in the range
   for (let date = new Date(startDate); date <= endDate; date.setDate(date.getDate() + 1)) {
       const startTime = new Date(date);
       const endTime = new Date(date);
       endTime.setHours(23, 59, 59, 999);
       const input = {
           InstanceId: process.env.InstanceId,
           Filters: {
               Channels: ['VOICE'],
               Queues: queues,
               Agents: agents,
           },
           CurrentMetrics: [
               { Name: "AGENTS_AFTER_CONTACT_WORK", Unit: "COUNT" },
               { Name: "AGENTS_ON_CALL", Unit: "COUNT" },
               { Name: "AGENTS_AVAILABLE", Unit: "COUNT" },
               { Name: "AGENTS_ONLINE", Unit: "COUNT" },
               { Name: "AGENTS_STAFFED", Unit: "COUNT" },
               { Name: "CONTACTS_IN_QUEUE", Unit: "COUNT" },
               { Name: "AGENTS_ERROR", Unit: "COUNT" },
               { Name: "AGENTS_NON_PRODUCTIVE", Unit: "COUNT" },
               { Name: "AGENTS_ON_CONTACT", Unit: "COUNT" },
               { Name: "CONTACTS_SCHEDULED", Unit: "COUNT" },
               { Name: "OLDEST_CONTACT_AGE", Unit: "SECONDS" },
               { Name: "SLOTS_ACTIVE", Unit: "COUNT" },
               { Name: "SLOTS_AVAILABLE", Unit: "COUNT" },
           ],
       };
       try {
           const command = new GetCurrentMetricDataCommand(input);
           const data = await connectClient.send(command);
           const metricsObject = convertToObject(data);
           dailyRealTimeMetrics.push({
               date: startTime.toISOString().split('T')[0],
               metrics: metricsObject,
           });
           console.log(`Real-time Metrics for ${startTime.toISOString().split('T')[0]}:`, metricsObject);
       } catch (err) {
           console.error(`Error fetching real-time metrics for ${startTime.toISOString().split('T')[0]}:`, err);
       }
   }
   return dailyRealTimeMetrics;
}
// Fetch historical metrics for each day over one month
async function getHistoricalMetrics() {
   const currentTime = new Date();
   const endDate = new Date(currentTime);
   const startDate = new Date();
   startDate.setMonth(startDate.getMonth() - 1);
   console.log("Fetching daily historical metrics from:", startDate.toISOString(), "to", endDate.toISOString());
   const queues = await getQueues();
   const agents = await getAgents();
   const dailyHistoricalMetrics = [];
   for (let date = new Date(startDate); date <= endDate; date.setDate(date.getDate() + 1)) {
       const startTime = new Date(date);
       const endTime = new Date(date);
       endTime.setHours(23, 59, 59, 999);
       const input = {
           ResourceArn: process.env.ResourceArn,
           StartTime: startTime,
           EndTime: endTime,
           Metrics: [
               { Name: "SUM_CONTACTS_ABANDONED_IN_X", Threshold: [{ Comparison: "LT", ThresholdValue: 7200 }] },
               { Name: "SUM_CONTACTS_ANSWERED_IN_X", Threshold: [{ Comparison: "LT", ThresholdValue: 7200 }] },
               { Name: "ABANDONMENT_RATE", Unit: "Percent" },
               { Name: "AGENT_ANSWER_RATE", Unit: "Percent"},
               { Name: "CONTACTS_HANDLED" },
               { Name: "MAX_QUEUED_TIME" },
               { Name: "AVG_HANDLE_TIME" },
               { Name: "AGENT_ADHERENT_TIME", Unit: "Seconds" },
               { Name: "AGENT_NON_ADHERENT_TIME", Unit: "Seconds" },
               { Name: "AGENT_NON_RESPONSE", Unit: "COUNT" },
               { Name: "AGENT_NON_RESPONSE_WITHOUT_CUSTOMER_ABANDONS", Unit: "COUNT" },
               { Name: "AGENT_OCCUPANCY", Unit: "Percentage" },
               { Name: "AGENT_SCHEDULE_ADHERENCE", Unit: "Percent" },
               { Name: "AGENT_SCHEDULED_TIME", Unit: "Seconds" },
           ],
           Filters: [
               { FilterKey: "QUEUE", FilterValues: queues },
               { FilterKey: "AGENT", FilterValues: agents },
           ],
       };
       try {
           const command = new GetMetricDataV2Command(input);
           const data = await connectClient.send(command);
           const metricsObject = convertToObject(data);
           dailyHistoricalMetrics.push({
               date: startTime.toISOString().split('T')[0],
               metrics: metricsObject,
           });
           console.log(`Historical Metrics for ${startTime.toISOString().split('T')[0]}:` , metricsObject);
       } catch (err) {
           console.error(`Error fetching historical metrics for ${startTime.toISOString().split('T')[0]}:` , err);
       }
   }
   return dailyHistoricalMetrics;
}
// Function to push metrics to DynamoDB
async function pushMetricsToDynamoDB(metrics, metricType) {
   for (const metric of metrics) {
       try {
           // Prepare item for DynamoDB
           const itemParams = {
               TableName : TABLE_NAME,
               Item : {
                   'metricType': { S : metricType }, // Partition key
                   'date': { S : metric.date },       // Sort key
                   'metrics': { M : convertMetricsToMap(metric.metrics) }, // Store metrics as a map
                   'resources': "agents"
               }
           };
           // Put item into DynamoDB
           await dynamoDbClient.send(new PutItemCommand(itemParams));
           console.log(`Successfully pushed ${metricType} metrics for ${metric.date} to DynamoDB.`);
       } catch (err) {
           console.error(`Error pushing ${metricType} metrics for ${metric.date} to DynamoDB:` , err);
       }
   }
}
// Function to convert metrics object to DynamoDB Map format
function convertMetricsToMap(metrics) {
   const map = {};
   for (const [key, value] of Object.entries(metrics)) {
       map[key] = { N : value.toString() }; // Assuming all metric values are numbers
   }
   return map;
}
// Handler function to call both APIs and return all metrics
async function handler(event, context) {
   try {
        const currentMetrics = await getCurrentMetrics();
        await pushMetricsToDynamoDB(currentMetrics, 'realTime'); // Push real-time metrics
        const historicalMetrics = await getHistoricalMetrics();
        await pushMetricsToDynamoDB(historicalMetrics, 'historical'); // Push historical metrics
        const allMetrics = {
            realTimeMetrics : currentMetrics,
            historicalMetrics : historicalMetrics,
        };
        console.log("Combined metrics for one month:", JSON.stringify(allMetrics));
        return allMetrics;
    } catch (err) {
        console.error("Error in handler:", err);
        throw err;
    }
}
// Function to convert API response into a clean object
function convertToObject(data) {
    const result = {};
    if (data && data.MetricResults) {
        for (let i=0; i<data.MetricResults.length; i++) {
            for (let j=0; j<data.MetricResults[i].Collections.length; j++) {
                const metricName=data.MetricResults[i].Collections[j].Metric.Name;
                const metricValue=data.MetricResults[i].Collections[j].Value;
                result[metricName]=metricValue;
            }
        }
    }
    console.log(result,'Formatted Metrics');
    return result;
}
export { handler, getCurrentMetrics, getHistoricalMetrics };
