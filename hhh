import { ConnectClient, UpdateSecurityProfileCommand,CreateUserCommand } from "@aws-sdk/client-connect";
const client = new ConnectClient();
export const handler = async (event) => {
   try {
       const profilesToCreate = [
           {
               name: "tagsecurityprofile3",
               tags: {
                   "Country": "Argentina",
                   "Createdby": "ABC"
               }
           },
           {
               name: "tagsecurityprofile4",
               tags: {
                   "BPO":"Octank",
                   "Createdby": "ABC"
               }
           },
           {
               name: "tagsecurityprofile5",
               tags: {
                   "Country": "Argentina",
                    "BPO":"Octank",
                   "Createdby": "ABC"
               }
           }
       ];
       const createdProfiles = [];
       for (const profile of profilesToCreate) {
           const input = {
               SecurityProfileName: profile.name,
               Description: "Security profile with access to Routing Profiles, Queues, and Users",
               Permissions: ["RoutingPolicies.Create","RoutingPolicies.Edit","RoutingPolicies.View","Queues.Create","Queues.Edit","Queues.EnableAndDisable","Users.Create","Users.View","Users.Edit"],
               InstanceId: "bd16d991-11c8-4d1e-9900-edd5ed4a9b21",
               Tags: profile.tags,
               AllowedAccessControlTags: {
                   "Country": "Out"
               },
               TagRestrictedResources: ["User","Queue","RoutingProfile"]
           };
           const command = new CreateSecurityProfileCommand(input);
           const response = await client.send(command);
           console.log(`Security profile ${profile.name} created successfully!`);
           console.log(`SecurityProfileId ${profile.name}:`, response.SecurityProfileId);
           console.log(`SecurityProfileArn ${profile.name}:`, response.SecurityProfileArn);
           createdProfiles.push(response);
           
              // Define the security profile ID for the remaining three users
       const agentProfileId = "agent_profile_id"; // Replace with the actual security profile ID
       // Create users
       const usersToCreate = [
           {
              username: "tagadmin1",
               firstName: "Admin",
               lastName: "Tag",
               // email: "john@example.com",
               tags: {
                //   "Department": "IT",
                //   "Location": "USA"
               },
               securityProfileId: createdProfiles[0].SecurityProfileId
           },
           {
               username: "taguser11",
               firstName: "Test1",
               lastName: "Tag",
               // email: "jane@example.com",
               tags: {
                   "Country": "Argentina"
               },
               securityProfileId: agentProfileId
           },
           {
              username: "taguser22",
               firstName: "Test2",
               lastName: "Tag",
               // email: "alice@example.com",
               tags: {
                   
                   "BPO": "Octank"
               },
               securityProfileId: agentProfileId
           },
           {
              username: "taguser33",
               firstName: "Test3",
               lastName: "Tag",
               // email: "bob@example.com",
               tags: {
                   "BPO": "Octank",
                   "Country": "Argentina"
               },
               securityProfileId: agentProfileId
           }
       ];
       for (const user of usersToCreate) {
           const userInput = {
               Username: user.username,
               Password: "Test@123",
               IdentityInfo: {
                   FirstName: user.firstName,
                   LastName: user.lastName,
                   Email: user.email,
               },
               PhoneConfig: {
                   PhoneType: "SOFT_PHONE",
                   AutoAccept: true,
                   AfterContactWorkTimeLimit: 300,
               },
               SecurityProfileIds: [user.securityProfileId],
               RoutingProfileId: "routing_profile_id",
               InstanceId: "instance_id",
               Tags: user.tags,
           };
           const createUserCommand = new CreateUserCommand(userInput);
           const userResponse = await client.send(createUserCommand);
           console.log(`User ${user.username} created successfully!`);
           console.log(`UserId ${user.username}:`, userResponse.UserId);
           console.log(`UserArn ${user.username}:`, userResponse.UserArn);
       }
       return createdProfiles;
   } catch (error) {
       console.error("Error:", error);
       throw error; // 
   }
}
       
