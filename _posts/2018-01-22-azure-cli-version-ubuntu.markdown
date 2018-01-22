---
title: azure-cli download specific version on ubuntu
updated: 2018-01-22 06:10
layout: post
category: blog
author: canizarescarlos
description: How to download and install specific version of azure-cli on ubuntu
language: en
---

In our build scripts we use azure-cli often to be easier to talk with some Azure Services via Azure api. We noticed that some scripts that were working started to fail, to be more specific one script related with Azure Container Registry was failing to free space of old tags. 

When I addressed Microsoft support this problem first they told me is to update my az --version. Makes sense. I downloaded latest version from the Microsoft official download page (currently 2.0.24) and crash [it's currently bugged](https://github.com/Azure/azure-cli/issues/5303).

Next that Microsoft recommended me was to use Cloud Exporer to check if same commands success from that machines, and yes... was a version problem because I was able to use az acr without problems. I noticed that the version installed in the remote machine I used from cloud was version 2.0.23 so my conclusion is that if I'm able to install version 2.0.23 in my Linux machines I will solve the problem within my scripts. 

<img src='../assets/images/azure-cli-cloud-explorer.png' />

#### Get a specific package version

All I want to remember writing this post is how to download a specific version of azure-cli because was not very intuitive.. Finally I found how to achieve it, may be there is a easier way of do it but I did not found it. 

Here you'll find all the versions packages:
[Versions](https://packages.microsoft.com/repos/azure-cli/pool/main/a/azure-cli/)
{% highlight js %}
 wget https://packages.microsoft.com/repos/azure-cli/pool/main/a/azure-cli/azure-cli_2.0.23-1_all.deb 
{% endhighlight %}

To install a deb package in ubuntu run this command: 
{% highlight js %}
 sudo dpkg -i azure-cli_2.0.23-1_all.deb 
{% endhighlight %}
