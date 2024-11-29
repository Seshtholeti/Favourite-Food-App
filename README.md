import { ConnectClient, SearchContactsCommand } from "@aws-sdk/client-connect"; // ES Modules import
// const { ConnectClient, SearchContactsCommand } = require("@aws-sdk/client-connect"); // CommonJS import

// Configure AWS SDK Client (No explicit credentials needed)
const config = {
  region: 'us-east-1', // Replace with your region
};

const client = new ConnectClient(config);

// Define input parameters for SearchContactsRequest
const input = {
  InstanceId: "your-instance-id", // Replace with your Amazon Connect instance ID
  TimeRange: {
    Type: "INITIATION_TIMESTAMP", // Change this based on your requirement
    StartTime: new Date("2024-01-01T00:00:00Z"), // Example Start Time
    EndTime: new Date("2024-01-31T23:59:59Z"), // Example End Time
  },
  SearchCriteria: {
    AgentIds: ["your-agent-id"], // Replace with agent ID(s) you're interested in
    Channels: ["VOICE"], // Change channel type if needed (e.g., "CHAT", "EMAIL")
    InitiationMethods: ["INBOUND"], // Modify initiation method as needed (e.g., "OUTBOUND", "TRANSFER")
    QueueIds: ["your-queue-id"], // Replace with the queue ID if needed
  },
  MaxResults: 10, // Set the maximum number of results you want to fetch
  Sort: {
    FieldName: "INITIATION_TIMESTAMP", // Field to sort by
    Order: "ASCENDING", // Sorting order
  },
};

// Create and send the SearchContactsCommand
const command = new SearchContactsCommand(input);

async function searchContacts() {
  try {
    const response = await client.send(command);
    console.log("Search Results:", response.Contacts);

    // Example of processing the response
    if (response.Contacts && response.Contacts.length > 0) {
      response.Contacts.forEach(contact => {
        console.log(`Contact ID: ${contact.Id}`);
        console.log(`Channel: ${contact.Channel}`);
        console.log(`Initiation Method: ${contact.InitiationMethod}`);
        console.log(`Queue ID: ${contact.QueueInfo ? contact.QueueInfo.Id : 'N/A'}`);
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

// Run the search function
searchContacts();