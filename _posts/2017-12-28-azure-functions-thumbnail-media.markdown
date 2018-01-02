---
title: Azure Functions - Thumbnail generator 
updated: 2017-12-28 06:10
layout: post
category: blog
author: canizarescarlos
description: How to create an Azure Function that is able to generate a thumbnail when a image is uploaded into a storage account.
image: https://ccanizares.github.io/KeepCoding/assets/images/MSAzure-functions.png
headerImage: true
---

In this post I'm gonna show how to create an azure function that have the mission of generate a thumbnail image for each picture that is being uploaded to a blob storage and save it in another container within the same storage account.

## Functions Overview

Azure Functions are the Microsoft Azure serverless proposal at this time, if you're an AWS guy this would be the Azure aproach for Lambda Functions.

As you may know Serverless aproach consist on having server side code that only run on demands no matter the server in wich this code is hosted.

### Hosting

In azure you can host and run your functions in a Windows or a Linux (preview) machine. You can bind the function to a Service Plan (same as you do within an AppService), the alternative is to create a consumption plan. [More info](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale)

### Code

By default Azure Functions let you code in C#, javascript, python, bash, powershell, and some more.. but you're running in a Windows or Linux machine you can use whatever.

### Triggers

The function run on demands, this means you can trigger this code 'manually' or let Azure trigger it when something happens in another resource that you own. For example is very easy to activate the function when an Event Hub receives a message in a topic or trigger it when somebody uploads blob to an storage, or when a document is stored in cosmosDb, etc. My personal opinion is that the integration with others resources in Azure is pretty cool. [More info](https://docs.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings)

### Input/Ouput

Functions are able to receive an input parameter and to throw another param as result. In our case we will receive as an input parameter the blob that has triggered my function and I will set as an output the thumbnail for that blob. 

## Thumbnail Generator

Let's go with my example, I know... It's not a very original example there is a lot of demos doing more expectacular things but it's useful for our team needs because we decieded not to use AMS for the moment and keep media files directly in a blob... so his post is a kind of proposal for my team mates...

To starters I recommend using the assistant and the online editor and once you know how this works, you can download code put in on your repo and work on local IDE as normal persons. 

Let's go to Azure and create a new Azure Function, we will use a template because it fits perfect to my needs.

<img src='../assets/images/azure-function-thumbnail-media-trigger.png' />

In the next step the assistant ask you for the connection string, I use WEBSITE_CONTENTAZUREFILECONNECTIONSTRING that is an ENV var that you can set as in any App Service (Manage Application Settings) to point to the storage Account where my media files are uploaded. Press create and you will see the function ready to code in the left panel. 

Before coding, we will set up the input and ouputs using the <b><i>Integrate</i></b> menu option that we have in the Azure Editor.

Once we are in, no mistery the trigger configuration is already set in the previous step and here you can define the input and output settings for your function. At the end this is sugar for hidding a json file where all this settings are stored, you can view this json if you click in <b><i>Advanced Editor</i></b>.

{% highlight js %}
{
  "bindings": [
    {
      "name": "myBlob",
      "type": "blobTrigger",
      "direction": "in",
      "path": "media/{name}",
      "connection": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING"
    },
    {
      "type": "blob",
      "name": "inputBlob",
      "path": "media/{name}",
      "connection": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
      "direction": "in"
    },
    {
      "type": "blob",
      "name": "outputBlob",
      "path": "media-thumbs/{rand-guid}",
      "connection": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
      "direction": "out"
    }
  ],
  "disabled": false
}
{% endhighlight %}

Now we are ready to code this thumbnail utility... we are using node.js so searching for libraries [this](https://www.npmjs.com/package/jimp) seem to fit. We will need to add a package.json (by default it's not provided) to our function so we can add this lib. For this purpose you can use App Settings editor or Kudu or directly open a console... Take a look to the picture below. Once we are in, we define an empty package.json and then as we would do in a node app we add the package using the npm command. 

<img src='../assets/images/azure-function-thumbnail-media-app-service-editor.png' />

Once this step is finished (you have to be very very patience my friend) we will notice that node_modules folder is added to our 'solution', so know we can use jimp in our code. Let's go.

In the online editor (index.js) I wrote this code:

{% highlight js %}
var Jimp = require("jimp");

module.exports = (context, myBlob) => {
    Jimp.read(myBlob).then((image) => {
        image
            .resize(200, Jimp.AUTO) 
            .greyscale()
            .getBuffer(Jimp.MIME_JPEG, (error, stream) => {
                if (error) {
                    context.log(error);
                    context.done(error);
                }
                else {
                    context.bindings.outputBlob = stream;
                }
            });
    });
};
{% endhighlight %}

When all this steps are finished we can test our function within the editor in Azure Portal, and see that every time I upload a file to the storage account that has been setted as input the function triggers and executes the code. If everything is working fine you should see something like this when testing your function. 

So now the function is ready to work, in production? I don't think so, this is a proof of concept I would never use this in production if I'm not able to manage this code in my IDE with the rest of the project and include this function or functions in our CI/CD pipelines. In the next post I will be covering with some proposal how to work with Azure Functions in a repository and how to configure CD/CI. 

