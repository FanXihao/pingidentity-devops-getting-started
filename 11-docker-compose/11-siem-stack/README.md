# Ping Stack with ElasticSearch SIEM -- BETA version 1.0
### Built by Ryan Ivis with Ping Identity   
### Project Salt Water
- Sailing to the NorthStar without understanding the water around you can be difficult. We want to help with that!

---------------

#### Project Goals
- Enable Customers to see the water in the pipes
- Enable some useful alerts that can be generated from our products
- Use industry standard tooling to boost your security program

---------------

## Latest Build News

- Implemented Impossible Distance Alert. Currently set to 1200km at 6hr.
- Modified many of the configurations to support logging in via LDAP.
- Authentication to the ES Stack / Kibana is now LDAP driven. There is a LDAP group `ESAdminGroup` that is in the example.com Root DSE. Adding a user to this group now gives them access to kibana and all data within elasticsearch as an admin.
- Migrated all indexes to using ILM. This means by default the stack will only store 2 days worth of logs and ensure indexes do not grow over 2GB. This is done because the enviroment is setup as a demo. HEAP sizes in the ES server are SMALL becasue this is a demo. I will soon have production ready documents written to show customers how this can be brought to production.
- Tested updating to elastic 7.6.1 (all good)
- Added in Slack Alerting! 
  - The process requires running a script after you start your stack. 
  - The script will ask you for your webhook url, then add the configuration into the elasticsearch keystore.
  - To run the script run the config_slack_alerts.sh script from the project directory end enter in your webhook URL.

---------------

## Status
| Phase    | Ping Product                                |  Status     |
|----------|---------------------------------------------|-------------|
| Phase 1  | PingFederate Audit Logs                     | Complete    |
| Phase 1a | PingFederate Provisioner Logs               | Complete    |
| Phase 1b | PingFederate System Logs                    | Complete    |
| Phase 2  | LDAP Integrate ElasticSearch / Kibana       | Complete    |
| Phase 2a | Ping SIEM Dashboard                         | Beta        |
| Phase 2b | PingDirectory Load Generator (thanks arno)  | Complete    |
| Phase 2c | Index Mapping rework for PD data index      | Complete    | 
| Phase 2d | Migrate All Indexes to use ILM vs Date/Time | Complete    |
| Phase 2e | Ping Federate Threat Detection Dashboard    | Beta        |
| Phase 3  | PingDirectory Logs                          | Complete    | 
| Phase 4  | PingAccess Logs                             | Complete    |
| Phase 5  | Test and Implement 3  Trial ES Watchers     | Complete    |
| Phase 6  | Help GTE / RSA Implement Customer Demos     | Not Started |
| Phase 7  | Slack Integrate Alerts from SIEM.           | Complete    |
| Phase 8  | Develop Several Custom Alerts               | In Progress |
| Phase 9  | Develop Production Grade Deployment Doc     | In Progress |
| Phase 10 | Implement GEO-Distance Impossible detection | Beta        |


---------------

## What is this ?
![alt text](https://github.com/ryanivis/ping-devops-es-siem/blob/master/images/Architecture.png "Architecture Overview")
- Threat Intel and TOR Endpoints are being provided by AlienVault and the TOR Network Endpoint List. 
- Threat Feeds are updated on an interval via setting a var in docker-compose 

This project will start a Ping Stack with Elastic Search Infrastructure built in for Visualizing traffic and other security / log data. Showing you the pipes.

---------------

## Important Note
- **THIS EXISTING SETUP IS NOT INTENDED FOR PRODUCTION**...   
     
- **THERE ARE PERSISTENT DISKS USED FOR ELASTIC SEARCH DATA, AND ELASTIC SEARCH CERTS. TO CLEAR THEM WHEN YOU ARE DONE TESTING RUN**...   
  - `docker volume prune`  

- **YOU MUST RUN THE FOLLOWING COMMAND ON UBUNTU (LIKELY OTHER DISTRO'S) TO SUPPORT HEAP SIZES**
  - `sudo sysctl -w vm.max_map_count=262144`

- **YOU NEED AT LEAST 12GB OF RAM (FOR DOCKER) TO RUN THIS STACK**

---------------

# Directions
- If Mac or Windows, ensure Docker Subsystem has at a *minimum 10GB of RAM* or the containers will crash.
- Contact DevOps / Ping Sales Team and collect a DevOps user account / key.
- (Optional / *Recommended* for Alerting) Pre work: Generate a Slack Webhook URL from Slack Admin. https://api.slack.com/messaging/webhooks
- To setup on AWS use a M5.XL or M5a.XL (16GB RAM REQUIRED // 50GB MIN STORAGE Recommended)
  - Install Docker / Docker Compose on your EC2 or Physical System
  - `sudo sysctl -w vm.max_map_count=262144` This step is required on linux systems.
- Clone this project to your Docker System.  
  - `git clone https://github.com/pingidentity/pingidentity-devops-getting-started.git`
- Within the Project change directory to the following path. `pingidentity-devops-getting-started/11-docker-compose/11-siem-stack/`
- Create and place a file `.env` in the above path and place these lines in it (UPDATE YOUR DEVOPS KEY AS SHOWN).

```
COMPOSE_PROJECT_NAME=es
ELASTIC_VERSION=7.6.1

ELASTIC_PASSWORD=2FederateM0re
ES_ADMIN_PD_USER_PASS=FederateTheB3st!

PING_IDENTITY_DEVOPS_USER={YOUR DEVOPS USER NAME HERE}       <====== NOTICE THIS
PING_IDENTITY_DEVOPS_KEY={YOUR DEVOPS KEY HERE}              <====== NOTICE THIS
```

- Start the stack with `docker-compose up -d`  

- (Optional) Run the Slack configuration script to configure slack alerts `./config_slack_alerts`
  - The first time you must provide your webhook URL that you created above.
  - You can re-run this script any time which will update and push new watchers you create from the `./elasticsearch-siem/watchers` folder! 
  - You don't need to provide your webhook in the future. If you don't provide it, it simply will not update it.
  - The script asks for webhook URL and elasticsearch password.
    - The webhook URL updates the destination for your alerts within slack
    - The password is used to push watchers into elasticsearch

- Monitor the stack with `docker-compose logs --follow`

- Login to kibana at `https://localhost:5601/` (Wait for PingDirectory to full start)
  - The stack is now LDAP INTEGRATED so you can login with `es_admin` or `elastic` (local user) which is a user in PingDirectory (Password is in the .env file configuration listed above (ES_ADMIN_PD_USER_PASS). Allow time for Ping Directory to load.

------------

## Included Slack Alerts (these can be customized through Watchers)
 - User Authenticates over 1200km away within 6 hour period.
 - User Authenticates successfully from TOR through Ping Federate. (potential credential theft)
 - User Authenticates successfully from Known Malicious IP through Ping Federate. (potential credential theft)
 - Account Lockout detected through Ping Federate. (potential brute force)
 - Likely SAML Signature Modifications (Forced Tampering with Authentication Protocols)

------------


## Slack Alert Examples (not all are shown)

### Low / Medium / High Alert Examples
![alt text](https://github.com/pingidentity/pingidentity-devops-getting-started/blob/master/11-docker-compose/11-siem-stack/images/slack_alert_examples.png "Successful Login From TOR Networks.")

------------
## Dashboard Examples

### Demo Ping Federate Threat Intel Dashboard
![alt text](https://github.com/pingidentity/pingidentity-devops-getting-started/blob/master/11-docker-compose/11-siem-stack/images/threat_intel_dash.png "SIEM Dashboard")

### Demo Ping SIEM Dashboard (Beta 4) - More security use cases are coming soon.
![alt text](https://github.com/pingidentity/pingidentity-devops-getting-started/blob/master/11-docker-compose/11-siem-stack/images/dashboard.png "SIEM Dashboard")

### Ping Federate Demo Dashboard
![alt text](https://github.com/pingidentity/pingidentity-devops-getting-started/blob/master/11-docker-compose/11-siem-stack/images/pingfederate_dashboard.png "PingFederate Basic Demo Dashboard")

### Demo Access Demo Dashboard
![alt text](https://github.com/pingidentity/pingidentity-devops-getting-started/blob/master/11-docker-compose/11-siem-stack/images/pingaccess_dashboard.png "PingAccess Basic Demo Dashboard")

### Demo Directory Demo Dashboard
![alt text](https://github.com/pingidentity/pingidentity-devops-getting-started/blob/master/11-docker-compose/11-siem-stack/images/pingdirectory_dashboard.png "PingDirectory Demo Dashboard")

------------

## PingDirectory
- Audit Logs are being delivered
- There are 2 containers that produce load
  -These are disabled by default, uncomment in docker-compose.yml if you wish to use.
    - authrate_ok
    - authrate_ko

------------

## PingFederate
- Audit and System logs are delivered. (DEBUG by DEFAULT)

------------

## PingAccess
 - Audit and System Logs are being delivered. (DEBUG by DEFAULT)

------------

## Kibana Access
- Kibana listens on https://{IP}:5601

- DEFAULT PASSWORDS  
    - UN is configured as `es_admin` (LDAP USER)
    - PASSWORD is configured in your `.env` file 

------------

## Kibana Saved Objects
- Saved Objects can be loaded by going to 'saved objects' under kibana settings and exporting all. Save the file in the...  
	- `./elasticsearch-siem/kibana_config/kib_base.ndjson` 

------------

## ElasticSearch Templates for Indexes
- Index mappings and config are stored in the index_templates folder
- The Scripts will load this template(s) once cluster state is green.
	- ./elasticsearch-siem/index_templates/****

------------
## Logstash Pipeline
- TOR Enrichment
- Threat Intel (Alien Vault Provided)
- GEO IP Lookup
- GEO Distance Query (template driven)
- Data Parsing
- Logstash Pipeline is stored in the folder structure. It includes Parsers for All Ping Log Sources.

------------
## PingFederate (log4J)
- Ping Fed ships logs on 2 different SYSLOG PORTS, with a CUSTOM mappings.
------------

------------
## PingDirectory (log4J)
- Ping Fed ships logs on 1 SYSLOG PORT, with a CUSTOM mapping.
------------

------------
## PingAccess (log4J)
- Ping Fed ships logs on 2 different SYSLOG PORTS, with a CUSTOM mappings.
------------
