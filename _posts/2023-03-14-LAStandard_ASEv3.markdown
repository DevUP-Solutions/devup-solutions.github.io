---
layout: post
title: "Logic App Standard on ASE v3"
description: Finally Logic App's can be run inside VNet.  | This post gives an overview of how a setup could look like and how it works.
date: 2023-03-14 14:00:00 +0200
categories: LogicApps Security
tags: [Logic App Standard, Integration, Security ]
author: "Mattias Lögdberg"
comments: true
---

Logic App Standard is the first version of Logic Apps that can run outside of the dedicated space in Azure. In this post we will show how we helped a customer use Logic App Standard without going outside their VNet boundaries, full lockdown.

### Background

Logic App Standard is built on the Azure Functions runtime which means that the runtime has the same functionality and therefor also gets the benefits of running on an App Service Environment (ASE). This makes it possible to deploy your Logic App in a totally isolated environment and that is what we are going to look into in this post.

### How does it work?
To make this setup work, we need to configuration a couple of things for traffic to flow.

* VNet with atleast 2 Subnets
* Each Subnet should have a Network Security Group (NSG)
    * To protect from unwanted traffic in and out of the subnet.
* App Service Environment, deployed in a dedicated subnet
* Logic App Standard, deployed on the App Service Environment
* Storage Account, deployed with private endpoint in to the other subnet.

The 2 subnets is divided as follow, *Subnet A* dedicated for the *App Service Environment* due to the need for it's own subnet. We then create a second subnet *Subnet B* so we can use it for adding private links or other services. It will look like this:

![Network overview Overview](/assets/images/2023/march/subnetdrawing.jpg)

Now we can add the storage account and the private links. To lock down our storage account we need four private endpoints, one for each service (Blob, Files, Table, Queues). Private links require a private DNS record to be created inside the VNet. This is so that resources can find them. We can just use the built in suggestion and put the standard generated private DNS zone in a Production subscription (since we used hub-and-spoke network setup) attached to the VNet. Now DNS lookup will be correct inside the network.


![Network Overview](/assets/images/2023/march/privateendpoints.jpg)

We also need to make sure that traffic can flow, all these endpoints are accessed over HTTPS so rules need to be added in the NSG's.
In the *NSG* on *Subnet A* we need to allow outboud HTTPS(443) and SMB(445) traffic, the rules below allows connecting to *Subnets* from *Subnet A*.(You can ofcourse narrow this down by changing *destination* property)

![Outbound Rule](/assets/images/2023/march/outbundhttpsnsgrule.png)

The same goes for the NSG that is attached to *Subnet B* but inbound rule to allow HTTPS(443) and SMB(445) traffic. The rules below allow all *Subnets* to sen requests to *Subnet B* over HTTPS. (You can ofcourse narrow this down by changing *source* property)

![Inbound Rule](/assets/images/2023/march/inboundhttpsnsgrule.png)

We can now deploy our *Logic App Standard* instance to our *App Service Environment*, this is done via picking the *App Service Environment* as the **region** when creating the *Logic App Standard* instance. It will then connect to the *Storage Account* as the image describes and the traffic will be allowed due to our rules in the NSG's.

![Full Overview](/assets/images/2023/march/asedrawing.jpg)

Now we can start to add workflows as normal, we will be restricted to use *Built-in* connectors or open up access to internet from Subnet A, there is a TAG to only allow traffic to Managed Connectors and not open for HTTPS to the whole internet, use that if needed to keep openings to a minimal.


> **Note:** that you need to have direct access to the *App Service Environment* in order to access logs in the Workflow runs, this is a security feature and by design. If you are not able to reach the *App Service Environment* there will be an error on the actions "Unexpected error. Failed to fetch", make sure to be on the network in order to get the data and also make sure routing/dns etc is working from your location, [read more here](https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/common-errors-in-azure-logic-apps-standard-unexpected-error/ba-p/3293197). 

### Improvements
When using full VNet protection we cannot use *Managed Connectors* and can only use *Built-in connectors* ([see the full list](https://learn.microsoft.com/en-us/azure/connectors/built-in)) or build our own [read more](https://learn.microsoft.com/en-us/azure/logic-apps/create-custom-built-in-connector-standard). In our case we had alot of *Oracle databases* and the only way is to use a generic *JDBC* connector or build our own.

As in many cases error messages are not very helpful and it took a long time to figure out that all 4 Private Endpoints where needed on the storage account before we could set *Public Network access* to *Disabled*.

### Why use it?
Full VNet protection is required in many companies and finally we can use Logic App Standard in these scenarios and utilize the power of Logic Apps orchestration engine in closed environments.

### Summary
When everything is up and running. this is a nice solution. We can manage our logic apps from Azure but make sure to have routing up if you want to see the input and outputs of the actions when troubleshooting. Being able to use Logic App's inside our network and in a full network lockdown proves that Microsoft is really pushing Logic Apps.

During these and many other cases it is important to understand that security now is co-managed by developers and infrastructure teams and managed via infrastructure as code. It's a long journey and it's important to take ownership of security in order to be successful over time. We also love to help you out here with **Helium**, our continuous review tool. To make sure that you are building robust and secure solutions that will continue to be secure and robust over time, all so you can sleep well at night knowing things are as expected.


[App Service Environment read more](https://learn.microsoft.com/en-us/azure/app-service/environment/overview)
[Deploy single-tenant Standard logic apps to private storage accounts using private endpoints](https://learn.microsoft.com/en-us/azure/logic-apps/deploy-single-tenant-logic-apps-private-storage-account)
[Common errors in Azure Logic Apps (Standard) - Unexpected error. Failed to fetch](https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/common-errors-in-azure-logic-apps-standard-unexpected-error/ba-p/3293197)
