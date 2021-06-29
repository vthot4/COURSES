# Monitoring and Dashboarding Multiple Projects from a Single Workspace

2 hoursFree

## Overview

Google Cloud Monitoring empowers users with the ability to monitor 1..100 projects from a single monitoring Workspace. In this exercise, you start with three Google Cloud projects, two with monitorable resources, and the third you use to host a central monitoring workspace. You attach the two resource projects to the monitoring Workspace, build uptime checks, and construct a centralized dashboard.

### Objectives

- Configure a Worker project.
- Create a Monitoring Workspace and link the two worker projects into it.
- Create and configure Monitoring Groups.
- Create and test an uptime check.

## Task 0. Lab Setup

### Access Qwiklabs

For each lab, you get a new GCP project and set of resources for a fixed time at no cost.

1. Make sure you signed into Qwiklabs using an **incognito window**.
2. Note the lab's access time (for example, ![img/time.png](https://cdn.qwiklabs.com/aZQJ4BT7uCmM9XR6BTXgTRP1Hfu1T7q6V%2BcnbdEsbpU%3D) and make sure you can finish in that time block.

There is no pause feature. You can restart if needed, but you have to start at the beginning.

1. When ready, click ![img/start_lab.png](https://cdn.qwiklabs.com/XE8x7uvQokyubNwnYKKc%2BvBBNrMlo5iNZiDDzQQ3Ddo%3D).
2. Note your lab credentials. You will use them to sign in to Cloud Platform Console. 
3. Click **Open Google Console**.
4. Click **Use another account** and copy/paste credentials for **this** lab into the prompts.

If you use other credentials, you'll get errors or **incur charges**.

1. Accept the terms and skip the recovery resource page.

Do not click **End Lab** unless you are finished with the lab or want to restart it. This clears your work and removes the project.

After you complete the initial sign-in steps, the project dashboard appears.

![Google Cloud Project Dashboard](https://cdn.qwiklabs.com/dxfeoOcn1ObyC0BYyoqqqSi4rO%2FeMdbWPFjoK6C0YYk%3D)

### Activate Google Cloud Shell

Google Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Google Cloud Shell provides command-line access to your GCP resources.

1. In GCP console, on the top right toolbar, click the Open Cloud Shell button.

   ![Cloud Shell icon](https://cdn.qwiklabs.com/vdY5e%2Fan9ZGXw5a%2FZMb1agpXhRGozsOadHURcR8thAQ%3D)

2. Click **Continue**. ![cloudshell_continue.png](https://cdn.qwiklabs.com/lr3PBRjWIrJ%2BMQnE8kCkOnRQQVgJnWSg4UWk16f0s%2FA%3D)

It takes a few moments to provision and connect to the environment. When you are connected, you are already authenticated, and the project is set to your *PROJECT_ID*. For example:

![Cloud Shell Terminal](https://cdn.qwiklabs.com/hmMK0W41Txk%2B20bQyuDP9g60vCdBajIS%2B52iI2f4bYk%3D)

**gcloud** is the command-line tool for Google Cloud Platform. It comes pre-installed on Cloud Shell and supports tab-completion.

You can list the active account name with this command:

```
gcloud auth list
content_copy
```

Output:

```output
Credentialed accounts:
 - <myaccount>@<mydomain>.com (active)content_copy
```

Example output:

```Output
Credentialed accounts:
 - google1623327_student@qwiklabs.netcontent_copy
```

You can list the project ID with this command:

```
gcloud config list project
content_copy
```

Output:

```output
[core]
project = <project_ID>content_copy
```

Example output:

```Output
[core]
project = qwiklabs-gcp-44776a13dea667a6content_copy
```

Full documentation of **gcloud** is available on [Google Cloud gcloud Overview ](https://cloud.google.com/sdk/gcloud).

## Task 1. Configure the resource projects

Your Qwiklabs environment has three projects pre-created in it, the project IDs are displayed in the upper-left corner of the Qwiklabs lab steps page.

![Multiple projects](https://cdn.qwiklabs.com/YSYl63KqoOqGgz9Cj%2FqISbU6l9STKjwTXQEmXbdiTeE%3D)

The first project (ID 1) will be the monitoring workspace host project. Projects ID 2 and ID 3 will be the monitored/resource projects. Per Google's recommended best practices, the project we use to host the monitoring workspace will not be one of the projects actually housing monitored resources.

In this task, you:

- Create an NGINX web server in each of your worker projects.

### Configure two resource projects

Let's start by building some resources to monitor.

1. Open a text document on your computer and in it, make note of the three project IDs. Label Project ID 1 as **Monitoring Project**, label Project ID 2 as **Worker 1**, and Project ID 3 as **Worker 2**. In the rest of this exercise, the project IDs are referred to by these names.
2. In the Google Cloud Console page, use the project dropdown near the upper-left corner of the interface to switch to the `Worker 1` project. Remember, it is the project with the ID you labeled **Worker 1** in the text file you created in Step 1.

**Note:** If you don't see all three projects, type **qwiklabs** in the search box and the missing projects should appear.

1. Open the Google Cloud menu using the **Navigation menu** (![console_nav_small.png](https://cdn.qwiklabs.com/%2BnHJDKUfemAVQPMPb%2BAcqH0ZqkvrGALIBL1p582JNCQ%3D)) in the upper-left corner of the console interface, then click the link to the **Marketplace**.
2. In the search box, type **NGINX Bitnami**. Click the **NGINX Open Source Certified by Bitnami** that pops up in the results, and then click the **Launch** button.
3. Set the Deployment name to `worker-1-server` and **Deploy** it.
4. Once the `Deployment Manager` interface comes up, don't wait for the deploy to finish, move on to the next step.
5. Use the **Navigation menu** to head back to the **Home** page for the project.
6. Use the project dropdown to switch to `Worker 2` project.
7. Perform similar steps in Worker 2:

- Visit the Marketplace.
- Search for **NGINX Open Source Certified by Bitnami** and **Launch** it.
- Set **Deployment name** to `worker-2-server` and **Deploy** it.

1. Once Deployment Manager starts, use the **Navigation menu** to navigate to the Google Cloud Console home page.
2. Use the project dropdown to switch back to `Worker 1` project.
3. Use the **Navigation menu** to view your new **Compute Engine | VM instances**.
4. If you don't see an `External IP` for worker-1-server-vm, wait a minute and refresh the page. Continue to do so until you get an external IP.
5. Copy the `External IP` and paste it in a new browser tab. Make sure you can see your new Worker 1 web server.
6. In the text file you created in Step 1, add a new entry **worker-1-server-vm:**, and next to it, paste your copied external IP.
7. Use the project dropdown to switch to `Worker 2` project. You should still be on the `VM instances` page. Again:

- Copy the `External IP` value.
- Paste it in the browser and view the new server 2 home page
- Add a new **"worker-2-server-vm:** entry into your text file and add its IP.

## Task 2. Create a Monitoring Workspace and link the two worker projects into it

There are a number of ways you might want to configure the relationship between the host project doing the monitoring, and the project or projects being monitored. In general, if you're going for the multiple projects being centrally monitored approach, then it's recommended that the monitoring project contains nothing but monitoring related resources and configurations. That's exactly the approach taken here.

In this task, you:

- Configure the central monitoring link to the Worker 1 and 2 projects.

### Configure the monitoring link to the Worker 1 and 2 projects

1. Use the project dropdown to switch to the `Monitoring Project`.
2. Use the **Navigation menu** to switch to the **Monitoring | Overview** page. Because this is the first time you visited Monitoring in this project, Google Cloud will automatically create a Monitoring Workspace. You may have to wait for the Workspace to be created.
3. Once the monitoring workspace is created and the Cloud Monitoring overview displays, in the left-hand menu, click **Settings**.

![Workspace Settings](https://cdn.qwiklabs.com/FD6Yil%2BxIU672ywcwyME77lydjsUfl6C%2BSgOc0S9jxg%3D)

1. Press **ADD GCP PROJECTS**.
2. Select the `Worker 1` and `Worker 2` projects.
3. Choose **Use this project as the scoping project** under **Select Scoping Project** and press **ADD PROJECTS**.
4. Switch to the **Dashboards** page. If you don't see anything, refresh the page and after a minute or two, you'll see the Disks, Firewalls, Infrastructure Summary, and VM Instances from the other two projects.
5. Click **VM Instances**. Take a few minutes and explore the dashboard.

![dashboard_overview.png](https://cdn.qwiklabs.com/arhXk%2Fqbj1TuJD4FNrl14G0VepGlBthhRgqS%2B5%2Fn3LA%3D)

1. Click **Dashboards** and take a few minutes exploring the other available dashboards, especially Infrastructure Summary.

## Task 3. Create and configure Monitoring Groups

Google Cloud Monitoring lets you monitor a set of resources together as a single group. Groups can then be linked to alerting policies, dashboards, etc. Each Workspace can support up to five-hundred groups and up to six layers of sub-groups. Groups can be created using a variety of criteria, including labels, regions, and applications.

You add a label `component=frontend` to each of the web servers as a way to track your servers that are externally facing. In this case, it also allow you to easily add them to the same group.

In this task, you:

- Assign labels to the web servers to make them easier to track.
- Create a resource group and place the servers into it.
- Create a sub-group just for frontend dev servers.

### Assign labels to your web servers to make them easier to track

1. Use the **Navigation menu** to navigate to the Google Cloud Console home page.
2. Use the project dropdown to switch to the `Worker 1` project.
3. Use the **Navigation menu** to navigate to your **Compute Engine | VM instances**.
4. Click the link to navigate to the **worker-1-server-vm** settings.
5. Press the **Edit** button.
6. Press the **+Add label** button and create a label with the key `component` and the value `frontend`.
7. Press the **+Add label** button and create a label with the key `stage` and the value `dev`.
8. **Save** the configuration changes.
9. Use the project dropdown to switch to the `Worker 2` project.
10. Perform similar tasks:

- Edit the settings for the **worker-2-server-vm**.
- **+Add label** `component=frontend`
- **+Add label** `stage=test` (Note the **test** value)
- **Save** the changes.

### Create a resource group and place the servers into it

1. Use the project dropdown to switch to the `Monitoring Project`, then navigate to the **Monitoring | Overview**.
2. In the left-hand menu, navigate to the **Groups** settings.
3. Create a new monitoring group using the **+CREATE GROUP** link.
4. Name the group `Frontend Servers`.
5. For the criteria, use:

| Setting  | Value       |
| :------- | :---------- |
| Type     | `Tag`       |
| Tag      | `component` |
| Operator | `Equals`    |
| Value    | `frontend`  |

1. Make sure that `Resources Selected` on the right hand of the page is displaying `2 VM Instances currently selected.` If not, double-check your criteria.
2. **Create** the group.
3. Refresh the page and after a minute or so, you should see several metrics and charts for your two VMs.

### Create a sub-group just for frontend dev servers

1. In the `Frontend Servers` group, locate the `Subgroups` section and click **CREATE SUBGROUP**.
2. Configure the subgroup with the following settings:

| Setting        | Value          |
| :------------- | :------------- |
| Name           | `Frontend Dev` |
| **Criteria 1** |                |
| Type           | `Tag`          |
| Tag            | `component`    |
| Operator       | `Equals`       |
| Value          | `frontend`     |

1. Click **Done** for the first criteria, then **Add criterion**.

| Setting        | Value    |
| :------------- | :------- |
| **Criteria 2** |          |
| Type           | `Tag`    |
| Tag            | `stage`  |
| Operator       | `Equals` |
| value          | `dev`    |

Click **Done** for the second criteria, then select the `And` criteria operator to join them.

1. Click **Create**.
2. Navigate back to the `Groups` home page. Notice how the `Frontend Servers` group can now be expanded to show its sub-group. The UI also displays a clickable link containing a little information about the types of resources contained by the group.

## Task 4. Create and test an uptime check

Google Cloud uptime checks test the liveliness of externally facing HTTP, HTTPS, or TCP applications by accessing said applications from multiple locations around the world. The subsequent report includes information on uptime, latency, and status. Uptime checks can also be used in alerting policies and dashboards.

In this task, you:

- Create an uptime check for the Frontend Servers group.
- Investigate out how an uptime check handles failure.

### Create an uptime check for the Frontend Servers group

If you've already created a group that contains the same resources that need to be uptime checked, then it's easy to create a single uptime check for multiple servers.

1. In the Monitoring section of your `Monitoring Project`, click **Uptime checks**.
2. At the top of the page, follow the link to **+CREATE UPTIME CHECK**.
3. Configure a new uptime check with the following settings (don't press Save yet):

| Setting         | Value                                     |
| :-------------- | :---------------------------------------- |
| Title           | `Frontend Servers Uptime`, click **Next** |
| Target Protocol | `HTTP`                                    |
| Resource Type   | `Instance`                                |
| Applies To      | `Group`                                   |
| Group           | `Frontend Servers`                        |
| Path            | `/`                                       |
| Check Frequency | `1 minute`                                |

1. Click **Next**, to leave other options as defaults. If you like, in the **Alert & Notification** section, use **Manage Notification Channels** to add your email address as a valid notification option. The Alert will be enabled by default but won't actually notify anyone otherwise.
2. Click **Test** and verify a 200 response.
3. **Create** the uptime check.
4. In the list of uptime checks, click your new **Frontend Servers Uptime** to view its dashboard.
5. Wait a few minutes and refresh. The dashboard populates information about the check results. Explore the charts and data.
6. On the right side of the page in the `Configuration` box, copy the `Check ID` value and paste it in your notes text file. It should read *frontend-servers-uptime.*

### Check out how an uptime check handles failure

Our uptime check is working, but how about if there was a failure? Let's trigger a failure and investigate the resulting uptime check and alert behavior.

1. Make sure you have a few minutes of data in your uptime check's dashboard before proceeding.
2. Using the **Navigation menu** to navigate to the Google Cloud **Home** page.
3. Use the project dropdown to switch to the `Worker 1` project.
4. Use the **Navigation menu** to navigate to your **Compute Engine | VM instances**.
5. Select the box next to **worker-1-server-vm** and **STOP** it running.
6. Refresh the `VM instances` page and wait until you see the server has stopped running.
7. Use the project dropdown to switch to the `Monitoring Project`, then navigate to **Monitoring | Uptime checks | Frontend Servers Uptime**.
8. Examine the `Current Status` and the `Passed Checks` chart. It could take a minute for the failures to start displaying.

### What can Cloud Monitoring, Logging, and Alerting tell us?

1. Navigate to **Monitoring | Metrics explorer**.
2. In the search box under `Find resource type and metric`, type/select **Check passed**. On a side note, after you investigate the Check passed metric, you might try searching for "uptime_check" will display some of the other metrics you might like to investigate.
3. Set the `Group By` to **checked_resource_id** (you might have to scroll) to group the check results by server.
4. Take a moment to examine the results.
5. Use the **Navigation menu** to navigate to **Logging | Logs Explorer**.
6. Click the query window, and delete anything there. Click **Log name**. Locate and **Add** the **uptime_checks** log, then select **Run Query**.

![uptime-checks.png](https://cdn.qwiklabs.com/y5EVJi8oN8QtbwLAefUgjLRNAiccarpPaR4JDnG3ztA%3D)

1. Expand and examine one of the log entries. What useful information does it provide?
2. In the same entry, examine the `labels` section and locate the `check_id.` Consult your text notes document and compare the id you recorded there. They should match.
3. Use the **Navigation menu** to navigate to **Monitoring | Alerting**.
4. Investigate the firing alert. If you added yourself as a notification channel, review the email.

**Note:** If a Monitoring group is created based on labels, then the group will keep checking for powered off server for 5 minutes. After 5 minutes, Google Cloud determines the server should no longer be counted as a member of the group. This is important because if an uptime check is tied to the group, then it will only report failures while the group reports that missing server. When the group quits reporting the off server, the uptime check quits checking for it, and suddenly the check starts passing again. This can be a real issue if you're not careful.

## Task 5. Create a custom dashboard

There will be times when the humans running a Google Cloud system want to investigate its status. This may be related to curiosity, capacity planning, or perhaps in response to an alert. Regardless, when data is expressed in chart form, as opposed to a list of items or values, it tends to be easier to spot trends, anomalies, and highs or lows. In this task, we add a chart for our developers to use as a way to track some of the happenings on the frontend servers.

In this task, you:

- Create a developer dashboard and add an uptime chart to it.
- Add and test a CPU utilization chart to the dashboard.

### Create a developer dashboard and add an uptime chart to it

If you were interested in what was happening on your developer webserver, it would be helpful to have a dashboard of charts just for that single server. In this section, you create a dashboard with an uptime check summary chart.

1. Using the **Navigation menu**, navigate to the Google Cloud **Home** page.
2. Use the project dropdown to switch to the `Worker 1` project.
3. Use the **Navigation menu** to navigate to your **Compute Engine | VM instances**.
4. Select the box next to **worker-1-server-vm** and **START** it running.
5. Use the project dropdown to switch to the `Monitoring Project`, then navigate to **Monitoring | Dashboards**.
6. At the top of the page, press **+CREATE DASHBOARD**.
7. For **New Dashboard Name**, type **Developer's Frontend**.
8. Click **Line Chart**.
9. Click **ADVANCED**.

| Setting        | Value               |
| :------------- | :------------------ |
| Chart Title    | `Dev Server Uptime` |
| Resource type: | `VM Instance`       |
| Metric         | `check_passed`      |

1. Examine the values displayed on the chart. Our worker-1-server-vm is our core development server. Take note of the `checked_resource_id` value from one of its check response rows. You must pick the value from a list in the next step.
2. Click `ADD FILTER`:

| Setting | Value                  |
| :------ | :--------------------- |
| Label   | `instance_id`          |
| Value   | `Select from dropdown` |

1. Click **DONE**.
2. Select Processing step `Count true` and Alignment period `5` Alignment unit `minutes`.

### Add and test a CPU utilization chart to the dashboard

Another key piece of information about what's going on inside your development server is its CPU load. Overall it is nice to know what's happening with the NGINX server itself, but without the installation of the logging and monitoring agents, you can't do that yet.

1. Turn on auto-refresh from the top right click on the three dots on your Dashboard by toggling **Off** to **On** (located at the top, near TIME 1H 6H ...).
2. Add another chart to our dashboard. Press **LINE CHART**.
3. Click **ADVANCED**.
4. Select `Resource type` choose **VM Instance**.
5. Select `Metric` choose **CPU utilization**.
6. Click `ADD FILTER`:

| Setting | Value                |
| :------ | :------------------- |
| Label   | `instance_name`      |
| Value   | `worker-1-server-vm` |

1. Cloud Shell doesn't always work well as a platform for generating load test traffic, since it sometimes thinks you're doing something malicious and terminates your session. Use your second web server to generate the load instead. Expand the **Navigation menu**, right-click **Compute Engine**, and open the link in a new tab or window.
2. Use the project dropdown to switch to the `Worker 2` project.
3. **SSH** into your `worker-2-server-vm`.
4. Update the server's package database and install the apache2-utils. Apache Bench is a quick easy tool you can use to generate HTTP load:

```bash
sudo apt-get update
sudo apt-get install apache2-utilscontent_copy
```

1. Before you use Bench, set up a URL to your server. Find your `worker-1-server-vm` external IP in your notes file and use it to construct the URL environmental variable below. Don't omit the "http://".

```bash
URL=http://[worker-1-server-vm ip]content_copy
```

1. Throw some load at your server. The following command executes 100 requests at a time and continues to do so up to a total of 100,000 requests. Don't miss the trailing "/" after the URL.

```bash
ab -n 100000 -c 100 $URL/content_copy
```

1. Once Bench finishes its run, wait a minute or so and then execute:

```bash
ab -n 500000 -c 500 $URL/content_copy
```

1. While the second series of traffic is generated, switch back to your dashboard. After a little time, you see the two distinct spikes in CPU load.