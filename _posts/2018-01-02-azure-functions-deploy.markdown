---
title: Azure Functions - Deploy to Azure
updated: 2018-01-02 06:10
layout: post
category: blog
author: canizarescarlos
description: How to work deploy local functions to Azure
---

This post will end a serie of three posts talking about Azure Functions, the fist post was about basics I created a function directly using the Azure Portal, in the second we saw how to start locally and code functions in your environment and finally in this post I will show how to deploy your local code to Azure.

## Scenario

My function is able to generate thumbnails each time a push a picture in a blob container. This is a need we have in one of our projects and we wanted to achieve it using functions to evaluate new things but to cover our needs I need to complicate a little bit this sample. 

In our case we have a multitenant application that each tenant have a different storage account configured. I consider this code the same for all tenants, I don't want to repeate this code in N functions.

First of all, the Azure template provide some settings for you that you can use and are properly preconfigured for example I'm refering to ENV var WEBSITE_CONTENTAZUREFILECONNECTIONSTRING that contains the connection string to our storage account. In my case I will need to create one setting like this one for each storage account corresponding to each tenant. 

I will add this to local.settings.json:
{% highlight js %}
  "BARCELONA": "CONNECTIONSTRING_TO_STORAGE_ACCOUNT",
  "FREIBURG": "CONNECTIONSTRING_TO_STORAGE_ACCOUNT",
  "INTEGRATION": "CONNECTIONSTRING_TO_STORAGE_ACCOUNT",
  "TEST": "CONNECTIONSTRING_TO_STORAGE_ACCOUNT",
  "TIMISOARA": "CONNECTIONSTRING_TO_STORAGE_ACCOUNT",
{% endhighlight %}

Next, I would like to share same code but different config for each function. To achieve this I will put my existing code in a node module and create one function per tenant. 

The final structure of my folders is the following:
<img src='../assets/images/azure-functions-deploy-structure.png' />

Shared package.json
{% highlight js %}
{
    "name": "onboarding-media-processor",
    "version": "1.0.0",
    "description": "My shared function",
    "main": "index.js",
    "author": "Carlos Cañizares",
    "license": "MIT",
    "dependencies": {
        "jimp": "0.2.28"
    },
    "scripts": {
        "publish": "node publish.js"
    },
    "devDependencies": {
        "request": "^2.83.0",
        "zip-folder": "^1.0.0"
    }
}
{% endhighlight %}

Tenant package.json
{% highlight js %}
{
    "name": "onboarding-media-processor-integration",
    "private": true,
    "dependencies": {
        "onboarding-media-processor": "file:../shared"
    }
}
{% endhighlight %}

When I run the host in local I see how blobs in my integration and test storages start to be processed and my thumbnails are in the respective folder... so sharing code between local packages in node seem to work properly.

## Deploy

My next goal is to deploy all this changes I have done to my public Azure Function I created in my first post of this series. It's important to keep in my mind that Azure Functions are hosted in an app service wich means that is just a web application hosted in a web server. So I will use the same script I use always for publishing node web sites to App Services in Azure. This script was taken from a bot template in a github repo, for sure you can deploy using differents aproaches... 

shared/publish.js
{% highlight js %}
var zipFolder = require('zip-folder');
var path = require('path');
var fs = require('fs');
var request = require('request');

var rootFolder = path.resolve('.');
var zipPath = path.resolve(rootFolder, '../MediaProcessor.zip');
var kuduApi = 'https://your-site.scm.azurewebsites.net/api/zip/site/wwwroot';
var userName = '$your-user';
var password = 'password';

function uploadZip(callback) {
  fs.createReadStream(zipPath).pipe(request.put(kuduApi, {
    auth: {
      username: userName,
      password: password,
      sendImmediately: true
    },
    headers: {
      "Content-Type": "applicaton/zip"
    }
  }))
  .on('response', function(resp){
    if (resp.statusCode >= 200 && resp.statusCode < 300) {
      fs.unlink(zipPath);
      callback(null);
    } else if (resp.statusCode >= 400) {
      callback(resp);
    }
  })
  .on('error', function(err) {
    callback(err)
  });
}

function publish(callback) {
  zipFolder(rootFolder, zipPath, function(err) {
    if (!err) {
      uploadZip(callback);
    } else {
      callback(err);
    }
  })
}

publish(function(err) {
  if (!err) {
    console.log('Site published successfully');
  } else {
    console.error('Site publish failed', err);
  }
});
{% endhighlight %}

You'll need to set your username and password, to get this information you can go to the Azure Portal and in the root screen of the Azure Function press the button 'Download Publish Profile'. In the file you download you will find the user name and password, once set just go to the shared folder root and type:
{% highlight js %}
node publish.js
{% endhighlight %}

Once this publish script does his work your local changes will be deployed to the server and your functions will be ready to be triggered. Remember create the tenant connection string settings that we added into local.setting.json file in the production host (using the administration portal of the azure function).


