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
## Configuration of the server.

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
 .........

