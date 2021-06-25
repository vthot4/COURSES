# week 1. Notes.



## Monitoring Tools

**Google Cloud Observability**

![image-20210625211027234](./images/image-20210625211027234.png)

**Application and infrastructure observability**

![image-20210625211631826](./images/image-20210625211631826.png)



- Google Cloud has many products, from Kubernetes, to BigQuery, to Spanner, and they all stream metrics and logs into Google's Cloud Logging and Cloud Monitoring components.
- The Logs Router determines where the data goes and can be used to exclude some types of entries,  or to route logs to external locations like Pub/Sub or BigQuery, perhaps for automated handling and/or long term storage and analysis.



## Operations-Based Tools

Let's start with the products that tend to be of interest for the operations folk:

- Monitoring.
- Logging.
- Error reporting.
- Service monitoring.

As we stated previously, monitoring starts with signal data. And if those same DevOps teams want to augment the signal metrics coming out of their custom application wherever it's running, they could use the open source Open Telemetry and create their own metrics.



**Resource monitoring**

![image-20210625213146313](./images/image-20210625213146313.png)

- Cloud Monitoring provides visibility into the performance, uptime, and overall health of cloud-powered applications.
- It collects metrics, events, and metadata from projects, logs, services, systems, agents, custom code, and various common application components.
- Monitoring ingests that data and generates insights via dashboards, metrics Explorer charts, and automated alerts.



![image-20210625214237905](./images/image-20210625214237905.png)

- Google's Cloud Logging is all about collecting, storing, searching, analyzing, monitoring, and alerting on log entries and events.
- Export log data as files to Cloud Storage as messages through Pub/Sub or into BigQuery tables. Logs-based metrics may be created and integrated into Cloud Monitoring dashboards, alerts and service SLOs.
- Default log retention in Cloud Logging depends on the log type. Data access logs are retained by default for 30 days, but this is configurable up to a max of 3650 days. Admin logs are stored by default for 400 days. You can export logs to Cloud Storage
  or BigQuery to extend retention periods.



**Available logs**

![image-20210625214602042](./images/image-20210625214602042.png)

Three key log categories are audit logs, agent logs, and network logs.
1. Cloud Audit Logs help answer the question, "Who did what, where, and when?" Admin Activity tracks configuration changes. Data Access tracks calls that read the configuration or metadata of resources, as well as user-driven calls that create, modify, or read user-provided resource data. System Events are non-human Google Cloud administrative actions that change the configuration of resources. Access Transparency provides you with logs that capture the actions Google personnel take when accessing your content.
2. Agent logs use a Google-customized and packaged Fluentd agent that can be installed on any AWS or Google Cloud VM to ingest log data from Google Cloud instances (for example, Compute Engine, Managed VMs, or Containers) as well as AWS EC2 instances.
3. Network logs provide both network and security operations with in-depth network service telemetry. VPC Flow Logs record samples of VPC network flow and can be used for network monitoring, forensics, real-time security analysis, and expense optimization. Firewall Rules Logging allows you to audit, verify, and analyze the effects of your firewall rules. NAT Gateway logs capture information on NAT network connections and errors.



![image-20210625214857747](./images/image-20210625214857747.png)

- Error Reporting counts, analyzes, and aggregates the crashes in your running cloud services. Its management interface displays the results with sorting and filtering capabilities. A dedicated view shows the error details: time chart, occurrences, affected user count, first- and last-seen dates, and a cleaned exception stack trace. You can also create alerts to receive notifications on new errors.



![image-20210625215053153](./images/image-20210625215053153.png)

- Service Monitoring helps organizations understand and troubleshoot intra-service dependencies.



## Application Performance Management Tools.

