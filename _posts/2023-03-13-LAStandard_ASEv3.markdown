---
layout: post
title: "Logic App Standard on ASE v3"
description: Finally Logic App's can be run inside VNet.  | This post explains an overview of how a setup could look like and how it works.
date: 2023-03-13 14:00:00 +0200
categories: LogicApps Security
tags: [Logic App Standard, Integration, Security ]
author: "Mattias Lögdberg"
comments: true
---

Logic App Standard is the first version of Logic Apps that can be run outside of the dedicated space in Azure. In this post we will show how how we helped a customer use Logic App Standard without going outside theiere VNet boundaries.

### Background

Logic App Standard is built on the Azure Functions runtime wich means that the runtime has the same requirement as Functions and there for also get's the benefits of running on a App Service Environment (ASE) this makes it possible to deploy your Logic App in a totaly isolated enviornment and that is what we are going to look in to in this post.

### How does it work
To set the full enviornment up we need to have a few things in place and do some configuration in order for traffic to flow

* VNet with atelast 2 Subnets
* Each Subnet should have a Network Security Group (NSG)
    * To protect from unwanted traffic in and out of the subnet.
* App Service Environment, deployed in a dedicated subnet
* Logic App Standard, deployed on the App Service Environment
* Storage Account, deployed with private endpoint in to another subnet.

The 2 subnets is divided as follow, *Subnet A* dedicated for the *App Service Environment* due to the need for it's own subnet. We then create a second subnet *Subnet B* so we can use it for adding private links, it will look as follows.

![Network overview Overview](/assets/images/2023/march/subnetdrawing.jpg)

Now we can add the storage account and the private links and we need all four (4) of them Blob, Files, Tables and Queues if we want to be able to lock down our *Storage Account* from beeing accessible outside our VNet's. Private links require a private dns record to be created inside the VNet so that resoruces can find them. We just used the built in suggestion putting the standard generated private dns zone in a Production subscription (since we used hub'n spoke network setup) attached to the VNet so that DNS lookup will be correct inside the network.


![Network Overview](/assets/images/2023/march/privateendpoints.jpg)

We also need to make sure that traffic can flow, all these endpoints are accessed over HTTPS so rules need to be added in the NSG's.
In the *NSG* on *Subnet A* we need to allow outboud HTTPS(443) and SMB(445) traffic, the rules bellow allows connecting to *Subnets* from *Subnet A*.(You can ofcourse narrow this down by changing *destination* property)

![Outbound Rule](/assets/images/2023/march/outbundhttpsnsgrule.png)

Same goes on the NSG that is attached to *Subnet B* but inbound rule to allow HTTPS(443) and SMB(445) traffic. The rules bellow allows all *Subnets* to sen requests to *Subnet B* over HTTPS. (You can ofcourse narrow this down by changing *source* property)

![Inbound Rule](/assets/images/2023/march/inboundhttpsnsgrule.png)

Now we can now deploy our *Logic App Standard* instance to our *App Service Environment* 
It will then connect to the *Storage Account* as the image describes and the traffic will be allowed due to our rules in the NSG's.

![Full Overview](/assets/images/2023/march/inboundhttpsnsgrule.png)

Implemented it's just to start adding workflows as normal, you will be restricted to use *Built-in* connectors or open up access to internet from *Subnet A*, there is a TAG to use for only allow traffic to Managed Connectors and not open for HTTPS to the whole internet, use that if needed.

Make note that you need to have direct access to the *App Service Environment* i order to access logs in the Workflow runs, this is a security feature and by design. If you are not able to reach the *App Service Environment* there will be an erroron the actions "Unexpected error. Failed to fetch", make sure to be on the network in order to get the data and also make sure routing/dns etc is working from your lokation, [read more here](https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/common-errors-in-azure-logic-apps-standard-unexpected-error/ba-p/3293197). 

### Improvements
When using full VNet protection we cannot use *Managed Connectors* and can only use *Built-in connectors* ([see the full list](https://learn.microsoft.com/en-us/azure/connectors/built-in)) or build our own [read more](https://learn.microsoft.com/en-us/azure/logic-apps/create-custom-built-in-connector-standard). In our case we had alot of *Oracle databases* and the only way is to use a generic *JDBC* connector or build our own.

As in many cases error messages are porly helping and it took a long time to figure out that all 4 Private Endpoints where needed on the storage account before we could set *Public Network access* to *Disabled*. 

### Why use it
Full VNet protection is requried in companies and finally we can use Logic App Standard in these scenarios and utilize the power of Logic Apps orchestration engine in closed environments. 

### Summary
When everything is up working this is a nice solution, management works from Azure but make sure to have routing up so you can see the input and outputs of the actions when troubelshooting.
Beeing able to use Logic App's inside our network and in a full network lockdown proves that Microsoft is really pushing Logic Apps.

During these and many other cases it's important to understand that security now is co-managed by developers and infrastructure teams and managed via infrastructure as code, it's a long journey and it's important to take ownership of security in order to be successfull over time..
We also love to help you out here with **Helium** our continious review tool to make sure that you are building robust and secure soltuions that will continue to be secure and robust over time, all so you can sleep well at night knowing things are as expected.


[App Service Environment read more](https://learn.microsoft.com/en-us/azure/app-service/environment/overview)
[Deploy single-tenant Standard logic apps to private storage accounts using private endpoints](https://learn.microsoft.com/en-us/azure/logic-apps/deploy-single-tenant-logic-apps-private-storage-account)
[Common errors in Azure Logic Apps (Standard) - Unexpected error. Failed to fetch](https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/common-errors-in-azure-logic-apps-standard-unexpected-error/ba-p/3293197)