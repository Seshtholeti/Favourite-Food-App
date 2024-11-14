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
