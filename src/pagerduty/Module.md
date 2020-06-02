# PagerDuty Module

The Ballerina PagerDuty connector allows you to work with PagerDuty users, escalationPolicies, services, schedules, and incidents through the PagerDuty Rest API. 
It handles the API Token Authentication. Following tokens are can be used under this mechanism:

* **Account API token** - It can access all the data in an account and can either be granted read-only access or full access to read, write, update, and delete. 
Only account administrators have the ability to generate account API tokens.
* **User API token** - It can access all the data that the associated user account has access to.

>**Note:** According to the account's role permission, the user API token can access all the available functionality. However,
> the account API token can't access the operations, which need the `from email Id`. They are `createUser`, `createIncident`, `createEscalationPolicy`, `createSchedule`, `manageIncidents`, `updateIncident`, and `addNote`.

The following groups are provided by Ballerina to interact with the different API groups of the PagerDuty REST API. 
- **pagerDuty:Account** - The `Account`, which is used to initiate the contact with the PagerDuty API and create all the associated sub groups with the operations. 
- **pagerDuty:UserClient** - The `UserClient`, which will be used to create/get/delete the Users/Contact methods/ Notification rules.
- **pagerDuty:EscalationPolicyClient** - The `EscalationPolicyClient`, which will be used to create/get/update/delete the escalation policies.
- **pagerDuty:ScheduleClient** - The `ScheduleClient`, which will be used to create/get/delete the schedules.
- **pagerDuty:ServiceClient** - The `ServiceClient`, which will be used to create/update|delete the services/integrations. 
- **pagerDuty:ExtensionClient** - The `ExtensionClient`, which will be used to create/get/update/delete the extensions.
- **pagerDuty:IncidentClient** - The `IncidentClient`, which will be used to create/get/update/manage/delete/snooze the incidents and add notes into that.

## Compatibility

|                             |           Version           |
|:---------------------------:|:---------------------------:|
| Ballerina Language          |            1.2.X            |
| PagerDuty REST API          |            v2               |

## Sample

**Generating tokens**

* Create a [PagerDuty account](https://www.pagerduty.com/).
* Generate one of the below API tokens.
    * User API token:
        * Go to Configuration->Users->User setting-> Create API user token
    * Account API token:
        * Go to Configuration->API Access->Create New API Key
        
>**Note:** The user API token supports all the available functionalities of the role permission. However, the account API token can't access the operations, which need the `from email Id`(`createUser`, `createIncident`, `createEscalationPolicy`, `createSchedule`, `manageIncidents`, `updateIncident`, and `addNote`).

>**Create the `pagerduty:Account`**

First, execute the below command to import the `ballerinax/pagerduty` module into the Ballerina project.
```ballerina
import ballerinax/pagerduty;
```
Instantiate the `pagerduty:Account` by giving the token authentication details. 

You can create the account as follows. 
```ballerina
// Creates the PagerDuty account.
Account pagerduty = new("API_TOKEN");
```

**PagerDuty operations related to `Users`**

The `createUser` remote function can be used to create a user in PagerDuty `Users`. 

```ballerina
pagerduty:UserClient userClient = pagerduty.getUserClient();
pagerduty:User user =  { 'type: "user", name: "sayan", email: "exa@gmail.com", role: "admin"};
pagerduty:Error|pagerduty:User output = userClient->createUser(user);
if (output is pagerduty:Error) {
    io:println("Error" + output.toString());
} else {
    userId = output.get("id").toString();
    createdUser = output;
    io:println("User id " + userId);
}
```

**Pagerduty operations related to `EscalationPolicies`**

The `create` remote function can be used to create the escalation policy in PagerDuty `Escalation Policies`.

```ballerina
pagerduty:EscalationPolicyClient escalationClient = pagerduty.getEscalationPolicyClient();
pagerduty:EscalationPolicy escalationPolicy = { 'type: "escalationPolicy", name: "Escalation Policy for Test",
                                                 escalationRules: [{ escalationDelayInMinutes: 30,
                                                                     targets: [{id: userId, 'type: "user"}]
                                                                  }]
                                              };
pagerduty:EscalationPolicy|pagerduty:Error response = escalationClient->create(escalationPolicy);
if (response is pagerduty:Error) {
    io:println("Error" + response.toString());
} else {
    createdPolicy = response;
    io:println("EscalationPolicy id " + response.get("id").toString());
}
```

**Pagerduty operations related to `Schedules`**

The `create` remote function can be used to create the schedule in PagerDuty `Schedules`.  

```ballerina
pagerduty:ScheduleClient scheduleClient = pagerduty.getScheduleClient();
time:Time time = time:currentTime();
pagerduty:Schedule schedule = { 'type: "schedule", timeZone: "Asia/Colombo",
                                scheduleLayers: [{ 'start: time, rotationTurnLengthInSeconds: 86400,
                                                   rotationVirtualStart: time, users: [createdUser]
                                                }]
                               };
pagerduty:Schedule|pagerduty:Error createdSchedule = scheduleClient->create(schedule);
if (createdSchedule is pagerduty:Error) {
   io:println("Error" + createdSchedule.toString());
} else {
    io:println("Schedule id " + createdSchedule.get("id").toString());
}
```

**Pagerduty operations related to `Services`**

The `createService` remote function can be used to create the Services in PagerDuty `Services`. 

```ballerina
pagerduty:ServiceClient serviceClient = pagerduty.getServiceClient();
pagerduty:Service serv = { name: "New services", escalationPolicy: createdPolicy, alertCreation:"createAlertsAndIncidents"};
pagerduty:Service|pagerduty:Error resp = serviceClient->createService(serv);
if (resp is pagerduty:Error) {
    io:println("Error" + resp.toString());
} else {
    createdService = resp;
    io:println("Service id " + resp.get("id").toString());
}
```

**Pagerduty operations related to `Extensions`**

The `create` remote function can be used to create the extension in PagerDuty `Extensions`. 

```ballerina
pagerduty:ExtensionClient extensionClient = pagerduty.getExtensionClient();
pagerduty:Extension extension = {  'type: "extension", name: "webhook",
                                   endpointUrl: "http://fc321768.ngrok.io/webhooks",
                                   extensionSchema: {id: "PJFWPEP", 'type: "extensionSchemaReference",
                                   summary: "Generic V2 Webhook"}, services: [createdService]
                                };
pagerduty:Extension|pagerduty:Error createdExtension = extensionClient->create(extension);
if (createdExtension is pagerduty:Error) {
    io:println("Error" + createdExtension.toString());
} else {
    io:println("Extension id " + createdExtension.get("id").toString());
}
```

**Pagerduty operations related to `Incidents`**

The `createIncident` remote function can be used to create the incident in PagerDuty `Incidents`.  

```ballerina
pagerduty:IncidentClient incidentClient = pagerduty.getIncidentClient();
pagerduty:Incident incident = {'type: "incident", title: "Test", 'service: createdService};
pagerduty:Incident|pagerduty:Error createdIncident = incidentClient->createIncident(incident);
if (createdIncident is pagerduty:Error) {
    io:println("Error" + createdIncident.toString());
} else {
    io:println("Incident id " + createdIncident.get("id").toString());
}
```