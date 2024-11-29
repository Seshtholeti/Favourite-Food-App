import { ConnectClient, SearchContactsCommand, ListUsersCommand, DescribeQueueCommand } from "@aws-sdk/client-connect"; 

// Configure AWS SDK Client
const client = new ConnectClient({
  region: 'us-east-1', // Adjust the region
});

// Define your Amazon Connect instance ID and Queue ID
const instanceId = 'your-instance-id'; // Replace with your Connect Instance ID
const queueId = 'your-queue-id'; // Replace with your Queue ID

// Function to fetch agent IDs dynamically using ListUsersCommand
async function fetchAgentIds() {
  const agentIds = [];
  let nextToken = null;

  try {
    do {
      const listUsersCommand = new ListUsersCommand({
        InstanceId: instanceId,
        NextToken: nextToken, // For pagination
      });

      const response = await client.send(listUsersCommand);

      if (response.Users && response.Users.length > 0) {
        response.Users.forEach(user => {
          agentIds.push(user.Id);
        });
      }

      nextToken = response.NextToken; // Set NextToken for pagination if available
    } while (nextToken);

  } catch (error) {
    console.error("Error fetching agent IDs:", error);
  }

  return agentIds;
}

// Function to describe queue to see queue settings (you can manually track agents per queue)
async function describeQueue() {
  try {
    const describeQueueCommand = new DescribeQueueCommand({
      InstanceId: instanceId,
      QueueId: queueId,
    });

    const queueDetails = await client.send(describeQueueCommand);
    console.log("Queue Details:", queueDetails);
    return queueDetails;
  } catch (error) {
    console.error("Error describing queue:", error);
  }
}

// Function to search contacts assigned to the given queue
async function searchContactsByQueue(agentIds) {
  const input = {
    InstanceId: instanceId,
    TimeRange: {
      Type: "INITIATION_TIMESTAMP",
      StartTime: new Date("2024-01-01T00:00:00Z"),
      EndTime: new Date("2024-01-31T23:59:59Z"),
    },
    SearchCriteria: {
      AgentIds: agentIds,
      Channels: ["VOICE"],
      QueueIds: [queueId], // Use the Queue ID to filter contacts
    },
    MaxResults: 10,
    Sort: {
      FieldName: "INITIATION_TIMESTAMP",
      Order: "ASCENDING",
    },
  };

  const command = new SearchContactsCommand(input);

  try {
    const response = await client.send(command);

    if (response.Contacts && response.Contacts.length > 0) {
      console.log("Search Results:");
      response.Contacts.forEach(contact => {
        console.log(`Contact ID: ${contact.Id}`);
        console.log(`Agent ID: ${contact.AgentInfo ? contact.AgentInfo.Id : 'N/A'}`);
        console.log(`Queue ID: ${contact.QueueInfo ? contact.QueueInfo.Id : 'N/A'}`);
        console.log(`Channel: ${contact.Channel}`);
        console.log(`Initiation Method: ${contact.InitiationMethod}`);
        console.log("----------------------------");
      });
    } else {
      console.log("No contacts found for the given search criteria.");
    }
  } catch (error) {
    console.error("Error searching contacts:", error);
  }
}

// Main function to describe queue, fetch agents, and search contacts
async function main() {
  const queueDetails = await describeQueue();
  const agentIds = await fetchAgentIds(); // Fetch agent IDs dynamically
  if (agentIds.length > 0) {
    await searchContactsByQueue(agentIds); // Search contacts using the dynamic agent IDs
  } else {
    console.log("No agent IDs available to search contacts.");
  }
}

main();