import { ConnectClient, CreateSecurityProfileCommand } from "@aws-sdk/client-connect";
// Create a new instance of the ConnectClient
const client = new ConnectClient();

export const handler = async (event) =>{
// Define input parameters for creating a security profile
const input = {
   SecurityProfileName: "tagsecurityprofile4", // Required
   // SecurityProfileId: "df69a9a4-d9a2-40b2-8a89-ef44e150f1b3",
   Description: "Security profile with access to Routing Profiles, Queues, and Users",
   Permissions: ["RoutingPolicies.Create","RoutingPolicies.Edit","RoutingPolicies.View","Queues.Create","Queues.Edit","Queues.EnableAndDisable"],
   InstanceId: "bd16d991-11c8-4d1e-9900-edd5ed4a9b21", 
   Tags: {
       "Country": "Argentina", 
       "Createdby": "ABC"
   },
   AllowedAccessControlTags: { // AllowedAccessControlTags
       "Country": "In"
   },
   TagRestrictedResources: [ // TagRestrictedResourceList
       "User","Queue","RoutingProfile" // Update with actual values
   ]
   // Applications: [ // Applications
   //     {
   //        Namespace: "STRING_VALUE", // Replace with actual value
   //        ApplicationPermissions: [ "STRING_VALUE" ] // ApplicationPermissions - Replace with actual value
   //     },
   // ],
};

const command = new CreateSecurityProfileCommand(input);
try {
   
   const response = await client.send(command);
   // Handle the response
   console.log("Security profile created successfully!");
   console.log("SecurityProfileId:", response.SecurityProfileId);
   console.log("SecurityProfileArn:", response.SecurityProfileArn);
   return response;
   
        
} catch (error) {
   
   console.error("Error creating security profile:", error);
}
};
