---
title: Azure DNS - CNAME redirections
updated: 2018-01-11 06:10
layout: post
category: blog
author: canizarescarlos
description: Some experiences redirecting load from applications using Azure DNS
image: https://ccanizares.github.io/KeepCoding/assets/images/Azure-DNS-620x264.png
headerImage: true
---

In this post I'm sharing (problably only with myself and some colleague) some experiences with CNAME redirections in Azure DNS Service.

## CNAME redirections between k8s clusters.

First scenario where CNAME is useful for us is when doing a/b switches between two k8s clusters. Each has an A record pointing to the cluster load balancer.  Example: 

{% highlight js %}
a-someservice.ourdomain.com A record ->  a PUBLIC IP
b-someservice.ourdomain.com A record ->  b PUBLIC IP
someservice.ourdomain.com CNAME record ->  [a||b].someservice.ourdomain.com
{% endhighlight %}

To achieve this scenario we have a multidomain SSL certificate and we own the domain (of course), ingress service in each k8s cluster is configured within this ssl to respond a.someservice.ourdomain.. or b.someservice.ourdomain.. (depending on the ingress) and to respond someservice.ourdomain.. (always). This is an example of the mentioned part of the ingress configuration:

{% highlight js %}
  tls:
    - hosts:
      -  a.someservice.ourdomain.com
      -  someservice.ourdomain.com
      secretName: oursecret
  rules:
    - host: a.someservice.ourdomain.com
      http:
        paths:
        - path:
          backend:
            serviceName: someservice
            servicePort: 80
    - host: someservice.ourdomain.com
      http:
        paths:
        - path:
          backend:
            serviceName: someservice
            servicePort: 80
{% endhighlight %}

So with this aproach when we want to switch production load what we do is to change third entry to match blue or green and this does the trick. It's important to configure TTL in the DNS record shorter as posible. 

## CNAME redirections between K8s and Azure App Service Web Site. 

Next scenario is useful for us, when we do a switch between our cluster we need to restore all data from green and restore it in B for each data service we are running in the cluster. We have a script that automates all this process of moving data and changing the DNS entry but while the script is working we should ensure nobody is writing data for avoiding losing any. So should be a little down time to ensure the switch is done without problem. 

We want to show a page informing the user the service is under maintaniance operation some minutes, and we want this web site outside our kubernetes clusters because if some day k8s goes down we want to activate this CNAME redirection and show the page to inform the users the service is down. 

To achieve this you need to create an App Service and configure custom domain and SSL (supported in Basic B1 and higher plans, not in free tier). Not gonna detail all the steps there are plenty of tutorials better than this, just comment two parts that were a little bit more tricky: 

#### Setting up custom domain 

Addressed to Linux guys this step of setting up custom domains is the equivalent for configuring ingress. The only confusing part I've found is that you need to demonstrate to the App Service that you own the domain, this is done adding an entry in your DNS. 

<img src='../assets/images/azure-dns-cname-redirection.png' />

Here you need to go to the DNS and add this kind of record.

<img src='../assets/images/azure-dns-entry-cname-redirection.png' />

Once done you click on validate again and you should pass the validation. 

<img src='../assets/images/azure-dns-cname-redirection-ok.png' />

#### Setting up ssl binding

If your original site runs over https you need to configure SSL binding is this site aswell otherwise the CNAME redirection will show ssl restrictions in browser and problably won't work. 

About how to bind an ssl certificate in App Service there is no mistery pretty intuitive this part, only share this link where you can find info about how to convert a .cer/.key to .pfx file you need to upload it. 

[PFX](https://www.sherweb.com/blog/when-given-crt-and-key-files-make-a-pfx-file/)
