---
layout: post
title:  "Going Serverless with Azure Functions and DocumentDB - Part 1"
date:   2017-01-17 9:00:00 +0000
categories: azure functions serverless
---
Today, I had a need to get my head around Azure Functions, so I decided to build a Function App which would take data from a public API and store it in Azure. The idea I came up with was to extract a list of organisations from GitHub and then build a list of some of their repos and star counts and store it in a database.

I came up with the following solution.

* Use one Azure Function to retrieve a list of organisations from GitHub and write their ID, name and the URL for their repos list to a queue.
* Use a second Azure Function to process messages from the queue, querying GitHub for the list of repos, creating a JSON document with the organisation ID, name and repos with star counts and then writing the results to an Azure DocumentDB database.

The first function provides the input for the second function to process. In this tutorial, we will limit the amount of data retrieved from the GitHub API to keep things simple. If, however, you were working with a large data source then our approach is a good way of making sure that the functions don't spend too much time running. If we were to try and do this with one function, it would retrieve the list of organisations from GitHub, then loop through them querying for more data and building the results list, which could be time consuming. With the approach we will build here, the first function runs relatively quickly on a schedule, the second function is then triggered multiple times to process one queue entry.

The processed data will be written to Azure DocumentDB. Combined with Azure Functions this gives us a ***serverless*** solution, where we have no servers, no operating systems and no databases to patch, run or otherwise maintain.

###  Walkthrough

In part 1, we will set up some pre-requisites, create the Azure Function App and write our first function. In part 2, we will write the second function.

To create this tutorial I've already been through the process of creating my first Azure Functions and I can honestly say that they truly are very simple and fast to build. But as with any new technology, my learning began with that daunting feeling you always get when you start looking at the first page of unfamiliar documentation. *Where do I start with this?* So, here's a walkthrough of how I got my Azure Functions working.

###  0. Pre-Requisites

There are a couple of things to do prior to getting started with this tutorial. 

* First, it's highly recommended that you install the [Azure Storage Explorer](http://storageexplorer.com/) as it's helpful 
during development to see what's going on in the Azure Storage Queue that we'll be using.
* Second, create a DocumentDB account in your Azure Subscription. You don't need to create a database or a collection as we can do that via the Azure Function.

To create a DocumentDB account, go to the Azure Portal at [https://portal.azure.com](https://portal.azure.com) and once signed-in click the **+ New** link at the top left, choose **Databases** and then **NoSQL (DocumentDB)**

![](/images/2017/01/functions-create-documentdb.png)

For **account ID** you just need to provide a unique name for your DocumentDB account. Leave the **NoSQL API** value at its default of **DocumentDB** and choose the appropriate Azure subscription from the list. Provide a unique name for the Resource Group that will contain your DocumentDB resources and then select an Azure region from the **Location** list. If you don't know which region to choose, just pick one that's near you.

###  1. Create a Function App

If you haven't used Azure Functions before, the starting point is here [https://functions.azure.com/](https://functions.azure.com/). From there you can either sign in to your existing Azure subscription or create a free account to try Functions out. Once you're done with the sign-up / sign-in, you'll get to here:

![](/images/2017/01/getting-started-azure-functions.png)

If you have more than one Azure subscription, you can select it here. You then need to choose a unique name for your Function App and select an Azure Region where your function will be deployed to. If you're unsure which region to choose, just pick one that's nearest to whatever part of the world you're in.

###  2. Create the First Function

A few moments after you click the **Create + get started** button you'll be taken to the Azure Portal where the Function app blade will be open at the quick start page

![](/images/2017/01/functions-quick-start.png)

Using these options, it's really simple to get started quickly with a pre-built Azure Function. The first function we are going to create will run on a schedule, so we'll choose the **Timer** scenario and **C#** as the language. Click the **Create this function** button. After a few moments you'll find yourself in the Azure Portal on the Function App blade. You'll get a few useful hints to help you find your way around, but once you're done with them you should find yourself in the **Code** view.

![](/images/2017/01/functions-code-view-start.png)

Near the top right, click the "View Files" and "Logs" buttons. This will open up some additional panels which will make things a little easier. In the files panel, which will appear on the right when you click **View Files**, there are two files where all the magic of Azure Functions happens!

* **run.csx** is a C# script file which is where we will write our code.
* **function.json** provides some configuration information for our function, including details of its inputs and outputs.

At this point, you have a fully working Azure Function. It's a timer function and it's configured to fire every five minutes. If you click the **Run** button at the top of the code panel, the function will run. Nothing will happen, but if you click **Logs** near the top right, you'll get a log panel appear at the bottom and in there you should see something like:

    2017-01-17T11:25:00.013 Function started (Id=de84439b-b897-4890-8803-4ecfd59a670d)
    2017-01-17T11:25:00.013 C# Timer trigger function executed at: 1/17/2017 11:25:00 AM
    2017-01-17T11:25:00.013 Function completed (Success, Id=de84439b-b897-4890-8803-4ecfd59a670d)

The second line in the log entry is the one that's written out by our function. So, let's get this function doing something more interesting than just writing to a log.

###  3. Modify the Function Configuration

There are two things we want to modify. First, we don't want this to run every 5 minutes. Instead, we want this to run hourly. Second, we need to output the results of the function to an Azure Storage queue.

Making sure you have the files panel open, click on the `function.json` file and you should see this:

    {
      "disabled": false,
      "bindings": [
        {
          "name": "myTimer",
          "type": "timerTrigger",
          "direction": "in",
          "schedule": "0 */5 * * * *"
        }
      ]
    }

The `disabled` property is set to `false` by default, which means our function is running, or ready to run. In some circumstances you may want to stop the function from firing, which is easily achieved by changing `disabled` to `true`. 

The `bindings` section configures the input and outputs for the function. In this case, we have a `timerTrigger` as an input, indicated by the `direction` property being set to `in`. The `schedule` indicates how often the function will run and uses cron format. In this case, it's set to run every 5 minutes. 

We can edit the `function.json` but Azure Functions provides another way to do this. On the left side of the portal we have four options:

![](/images/2017/01/functions-dev-int-man.png)

**Develop** is the view we're looking at now. If you click on **Integrate** you'll see a page with three options at the top - **Triggers**, **Inputs** and **Outputs**. There should be a box underneath **Triggers** marked **Timer (myTimer)**. Click that box and you'll see the following:

![](/images/2017/01/functions-trigger-1.png)

At the lower right, you can see **Schedule**. Here we can enter a new schedule in cron format. The following will set the schedule to run hourly.

    0 0 * * * *

Update the schedule with the above and click the Save button.

Next, we want to add an Azure Storage Queue output, so that we can send the results of our Function to a queue. Azure Functions makes this simple. Whilst still on the **Integrate** page click on **+ New Output** and you'll get some choices:

![](/images/2017/01/functions-outputs.png)

We want to add a message queue, so click on **Azure Queue Storage** and then click the **Select** button. We now have to provide some information.

![](/images/2017/01/functions-queue-output.png)

The **Message parameter name** will be a parameter in our code which will contain the data we want to place into the queue. The **Queue Name** is the name of the queue we will be placing our data into. The **Storage account connection** is the name of an environment variable in the Azure Function App which contains the connection string for our storage account. The default values will be fine for our purposes, so click the **Save** button.

If you click back to the **Develop** view and open `function.json` in the Code panel, you'll see that the configuration has been updated:
```
{
  "bindings": [
    {
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in",
      "schedule": "0 0 * * * *"
    },
    {
      "type": "queue",
      "name": "outputQueueItem",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Our `in` bindings are still there, but we now also have an `out` binding for the queue. You'll notice that the timer schedule has been modified as well.

### 4. Create a Queue in Azure Storage

When you created the Azure Function App, it automatically created a storage account and pre-configured the connection string. This all helps to simplify the setup. However, we still need to do one or two things ourselves. We need to create a queue in our storage account. This is where the Azure Storage Explorer comes in handy. Start it up and login to your Azure account and you should see that you have a Storage Account named with an "azurefunctions" prefix.

![](/images/2017/01/functions-storage-explorer.png)

Right click **Queues**, choose **Create Queue** from the menu and then name the queue `outqueue` to match the value we provided when we added the queue output earlier.

![](/images/2017/01/functions-storage-explorer-outqueue.png)

### 5. Write the Code

We're ready to write some code! Return to the Azure Portal and make sure you're back in the **Develop** view with the **Code** panel visible. If it's not open already, open the **View Files** panel and make sure `run.csx` is selected. It should currently look like this:

    using System;
    
    public static void Run(TimerInfo myTimer, TraceWriter log)
    {
        log.Info($"C# Timer trigger function executed at: {DateTime.Now}");    
    }

We need to add in the output parameter which we do by adding `outputQueueItem` as follows

    public static void Run(TimerInfo myTimer, ICollector<string> outputQueueItem, TraceWriter log)

We add the parameter as an `ICollector` of type `string` because the output binding supports multiple items. Each run of our function will generate multiple message queue items and we will build a collection of those items in our code.

We're going to make a call to the GitHub API and then process the JSON data returned. Azure Functions includes a selection of frequently used namespaces and assemblies, including those used for making HTTP calls and processing JSON data, so we need to modify the start of our code as follows:

```
#r "Newtonsoft.Json"

using System;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

We can now remove the `log.Info ...` line at the start of our code and replace it with the code to make the HTTP calls to the GitHub API.

    {
        HttpClient client = new HttpClient();
        client.DefaultRequestHeaders.UserAgent.Add(new System.Net.Http.Headers.ProductInfoHeaderValue("Mozilla", "5.0"));
        JArray response = JArray.Parse(await client.GetStringAsync("https://api.github.com/organizations?per_page=5"));

The `client.DefaultRequestHeaders ...` line is necessary as the call to the GitHub API will fail if we don't set a user agent in the HTTP header that we send.

The third line makes the call to the GitHub API. You'll notice that it uses `await` so we'll need to modify our method to make it `async` and return a `Task`.

    public static async Task Run(TimerInfo myTimer, ICollector<string> outputQueueItem, TraceWriter log)

Finally, we need to process the data returned from the API and add the data that we want written to the queue. I'm creating a new JSON document which contains a subset of the information returned by the GitHub API - the organisation name, ID and URL to query for their list of repos.

    foreach (JObject org in response)
    {
        object document = new {
            id = org["id"].ToString(),
            orgname = org["login"].ToString(),
            repos = org["repos_url"].ToString()
            };
        string json = JsonConvert.SerializeObject(document);
        outputQueueItem.Add(json);
    }

The really nice thing about functions is that simply adding to my `outputQueueItem` collection will result in the data being written to the queue. I don't need to write any code, or include any external assemblies to specifically perform the write of the data to Azure Queue Storage. When the function completes, the content of `outputQueueItem` will be sent to the queue we specified in the `function.json` file.

Here's our complete code:
```
#r "Newtonsoft.Json"

using System;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public static async Task Run(TimerInfo myTimer, ICollector<string> outputQueueItem, TraceWriter log)
{
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.UserAgent.Add(new System.Net.Http.Headers.ProductInfoHeaderValue("Mozilla", "5.0"));
    JArray response = JArray.Parse(await client.GetStringAsync("https://api.github.com/organizations?per_page=5"));
    foreach (JObject org in response)
    {
        object document = new {
            id = org["id"].ToString(),
            orgname = org["login"].ToString(),
            repos = org["repos_url"].ToString()
            };
        string json = JsonConvert.SerializeObject(document);
        outputQueueItem.Add(json);
    }
}
```
If we now click the **Save** button at the top of the **Code** panel, we will see a message in the logs indicating that the script has changed and, hopefully, that compilation has been successful:
```
2017-01-17T16:16:13.623 Script for function 'TimerTriggerCSharp1' changed. Reloading.
2017-01-17T16:16:13.685 Compilation succeeded.
```

Finally, we can click the **Run** button to see what our function does.
```
2017-01-17T16:17:37.480 Function started (Id=dacedfbc-faaf-467f-94d0-153f1bf61da8)
2017-01-17T16:17:38.691 Function completed (Success, Id=dacedfbc-faaf-467f-94d0-153f1bf61da8)
```
Not much, apparently. But wait, let's take a look in the Azure Storage Explorer:

![](/images/2017/01/functions-storage-explorer-queue-content-3.png)

A-ha, there's our data!

That's the first part of our solution completed. Next we need to process the items in the queue, query the data we need from the GitHub API and write it out to DocumentDB. All of which is really simple with Azure Functions! Find out how in [Part 2](https://markw.me/going-serverless-with-azure-functions-and-documentdb-part-2/).