# Compute Logging and Monitoring

2 hoursFree

## Overview

Google Cloud has a number of compute-related resources including Compute Engine, Kubernetes, and Cloud Run. In this exercise, you install the logging and monitoring agents into a VM running a Nginx server to maximize the metrics to easily view. You also set up a Kubernetes cluster and monitor an application deployed into it.

### Objectives

In this lab, you learn how to perform the following tasks:

- Set up a VM and a GKE cluster.
- Install and use the logging and monitoring agents for Compute Engine.
- Add a service to the GKE cluster and examine its logs and metrics.

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

## Task 1. Set up a VM and a GKE cluster

In this task, you:

- Set up a VM running Nginx.
- Create a GKE cluster.

### Set up a VM to host Nginx

1. Open the Google Cloud **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) in the upper-left corner of the console, then click the link to the **Compute Engine**.
2. Create a new VM instance. Give it the name **web-server-vm** and tick the box to **Allow HTTP traffic**. **Create** and then move onto the next step.

### Create a GKE cluster

To explore a little of what Google Kubernetes Engine offers in the way of logging and monitoring, let's add a cluster, which will come with logging and monitoring enabled by default.

1. Use the **Navigation menu** to navigate to the **Kubernetes Engine** > **Clusters** page.
2. Select **Create** and then click on the **Configure** button for **Standard** option.
3. Set the `Name` to **gke-cluster**, then under the **Cluster** settings on the left, select **Features**.
4. Verify that `Enable Cloud Operations for GKE` is **Checked** and that the choice in the dropbox is set to **System and workload logging and monitoring**, and then press **Create**.
5. Once you see the cluster creation start, move on to the next step.

### Install Nginx

1. Switch back to **Compute Engine** and **SSH** in to the `web-server-vm`.
2. Use APT to install Nginx.

```bash
sudo apt-get update
sudo apt-get install nginxcontent_copy
```

1. Verify that Nginx installed and is running.

```bash
sudo nginx -vcontent_copy
```

1. Switch back to **Compute Engine** and if you enabled the HTTP access firewall rule, then you should be able to click the external IP. Do so and make sure the Nginx server is up and running. Copy the URL to the server.
2. Change to the Cloud Shell terminal window. Create a URL environmental variable and set it equal to the URL to your server.

```bash
URL=URL_to_your_servercontent_copy
```

1. Use a bash `while` loop to place some load on the server. Make sure you are seeing the `Welcome to nginx` responses.

```bash
while true; do curl -s $URL | grep -oP "<title>.*</title>"; \
sleep .1s;donecontent_copy
```

## Task 2. Install and use the logging and monitoring agents for Compute Engine

In this task, you:

- Examine the logs and see some data that isn't included.
- Install the logging and monitoring agents in the Compute Engine instance.
- Re-examine the logs to see the new data.

### Examine the logs and see some data that isn't included

A standard Debian Linux image from Google will not have the logging or monitoring agents installed, so you won't have access to metrics from googleapis.com/nginx/ such as:

- connections/accepted_count
- connections/current
- connections/handled_count
- request_count

1. Switch to the Google Console window and navigate to **Monitoring** > **Overview**. Wait for the monitoring workspace to create, and then navigate to **Metrics explorer**.
2. Set the Metrics explorer `Resource type` to **VM Instance**, the Metric to **CPU utilization**, and the `Filter` to **instance name = web-server-vm**
3. You may not see a spike when you applied the load, since it's not a big load, but you are able to access the metric.
4. Delete the Filter and the Metric.
5. In the `Select a metric` box, enter **nginx** and select **Requests**.

Why is there no data?

### Install the logging and monitoring agents in the Compute Engine instance

The data for Nginx requests is missing because without the logging and monitoring agents being installed, the best Google Cloud can do is black-box monitoring. If you want to see more details, then you need to enable white-box monitoring by installing the agents.

1. Use the **Navigation menu** to navigate to **Compute Engine** > **VM instances**.
2. Click the **SSH** link for the `web-server-vm`
3. Check to see if the logging agent is installed/running.

```bash
sudo service google-fluentd status
sudo service stackdriver-agent statuscontent_copy
```

1. Not surprisingly, the services could not be found. Check to make sure you have the requisite scopes to perform logging and monitoring.

```bash
curl --silent --connect-timeout 1 -f -H "Metadata-Flavor: Google" \
http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/scopescontent_copy
```

1. Note in the response the `logging.write` and `monitoring.write` scopes.
2. Download the script, add the monitoring agent repo to apt, and install the agent.

```bash
curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
sudo bash add-monitoring-agent-repo.shcontent_copy
sudo apt-get update
sudo apt-get install stackdriver-agentcontent_copy
```

1. Start the monitoring agent.

```bash
sudo service stackdriver-agent startcontent_copy
```

1. Install the logging agent.

```bash
curl -sSO https://dl.google.com/cloudagents/install-logging-agent.sh
sudo bash install-logging-agent.shcontent_copy
```

1. Retest the two agents again and verify they are both active. If you see any "can not take infinite value" warnings, ignore them and if the status is **active (running)**, press Crtl+c and move on to the next step.

```bash
sudo service google-fluentd statuscontent_copy
sudo service stackdriver-agent statuscontent_copy
```

1. To fully integrate the server, you enable the status information handler in Nginx by adding a configuration file to the Nginx configuration directory.

```bash
(cd /etc/nginx/conf.d/ && sudo curl -O https://raw.githubusercontent.com/Stackdriver/stackdriver-agent-service-configs/master/etc/nginx/conf.d/status.conf)content_copy
```

1. Reload the Nginx service.

```bash
sudo service nginx reloadcontent_copy
```

1. Enable the Nginx monitoring plugin.

```bash
(cd /opt/stackdriver/collectd/etc/collectd.d/ && sudo curl -O https://raw.githubusercontent.com/Stackdriver/stackdriver-agent-service-configs/master/etc/collectd.d/nginx.conf)content_copy
```

1. Restart the monitoring agent.

```bash
sudo service stackdriver-agent restartcontent_copy
```

### Re-examine the metrics to see the new data

Now that the agents are installed and the Nginx plugins have been added, retest the Nginx metric that we examined earlier.

1. Check the Cloud Shell window and verify that the test loop is still running, and that you are still receiving responses. If not, reset the URL property and restart the while.
2. Navigate to the **Monitoring** > **Metrics explorer**.
3. Set the `Resource type` to **VM Instance**, in the `Select a metric` box enter **nginx**, and then select **Requests**.

You may have to wait a few minutes and refresh the page, but you should soon see metric data coming in from Nginx. The monitoring agent is working as it should.

## Task 3. Add a service to the GKE cluster and examine its logs and metrics

You created a GKE cluster with logging and monitoring enabled earlier in this exercise. Now, load the HelloLoggingNodeJS application to it, put it behind a load balancer service, and view some metrics for your cluster.

1. Switch to or reopen the Cloud Shell console. Break the running test loop with CTRL+C.
2. Enable the Cloud Build API as it is needed it in a few steps.

```bash
gcloud services enable cloudbuild.googleapis.com content_copy
```

1. Clone the `https://github.com/haggman/HelloLoggingNodeJS.git` repo.

```bash
git clone https://github.com/haggman/HelloLoggingNodeJS.gitcontent_copy
```

1. This repository contains a basic Node.js web application that is used for testing. Change into the `HelloLoggingNodeJS` folder and open the **index.js** in the Cloud Shell editor.

```bash
cd HelloLoggingNodeJS
edit index.jscontent_copy
```

1. Take a few minutes to peruse the code.
2. In the editor, also take a look at the `package.json` file which contains the dependencies, and the `Dockerfile` which plans the Docker container we generate and deploy to GKE.
3. Submit the Dockerfile to Google's Cloud Build to generate a container and store it in your Container Registry.

```bash
gcloud builds submit --tag gcr.io/$DEVSHELL_PROJECT_ID/hello-logging-js .content_copy
```

1. Open the `k8sapp.yaml` file and explore the instructions to Kubernetes to create three hello-logging pods, and then expose them through a LoadBalancer service.
2. In the `k8sapp.yaml` file, replace the **$GCLOUD_PROJECT** with your actual project ID. Remember, your project ID is located on the Home page of your Google Cloud Console and in Qwiklabs just below your temporary Google Cloud password.
3. Use the **Navigation menu** to navigate to **Kubernetes Engine**. Click on the **triple dot** icon for your cluster and select **Connect**. Copy the command line to configure `kubectl`.

![dot_icon](https://cdn.qwiklabs.com/rDedlXDeoZq5FJGG7xgH07s%2FtbKxW2CYrNNWhPLRcHE%3D)

1. Switch back to Cloud Shell and execute the command.
2. Use `kubectl` to apply your k8sapp.yaml.

```bash
kubectl apply -f k8sapp.yamlcontent_copy
```

1. Get a list of your services. It may take a minute or two for the new `hello-logging-service` to appear, and for it to get an External IP.

```bash
kubectl get servicescontent_copy
```

1. Once the service appears with the external IP, copy the external IP value.
2. Open a tab in the browser and paste in the IP. Make sure you see the `Hello World` message.
3. Copy the page URL from the browser and switch back to Cloud Shell. Update the URL environmental variable and restart the `while` loop. Make sure you are seeing the `Hello World` responses.

```bash
URL=url_to_k8s_app
while true; do curl -s $URL -w "\n"; sleep .1s;donecontent_copy
```

1. Navigate to the **Monitoring** > **Dashboards** and open the **GKE** dashboard.
2. On the `GKE` dashboard, for **gke-cluster** scroll horizontally and see the charts. If all of the small charts are reading *No Data,* refresh the page until they start showing readings for CPU and Memory Utilization. You might also want to toggle **Off** to **On** to enable auto-refresh of the data.
3. If you want to examine the different cluster nodes, you can scroll horizontally in **Nodes** tab and see that the three hello-logging-js-deployments are spread across the nodes. This tab focuses on what's running where, from Cluster, to Node, to Pod.
4. Switch to the **Workloads** tab. This is focused on the deployed workloads, grouped by namespace.
5. Finally, scroll to the **Kubernetes Services** tab and expand **hello-logging-service**. This view is more about how services relate to their pods.
6. In any of the views, if you drill all the way down to one of our hello-logging-js-containers, a right-hand window appears with details. Investigate Incidents, Metrics, and Logs.