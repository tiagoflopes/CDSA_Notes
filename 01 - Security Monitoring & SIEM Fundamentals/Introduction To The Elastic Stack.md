## What Is The Elastic Stack

Created by Elastic, is an open-source collection of mainly 3 applications (Elasticsearch, Logstash and Kibana) that work in harmony to offer users comprehensive search and visualization capabilities for real-time analysis and exploration of log file sources.
The high-level architecture of the Elastic stack can be enhanced in resource-intensive environments with the addition of Kafka, RabbitMQ and Redis for buffering and resiliency, and nginx for security.
![[elastic_architecture.webp]]
Note: *Beats* is an additional component of the Elastic stack. These are data shippers designed to be installed on remote machines, to forward logs and metrics.
##### Elasticsearch
It is a distributed and JSON-based search engine, designed with RESTful APIs. As the core component, it handles indexing, storing and querying. It empowers users to conduct analysis operations on the log file records processed by Logstach.
##### Logstash
It is responsible for collecting, transforming and transporting log file records. Its strength lies in the ability to consolidate data from various sources and normalize them. It operates in 3 main areas:
- Process input - ingests log file records from remote locations, converting them into a format that machines can understand. It can receive records through different input methods such as TCP sockets and direct syslogs.
- Transform and enrich log records - modifies a log record's format and even content, often based on a predefined condition.
- Send log records to Elasticsearch - utilizes output plugins to transmit log records to Elasticsearch.
##### Kibana
It serves as the visualization tool for Elasticsearch documents. Users can view data and execute queries through Kibana.

## The Elastic Stack As A SIEM Solution

The Elastic stack can be used as a SIEM solution to collect, store, analyze and visualize security-related data from various sources.
To implement it, security-related data should be ingested into the Elastic stack using Logstash. Elasticsearch should be configured to store and index the security data and Kibana should be used to create custom dashboards and visualizations to provide insights into security-related events.
As SOC analysts, we are likely to extensively use Kibana as our primary interface, therefore it is essential to become proficient with its functionalities and features.
Kibana Query Language (KQL) is a query language designed specifically for searching and analyzing data in Kibana. It simplifies the process of extracting insights from indexed Elasticsearch data and it is more intuitive then Elasticsearch's Query DSL.
##### Basic Structure
KQL queries are composed of **field:value** pairs, with the field representing the data's attribute and the value representing the data we are searching for. For example: *event.code:4625*.
This query filters data to show events that have the [Windows event code 4625](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4625) (failed login attempts in a Windows OS).
##### Free Text Search
It allows searches for a specific term across multiple fields without specifying a field name. For instance: *"svc-sql1"*.
This returns records containing the string "svc-sql1" in any indexed field.
##### Logical Operators
It supports **AND**, **OR** and **NOT** for constructing more complex queries. For example: *event.code:4625 AND winlog.event_data.SubStatus:0xC0000072*.
This query filters data to show events that have the Windows event code 4625 and the SubStatus value of 0xC0000072 (the account is currently disabled).
##### Comparison Operators
It supports **:**, **:>**, **:>=**, **:<**, **:<=** and **:!** to define precise conditions for matching field values. 
For instance: *event.code:4625 AND winlog.event_data.SubStatus:0xC0000072 AND @timestamp >= "2023-03-03T00:00:00.000Z" AND @timestamp <= "2023-03-06T23:59:59.999Z"*.
##### Wildcards and Regular Expressions
It supports wildcards and regex to search for patterns in field values. For example: *event.code:4625 AND user.name: admin\**.
This query filters data to show events that have the Windows event code and where the username starts with "admin".

## How To Identify The Available Data

##### Leverage KQL's free text search
By using the Discover feature, we can gain insights into the architecture of the available fields, before we start constructing KQL queries.
##### Leverage Elastic's documentation
- [Elastic Common Schema (ECS)](https://www.elastic.co/guide/en/ecs/current/ecs-reference.html)
- [Elastic Common Schema (ECS) event fields](https://www.elastic.co/guide/en/ecs/current/ecs-event.html)
- [Winlogbeat fields](https://www.elastic.co/guide/en/beats/winlogbeat/current/exported-fields-winlog.html)
- [Winlogbeat ECS fields](https://www.elastic.co/guide/en/beats/winlogbeat/current/exported-fields-ecs.html)
- [Winlogbeat security module fields](https://www.elastic.co/guide/en/beats/winlogbeat/master/exported-fields-security.html)
- [Filebeat fields](https://www.elastic.co/guide/en/beats/filebeat/current/exported-fields.html)
- [Filebeat ECS fields](https://www.elastic.co/guide/en/beats/filebeat/current/exported-fields-ecs.html)

## The Elastic Common Schema (ECS)

ECS is a shared and extensible vocabulary for events and logs across the Elastic Stack, which ensures consistent field formats across different data sources. Here are some key advantages in using them:
- Unified Data View - it enforces a structured and consistent approach to data, allowing for unified views across multiple data sources.
- Improved Search Efficiency - by standardizing the field names across different data types.
- Enhanced Correlation - you can correlate an IP address involved in a security incident with network traffic logs, firewall logs and endpoint data.
- Better Visualizations - all data sources adhere to the same schema, so creating dashboards and visualizations becomes easier and more intuitive.
- Interoperability with Elastic Solutions - ensures full compatibility with advanced features such as Elastic Security, Elastic Observability and Elastic Machine Learning.
- Future-proofing - ensures future compatibility with enhancements and new features that are introduced into the Elastic ecosystem.

## Questions

- Navigate to http://[Target IP]:5601, click on the side navigation toggle, and click on "Discover". Then, click on the calendar icon, specify "last 15 years", and click on "Apply". Finally, choose the "windows*" index pattern. Now, execute the KQL query that is mentioned in the "Comparison Operators" part of this section and enter the username of the disabled account as your answer. Just the username; no need to account for the domain.
	- anni

- Now, execute the KQL query that is mentioned in the "Wildcards and Regular Expressions" part of this section and enter the number of returned results (hits) as your answer.
	- 8