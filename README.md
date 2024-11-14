import { ConnectClient, GetCurrentMetricDataCommand, ListQueuesCommand, ListUsersCommand } from "@aws-sdk/client-connect";
// Initialize the Connect client
const client = new ConnectClient({ region: 'us-east-1' });
// Function to fetch queues dynamically using ListQueues
async function getQueues() {
  const input = { InstanceId: process.env.InstanceId };
  try {
      const command = new ListQueuesCommand(input);
      const data = await client.send(command);
      const queueIds = data.QueueSummaryList.map(queue => queue.Id);
      console.log("Fetched Queues:", queueIds);
      return queueIds.slice(0, 10); // Limiting to a maximum of 10 queues
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
      const data = await client.send(command);
      const agentIds = data.UserSummaryList.map(user => user.Id);
      console.log("Fetched Agents:", agentIds);
      return agentIds.slice(0, 10); // Limiting to a maximum of 10 agents
  } catch (err) {
      console.error("Error fetching agents:", err);
      throw err;
  }
}
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
// Main function to fetch metrics for agents and queues
export const handler = async (event) => {
  const { resourceType, selectedResources, startDate, endDate } = event;
  // Validate that selectedResources is not empty
  if (!selectedResources || selectedResources.length === 0) {
      throw new Error("You must select at least one resource.");
  }
  const instanceId = process.env.InstanceId; // Make sure the instanceId is set in the environment
  const dateRange = getDateRange(startDate, endDate);
  const response = {
      resourceType,
      startDate,
      endDate,
      selectedResources,
      metricsByDate: [],
  };
  // Fetch queues and agents dynamically
  const queues = await getQueues();
  const agents = await getAgents();
  // Loop through each day in the range
  for (const date of dateRange) {
      const dateMetrics = { date, queueMetrics: [], agentMetrics: [] };
      // Fetching Queue Metrics
      if (resourceType === "Queues") {
          const queueInput = {
              InstanceId: instanceId,
              Filters: {
                  Queues: selectedResources,  // selected queue IDs
                  Channels: ["VOICE"],
              },
              Groupings: ["QUEUE"],  // Group by queue
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
              const queueCommand = new GetCurrentMetricDataCommand(queueInput);
              const queueData = await client.send(queueCommand);
              if (queueData.MetricResults && queueData.MetricResults.length > 0) {
                  queueData.MetricResults.forEach((result) => {
                      const metrics = result.Collections.map((collection) => ({
                          metricName: collection.Metric.Name,
                          metricValue: collection.Value,
                      }));
                      dateMetrics.queueMetrics.push({
                          queueId: result.Dimensions.Queue.Id,
                          queueName: result.Dimensions.Queue.Name || "Unknown Queue",
                          metrics,
                      });
                  });
              }
          } catch (err) {
              console.error(`Error fetching queue metrics for date ${date}:`, err);
              dateMetrics.queueMetrics.push({
                  error: err.message,
              });
          }
      }
      // Fetching Agent Metrics
      if (resourceType === "Agents") {
       const queues = await getQueues();
       const agents = await getAgents();
          const agentInput = {
              InstanceId: instanceId,
              Filters: {
                  UserIds: selectedResources,  // selected agent IDs
                  Channels: ["VOICE"],
                  Queues:queues,
                  Agents: agents,
              },
              Groupings: ["QUEUE"],  // Group by both agent and queue
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
              StartTime: new Date(`${date}T00:00:00Z`).toISOString(),
              EndTime: new Date(`${date}T23:59:59Z`).toISOString(),
          };
          try {
              const agentCommand = new GetCurrentMetricDataCommand(agentInput);
              const agentData = await client.send(agentCommand);
              if (agentData.MetricResults && agentData.MetricResults.length > 0) {
                  agentData.MetricResults.forEach((result) => {
                      const metrics = result.Collections.map((collection) => ({
                          metricName: collection.Metric.Name,
                          metricValue: collection.Value,
                      }));
                      dateMetrics.agentMetrics.push({
                          agentId: result.Dimensions.Agent.Id,
                          agentName: result.Dimensions.Agent.Name || "Unknown Agent",
                          queueId: result.Dimensions.Queue.Id || "Unknown Queue",
                          queueName: result.Dimensions.Queue.Name || "Unknown Queue",
                          metrics,
                      });
                  });
              }
          } catch (err) {
              console.error(`Error fetching agent metrics for date ${date}:`, err);
              dateMetrics.agentMetrics.push({
                  error: err.message,
              });
          }
      }
      response.metricsByDate.push(dateMetrics);
  }
  return response;
};
