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

