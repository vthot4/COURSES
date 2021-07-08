# Application Performance Management

2 hoursFree

## Overview

In this lab, you use the Google Cloud Debugger to track down a bug in an application deployed to App Engine. You see how to create breakpoints, inspect values, and dynamically add logging information to a running Google Cloud application. You also examine errors and trace spans in a Cloud Run application.

### Objectives

In this lab, you learn how to perform the following tasks:

- Download and deploy our two applications.
- Debug your App Engine app using the Google Cloud Debugger.
- Add Log Data.
- Fix the bug and deploy a new version.
- Examine an error report coming out of Cloud Run in Error Reporting.
- Examine a default and custom trace span.

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

## Task 1. Download a pair of sample apps from GitHub

1. Switch to your Cloud Shell terminal and enable the APIs for Cloud Build and the Container Registry as we will need them in a few steps.

   ```bash
   gcloud services enable cloudbuild.googleapis.com
   gcloud services enable containerregistry.googleapis.com
   gcloud services enable run.googleapis.comcontent_copy
   ```

2. We are going to deploy two applications we can use to explore the Google Cloud Operations suite of Application Performance Management (APM) products. Let's start by deploying the HelloLoggingNodeJS application we've used previously into Cloud Run. Clone the repo into Cloud Shell.

   ```bash
   cd ~/
   git clone https://github.com/haggman/HelloLoggingNodeJS.gitcontent_copy
   ```

3. Change into the **HelloLoggingNodeJS** folder and use the `rebuildService.sh` script to deploy the application into Cloud Run.

   ```bash
   cd ~/HelloLoggingNodeJS
   sh rebuildService.shcontent_copy
   ```

4. While that application is deploying, open a new tab in the Cloud Shell Terminal window using the ＋ at the top. Here we work on getting the second application deployed.

5. Clone the `haggman/gcp-debugging` repository from GitHub into your Cloud Shell instance.

   ```bash
   git clone https://github.com/haggman/gcp-debuggingcontent_copy
   ```

6. Change into the **gcp-debugging** folder containing the sample app.

   ```bash
   cd ~/gcp-debuggingcontent_copy
   ```

7. Use the following command to open `main.py` in the Cloud Shell editor.

   ```bash
   edit main.pycontent_copy
   ```

8. Take a moment and explore the code. This is a simple Python Flask web app. Initially, the app displays a basic home page. When you click `Convert Temp`, you are sent to the `/temp` endpoint and the GET request presents a *Convert Fahrenheit to Celsius* page, prompting you for a temperature value. Submitting the form POSTs back to the same `/temp` endpoint. The code then returns a page displaying the converted value. Both pages are built from templates found in the `templates` folder.

9. Before deploying the application to the cloud, test it locally in Cloud Shell. First, install the application dependencies using the Python `requirements.txt` file, and then execute `main.py`.

   ```bash
   sudo pip3 install -r requirements.txt
   python3 main.pycontent_copy
   ```

10. You see that the terminal window is in a hold mode as the application runs in the background. To see the program running, click the **Web Preview** button in the toolbar of Google Cloud Shell, and then select **Preview on port 8080**.

![Web Preview](https://cdn.qwiklabs.com/8rE%2Fz5TD4tZmrL1ttQgU%2F9V7T%2BjG7BGAuKgXzItjbIQ%3D)

1. At the top of the page, click the **Convert Temp** link. The following screen is displayed.

![convert_temp](https://cdn.qwiklabs.com/ur6P8DIuSMpbeTPKZx4lwLzZ%2BibHEr6ktVDRTVjLqWk%3D)

1. Delete the text in the text box, enter **212**, and click the **To Celsius** button. The result should be 100.
2. Enter some garbage text like **foo**. How does the application handle this?
3. Close the browser tab for the running application and switch back to Cloud Shell. Press CTRL+C to stop the Flask application.

![temp() Code](https://cdn.qwiklabs.com/lNtCM3JMuXCfwR3o8v16V7IT2hRqb9KxGEbnfgT8hl8%3D)

1. Take a moment to re-examine the above code, specifically the `temp()` function. When the button on the Convert Temp page is pressed, a POST is made. That is handled by the if block starting at line 20. At line 23 the user input is retrieved; this is the temperature in Fahrenheit. Notice below that the `isnumeric()` function is used to ensure a number was entered. If the input is valid, the conversion to celsius is done. If not, an error message is returned.

## Task 2. Deploy the converter application to App Engine

The code is working, but only locally on the Cloud Shell VM. To test our code in a more production-like environment, deploy it to Google's fully-managed App Engine.

1. Open `app.yaml` in the Cloud Shell editor.

   ```bash
   edit app.yamlcontent_copy
   ```

2. Take a moment to explore the file. This is a configuration file telling App Engine the type of environment the code needs when it runs.

3. App Engine needs an application created before it can be used. This is done just once using the `gcloud app create` command and specifying the region where you want the app to be created. This command takes a minute or so. Please wait for it to complete before moving on.

   ```bash
   gcloud app create --region=us-centralcontent_copy
   ```

4. Deploy the Flask application into App Engine. This command takes a minute or three to complete. Please wait for it before moving on.

   ```bash
   gcloud app deploy --version=one --quietcontent_copy
   ```

5. In the Google Cloud Console window, use the **Navigation menu** (![Navigation menu](https://cdn.qwiklabs.com/tkgw1TDgj4Q%2BYKQUW4jUFd0O5OEKlUMBRYbhlCrF0WY%3D)) to navigate to **App Engine | Dashboard**.

6. In the upper-right corner of the dashboard, click the link (`qwiklabs-gcp-???????.appspot.com`) to your application.

7. Click the **Convert Temp** link and replace the text in the text box with **212**. Click the button and it returns to 100.

## Task 3. Debug the application

Even after all of the *thorough* testing, users have reported that the program doesn’t always return the correct answer. Use Google Clouds Debugging to investigate.

1. Take a moment and test the application with various possible numeric temperature values. Can you find any that won't work? If not, don't let that keep you from moving to the next step.
2. In the Google Cloud Console, use the **Navigation menu** to navigate to **Debugger**.
3. On the **Debugger** console, check mark the box for **I consent to Google collecting and storing my OAuth token** and then click **AUTHORIZE**.

![authorize_code](https://cdn.qwiklabs.com/cTFaqTZWFHYwKFCUXv%2B1j2Id6cEF5maxtDz3gb9LJG4%3D)

1. Refresh the **Debugger** console page.
2. Since the Debugger is tightly integrated with App Engine, it has a copy of your application code and has your application. In the left-hand tree, select **main.py**.
3. Add a breakpoint so you have a snapshot of the application at the moment in time when it returns the temperature page. Locate the line which returns the rendered `temp.html` page (around line 34). Click the grey area next to code line number to set a breakpoint.

![Add Breakpoint](https://cdn.qwiklabs.com/HOcytuNqlpRiN%2BSqYNOfNr55tcksP%2BqQAO0iLFYhnbQ%3D)

1. Click **CREATE SNAPSHOT**.
2. Notice the window to the right of the debugger. We could set a condition that must be met before a snapshot is taken. At the bottom, we see a message that the Debugger is waiting to take a snapshot.

![Snapshot Condition Pane](https://cdn.qwiklabs.com/RNwhjwxBDyGtGXwG%2F%2BUUP9h5nEdBdkNIUyVslOjlv6s%3D)

1. Go back to the temperature converter page and convert 212 (this should work). Switch back to the Debug screen and check the results.
2. Take a moment to explore the data available in the snapshot. The `input` was the string 212. That was converted to the numeric `fahrenheit` 212.0. The calculation was correct and resulted in the 100.0 in `celsius`.

![Snapshot 212 to 100](https://cdn.qwiklabs.com/hDMuvfhEmJceoXr6rJIe13EGy728m6eQo89xmumgm5Y%3D)

1. In the Snapshot window, click the **RETAKE** to re-enable the breakpoint.

![retake_snapshot](https://cdn.qwiklabs.com/gDbtfsbbSQuhH8c4UELmR06LgTBUFDgrc1TBAbGj0jQ%3D)

1. Go back to your app converter page, enter **-40** in the text box, and click the button. Switch back to the Debugger and investigate the results.

![Bug Snapshot](https://cdn.qwiklabs.com/9JahG80Ba01lous7jZSwrI8B38BN3cs6RhFtQDHvAD0%3D)

1. Notice that the input was correct, the string of -40, but clearly the value was not converted correctly to a number. The -40 indicates that it's still in a string format. Take a look at code lines 20 through 34 and speculate what might be wrong.

## Task 4. Adding log data

In addition to seeing the code and current variable values using Debugger snapshots, it's also possible to easily add free-form logging information using Logpoints. Add some Logpoints to further explore our code.

1. The `isnumeric()` function is not working correctly. In the upper-right corner of the Debug screen, click the **Logpoint** link.

![Logpoint Link](https://cdn.qwiklabs.com/oI%2Fmt2dnYSXO0yhTexedhK2VbBwxG%2BnVE%2B00PyKfVQE%3D)

1. Click next to line 31 and then click **CREATE LOGPOINT**, the logpoint form appears.

![New Logpoint](https://cdn.qwiklabs.com/0eXctIDSxAgI%2FjjvUlqTFnVMVlODCmbh6BvNRZjQGNI%3D)

1. Set the log level from Info to **Warning**, then change the `var = {var}` placeholder to: **{input} was cast to {fahrenheit}**, and press the **Add** button.

![log_level_warning](https://cdn.qwiklabs.com/UtGKvYclRJ6EHVk1YuwE76xF445icxVa8REfHu0O5EU%3D)

1. Go back to your app converter page, test the values 212, 32, -40, and 98.6.
2. Return to the Debugger screen and click the **View logs** link in the Logpoints window.

![view_logs](https://cdn.qwiklabs.com/sccAV7jSbFFfvelNKlGa75tCiE9ptDA%2BcGXmX2tYhRI%3D)

1. Take a moment and explore the log values. It appears that the `isnumeric()` function, used in the test on line 24, is returning a `false` for negative numbers and numbers with a decimal. That isn't correct.

![Logpoint Log Values](https://cdn.qwiklabs.com/TTf%2F02FfYDRLVjF3Pm%2FGqerx7MLBG0F02gtEZh%2BXKSs%3D)

## Task 5. Fix the bug and deploy a new version

Now that we have a fairly good idea where the bug is, fix the code, deploy a new version of the application, and test to make sure the bug is actually resolved.

1. Return to your Cloud Shell window, find the `main.py` file in the `gcp-debugging` folder, and open it in the Cloud Shell editor.

2. If you’re highly motivated (and a coder), try to fix the code yourself. If not, replace the `if-else` block on lines 24 through 29 with the following `try-catch` block. This is Python, so make sure you get the spacing correct.

   ```python
           try:
               fahrenheit = float(input)
               celsius = int((fahrenheit - 32.0) * 5.0 / 9.0)
           except ValueError:
               fahrenheit = 'Enter a number'
               celsius = 'Invalid Input'content_copy
   ```

3. Make sure your finished code resembles the following screenshot:

![Fixed Code](https://cdn.qwiklabs.com/JTWKayy7X0T8EDKq1Fa%2BLcdIYBmO5Z9A7RryN5%2FoJAk%3D)

1. In the Cloud Shell terminal window, change into the `~/gcp-debugging` directory and deploy a new version of the application to App Engine. It will take a few moments for this to complete; don't move on until it does.

   ```bash
   cd ~/gcp-debugging
   gcloud app deploy --version=two --quietcontent_copy
   ```

2. After the command completes, go back to your temperature converter web page and test it. It should work now.

## Task 6. Examine an error report coming out of Cloud Run in Error Reporting

Now that you've seen the debugger at work, let's see how it integrates with Error Reporting.

1. Switch back to the Cloud Shell tab you used to deploy the HelloLoggingNodeJS to Cloud Run. Open the index.js file in the Cloud Shell editor.

```bash
cd ~/HelloLoggingNodeJS
edit index.jscontent_copy
```

1. Notice that Error Reporting is already enabled in the code (look for the `//Enable Error Reporting` comment).

2. Use the **Navigation menu** navigate to **Cloud Run**. Select **hello-logging** and click the link to the application. We've see this before.

3. Modify the URL by adding **/uncaught** to the end. You might recall from an earlier module that the `/uncaught` route in our application throws an uncaught exception. Refresh the page a few times.

4. Use the **Navigation menu** to view **Error Reporting**. You should see the `ReferenceError...` message generated from hello-logging. Click the error to view the details.

5. Locate the stack trace and notice that the error is coming out of index.js. Click the link to the code and the view should switch to the Debugger. At the top of the Debugger, the hello-logging application is not selected by default, we have to select it manually.

6. Switch to the Cloud Shell terminal and make sure you are in the HelloLoggingNodeJS folder. Create a new Google Cloud Source Repository git repo named **hello-world**

   ```bash
   cd ~/HelloLoggingNodeJS
   gcloud source repos create hello-worldcontent_copy
   ```

7. Push a copy of the code into the project Git repository.

   ```bash
   git push --mirror \
   https://source.developers.google.com/p/$GOOGLE_CLOUD_PROJECT/r/hello-worldcontent_copy
   ```

8. Switch to the Google Cloud Console window and use the **Navigation menu** to pull up **Error Reporting**. Once again, drill in to the **ReferenceError...**, locate the `Stack trace sample` and click the **/usr/src/app/index.js** link to launch the Debugger.

9. Verify the project is your Qwiklabs project, thn select the **hello-world** repository, and choose the **master** branch, and then click **Select source**.

10. Refresh the **Debugger** console page. This time, click **Select source** in the Cloud Source Repositories section.

11. Once again, use the **Navigation menu** to pull up **Error Reporting**. Drill in to the **ReferenceError...**, locate the `Stack trace sample` and click the **/usr/src/app/index.js** link to launch the Debugger. What did it do this time?

12. Refresh the page generating the error. Debugger and you see it's taken a debugger snapshot. Take a moment to explore it.

## Task 7. Examine a default and custom trace span

Now that you have spent some time with Debugger and Error Reporting, you examine a Trace.

### Default trace

1. Switch back to the Cloud Shell tab and open the **index.js** file in the Cloud Shell editor. Examine the first statement in the application code and you see the Google Cloud trace agent has been loaded and started.
2. Use the **Navigation menu** to return to **Cloud Run**. Select your `hello-logging` application and visit its URL. The Hello World message should appear.
3. Back in the Console window, use the **Navigation menu** to select **Trace**.
4. Take a moment to explore some of the trace spans by selecting the dots under `Select a trace`.
5. Switch back to the browser tab where you viewed your application and modify the URL by adding **/slow** to the end. Wait for the page to load. It should take a couple of minutes.
6. Switch back to **Trace**. Use the switch at the top of the page to enable **Auto Reload**. Locate and select the latest, and highest, trace span. What information do you get from the trace?

### Custom trace

1. By default, Trace only captures spans where services call each other. Since the `/slow` route does a calculation but doesn't call any other services, the telemetry Trace provides is limited. Fix that by adding a couple of manual trace spans. Switch back to the Cloud Shell tab and open the **index.js** file in the Cloud Shell editor.

2. Scroll down to the `/slow` route. Edit or replace the method so it resembles the following:

   ```javascript
   //Generates a slow request
   app.get('/slow', (req, res) => {
       const span1 = tracer.createChildSpan({name: 'slowPi'});
       let pi1=slowPi();
       span1.endSpan();
   
       const span2 = tracer.createChildSpan({name: 'slowPi2'});
       let pi2=slowPi2();
       span2.endSpan();
       res.send(`Took it's time. pi to 1,000 places: ${pi1}, pi to 100,000 places: ${pi2}`);
   });
   content_copy
   ```

3. Return to the Cloud Shell terminal window and rebuild/redeploy the application. Wait for the redeploy to complete before moving on.

   ```bash
   sh rebuildService.shcontent_copy
   ```

4. Return to the browser tab where you tested your application. Edit the URL and visit the application home page to verify that the application is up and running again. Modify the URL again back to **/slow**. Wait for the page load to complete.

5. In the Google Cloud Console, navigate back to **Trace**. Investigate the latest trace, what additional information can you see in the latest `/slow` span? Where it the majority of the code latency being spent?