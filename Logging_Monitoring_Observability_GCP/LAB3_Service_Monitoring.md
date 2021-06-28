# Service Monitoring



## Overview

Google Cloud's Service Monitoring streamlines the creation of microservice Service Level Objectives (SLOs) based on availability, latency, or custom Service Level Indicators (SLIs). In this lab, you use Service Monitoring to create a 99.5% availability SLO and corresponding alert.

### Objectives

In this lab, you learn how to perform the following tasks:

- Deploy a test application.
- Use Service Monitoring to create an SLO.
- Tie an alert to the SLO.

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

## Task 1. Deploy a test application

In this task, you:

- Deploy a test application to App Engine.

### Deploy a test application to App Engine

To have something for Service Monitoring to connect to, deploy a basic Node.js application to App Engine standard.

1. In your Cloud Shell terminal, clone `https://github.com/haggman/HelloLoggingNodeJS.git` repo.

```bash
git clone https://github.com/haggman/HelloLoggingNodeJS.gitcontent_copy
```

1. This repository contains a basic Node.js web application used for testing. This is the same application you saw pieces of in the lecture module. Change into the `HelloLoggingNodeJS` folder and open the `index.js` in the Cloud Shell code editor.

**Note:** If an error indicates that the code editor could not be loaded because third-party cookies are disabled, click **Open in New Window** and switch to the new tab.

```bash
cd HelloLoggingNodeJS
edit index.jscontent_copy
```

1. Take a few minutes to peruse the code.
2. In the cloud shell code editor, look at the `app.yaml` file. App Engine standard uses this file to define the runtime required by the application.
3. In the cloud shell code editor, look at the `package.json` file. Not only does this define the Node.js application dependencies, but it also defines the start script App Engine uses to serve requests.
4. Return to the Cloud Shell window. If the Cloud Shell is not visible, click **Open Terminal**.
5. In the Cloud Shell terminal, create a new App Engine app. This must be done once in each new project that is running App Engine applications. App Engine is a regional technology, thus the *region* switch.

```bash
gcloud app create --region=us-centralcontent_copy
```

1. Deploy the Hello Logging app to App Engine. Wait until the deploy completes before moving on.

```bash
gcloud app deploycontent_copy
```

1. When prompted, type **y** and press **Enter**.
2. Copy the URL to your newly deployed app from the console (https://qwiklabs-gcp-****************.appspot.com) and open it in a new browser tab. Verify a `Hello World!` response.

## Task 2. Use Service Monitoring to create an availability SLO

In this task, you:

- Use Service Monitoring to create an availability SLO.
- Create an alert tied to your SLO.
- Trigger the alert.

### Place some load on the application

1. At the top of the Cloud Shell interface, press the ＋ to **Open a new tab**.
2. In the new tab, use a simple bash *while* loop to generate load on your application. The loop below generates ten requests per second. The URL is to the /random-error route, which generates an error about every 1000 requests, so you should see approximately 1 error every 100s.

```bash
while true; \
do curl -s https://$DEVSHELL_PROJECT_ID.appspot.com/random-error \
-w '\n' ;sleep .1s;donecontent_copy
```

1. Leave the loop running in its Cloud Shell tab and move on to the next step.

### Use Service Monitoring to create an availability SLO

We have a working App Engine application that is currently throwing an error approximately every 1000 requests. Imagine we want to create an availability SLO with a target of 99.5%, and an alert that will notify us if our SLO is in danger. That's exactly what Service Monitoring makes easy.

1. In the Google Cloud Console, use the **Navigation menu** (![console_nav_small.png](https://cdn.qwiklabs.com/%2BnHJDKUfemAVQPMPb%2BAcqH0ZqkvrGALIBL1p582JNCQ%3D)) to navigate to **App Engine | Dashboard**. You can already see information on your running service and the load you are placing on it.
2. Scroll down to the `Application Errors` section. Have any errors been generated yet? If not, wait a couple of minutes and refresh the page. You should see one every few minutes.
3. Use the **Navigation menu** to navigate to **Error Reporting**. Notice the error is also being caught here. We will discuss Error Reporting in a later module.
4. Use the **Navigation menu** to navigate to **Monitoring**. It takes a moment for the monitoring workspace to create. Once it does, click **Services**.
5. Notice that Service Monitoring already sees your `default` App Engine application. If it doesn't, wait a minute and refresh the page until it appears in the table.
6. Click the `default` App Engine application to drill into it.
7. Click **+Create SLO** to start the new SLO dialog.
8. Select the **Availability** metric, leave the evaluation method set to **Request-based**, and then click **Continue**.
9. Take a moment to investigate the details the SLI details displayed, then click **Continue**.
10. To define the SLO, set the `Period type` to **Rolling** and the `Period length` to **7** days to calculate the SLO on a constantly moving 7-day window of time.
11. Set the `Goal` to **99.5**% and the charts fill in, though it's typically difficult to see that 99.5 to 99.9 difference. Click the red dashed line, and the chart will zoom in to make things easier to see.
12. Click **Continue**, notice the default name, and submit the new SLO by clicking **Create SLO**.

### Investigate the new SLO and create an alert for it

1. Under the `Current status of 1 SLO` section, expand the new SLO and investigate the information it displays. Move between the three tabs, **Service level indicator**, **Error budget**, and **Alerts firing**, investigating each.

![Working SLO](https://cdn.qwiklabs.com/TeRlWXNSj7v5J%2B4iGo0rGr%2BR0BMH0MyMem6JeafW01I%3D)

### Create an alert tied to the availability SLO

The SLO has been created and so far, you are well within your objective. Since the SLO target is 99.5%, and the SLI should be showing a current measurement level of about 99.9%, that means that your application is using approximately 1/5 of its error budget, so the error budget should be displaying about 80%. If you start to burn through your error budget at an unexpectedly fast rate, it would be nice for an alert to fire to let you know.

1. There are several ways to create an alert for an SLO in Service Monitoring. Because you are looking at the expanded SLO interface, click the **Alerts firing** tab and select **CREATE SLO ALERT**.

![SLOALERT](https://cdn.qwiklabs.com/dvI2Wh9%2Bm2w46RSlI9UvxPvNzoeBk8U3C00ysY0jjhQ%3D)

1. Set the `Display name` to **Really short window test**. Because you are doing a test and not setting values, that would make sense in production.
2. Set the `Lookback duration` to **10** minutes and the burn rate threshold to **1.5**.
3. Click **Next**.
4. Click on drop down arrow next to **Notification Channels**, then click on **Manage Notification Channels**.

![email.png](https://cdn.qwiklabs.com/akOFdDyEuKOmhn6BWOrBP7cGEvEy5jTDg28Q71R6Xms%3D)

A **Notification channels** page will open in new tab.

1. Scroll down the page and click on **ADD NEW** for **Email**.

![add_email.png](https://cdn.qwiklabs.com/%2B3S74DM%2BjBdvWDyjMwuEzhk76T8GS6jA3aSpeeTsfvU%3D)

1. In **Create Email Channel** dialog box, enter your personal email address in the **Email Address** field and a **Display name**.
2. Click on **Save**.
3. For `Who should be notified`, use the `Manage notification channels` link to add your email address as a notification channel and select that. Remember, this link opens a new tab so close it once your email address has been added, and then **Save** the new alert.
4. Click on **Notification Channels** again, then click on the **Refresh icon** to get the display name you mentioned in the previous step.

![refresh_icon.png](https://cdn.qwiklabs.com/fXnoFzeQzHQdc905vD6QLzfKuwfk3F0BXZZADgsaUaQ%3D)

1. Now, select your **Display name** and click **OK**.
2. Click **Next**.
3. Skip **What are the steps to fix the issue? (optional)** and click **Save**.
4. On the SLO page, switch back to the **Service level indicator** tab. It should not display our alert as a red dotted line. Once again, clicking the line will zoom in the view. In the upper-right corner of the page, click **Auto Refresh** so the charts update automatically.

### Trigger the alert

1. Modify our application and trigger the alert. Switch back to your Cloud Shell view and **Open Editor**, if it's not already displayed, and re-open `index.js`.
2. Scroll to the `/random-error` route found at approximately line 126 and modify the value next to `Math.random` from 1000 to 20. So instead of generating an error every 1000 requests, we are not going to get an error every 20 requests. That will drop our availability from 99.9$ to about 95%, which should trigger the alert.
3. Close the Cloud Shell code editor and switch to the terminal window. You have two tabs, one that's running the test loop and one that's standard. In the standard (non-busy) tab, redeploy the change to App Engine.

```bash
gcloud app deploycontent_copy
```

1. When prompted, type **y** and press **Enter**.
2. Once the redeploy completes, switch to the tab running the test loop and verify the uptick in errors.
3. Switch back to the your Service Monitoring page and in the upper-right corner, verify a green check next to auto refresh. Verify that your SLO is expanded and that you can see the Service level indicator. After a few minutes, the SLI value and chart should show clearly the decrease in performance down to about the 95% level. Within a few minutes, you should also receive the alert notification email.

![SLO Violation](https://cdn.qwiklabs.com/gSlkDwX2X0s%2Fi3fxjptyMXedx5DtHOF9HIaRpkjwiA4%3D)