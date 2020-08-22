# Agnostic Watcher 👁

An observability tool designed to be agnostic to data sources.

⚡️ Create rules using the language syntax from a data source which rule will look at

⚡️ Emit alerts to multiple targets

## Architecture
![alt text](https://i.imgur.com/SQcGAqu.png "Agnostic Watcher architecture")

👉 &nbsp;&nbsp;__Rule Register__ is responsible for rule registration. This process involves getting the list of available data sources, get the list of available alert targets, and notify rule worker about the new registered rule. In other words, it acts as an API Gateway: wait for the payload to be filled and send it to downstream services. A sample payload of a rule could be:
```json
{
    "name": "Rule to find dogs that didn't bark in last 10 minutes",
    "datasource_database": "MySQL",
    "datasource_address": "sampleds.mysql.mydomaim.com",
    "cron": "*/10 * * * *",
    "query": "SELECT * FROM dogs WHERE last_time_barked <= NOW() - INTERVAL 10 MINUTE",
    "minimum_results_threshold": 1,
    "alert": {
        "target": "slack",
        "channel": "#kennel-alerts"
    }
}
```

👉 &nbsp;&nbsp;__Rule Worker__ is responsible to spread new rules through services, executing these steps:
1. Create a job for a new rule. This job must run according to the rule's cron and publish eventual matches to rule's Kafka topic.
2. Create a topic for the new rule in Kafka. This topic will receive rule's matches, as described on the first item.
3. Create a new Matches Consumer for the new rule. It is charged with getting rule's matches from rule's Kafka topic.

👉 &nbsp;&nbsp;__Data Source Register__ is charged with keeping the catalogue of data sources and perform CRUD operations on it.

👉 &nbsp;&nbsp;__Alert Emitter__ is responsible for sending the alerts to the designated target and to keep the catalogue of available alert targets and perform CRUD operations on it.
