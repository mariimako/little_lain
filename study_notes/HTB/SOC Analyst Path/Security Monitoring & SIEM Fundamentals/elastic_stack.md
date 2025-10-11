---
title: Elastic Stack
parent: Security Monitoring & SIEM Fundamentals
layout: default
---
# The Elastic Stack
Collection of three applications, Elasticsearch, Logstash, and Kibana. Search and visualization for real-time analysis and exploration of log files.

![[Screenshot 2025-10-06 at 11.42.20 PM.png]]

Is enhanced for resource intensive environments with Kafka, RabbitMQ, and Redis for buffering, resieleincy, and nginx for security.
![[Screenshot 2025-10-06 at 11.46.27 PM.png]]

- **Elastic Search** distributed, JSON based search engine. Handles indexing, storing and querying. Used to conduct analysis on Logstash
- **Logstash** collecting, transofmring, transporting log gile records. consolidate data from various sources and normalize them
	- Process input (converts log files from remote locations)  :: input methods can be reading from flat file, TCP socket, syslog messages
	- Transform and enrich log recorsd: filter plugins can perform intermediary processing on an event
	- Send log records to Elasticsearch
- **Kibana** visualization tool. View the data stored in Elasticserch and execute queries through Kibana
- **Beats** forward logs and metrics to Logstash or Elasticssearch
## Elastic Stack as SIEM Solution

Can be used as Security Information and Event Management.

Kibana Query Language is a good skill to have.

KQL queries have `field:value` pairs, field being attribute and value representing data.

`event.code:4625`
Show events that have event code 4625, failed log in events.
`"svc-sql1"`
free search using quotes

Support `AND OR NOT` 
``event.code:4625 AND winlog.event_data.SubStatus:0xC0000072``
Failed login + substatus code, indicating account disabled 
Also,
`: :> :>= :< :<= :!` 
Wildcards and regex
```shell-session
event.code:4625 AND winlog.event_data.SubStatus:0xC0000072 AND @timestamp >= "2023-03-03T00:00:00.000Z" AND @timestamp <= "2023-03-06T23:59:59.999Z"
```
Additional Resources:
- [Elastic Common Schema (ECS)](https://www.elastic.co/guide/en/ecs/current/ecs-reference.html)
- [Elastic Common Schema (ECS) event fields](https://www.elastic.co/guide/en/ecs/current/ecs-event.html)
- [Winlogbeat fields](https://www.elastic.co/guide/en/beats/winlogbeat/current/exported-fields-winlog.html)
- [Winlogbeat ECS fields](https://www.elastic.co/guide/en/beats/winlogbeat/current/exported-fields-ecs.html)
- [Winlogbeat security module fields](https://www.elastic.co/guide/en/beats/winlogbeat/current/exported-fields-security.html)
- [Filebeat fields](https://www.elastic.co/guide/en/beats/filebeat/current/exported-fields.html)
- [Filebeat ECS fields](https://www.elastic.co/guide/en/beats/filebeat/current/exported-fields-ecs.html)

## Elastic Common Schema
A shared extensible vocabulary for events and logs across Elastic Stack. Ensures consistent field formats across different data sources

- Unified Data View, regardless of data origin
- Better search efficiency
- Easier correlation of events across sources: e.g. IP addrs 
- Visualizations
Zeek can also be used: Network Security Monitor