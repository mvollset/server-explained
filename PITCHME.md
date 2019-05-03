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
 - Differnet technologies
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
    //icapServer:'155.55.65.11', If needed the integration server can call an icap server to do virus control on incoming files.
    
    insight: {
        //Insight configuration  
       
    },
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
    }
    ,
    maps: {
        //Common maps can be placed here
    }
};
```

---

## Maps
Maps are meant to be configurable way to transform an object, either on the way out from Jira or in from the vendor.

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
                //bytt til id 13101 hvis dette ikke funker (kanskje ikke riktig ID i Q og P)
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
            key: "ITSD
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
