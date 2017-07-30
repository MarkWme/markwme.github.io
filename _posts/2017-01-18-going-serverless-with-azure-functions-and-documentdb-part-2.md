---
layout: post
title:  "Going Serverless with Azure Functions and DocumentDB - Part 2"
date:   2017-01-18 9:00:00 +0000
categories: azure functions serverless
---
In [Part 1](https://markw.me/going-serverless-with-azure-functions-and-documentdb-part-1/) we created an Azure Function that queried the GitHub API to retrieve a list of organisations and write the organisation name, id and URL for their GitHub repos to a queue in Azure Storage. In Part 2, we will write an Azure Function that listens for incoming messages on that queue, queries the GitHub API for details of the repos and then writes the data out to DocumentDB.

 0. Pre-Requisites

If you've followed through Part 1, then you will already have these completed, but just as a reminder:

* First, it's highly recommended that you install the [Azure Storage Explorer](http://storageexplorer.com/) as it's helpful 
during development to see what's going on in the Azure Storage Queue that we'll be using.
* Second, create a DocumentDB account in your Azure Subscription. You don't need to create a database or a collection as we can do that via the Azure Function.

To create a DocumentDB account, go to the Azure Portal at [https://portal.azure.com](https://portal.azure.com) and once signed-in click the **+ New** link at the top left, choose **Databases** and then **NoSQL (DocumentDB)**

![](/images/2017/01/functions-create-documentdb.png)

For **account ID** you just need to provide a unique name for your DocumentDB account. Leave the **NoSQL API** value at its default of **DocumentDB** and choose the appropriate Azure subscription from the list. Provide a unique name for the Resource Group that will contain your DocumentDB resources and then select an Azure region from the **Location** list. If you don't know which region to choose, just pick one that's near you.

###  1. Create the Second Azure Function

Open the **Integrate** view of the first Azure Function that we created and click on the **Azure Storage Queue (outputQueueItem)** that we created. This will display the configuration information for this output. Towards the bottom of the screen is an **Actions** section, with a big **Go** button next to a message that says "Create a new function triggered by this output".

![](/images/2017/01/functions-outputs-actions.png)

As it suggests, this will create a new function that's set up to be trigged by the output from our current function. Click the **Go** button and you'll see the following:

![](/images/2017/01/functions-actions-create-queue-trigger.png)

As our output on the first function was a queue, the second function will consume that output, so its trigger has been set to **queue**. The details of the queue from our first function has also been pre-populated, so we don't need to change anything and can just accept the defaults by clicking the **Create** button.

Several things now happen in quick succession!  First, our second function is created. It's configured with a trigger binding, set up to watch for messages arriving in the queue that our first function uses.

```
{
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    }
  ],
  "disabled": false
}
```

Second, we have some starter code.

```
using System;

public static void Run(string myQueueItem, TraceWriter log)
{
    log.Info($"C# Queue trigger function processed: {myQueueItem}");
}
```

When the function is triggered by a message arriving in the queue, the message is passed into our code via the `myQueueItem` parameter. The starter code simply writes the contents of that message out to the log.

If you followed along with Part 1 of this tutorial, then a third thing will also have happened. Take a look at the logs:

```
2017-01-18T09:22:39.287 Function started (Id=d6d4b3f1-430a-4afc-999e-e8a06d1603b1)
2017-01-18T09:22:39.287 C# Queue trigger function processed: {"id":"44","orgname":"errfree","repos":"https://api.github.com/orgs/errfree/repos"}
2017-01-18T09:22:39.287 Function completed (Success, Id=d6d4b3f1-430a-4afc-999e-e8a06d1603b1)
2017-01-18T09:22:39.318 Function started (Id=4badfaee-6d33-43bc-ae25-65bbd162de6f)
2017-01-18T09:22:39.318 C# Queue trigger function processed: {"id":"81","orgname":"engineyard","repos":"https://api.github.com/orgs/engineyard/repos"}
2017-01-18T09:22:39.318 Function completed (Success, Id=4badfaee-6d33-43bc-ae25-65bbd162de6f)
2017-01-18T09:22:39.318 Function started (Id=9260fc39-6840-4936-ba95-7af3db1f4df3)
2017-01-18T09:22:39.318 C# Queue trigger function processed: {"id":"119","orgname":"ministrycentered","repos":"https://api.github.com/orgs/ministrycentered/repos"}
2017-01-18T09:22:39.318 Function completed (Success, Id=9260fc39-6840-4936-ba95-7af3db1f4df3)
2017-01-18T09:22:39.365 Function started (Id=5c3d4510-208b-41ab-b109-7e536d96278a)
2017-01-18T09:22:39.365 C# Queue trigger function processed: {"id":"128","orgname":"collectiveidea","repos":"https://api.github.com/orgs/collectiveidea/repos"}
2017-01-18T09:22:39.365 Function completed (Success, Id=5c3d4510-208b-41ab-b109-7e536d96278a)
2017-01-18T09:22:39.380 Function started (Id=25f2b51f-a6ef-4fdb-9f27-5877289a159f)
2017-01-18T09:22:39.380 C# Queue trigger function processed: {"id":"144","orgname":"ogc","repos":"https://api.github.com/orgs/ogc/repos"}
2017-01-18T09:22:39.380 Function completed (Success, Id=25f2b51f-a6ef-4fdb-9f27-5877289a159f)
```

When the function was created, it was instantly triggered by the messages we'd place into the queue with our first function.

Now, I know this doesn't have a huge amount of functionality right now, but think about what just happened. We created a function to place messages into a queue. Then, with nothing more than a couple of mouse clicks and without writing any code, we created a second function that consumed those messages. How awesome is that?

Ok, so let's make our function a little more, err, functional.

###  2. Add DocumentDB Output Bindings

As we did with our first function, let's click on the **Integrate** button and then click **+ New Output** to add a new output binding. This time, we will select **Azure DocumentDB Document** from our list of options and then click **Select**.

![](/images/2017/01/functions-documentdb-output.png)

We will be prompted to provide configuration details for DocumentDB. The defaults are mostly fine for our purposes, but there are a couple of changes. 

First, check the box that says **Would you like the DocumentDB database and collection to be created for you?**. 

Second, we need to provide connection information. To the right of the box underneath **DocumentDB account connection** you will see a ***new*** link. Click that link and you will see a list of DocumentDB accounts. Select the one you created for this tutorial and the connection information will be configured for you. Then click the **Save** button to finish the configuration.

![](/images/2017/01/functions-documentdb-output-complete.png)

Click back on **Develop** and let's have a look at our `function.json` file.

```
{
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    },
    {
      "type": "documentDB",
      "name": "outputDocument",
      "databaseName": "outDatabase",
      "collectionName": "MyCollection",
      "createIfNotExists": true,
      "connection": "t-dd-eun-fn170118_DOCUMENTDB",
      "direction": "out"
    }
  ],
  "disabled": false
}
```
Now we have the input trigger and our DocumentDB output configured. In the log file, you may notice the following:

    warning AF004: Missing binding argument named 'outputDocument'. Mismatched binding argument names may lead to function indexing errors.

This is warning us that we haven't provided an `outputDocument` parameter in our function. This parameter will eventually contain the data we want to write to DocumentDB. The `out` keyword is needed so that the parameter is passed by reference. So, let's add that to our method signature.

    public static void Run(string myQueueItem, out object outputDocument, TraceWriter log)

Let's update our `run.csx` file and get it to do something more interesting. As with our first function, we need to handle some JSON formatted documents, so let's add the following to the top of our code.

```
#r "Newtonsoft.Json"

using System;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

The data we wrote to the queue was a JSON document which contained the organisation ID, name and URL for their repo list. That data is passed to our function through the `myQueueItem` parameter as a `string`. So, we'll parse that string and turn it into a JSON object and then make an HTTP call to GitHub using the repos URL in the message from the queue. The following code will achieve that:

```
JObject message = JObject.Parse(myQueueItem);
HttpClient client = new HttpClient();
client.DefaultRequestHeaders.UserAgent.Add(new System.Net.Http.Headers.ProductInfoHeaderValue("Mozilla", "5.0"));
JArray response = JArray.Parse(await client.GetStringAsync(message["repos"].ToString()));

```

And as the HTTP request uses `await` we need to make our method `async`, so let's update that.

    public static async Task Run(string myQueueItem, out object outputDocument, TraceWriter log)

If you hit the **Save** button at this point, you should spot an error message in the log.

    error CS1988: Async methods cannot have ref or out parameters

We have a minor issue here. We need to have an async method because we're using the `HttpClient` and it's designed to work asynchronously. However, this error is saying we cannot mix async methods and ref or out parameters.

Fortunately, there is a simple workaround. Another way to get an Azure Function to write to its output bindings is to configure it to use the function's `return` value. This means that instead of Azure using the value of the `outputDocument` parameter, it will instead use whatever value the function returns when it's finished. So, let's change our method one more time, this time to remove the `outputDocument` parameter and as we'll now be returning a value, we'll need to specify what type of value we'll be returning.

    public static async Task<object> Run(string myQueueItem, TraceWriter log)

The next piece of code will iterate through the returned results, extract the information we're interested in, create a new object with that information and add the objects to a `List`.

```
List<object> repoList = new List<object>();
foreach (JObject repo in response)
{
    object repoInfo = new {
        id = repo["id"].ToString(),
        name = repo["full_name"].ToString(),
        stars = repo["stargazers_count"].ToString()
    };
    repoList.Add(repoInfo);
}
```

Then we create the document that will be written to DocumentDB. This document contains the ID and name of the organisation and then the list of repos that we just built.

```
object document = new {
    id = message["id"].ToString(),
    orgName = message["orgname"].ToString(),
    repos = repoList
};
```
Finally, we convert the object into a JSON document and then terminate the function by returning that JSON document.

```
string json = JsonConvert.SerializeObject(document);
return json;
```

The final code looks like this

```
#r "Newtonsoft.Json"

using System;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public static async Task<object> Run(string myQueueItem, TraceWriter log)
{
    JObject message = JObject.Parse(myQueueItem);
    HttpClient client = new HttpClient();
    client.DefaultRequestHeaders.UserAgent.Add(new System.Net.Http.Headers.ProductInfoHeaderValue("Mozilla", "5.0"));
    JArray response = JArray.Parse(await client.GetStringAsync(message["repos"].ToString()));
    
    List<object> repoList = new List<object>();
    foreach (JObject repo in response)
    {
        object repoInfo = new {
            id = repo["id"].ToString(),
            name = repo["full_name"].ToString(),
            stars = repo["stargazers_count"].ToString()
        };
        repoList.Add(repoInfo);
    }

    object document = new {
        id = message["id"].ToString(),
        orgName = message["orgname"].ToString(),
        repos = repoList
    };

    string json = JsonConvert.SerializeObject(document);
    return json;
}
```

Click the **Save** button to save the code. At this point, we're nearly ready to run, but we need to make one more change. Open the `function.json` file and modify the DocumentDB output binding so that the `name` is `$return`.

```
{
  "type": "documentDB",
  "name": "$return",
  "databaseName": "outDatabase",
  "collectionName": "MyCollection",
  "createIfNotExists": true,
  "connection": "t-dd-eun-fn170117_DOCUMENTDB",
  "direction": "out"
}
```

Now, we're ready to run. First, we need some data, so go to the first function that we created in Part 1 and click the **Run** button. Once it's completed, go back to our newly created second function. As it's triggered by new messages arriving in the queue, it should have run already. To confirm, you should see a bunch of messages in the log similar to this:

```
2017-01-18T09:43:59.200 Function started (Id=4ad152b4-d1bc-43e5-a733-d9ac82a31dd3)
2017-01-18T09:44:00.611 Function completed (Success, Id=4ad152b4-d1bc-43e5-a733-d9ac82a31dd3)
```

And now to verify that everything worked. Let's go to our DocumentDB instance in the Azure Portal. From the **Overview** tab, first you should see that the database and collection we specified have been automatically created.

![](/images/2017/01/functions-docdb-collection.png)

Go to the **Document Explorer** tab and you should see a list of 5 ID's.

![](/images/2017/01/functions-docdb-documents-list.png)

Click on one of the ID's and you should see something like this.

![](/images/2017/01/functions-docdb-document.png)

The DocumentDB output provided by Azure Functions will create a new document in DocumentDB with a new document ID. If you provide an ID value as part of your document, then that value will be used, as we have done here. If a document already exists with that ID, the document will be overwritten.

Again, this was all relatively simple to put together, especially the input from the Azure Storage queue and the output to DocumentDB, neither of which required us to write any code to retrieve the messages or write to the database.

Azure Functions is very simple to use and quick to get up and running. At the time of writing, Azure Functions are still relatively new, so I can't wait to see how they develop.
