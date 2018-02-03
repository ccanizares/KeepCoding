---
title: Dockerize VSTS Agents
updated: 2018-02-03 06:10
layout: post
category: blog
author: canizarescarlos
description: How to dockerize VSTS Agents
language: en
---

In this post I'm gonna explain how to set up a custom agent and take profit of that machine using docker images and containers for <strong>running more than one agent instance in that machine</strong>. The information I'm showing in this post could be useful for your time if you use VSTS, you define CD/CI tasks, automatic testing, etc... as the team and project is growing you can run into a bottleneck scenario where pipe tasks expend more time than desired queued waiting for a machine to run.

<img src='../assets/images/dockerize-vsts-agent-waiting.jpeg' />

First you need to know is how agents work in VSTS. 

#### Build pipes 

When you define a build pipe you need to select an Agent Queue where the task related to the build will be executed.

<img src='../assets/images/dockerize-vsts-agents-build.png' />

#### Agents Queues 

Agents Queues are a collection of machines within the same capabilities. If you press "Manage" button you see in the picture above you will be redirected to the Agent Queues Management Page where you can see default queues, check capabilities and create your own queues and bind new machines to queues. VSTS provides oob different Queues with different capabilities. You can use it for free (free quota per month) and this queue is shared with other vsts users.

<img src='../assets/images/dockerize-vsts-agents-queues.png' />

#### Custom Agents 

You can bind any machine, you'll need to install capabilities and run a vsts service to act as an Agent. In the section shown in the picture above you're able to create new Queues. If you press Download Agent you will get the instructions to install capabilities and how to run the vsts service to act as agent for different platforms. You will find a lot of official tutorials explaining this scenario, I'm gonna skip this part, the interesting topic I want to show is described in next section. 

### Dockerize Agents

[Microsoft](https://github.com/Microsoft/vsts-agent-docker) has a github repo where you can find prepared docker files within standard capabilities. If you have docker installed in your machine it's easy to test... you only need to generate a PAT and run the container in docker replacing `<name>,<agentname>,<pat>` with your values (PAT you can generate under security section in your VSTS profile).

{% highlight js %}
docker run \
  -e VSTS_AGENT=<agentname>
  -e VSTS_ACCOUNT=<name> \
  -e VSTS_TOKEN=<pat> \
  -it microsoft/vsts-agent
{% endhighlight %}

This will add an agent into the default queue... you can specify a different queue using VSTS_POOL env var. 

#### Customize docker file

You can add capabilities to this image or create a new one from this base. For example, I'm gonna extend the microsoft agent for ubuntu adding kubernetes client, helm and acs engine as capabilities.

{% highlight js %}
FROM microsoft/vsts-agent:ubuntu-16.04

# Configure AzCLI 2.0  Instructions taken from https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
RUN echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ wheezy main" | tee /etc/apt/sources.list.d/azure-cli.list \
 && apt-key adv --keyserver packages.microsoft.com --recv-keys 417A0893 \
 && apt-get install -y --no-install-recommends \
    apt-transport-https \
 && apt-get update \
 && apt-get install -y --no-install-recommends \
    azure-cli \
 && rm -rf /var/lib/apt/lists/* \
 && az --version

# Install kubernetes
RUN curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl \
  && chmod +x ./kubectl \
  && sudo mv ./kubectl /usr/local/bin/kubectl

# Install helm
RUN wget https://storage.googleapis.com/kubernetes-helm/helm-v2.5.1-linux-amd64.tar.gz \
  && tar -xzf helm-v2.5.1-linux-amd64.tar.gz \
  && chmod +x ./linux-amd64/helm \
  && sudo mv ./linux-amd64/helm  /usr/local/bin/helm \
  && rm -rf linux-amd64

# Acs client
RUN wget https://github.com/Azure/acs-engine/releases/download/v0.10.0/acs-engine-v0.10.0-linux-amd64.tar.gz \
  && tar -xzf acs-engine-v0.10.0-linux-amd64.tar.gz \
  && chmod +x ./acs-engine-v0.10.0-linux-amd64/acs-engine \
  && sudo mv ./acs-engine-v0.10.0-linux-amd64/acs-engine  /usr/local/bin/acs-engine \
  && rm -rf acs-engine-v0.10.0-linux-amd64 \
  && rm acs-engine-v0.10.0-linux-amd64.tar.gz 
{% endhighlight %}

#### Build and run new image

Open a terminal in the same folder where you saved this file and run this command to build the new image replacing `<yournewimagename>`. 

{% highlight js %}
docker build -t <yournewimagename> .
{% endhighlight %}

Once image is created and stored in your local images repository you can run the agent replacing the gaps with your values.

{% highlight js %}
docker run -d --rm -e VSTS_AGENT=<youragentname> -e VSTS_ACCOUNT=<yourvstsaccount> -e VSTS_TOKEN=<pat> -e VSTS_POOL=<queue> -it <yourimagename>
{% endhighlight %}

After few minutes you should see in VSTS Agents Queues management section this agent appearing in the list ready to queue new tasks. If you run the same command changing `<youragentname>` you'll see another agent in the list ready... 

### Conclusion

Using docker you can take more profit of a machine and paralelice tasks avoiding tasks waiting to be executed for long periods... <strong>You save money and time</strong>... The only thing you have to care is to <strong>don't saturate the machine</strong>, depending on what are you running on the agent you can run more instances per machine or not.







