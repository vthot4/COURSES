# Week 3. notes.



## Monitoring Network Security

### VPC Flow Logs.

- VPC Flow Logs record a sample (about 1 out of 10 packets) of network flows sent from and received by VM instances, including Kubernetes Engines nodes. These logs can be used for network monitoring, traffic analysis, forensics, real-time security analysis, and expense optimization.
- VPC Flow Logs is part of Andromeda, the software that powers VPC networks. VPC Flow Logs introduces no delay or performance penalty when enabled.

![image-20210701001845915](./images/image-20210701001845915.png)

![image-20210701001924615](./images/image-20210701001924615.png)

- You can enable or disable VPC Flow Logs per VPC subnet. Once enabled for a subnet, VPC Flow Logs collect data from all VM instances in that subnet.
- Each log entry contains a record of different fields. For example, this table illustrates the IP connection information that is recorded. This consists of the source IP address and port, the destination IP address and port, and the protocol number. This set is commonly referred to as 5-tuple.
- Other fields include the start and end time of the first and last observed packet, the bytes and packets sent, instance details including network tags, VPC details, and geographic details.

![image-20210701002055989](./images/image-20210701002055989.png)

- Exporting VPC Flow logs to BigQuery allows you to analyze your network traffic with SQL, to understand traffic growth patterns and network usage better.



### Firewall Rules Logging.

- VPC firewall rules let you allow or deny connections to or from your virtual machine (VM) instances based on a configuration that you specify.
- Enabled VPC firewall rules are always enforced, protecting your instances regardless of their configuration and operating system, even if they have not started up.

- Firewall Rules Logging allows you to audit, verify, and analyze the effects of your firewall rules. It can help answer questions like:
  - Did my firewall rules cause that application outage?
  - How many connections match the rule I just created?
  - Are my firewall rules stopping (or allowing) the correct traffic?

![image-20210701002635618](./images/image-20210701002635618.png)

![image-20210701002715592](./images/image-20210701002715592.png)

- Firewall rule logging can also be enabled on existing firewall rules using the CLI. See these two examples on this slide. In both, [NAME] would be the name of your firewall rule.

![image-20210701002807440](./images/image-20210701002807440.png)

![image-20210701002911853](/home/vthot4/FORMACION/COURSES/Logging_Monitoring_Observability_GCP/images/image-20210701002911853.png)

- Google Cloud VPC Firewalls are micro-segmentation firewalls.
- These function more like a bunch of micro firewalls, each sitting on the NIC of every VM connected to the VPC.
- The micro firewalls can then grant or deny any configured incoming or outgoing traffic. 
- Now, imagine we have an issue. We have two different web servers, and after some configuration changes by a particular DevOps team, the web servers can no longer access the application server they both share.

![image-20210701003008042](./images/image-20210701003008042.png)



### Cloud NAT Logs

![image-20210701004054753](./images/image-20210701004054753.png)



- Cloud NAT (network address translation) allows Google Cloud virtual machine (VM) instances without external IP addresses and private Google Kubernetes Engine (GKE) clusters to send outbound packets to the internet and receive any corresponding established inbound response packets.
- Cloud NAT is a distributed, software-defined, fully managed service, grounded in the Andromeda software that powers your VPC network. It provides source network address translation (SNAT) for VMs without external IP addresses, as well as destination network address translation (DNAT) for established inbound response packets.
- Cloud NAT benefits include:
  - Security: You can reduce the need for individual VMs to have external IP addresses, lessening the surface area for attack. You can also confidently share a set of common external source IP addresses with a destination party.
  - Availability: Cloud NAT is a distributed, software-defined, managed Google Cloud service. It doesn't depend on any VMs in your project or a single physical gateway device.
  - Scalability: Cloud NAT can be configured to automatically scale the number of NAT IP addresses it uses, and it supports VMs that belong to managed instance groups, including those with autoscaling enabled.
  - Performance: Cloud NAT does not reduce the network bandwidth per VM. Cloud NAT works directly with Google's Andromeda software-defined
  - networking.

![image-20210701004343908](./images/image-20210701004343908.png)

- Cloud NAT logging allows you to log NAT TCP and UDP connections and errors. When Cloud NAT logging is enabled, a log entry can be generated when a network connection using NAT is created, and/or when an egress packet is dropped because no port was available for NAT.
- You can opt to log both kinds of events, or just one or the other. Logs contain TCP and UDP traffic only, and the log rate threshold will max out at 50-100 log events per vCPU before log filtering.
- Cloud NAT logging may be enabled when a new Cloud NAT gateway is first created, or by editing an existing gateway's settings.
- To view the collected logs in Logs Explorer, filter to the Cloud NAT Gateway resource and optionally, restrict to a particular region or Gateway.
- The full query will look something like:
  `resource.type="nat_gateway"`
  `logName="projects/{#project_id}/logs/compute.googleapis.com%2Fnat_flows"`



### Packet Mirroring

- Packet Mirroring clones the traffic of specified instances in your Virtual Private Cloud (VPC) network and forwards it for examination. Packet Mirroring captures all ingress and egress traffic and packet data, such as payloads and headers.
- The mirroring happens on the virtual machine (VM) instances, not on the network. Consequently, Packet Mirroring consumes additional bandwidth on the hosts.
- Packet Mirroring is useful when you need to monitor and analyze your security status. It exports all traffic, not only the traffic between sampling periods. For example, you can use security software that analyzes mirrored traffic to detect all threats or
  anomalies.
- Additionally, you can inspect the full traffic flow to detect application performance issues and to provide network forensics for PCI compliance and other regulatory use cases.
- Obviously, this can generate a lot of data, so the recommended target is a load-balanced Compute Engine Managed Instance Group or equivalent technology.

![image-20210701005058198](./images/image-20210701005058198.png)



- Packet Mirroring exports monitoring data about mirrored traffic to Cloud Monitoring. You can use monitoring metrics to check whether traffic from a VM instance is being mirrored as intended.

![image-20210701005308975](./images/image-20210701005308975.png)



### Network Intelligence Center.

- Google's Network Intelligence Center is all about giving you centralized monitoring and visibility into your network, reducing troubleshooting time and effort, increasing network security, all while improving the overall user experience.
- Currently, it offers four modules: network topology, connectivity testing, a performance dashboard, and firewall insights.

![image-20210701005711304](./images/image-20210701005711304.png)

![image-20210701005806439](./images/image-20210701005806439.png)

![image-20210701005844690](./images/image-20210701005844690.png)

![image-20210701005920859](./images/image-20210701005920859.png)

- Firewall Insights enables you to better understand and safely optimize your firewall configurations by analyzing Firewall Rules logs and providing reports on firewall usage, and the impact of various firewall rules on your VPC. For each firewall rule with logging enabled, you can see:
  - How many times the firewall rule has blocked or allowed connections.
  - The last time a particular firewall rule was applied to allow or deny traffic.
  - A list of firewall rules that haven't been used in the last six weeks.
  - and Shadowed firewall rules. (A shadowed rule is a firewall rule that has all of its relevant attributes, such as IP address range and ports, overlapped by attributes from one or more other firewall rules with higher or equal priority. A firewall doesn't evaluate a shadowed rule because of the overlap and because the shadowed rule has a lower priority than its shadowing rules.)



## Monitoring Audit Logs

### Audit Logs.

- Cloud Audit Logs help answer the question, "Who did what, when, and where?” It maintains three audit logs for each Google Cloud project, folder, and organization: Admin Activity, Data Access, and System Event.
- All Google Cloud services will eventually provide audit logs. For now, see the Google services with audit logs documentation for coverage details.
- Admin Activity audit logs contain log entries for API calls or other administrative actions that modify the configuration or metadata of resources. For example, these logs record when users create VM instances or change Cloud Identity and Access Management permissions. They are always on, are retained for 400 days, and are available at no charge. To view these logs, you must have the Cloud IAM role Logging/Logs Viewer or Project/Viewer.
- System Event audit logs contain log entries for Google Cloud administrative actions that modify the configuration of resources. System Event audit logs are generated by Google systems; they are not driven by direct user action. They are always enabled, free, retained for 400 days, and to view these logs, you must have the Cloud IAM role Logging/Logs Viewer or Project/Viewer.
- Data Access audit logs contain API calls that read the configuration or metadata of resources, as well as user-driven API calls that create, modify, or read user-provided resource data. Data Access audit logs do not record the data-access operations on resources that are publicly shared (available to All Users or All Authenticated Users), or that can be accessed without logging into Google Cloud. They are disabled by default (except for BigQuery), and when enabled, the default retention is 30 days.
  To view these logs, you must have the Cloud IAM roles Logging/Private Logs Viewer or Project/Owner.

![image-20210701010850563](./images/image-20210701010850563.png)

![image-20210701010946802](./images/image-20210701010946802.png)

![image-20210701011038889](./images/image-20210701011038889.png)



### Data Access Logging.

- Data Access logs can be enabled at the organization, folder, project, or resources levels.

![image-20210701011622317](./images/image-20210701011622317.png)

![image-20210701011709583](./images/image-20210701011709583.png)

- There are three types of Data Access audit log information:
  - Admin-read: Records operations that read metadata or configuration information. For example, you looked at the configurations for your bucket.
  - Data-read: Records operations that read user-provided data. For example, you listed files and then downloaded one from GCS.
  - Data-write: Records operations that write user-provided data. For example, you created a new GCS file.

![image-20210701011821575](./images/image-20210701011821575.png)

![image-20210701011902002](./images/image-20210701011902002.png)

- You can also use gcloud or the API to enable data access logging. 
- If you're using gcloud, frequently, the easiest way is to get the current IAM policies, as seen in the bullet, and write them to a file.
- Then you can edit the file to add or edit the auditLogConfigs. You can also add the log details per service, like this example is enabling logging for cloud run, or even enable logging on all services.
- Then, as seen in the bullet, you would set that as the new IAM policy.



### Understanding Audit Logs.

![image-20210701012254935](./images/image-20210701012254935.png)

- Every audit log entry in Cloud Logging is an object of type LogEntry.
- What distinguishes an audit log entry from other log entries is the protoPayload field, which contains an AuditLog object that stores the audit logging data.



### Best Practices.

![image-20210701012612937](./images/image-20210701012612937.png)

- Spend time and create a solid plan for Data Access logs. Think organization, folder, then project. Like most organizations, some of your projects will be very specialized, but usually, they do break down into common organizational types.

![image-20210701012708288](./images/image-20210701012708288.png)

![image-20210701012814534](./images/image-20210701012814534.png)

- We've discussed the options and benefits of exporting logs. Again, make this part of your plan.
- Start by deciding what, if anything, you will export from Aggregated Exports at the organization level.
- Then decide what options you will use, project by project, folder by folder, etc.
- Then, carefully consider your filters - both what they leave in, and what they leave out.
- This applies to all logging, not just to exports.
- Lastly, carefully consider what, if anything, you will fully exclude from logging.
- Remember that excluded entries will be gone forever.

![image-20210701012914300](./images/image-20210701012914300.png)

- Side-channel leakage of data through logs is a common issue. You need to be careful who, gets what kind of access, to which logs.
- Remember some of the discussions earlier in this course on monitoring workspaces and how a monitoring workspace can monitor the current project, or it can also monitor up to 100 other projects? That's where your security starts. Are you monitoring project by project, or are you selectively grouping work projects into higher-level monitored projects?
- Use appropriate IAM controls on both Google Cloud-based and exported logs, only allowing the minimal access required to get the job done.
- Especially scrutinize the Data Access Log permissions, as they will often contain Personally Identifiable Information (PII)

![image-20210701013035653](./images/image-20210701013035653.png)

![image-20210701013115300](./images/image-20210701013115300.png)

![image-20210701013150025](./images/image-20210701013150025.png)



## Incident Management.

- An alert indicates that something bad is happening, or is on track to happen. As we've discussed in this course, "something bad" is a generic way of saying that one of our Service Level Objectives is being violated, or might be on track to be violated if things are left as they are. An alert doesn't mean your client is yelling at you; in fact, in an ideal world, they may not have noticed anything, yet.
- An incident is the declared start of an Incident response process, and that’s all about what you do next. If you declare an ankle incident then you might have to go easy, perhaps put ice on it for awhile, or you might need to go to the emergency room. With software systems you need to answer the question, does this alert (pain) trigger a response, or is it a simple anomaly?
  In other words, can I safely ignore it or do I need to start some form of incident response (for example does it need a doctor)? Who is it that makes that decision? Does the client need to be informed? How about the boss? Does someone need to roll out of bed, or can it wait? All of this should be part of your standard incident response methodology, and this should all kicks in when an incident is declared.



![image-20210707103733568](./images/image-20210707103733568.png)

- Start in the lower left. Things are going along fine when bang! Alerts start firing. This moves you into the first major incident state: Detected. Something untoward is happening, but that's all you know.
- Triage is started. How bad is it? What's happening? And someone needs to make a decision: is this an incident or a simple anomaly. That takes you to the next incident stage: Triaged. Let's say that triage decides that this is something that has to be dealt with, so an incident is declared, and an initial severity is set.
- You and your team are working on the problem. The longer it goes on, the worse it gets. Your goal here is to make things better and, if possible, to fix the problem.
- When the patch is in place, your incident moves into the next stage, Mitigated. At worst, you've lessened the problem. At best, the client is no longer being impacted.
- Now you do your root cause analysis. Why did this happen? How can you make sure it never happens again. Once you can answer those questions, your incident moves into its final phase: Resolved.
- You write up your post mortem documentation, so everyone knows what happened, why, and what we've done to make sure it doesn't happen again. You submit it and move on.



- The time to decide how to handle an incident is not when it's happening. You, and your organization, need to develop a structured incident response methodology. You want to build an SRE culture:
  - On collaboration and clear communication.
  - Which learns and shares that knowledge.
  - With a predefined, and organized information flow
  - And, with a clearly organized approach to client notification.

- Once your SRE foundations are in place, then it's time to practice. That way, you can be prepared and respond quickly, because you'll have the checklists, and you know what to do when.
- As you and your team train, you'll also learn to be consistent. Like the pilots, you want to reduce duplication of effort and know who's doing what.

![image-20210708181136139](./images/image-20210708181136139.png)

![image-20210708181353048](./images/image-20210708181353048.png)

- The **Incident Commander (IC)** is at the top of the team chain of command, and during an incident, every member of the response team reports to the Incident Commander. The IC's role isn't to personally fix the problem, but to make sure the problem gets fixed. Think management and organization, not technical wizardry.
- IC responsibilities include building and maintaining the team, coordinating the parts of the response, deciding priorities, and delegating activities. It's the IC that always knows the current status of significant events during the incident, and the IC will also handle communication if a communication lead is not assigned.
- Next, we have the **Communications Lead (CL).** The Communications Lead is responsible for clear, timely communication. The CL keeps everyone outside the response team informed, and it's the CL that fields questions, and who decides what should be released and through which communication channels. So the CL faces the outside world and lets the team work in peace. Direct communication between the client and other team members should be actively discouraged.
- The **Operations Lead (OL)** manages the immediate, detailed, technical, and tactical work of the incident response. The OL develops and executes the incident action plan, requests additional resources as needed, allocates resources among the various operational response, and maintains close contact with the Incident Commander, Communications Lead, Primary and Secondary Responders, and others involved with handling the incident.
- The **Operations Lead leads**, and works closely with, the responders.
- The **Primary Responder** executes the operation lead's technical response plan, and maintains close contact with the Operations Lead and, if there is one assigned, uses the Secondary Responder for additional technical support.
- The **Secondary Responder** role assists the Primary Responder if they need help on the particular incident.



- Best practices before the incident:

  - Establish clear criteria for declaring an incident and remember, err on the side of caution. Better false incidents, than missed.
  - Choose a communications channel in advance and make sure SRE members and clients know the plan.
  - Create templates for communications. Templates are easy to fill in, save time, and help avoid missing information.
  - Train. Train. Train. Train the team on how to respond in theory, and then drill them to make sure they know how to practice what they've learned. Drilling also helps avoid panic.

  

- A postmortem report is a detailed and organized document outlining the key events of the incident. It's a blameless report containing the impact, root cause, triggering event, the resolution, any quantifiable metrics, action items, lessons learned, and a timeline.

![image-20210709010637690](./images/image-20210709010637690.png)

- Postmortems are letters to your future self and team. They help with future troubleshooting and provide starting points and clarity on future mitigations.

![image-20210709010848128](./images/image-20210709010848128.png)

