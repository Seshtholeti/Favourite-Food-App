import { ConnectClient, GetContactAttributesCommand } from "@aws-sdk/client-connect"; // ES Modules import
// const { ConnectClient, GetContactAttributesCommand } = require("@aws-sdk/client-connect"); // CommonJS import
const client = new ConnectClient(config);
const input = { // GetContactAttributesRequest
  InstanceId: "STRING_VALUE", // required
  InitialContactId: "STRING_VALUE", // required
};
const command = new GetContactAttributesCommand(input);
const response = await client.send(command);
// { // GetContactAttributesResponse
//   Attributes: { // Attributes
//     "<keys>": "STRING_VALUE",
//   },
// };

api syntax for GetCOntact.

below is the syntax code for Searchcontact command
import { ConnectClient, SearchContactsCommand } from "@aws-sdk/client-connect"; // ES Modules import
// const { ConnectClient, SearchContactsCommand } = require("@aws-sdk/client-connect"); // CommonJS import
const client = new ConnectClient(config);
const input = { // SearchContactsRequest
  InstanceId: "STRING_VALUE", // required
  TimeRange: { // SearchContactsTimeRange
    Type: "INITIATION_TIMESTAMP" || "SCHEDULED_TIMESTAMP" || "CONNECTED_TO_AGENT_TIMESTAMP" || "DISCONNECT_TIMESTAMP", // required
    StartTime: new Date("TIMESTAMP"), // required
    EndTime: new Date("TIMESTAMP"), // required
  },
  SearchCriteria: { // SearchCriteria
    AgentIds: [ // AgentResourceIdList
      "STRING_VALUE",
    ],
    AgentHierarchyGroups: { // AgentHierarchyGroups
      L1Ids: [ // HierarchyGroupIdList
        "STRING_VALUE",
      ],
      L2Ids: [
        "STRING_VALUE",
      ],
      L3Ids: [
        "STRING_VALUE",
      ],
      L4Ids: [
        "STRING_VALUE",
      ],
      L5Ids: [
        "STRING_VALUE",
      ],
    },
    Channels: [ // ChannelList
      "VOICE" || "CHAT" || "TASK" || "EMAIL",
    ],
    ContactAnalysis: { // ContactAnalysis
      Transcript: { // Transcript
        Criteria: [ // TranscriptCriteriaList // required
          { // TranscriptCriteria
            ParticipantRole: "AGENT" || "CUSTOMER" || "SYSTEM" || "CUSTOM_BOT" || "SUPERVISOR", // required
            SearchText: [ // SearchTextList // required
              "STRING_VALUE",
            ],
            MatchType: "MATCH_ALL" || "MATCH_ANY", // required
          },
        ],
        MatchType: "MATCH_ALL" || "MATCH_ANY",
      },
    },
    InitiationMethods: [ // InitiationMethodList
      "INBOUND" || "OUTBOUND" || "TRANSFER" || "QUEUE_TRANSFER" || "CALLBACK" || "API" || "DISCONNECT" || "MONITOR" || "EXTERNAL_OUTBOUND" || "WEBRTC_API" || "AGENT_REPLY" || "FLOW",
    ],
    QueueIds: [ // QueueIdList
      "STRING_VALUE",
    ],
    SearchableContactAttributes: { // SearchableContactAttributes
      Criteria: [ // SearchableContactAttributesCriteriaList // required
        { // SearchableContactAttributesCriteria
          Key: "STRING_VALUE", // required
          Values: [ // SearchableContactAttributeValueList // required
            "STRING_VALUE",
          ],
        },
      ],
      MatchType: "MATCH_ALL" || "MATCH_ANY",
    },
    SearchableSegmentAttributes: { // SearchableSegmentAttributes
      Criteria: [ // SearchableSegmentAttributesCriteriaList // required
        { // SearchableSegmentAttributesCriteria
          Key: "STRING_VALUE", // required
          Values: [ // SearchableSegmentAttributeValueList // required
            "STRING_VALUE",
          ],
        },
      ],
      MatchType: "MATCH_ALL" || "MATCH_ANY",
    },
  },
  MaxResults: Number("int"),
  NextToken: "STRING_VALUE",
  Sort: { // Sort
    FieldName: "INITIATION_TIMESTAMP" || "SCHEDULED_TIMESTAMP" || "CONNECTED_TO_AGENT_TIMESTAMP" || "DISCONNECT_TIMESTAMP" || "INITIATION_METHOD" || "CHANNEL", // required
    Order: "ASCENDING" || "DESCENDING", // required
  },
};
const command = new SearchContactsCommand(input);
const response = await client.send(command);
// { // SearchContactsResponse
//   Contacts: [ // Contacts // required
//     { // ContactSearchSummary
//       Arn: "STRING_VALUE",
//       Id: "STRING_VALUE",
//       InitialContactId: "STRING_VALUE",
//       PreviousContactId: "STRING_VALUE",
//       InitiationMethod: "INBOUND" || "OUTBOUND" || "TRANSFER" || "QUEUE_TRANSFER" || "CALLBACK" || "API" || "DISCONNECT" || "MONITOR" || "EXTERNAL_OUTBOUND" || "WEBRTC_API" || "AGENT_REPLY" || "FLOW",
//       Channel: "VOICE" || "CHAT" || "TASK" || "EMAIL",
//       QueueInfo: { // ContactSearchSummaryQueueInfo
//         Id: "STRING_VALUE",
//         EnqueueTimestamp: new Date("TIMESTAMP"),
//       },
//       AgentInfo: { // ContactSearchSummaryAgentInfo
//         Id: "STRING_VALUE",
//         ConnectedToAgentTimestamp: new Date("TIMESTAMP"),
//       },
//       InitiationTimestamp: new Date("TIMESTAMP"),
//       DisconnectTimestamp: new Date("TIMESTAMP"),
//       ScheduledTimestamp: new Date("TIMESTAMP"),
//       SegmentAttributes: { // ContactSearchSummarySegmentAttributes
//         "<keys>": { // ContactSearchSummarySegmentAttributeValue
//           ValueString: "STRING_VALUE",
//         },
//       },
//     },
//   ],
//   NextToken: "STRING_VALUE",
//   TotalCount: Number("long"),
// };

below is the error I am getting
MyEventName

Response
{
  "statusCode": 200,
  "body": "{\"message\":\"Process completed successfully\",\"details\":[{\"phoneNumber\":\"+919949921498\",\"agentId\":\"Error\",\"disposition\":\"Error Fetching Record\"},{\"phoneNumber\":\"+918639694701\",\"agentId\":\"Error\",\"disposition\":\"Error Fetching Record\"}]}"
}

Function Logs
START RequestId: fd83af81-9023-47ba-850d-e914361eb0db Version: $LATEST
2024-11-29T02:10:03.100Z	fd83af81-9023-47ba-850d-e914361eb0db	ERROR	Error processing phone number +919949921498: BadRequestException: Invalid request body
    at throwDefaultError (/var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:840:20)
    at /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:849:5
    at de_CommandError (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:9806:14)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-serde/dist-cjs/index.js:35:20
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/core/dist-cjs/index.js:165:18
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-retry/dist-cjs/index.js:320:38
    at async /var/runtime/node_modules/@aws-sdk/middleware-logger/dist-cjs/index.js:34:22
    at async Runtime.handler (file:///var/task/index.mjs:250:33) {
  '$fault': 'client',
  '$metadata': {
    httpStatusCode: 400,
    requestId: '004b87f4-bb84-4892-91e4-1c76cd08df7b',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  }
}
2024-11-29T02:10:03.160Z	fd83af81-9023-47ba-850d-e914361eb0db	ERROR	Error processing phone number +918639694701: BadRequestException: Invalid request body
    at throwDefaultError (/var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:840:20)
    at /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/smithy-client/dist-cjs/index.js:849:5
    at de_CommandError (/var/runtime/node_modules/@aws-sdk/client-connect/dist-cjs/index.js:9806:14)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-serde/dist-cjs/index.js:35:20
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/core/dist-cjs/index.js:165:18
    at async /var/runtime/node_modules/@aws-sdk/node_modules/@smithy/middleware-retry/dist-cjs/index.js:320:38
    at async /var/runtime/node_modules/@aws-sdk/middleware-logger/dist-cjs/index.js:34:22
    at async Runtime.handler (file:///var/task/index.mjs:250:33) {
  '$fault': 'client',
  '$metadata': {
    httpStatusCode: 400,
    requestId: 'f4b8078f-a5ed-4d5c-b64b-9c18db286492',
    extendedRequestId: undefined,
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  }
}
2024-11-29T02:10:03.161Z	fd83af81-9023-47ba-850d-e914361eb0db	INFO	Call Details: [
  {
    phoneNumber: '+919949921498',
    agentId: 'Error',
    disposition: 'Error Fetching Record'
  },
  {
    phoneNumber: '+918639694701',
    agentId: 'Error',
    disposition: 'Error Fetching Record'
  }
]
END RequestId: fd83af81-9023-47ba-850d-e914361eb0db
REPORT RequestId: fd83af81-9023-47ba-850d-e914361eb0db	Duration: 732.77 ms	Billed Duration: 733 ms	Memory Size: 128 MB	Max Memory Used: 99 MB
