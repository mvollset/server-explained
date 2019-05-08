---?color=linear-gradient(100deg, #cf022b 50%, white 50%)

@snap[west span-60 text-left]
### Jira Integration Server
* Siam for Jira Servicedesk *

@snapend

@snap[east span-40 text-center]
![](assets/img/integration.png)
@snapend

---

## Background

 When XXX switched from [SD vendor] to [SD vendor] every integration needed to be rewritten
 - Both on their side 
 - And for the suppliers/vendors/customers 


---

@snap[west span-40 text-center]

### Before
![](assets/img/before_integratons.png)
@snapend

@snap[east span-40 text-center]
 
 @ul
 - Directly coupled
 - Different technologies
@ulend

@snapend
---

@snap[west span-40 text-center]

### Before
![](assets/img/before_integratons.png)
@snapend

@snap[east span-40 text-center]
### After
![](assets/img/after_integratons.PNG)
@snapend

---

## Configuration.
There is a _lot_ of configuration, the most important files are
@ul
    - .env 
    - config/config.[environment].js
    - partnerfiles
    - mapfiles 
@ulend

---

### .env file
This file contains username and password for jira, and environment setting.
```
PORT=8080 #Port the server will listen on
NODE_ENV=test #Environment
JIRA_USERNAME=srvjira
JIRA_PASS=secret

```
NODE_ENV decides what config files to read. 
 - test->config.test.js
 - preprod->config.preprod.js

---

### config.[environment].js
This is the main configuration file. 
```js
 server: {
        port: process.env.PORT || 3000
    },
    jiraAuth: {
        user: process.env.JIRA_USERNAME,
        password: process.env.JIRA_PASS
    },
    //How long to cache insight objects
    cache: {
        stdTTL: 1000,
        checkPeriod: 1200
    },
    loadPlugins:[
        //Aray of plugins/routes to enable for this server
        {
            path:'./nav',
            root:'skatt'
        },
        {
            path:'./partner',
            root:'api/siam/partner'
        }
    ],
    jiraRoot: 'https://ref-jira.sits.no',
    jiraKeySuffix: 'ref-jira.sits.no',
    mapFolder: 'config/map-preprod', //Folder to read maps from
    partnerFolder: 'config/partner-preprod', //Folder to read partners from
    jiraEditMetaUrl: '/rest/api/2/issue/createmeta?expand=projects.issuetypes.fields',
    fields: {
        //Well known fields 
        externalRef: 'customfield_16661',
        partner: 'customfield_16660',//Skatt
        externalId: 'customfield_16662',
        closedPartnerDate:'customfield_16665',
        receivedPartnerDate:'customfield_16663'

    },
    transitions: {
        //The id of transitions that are performed by the integration server
        receivedPartner: "161",
        backoutPartner: "151",
        finishedPartner: "171"
    },
    //icapServer:'155.55.65.11', If needed the integration server can call an icap server to do virus control on incoming files.
    
    insight: {
        //Insight configuration  
       
    }
    ,
    maps: {
        //Common maps can be placed here
    }
};
```

---

## Maps
Maps are ,meant to be, a configurable way to transform an object, either on the way out from Jira or in from the vendor.

---

#### Simple example

```js 
const themap = {
        "val1": "value1",
        "val2": "value2"
}
const map=mapper.create(themap);
const result=map.map({
    value1:2,
    value2:3
});
console.dir(res,true);
//Outputs
//{ val1: 2, val2: 3 }
//

```

---

#### Fixed values
This is your typical jira map.
```js
const map = {
    fields: {
        project: {
            key: {
                value: "ITSD",
                $fixed: true
            }
        },
        summary: "summary",
        description: "description",
        issuetype: {
            name: {
                value: "Service Request",
                $fixed: true
            }
        }
    }
}
const result = map.map({
    summary:"My Summary",
    description:"My Description"
});
result ={
    fields: {
        project: {
            key: "ITSD"
        },
        summary: "My Summary",
        description: "My Description",
        issuetype: {
            name: {
                value: "Service Request",
            }
        }
    }
}

```

---

#### More examples

```js
const map = {"foo.bar":"field1"} 
/*Will translate into 
 foo:{
     bar:"field1"
 }*/
 const map = {value:"object1.property"} //Will also work.
 ```

---

 #### Mapping with functions
 Sometimes you can not simply map one value to another, so the object mapper allows you to use define functions

 ```
var themap = {
        "value1": "value1",
        "value2": "value2",
        "sumOfValue":function(value,sourceobject){
            return {
                value: sourceobject.value1 + sourceobject.value2
            }
        }
}
var map=mapper.create(themap);
var res=map.map({
    value1:2,
    value2:3
});
console.dir(res,true);
//Outputs
//{ value1: 2, value2: 3, sumOfValue: 5 }
//
```
The function always has two arguments, val and sourceobject, where val is sourceobject[val], and sourceobject is the full object passed to the map function.
The return value should be an object with a value property, you can also return a name property that will lead to a renaming of the return property.

---

# Partner configuration
A partner/ vendor is configured in a .js file in the directory config.partner folder. For a partner to work you have to define the following properties.
@ul
    - User information
    - Name
    - requests
    - maps
@ulend

---

# Partner user information

```js
    user: {
       id: "976239997", //For Skatteetaten this is the partners org.no
       partner: "atea",
       name: "Atea", //This MUST match the the value in the partner dropdown.
       scope:["jira-comment"] //Now there are two scopes available, jira-comment and jira-create
   }
```

---

# Partner requests configuration

A partner must have a transfer request and a addComment request. In addition you may add a addAtttachment request, if the attachments are sent as separate calls to the vendor.

> The request object is an extension to the [node request package](https://www.npmjs.com/package/request), so most of the options available here should work.

---

# Atea transfer explained

```js

          method: 'POST', //HTTP Method
           uri: `https://servicehub.atea.com/partner/servicedesk/NO/customer/1207521/incident`, //URL 
           bodyMap: "ateaCreate", // Map  for translation of 
           json: true, //Use JSON
           headers: { "Ocp-Apim-Subscription-Key": "c4423dc2ab7e4772ad22f0ddbe85c4cf" }, //Any additional http headers
           rejectUnauthorized: false, //Check https certificate.
           //Transform function should always return an object with success true or false, and a requestId and or request number
           transform: function (body, response /*, resolveWithFullResponse*/) {
               if (response.statusCode == 202)
                   return {
                       requestId: body,
                       requestNumber:body,
                       success: true,
                       message: 'Atea accepted the call'
                   };
           },
           //Should we handle attachments? If attachments is not defined  attachments will not be transferred. If mode is split - we will send the attachments as seperate calls after the case is transferred.
           attachments:{
               mode:"split",
               transform: async function(attachmentInfo,user,pass){
                   return attachmentInfo;
               }
           }
       }
```

---


# The End!!


