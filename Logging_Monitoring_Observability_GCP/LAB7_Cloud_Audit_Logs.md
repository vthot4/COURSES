# Cloud Audit Logs

2 hoursFree

## Overview

In this lab, you investigate Google Cloud Audit Logs. Cloud Audit Logging maintains multiple audit logs for each project, folder, and organization, all of which help answer the question, "Who did what, when, and where?"

## Objectives

In this lab, you learn how to:

- Enable data access logs on Cloud Storage.
- Generate admin and data access activity.
- View Audit logs.

## Task 0. Lab Setup

### Access Qwiklabs

For each lab, you get a new GCP project and set of resources for a fixed time at no cost.

1. Make sure you signed into Qwiklabs using an **incognito window**.
2. Note the lab's access time (for example, ![img/time.png](https://cdn.qwiklabs.com/aZQJ4BT7uCmM9XR6BTXgTRP1Hfu1T7q6V%2BcnbdEsbpU%3D) and make sure you can finish in that time block.

There is no pause feature. You can restart if needed, but you have to start at the beginning.

1. When ready, click ![img/start_lab.png](https://cdn.qwiklabs.com/XE8x7uvQokyubNwnYKKc%2BvBBNrMlo5iNZiDDzQQ3Ddo%3D).
2. Note your lab credentials. You will use them to sign in to Cloud Platform Console
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

## Task 1. Enable data access logs on Cloud Storage

Data Access audit logs (except for BigQuery) are disabled by default. Let's start by enabling data access logs on Cloud Storage.

1. In the Google Cloud Console window, use the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) to navigate to **IAM & Admin > Audit Logs**.
2. Scroll or use `Filter table` to locate `Google Cloud Storage`, then check the box next to it. This should display the `Info Panel` with options on `LOG TYPE`.

Data Access audit logs are divided into different categories:

- Admin Read: Records operations that read metadata or configuration information. Admin Activity audit logs record writes of metadata and configuration information. They can't be disabled.
- Data Read: Records operations that read user-provided data.
- Data Write: Records operations that write user-provided data.

1. Select **Admin Read**, **Data Read** and **Data Write**, and then click **SAVE**.

## Task 2. Generate some admin and data access activity

The data access logs are now enabled for Cloud Storage. Use Cloud Shell to upload a file to a new storage bucket, create a network, create a VM, and then examine the Cloud Audit Logs.

1. Open or switch to your Cloud Shell terminal.
2. Use `gsutil` to create a Cloud Storage bucket with the same name as your project.

```bash
gsutil mb gs://$DEVSHELL_PROJECT_IDcontent_copy
```

1. Make sure the bucket successfully created.

```bash
gsutil lscontent_copy
```

1. Create a simple "Hello World" type of text file and upload it to your bucket.

```bash
echo "Hello World!" > sample.txt
gsutil cp sample.txt gs://$DEVSHELL_PROJECT_IDcontent_copy
```

1. Verify the file is in the bucket.

```bash
gsutil ls gs://$DEVSHELL_PROJECT_IDcontent_copy
```

1. Create a new auto mode network named **mynetwork**, then create a new virtual machine and place it on the new network.

```bash
gcloud compute networks create mynetwork --subnet-mode=auto
gcloud compute instances create default-us-vm \
--zone=us-central1-a --network=mynetworkcontent_copy
```

1. Delete the storage bucket.

```bash
gsutil rm -r gs://$DEVSHELL_PROJECT_IDcontent_copy
```

## Task 3. Viewing audit logs

Admin Activity logs contain log entries for API calls or other administrative actions that modify the configuration or metadata of resources. For example, the logs record when VM instances and App Engine applications are created, or when permissions are changed. To view the logs, you must have the Cloud Identity and Access Management roles Logging/Logs Viewer or Project/Viewer.

Admin Activity logs are always enabled so there is no need to enable them. There is no charge for your Admin Activity audit logs.

### Using the Activity page

You can view abbreviated audit log entries in your project's **Activity page** in the Google Cloud Console. The actual audit log entries might contain more information than you see in the Activity page. The Activity page is good for a quick check of account activity.

1. Switch to the browser tab showing the Google Cloud Console and use the **Navigation menu** to navigate to **Home**.
2. Click the **ACTIVITY** button near the top left. At the top of the table, you see the changes we made in the previous part of this exercise (Delete bucket, Create VM, Create network, etc.). If you do not see the activity, reload the page.

![6f38d7067538c77.png](https://cdn.qwiklabs.com/6QKDxOZM7kg%2FSIdwC7j88BKJAwsitZWKNAbPolrwtqw%3D)

1. If the Filter pane is not displayed on the right, click the **Filter** button on the top right.
2. In the Filter pane, click the **Activity types**, select all, and click **OK**. Note the Cloud Storage file operations are displayed as well.
3. In the filter pane, click the **Resource type**, uncheck **All XX selected** checkbox, select **GCE Network**, and click **OK**. The Activity table now only shows the network that was created at the start of the lab.
4. Feel free to explore other filters to help locate specific events. Filters can help locate events or verify which events occurred.

### Using the Logging page

1. Use the **Navigation menu** to navigate to **Logging > Logs Explorer**.
2. If the top of the Logging page looks like the below, use the **Classic** dropdown to **Preview the new Logs Explorer**.

![Classic Log View](https://cdn.qwiklabs.com/X%2BGirH3HFxt%2BRH6C6y%2BQHZKgI%2BpDbnDLntTWAjLEA6E%3D)

1. Click into the `Query builder` and delete the contents.
2. Click the **Log name** dropdown and use the filter to locate the **activity** log under **CLOUD AUDIT** section., and **Add** it to the query.
3. Press the **Run query** button, and then use the `Log fields` explorer to filter to **GCS Bucket** entries.
4. Locate the log entry for when the Cloud Storage was deleted.
5. Expand the delete entry, then drill into **protoPayload** > **authenticationInfo** field and notice you can see the email address of the user that performed this action.
6. Feel free to explore other fields in the entry. Also, notice how many of the values can be clicked to add inclusions/exclusions to the query.
7. Delete the existing query and use **Log Name** to view the **data_access** logs. What operations can you see now?

### Using the Cloud SDK

Log entries can also be read using the Cloud SDK command:

Example (**Do not copy**):

```bash
gcloud logging read [FILTER]content_copy
```

1. Switch to or reopen a Cloud Shell terminal.
2. If we wanted to see those same data access logs using the command line, we could run the following:

```bash
gcloud logging read \
"logName=projects/$DEVSHELL_PROJECT_ID/logs/cloudaudit.googleapis.com%2Fdata_access"
```