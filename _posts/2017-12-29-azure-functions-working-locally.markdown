---
title: Azure Functions - Working Locally
updated: 2017-12-28 06:10
layout: post
category: blog
author: canizarescarlos
description: How to work locally with Azure Functions
---

In the [previous post](https://ccanizares.github.io/KeepCoding/azure-functions-thumbnail-media/) I wrote an introduction to Azure Functions. That sample was created directly within the Azure Portal. That function is triggered when somebody push a picture to a blob storage and is responsible of generating thumbnails. As I told at the end of the previous post, creating, coding and testing the function directly in the Azure Portal is not an ideal solution ...

## Tooling

I'm currently working on mac so my first challenge was to see if it's possible to work locally in a non Windows environment and... Microsoft did it, I'm able to create, code and test locally within my mac. Kudo's for Microsoft. (Preview)

In [this page](https://docs.microsoft.com/es-es/azure/azure-functions/functions-run-local) you will find all you need... In my case I'm using v2 because I need to run it on a mac. I'm gonna skip in this post the explanation about how to create the function within this tool because I won't do it better than this doc, it's pretty clear. 

## Running previous sample in local

I've already a function created in the latest post, so in my case I will avoid creating it locally with this tool... instead I will download the one that I already have in Azure and I'll execute in local to try this tool. 

First step is to open the function in the portal and press the 'Download app content' button. Once you have downloaded the app, unzip it and open it in your favourite IDE.

You'll find a local.settings.json file. In this file you place the configuration for example the connection string to the storage account that triggers the function. 

{% highlight js %}
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_EXTENSION_VERSION": "~1",
    "ScmType": "None",
    "WEBSITE_AUTH_ENABLED": "False",
    "AzureWebJobsDashboard": "DefaultEndpointsProtocol=https;AccountName=asfsdfdsafsdfad5;AccountKey=eJFabMS0ENBx4e7TYc+808rvJkfadfadfasdfasdfasdfaasdasfY5g728i9ldYvckLGjwJGUw/Eo1cJ/PF/5izIfbyoDGdjiNdrpIa++kTU93Lg==",
    "WEBSITE_NODE_DEFAULT_VERSION": "6.5.0",
    "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING": "DefaultEndpointsProtocol=https;AccountName=blablabla;AccountKey=blabla/VYQnCKn2bFBsihvkAnyQzKHJPDTDg==;EndpointSuffix=core.windows.net",
    "WEBSITE_CONTENTSHARE": "balbalba",
    "WEBSITE_SITE_NAME": "balbalbalbalba",
    "WEBSITE_SLOT_NAME": "Production",
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=babababa;AccountKey=eJFabMS0ENBx4e7TYc+808rvJkY5g728i9ldYvckLGjwJGUw/Eo1cJbalbalblablalbalblalabla/PF/5izIfbyoDGdjiNdrpIa++kTU93Lg=="
  }
}
{% endhighlight %}

To start the host and test locally your function:
{% highlight js %}
func host start
{% endhighlight %}
<img src='../assets/images/azure-functions-working-locally.png' />

If I push a new picture to the configured blob container that triggers this function I wil see in my terminal how the function reacts and do his stuff. Let's see:

{% highlight js %}
[12/29/17 6:40:53 AM] <Buffer ff d8 ff e0 00 10 4a 46 49 46 00 01 01 00 00 01 00 01 00 00 ff db 00 84 00 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 ... >
[12/29/17 6:40:53 AM] Successfully processed the image
[12/29/17 6:40:54 AM] Function completed (Success, Id=df79dcf0-040f-478c-82de-2fc395f1d742, Duration=2862ms)
[12/29/17 6:40:54 AM] Executed 'Functions.MediaFiles' (Succeeded, Id=df79dcf0-040f-478c-82de-2fc395f1d742)
{% endhighlight %}

Now connect to the storage output container where I have configured to store the results and I see my Nicolas Cage picture thumbnail. 

<img src='../assets/images/azure-functions-locally-cage.png' />

### Recopilation

In the [previous post](https://ccanizares.github.io/KeepCoding/azure-functions-thumbnail-media/) I wrote directly a function in Azure and introduce the basics, in this one I have shown how to work local within Azure Functions tooling for mac, linux or windows... this allows us to put our code in a repo and work properly. In the next post I will show how to deploy this functions from your machine or from a tool like VSTS.