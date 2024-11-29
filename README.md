import { ConnectClient, SearchContactsCommand, DescribeUsersCommand } from "@aws-sdk/client-connect"; // ES Modules import

// Configure AWS SDK Client
const config = {
  region: 'us-east-1', // Replace with your region
};

const client = new ConnectClient(config);

// Define your Amazon Connect instance ID
const instanceId = 'your-instance-id'; // Replace with your Amazon Connect instance ID

// Function to fetch agent IDs dynamically
async function fetchAgentIds() {
  const agentIds = [];
  
  try {
    const describeUsersCommand = new DescribeUsersCommand({
      InstanceId: instanceId,
      UserStatus: "Active", // Optionally, filter by active users
    });

    const response = await client.send(describeUsersCommand);

    if (response.Users && response.Users.length > 0) {
      response.Users.forEach(user => {
        agentIds.push(user.Id);
      });
      console.log("Fetched Agent IDs:", agentIds);
    } else {
      console.log("No active agents found.");
    }
  } catch (error) {
    console.error("Error fetching agent IDs:", error);
  }

  return agentIds;
}

// Function to search contacts and check which agent is assigned
async function searchContactsAndCheckAssignedAgents(agentIds) {
  const input = {
    InstanceId: instanceId,
    TimeRange: {
      Type: "INITIATION_TIMESTAMP", // Adjust this as needed
      StartTime: new Date("2024-01-01T00:00:00Z"), // Example Start Time
      EndTime: new Date("2024-01-31T23:59:59Z"), // Example End Time
    },
    SearchCriteria: {
      AgentIds: agentIds, // Use the dynamic agent IDs
      Channels: ["VOICE"], // Example Channel (you can change this as needed)
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
        console.log(`Channel: ${contact.Channel}`);
        console.log(`Initiation Method: ${contact.InitiationMethod}`);
        console.log(`Agent ID: ${contact.AgentInfo ? contact.AgentInfo.Id : 'N/A'}`);
        console.log(`Initiation Timestamp: ${contact.InitiationTimestamp}`);
        console.log(`Disconnect Timestamp: ${contact.DisconnectTimestamp}`);
        console.log("----------------------------");
      });
    } else {
      console.log("No contacts found for the given search criteria.");
    }
  } catch (error) {
    console.error("Error searching contacts:", error);
  }
}

// Main function to fetch agents and then search contacts
async function main() {
  const agentIds = await fetchAgentIds(); // Fetch agent IDs dynamically
  if (agentIds.length > 0) {
    await searchContactsAndCheckAssignedAgents(agentIds); // Search contacts using the dynamic agent IDs
  } else {
    console.log("No agent IDs available to search contacts.");
  }
}

main();