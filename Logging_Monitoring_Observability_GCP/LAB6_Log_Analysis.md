# Log Analysis

2 hours 30 minutesFree

## Overview

In this exercise, you generate log entries from an application, filter and analyze logs in Cloud Logging, and export logs to a BigQuery log sink.

### Objectives

In this lab, you learn how to perform the following tasks:

- Set up and deploy a test application.
- Explore the log entries generated by the test application.
- Create and use a logs-based metric.
- Export application logs to BigQuery.

## Task 0. Lab Setup

In this task, you use Qwiklabs and perform initialization steps for your lab.

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

![GCP Project Dashboard](https://cdn.qwiklabs.com/dxfeoOcn1ObyC0BYyoqqqSi4rO%2FeMdbWPFjoK6C0YYk%3D)

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

## Task 1. Set up a test application, deploy it, and set up a load test VM

To experiment with logging, log exporting, and Error Reporting, we will deploy the HelloLoggingNodeJS application we used in a previous exercise to Cloud Run.

1. Make sure the Cloud Build, Compute Engine, and Cloud Run APIs are enabled, as it is needed in later steps.

```bash
gcloud services enable cloudbuild.googleapis.com \
run.googleapis.com \
compute.googleapis.com \
cloudprofiler.googleapis.comcontent_copy
```

1. Once the APIs successfully enable, clone the `https://github.com/haggman/HelloLoggingNodeJS.git` repo.

```bash
git clone https://github.com/haggman/HelloLoggingNodeJS.gitcontent_copy
```

1. This repository contains a basic Node.js application used for testing. Change into the `HelloLoggingNodeJS` folder and open the `index.js` in the Cloud Shell editor.

```bash
cd HelloLoggingNodeJS
edit index.jscontent_copy
```

1. If an error indicates that the code editor could not be loaded because third-party cookies are disabled, click **Open in New Window** and switch to the new tab.
2. Take a few minutes to peruse the code. You may recognize this code from some of the examples in the lecture module. Notice that it loads the Google Error Reporting library as well as the debugging library. Scroll lower and there are several methods that generate logs, errors, and error logs. Pay special attention to `/random-error`. It generates an error approximately every 1000 requests.
3. In the editor, also take a look at the `rebuildService.sh` file. This file uses Cloud Build to create a Docker container, then it creates (or updates) a Cloud Run application using the container. The Cloud Run service supports anonymous access, has a couple of labels on it for stage and department, and only allows a maximum of 5 concurrent connections to any one execution instance.
4. Return to the Cloud Shell window. If the Cloud Shell is not visible, click **Open Terminal**.
5. Build the container and start the Cloud Run application by executing the file. It takes a minute or two for this to complete.

```bash
sh rebuildService.shcontent_copy
```

1. In Cloud Shell, the final message contains the URL to your new service. Click the link to test the application in a new tab. It returns a simple Hello World! response. Once satisfied, copy the URL to your service from the browser and close the tab.
2. In the Cloud Shell terminal window, create a new URL environmental variable and set its value to the URL to your application.

```bash
URL=url_you_copied_herecontent_copy
```

1. Test that the variable was created correctly by echoing its value.

```bash
echo $URLcontent_copy
```

1. Use a bash while loop to generate load on the application's `random-error` route. Make sure you see the mostly "Worked this time" messages start to appear. If they don't, then double check that your application is running and that the URL property is properly set.

```bash
while true; \
do curl -s $URL/random-error \
-w '\n' ;sleep .1s;donecontent_copy
```

## Task 2. Explore the log files for a test application

You have an application now deployed to Cloud Run which, depending on the URL, can simply log, log through Winston, or generate error reports. Take a few minutes to explore those logs.

1. In the Google Cloud Console, use the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) to navigate to **Logging | Log Explorer**.
2. Click the `Query builder` and erase any query that is there by default.
3. Use the **Resource** drop-down menu to select the **Cloud Run Revision | hello-logging** Cloud Run service. Don't forget to **Add** your selection, or it won't actually filter by that resource.

![hellologging](https://cdn.qwiklabs.com/Z6qFCrZzNNLFiuuJITnjM5GoxFfauQCOZfQNQHXwFSk%3D)

1. **Run Query**.
2. Most likely you see nothing but successful `200` messages. How can you find the errors? One way is to exclude all of the 200 responses and see what's left. To do that, click one of the **200** status codes and select **Hide matching entries**.

![Hide matching entries](https://cdn.qwiklabs.com/nhddqIVNBpWdaUZkbAdYxEAt93Wo5CLvvKtwptSKmYc%3D)

1. While not perfect, you are able to find log messages tied to the error, and the stack traces generated by the error happening. Use the `Logs field explorer` to filter the display to only messages with a `Severity` of **Error**.
2. Now you see the responses when an error was thrown. Expand and explore one of the entries. Note which log file contains the 500 errors. When you are finished, **Clear** the Error severity filter.
3. To see the stack traces, which is what a developer might be most interested in, use the `Logs field explorer` to filter on the **run.googleapis.com/stderr** standard error log. That does show the exceptions, but where does each one start?
4. It is nice to see both the stderr log with the stack traces, and the requests log with the 500 status errors. Click the `Query builder`, select **Log name**, select both the Cloud Run **requests** and **stderr** logs, and then **Add** them to the query. Select **Run Query**.

**Note:** You need to wait for 10-12 minutes for the 500 status errors to appear.

1. Take a moment to explore the displayed logs. Now you see the response returning the error to the client request, and the subsequent stack trace sent to stderr.

![Error and stack trace](https://cdn.qwiklabs.com/YIJs8rQoJo4xe3qQIJ%2BP2bAzdMQeIcJxFkn0du382v0%3D)

## Task 3. Create and use a logs-based metric

You just looked at the log file for the Cloud Run service and examined an error entry, but what if you were doing some custom logging, not necessarily error related, and wanted to create a custom metric based on it? In this portion of the exercise, you change the loop to call the `/score` route, and then create a metric from the scores that are coming through.

In this task, you:

- Generate some load on the /score endpoint.
- Explore the logs generated by the load.
- Tweak the code to put the message in a more usable format.
- Create a new logs-based metric.

### Generate some load on the /score endpoint

1. Switch to or reopen Cloud Shell.
2. Use CTRL+C to break your test loop.
3. Modify the `while` loop to call the `/score` route, and restart the loop. Verify that the new messages are displaying random scores.

```bash
while true; \
do curl -s $URL/score \
-w '\n' ;sleep .1s;donecontent_copy
```

### Explore the logs generated by the load

1. Switch to your Google Cloud Console and reopen the **Logging | Logs Explorer** page.
2. Clear any existing query and use the **Resource** to display your **Cloud Run Revision | hello-logging** logs. Select **Run Query**.
3. All of the 200 status code entries should be from this latest test run. If not, refresh your logs view by clicking **Jump to Now**. Expand one of the 200 status code entries. How can you tell if it's from a /score request? Why isn't the score displaying in the log entry?
4. Use the `Logs field explorer` to filter on the **run.googleapis.com/stdout** log. Now you should see all of the messages printed by the code itself. Why would it be difficult to build a metric displaying the scores for this entry?
5. It would be better if the messages were generated as structured JSON, rather than unstructured text. That way you could easily access and extract just the scores in our logs-based metric. In the Cloud Shell terminal window, use **CTRL+C** to break the test `while` loop.

### Tweak the log format to make the score easier to access

1. Open the `index.js` file in the Cloud Shell editor and locate the code for the `/score` route (around line 90).
2. Replace the `/score` route with the below code. Notice how the message contents are now properties of the `output` object, and how the printed message is the JSON object stringified.

```javascript
//Basic NodeJS app built with the express server
app.get('/score', (req, res) => {
  //Random score, the contaierID is a UUID unique to each
  //runtime container (testing was done in Cloud Run).
  //funFactor is a random number 1-100
  let score = Math.floor(Math.random() * 100) + 1;

    let output = {
      message:  '/score called',
      score:    score,
      containerID: containerID,
      funFactor: funFactor
  };

  console.log(JSON.stringify(output));

  //Basic message back to browser
  res.send(`Your score is a ${score}. Happy?`);
});content_copy
```

1. Since you modified the code, make sure you didn't introduce any errors by starting the application locally in Cloud Shell. In the Cloud Shell terminal window, install the dependencies and start the application. Make sure you are in the HelloLoggingNodeJS folder.

```bash
cd ~/HelloLoggingNodeJS/
npm i
npm startcontent_copy
```

1. If you see a "Hello world listening on port 8080 message," use **CTRL+C** to stop the application and move on to the next step. If you see any errors, fix them and make sure the application starts before continuing.
2. Rebuild and redeploy the application by rerunning `rebuildService.sh`.

```bash
sh rebuildService.shcontent_copy
```

1. Wait for the application to finish rebuilding and redeploying, then restart the test loop. Make sure you see the score messages appearing once again.

```bash
while true; \
do curl -s $URL/score \
-w '\n' ;sleep .1s;donecontent_copy
```

1. Switch to your browser tab showing the **Logs Explorer** and display the latest logs by clicking **Jump to Now**. You should still have the entries filtered to display the Cloud Run `stdout` logs for `hello-logging`. Expand one of the entries and examine the new format.

### Create a score logs-based metric

1. Use the `Actions` menu to **Create Metric**. Click on **Actions** > **Create Metric**.
2. `Name` the metric **score_averages**, give it a `Unit` of **1**, and set the `Type` to **Distribution**. For `Field name`, choose **jsonPayload.score**, and then press the **ADVANCED** link.
3. Because scores are all going to be numbers in the 0-100 range, set the histogram bucket `Type` to **Linear**, the `Start value` to **0**, the `Number of buckets` to **20**, and the `Bucket width` to **5**.
4. Click **Create Metric**.
5. Use the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) to switch **Monitoring > Dashboards**. You may have to wait while the workspace is created.
6. Click **+Create Dashboard**.
7. Click **Line Chart**.
8. For **New Dashboard Name**, type **Score Fun**.
9. Set the `Chart Title` to **Score Info**.
10. Under `Resource type`, click inside and select **Cloud Run Revision** and in the `Metric` box, type **score_averages** as the name.
11. The aligner was set by default to `99th percentile` so the graph was showing that 99% of the values fell below scores in the high 90s. That's not really what you want. Set the Aligner to **50th percentile** (median).

![scoredash](https://cdn.qwiklabs.com/gXTo10%2BAqBlIkXlSSfvq8HcqWKHa0DihXAf57Uk1r1s%3D)

## Task 4. Export application logs to BigQuery

Exporting logs to BigQuery not only allows them to be stored for longer, but it also gives the ability to analyze them with SQL and the sheer power of BigQuery. In this task, you configure an export for the Cloud Run logs to BigQuery, and use it to find the error messages generated by the `random-error` application route.

In this task, you:

- Configure a log export to BigQuery.

### Configure an Export Sink to BigQuery

1. Switch to the Cloud Shell window with the `while` loop sending load to our application. Use **CTRL+C** to break the loop.
2. Modify the `while` to once again send load to the `random-error` route. Re-execute the loop.

```bash
while true; \
do curl -s $URL/random-error \
-w '\n' ;sleep .1s;donecontent_copy
```

1. Use the Google Cloud Console **Navigation menu** to navigate to **Logging | Logs Explorer**.
2. Delete any existing query and execute a query filtering the **Resource** to **Cloud Run Revision | hello-logging**. Expand an entry or two and verify that they relate to the `/random-error` requests.
3. Create a sink using **Actions | Create Sink**.

![Create Sink](https://cdn.qwiklabs.com/3G86rpkZPNI3oUEuSD2AIX9vIe%2FQQsPnUWiI6h6hxxc%3D)

1. Configure the `Sink Name` as **hello-logging-sink**.
2. Click **Next**.
3. Set the sink service to **BigQuery dataset**.
4. For `Sink Destination`, select **Create new BigQuery dataset** and name the dataset **hello_logging_logs** and click **CREATE**.
5. Click **Next**.
6. For `Choose logs to include in sink`, Click **Next**.
7. Click **CREATE SINK**. When you **Create Sink**, notice how it creates a service account which is used to write to the new BigQuery dataset.
8. Use the **Navigation menu** to switch to **BigQuery**.
9. In the `Resources` section, expand your projects node and the **hello_logging_logs** dataset.
10. Click the **requests** table. In the tabbed dialog below the query window, click the **Preview** tab and explore the information in this log. This contains general request-oriented information. What URL was requested, when, and from where.
11. Click the **Schema** tab and take a moment to investigate the schema of the table generated. Notice that nested and repeated data structures are used for columns like `resource` and `httpRequest`.
12. Select the **stderr** table. Standard out and Standard error are commonly used by applications to differentiate between simple log messages and messages that are related to an error. Node.js applications will dump any uncaught exception/error related information to standard error.

**NOTE:** Wait for 5~10 minutes, if the **stderr** table doesn't appear.

1. Once again, **Preview** the data in the table. Now you are seeing the random errors as they impact different parts of the request call stack. The entries with a `textPayload` that contain `ReferenceError` are the real errors itself.
2. Click the **Schema** tab and investigate the actual structure of the table. Once again, you see the mix of standard, and nested, repeated fields.
3. Create and execute a query to pull just the `textPayloads` that start with `ReferenceError`. Start the query by clicking the **Query Table** button. Modify the `SELECT` so it pulls the **textPayload**, and then add the `where` clause. Replace `[project-id]` with your GCP project ID and `[date]` with your date speacified in the table name.

```sql
SELECT
  textPayload
FROM
  `[project-id].hello_logging_logs.run_googleapis_com_stderr_[date]`
WHERE
  textPayload LIKE 'ReferenceError%'content_copy
```

1. To do a count, modify the query to count these entries. Execute again and check the number you are receiving. Replace `[project-id]` with your GCP project ID and `[date]` with your date speacified in the table name.

```sql
SELECT
  count(textPayload)
FROM
  `[project-id].hello_logging_logs.run_googleapis_com_stderr_[date]`
WHERE
  textPayload LIKE 'ReferenceError%'content_copy
```

1. To check the error percentage, build a query that compares the total requests to the `ReferenceError%` requests. Replace `[project-id]` with your GCP project ID and `[date]` with your date speacified in the table name. Is the error about 1/1000?

```sql
SELECT
  errors / total_requests
FROM (
  SELECT
    (
    SELECT
      COUNT(*)
    FROM
      `[project-id].hello_logging_logs.run_googleapis_com_requests_[date]`) AS total_requests,
    (
    SELECT
      COUNT(textPayload)
    FROM
      `[project-id].hello_logging_logs.run_googleapis_com_stderr_[date]`
    WHERE
      textPayload LIKE 'ReferenceError%') AS errors)content_copy
```