---
layout: post
title: "Service Endpoint vs Private Endpoint, network options for PaaS"
description: Let's talk about the network options we have in Azure for PaaS Services. | This post gives an overview of the different choices and the pros and cons of each choice.
date: 2033-09-11 09:00:00 +0200
categories: Network Security
tags: [Azure Network, Integration, Security ]
author: "Mattias Lögdberg"
comments: true
---

Azure has alot of great PaaS services that we want to be able to connect to securly from inside our VNet's. Some services have two different options and in order for you to take the best option we will try to explain what the difference is.
The two options are:
* Service Endpoint
* Private Endpoint


### Background
PaaS services has a tradition of beeing easily accessed and a fast way to get started on your cloud journey, some of them dident even have network options in the beginning. But as maturity evolved so did also the requirements for networking and customer wantet to build network protected solutions.
First out was the **Service Endpoint** who met the intitial requirements of protecting the resource with and made customers happy that they now could protect theire services.
But as complexity grew and security demands increased this was not enough and Microsoft created **Private Endpoints** and this is the newest addition.

### What is the difference
There is a significant difference between these offers and in short words enabling a **Service Endpoint** on a subnet allows any resource of that specific type to be connected to that subnet. Access to the resource will be securely via Azure backbone network but detection and identification is done via the public endpoint. Leaving the public endpoint still visible.
With **Private Endpoint** one specific resource is exposed inside the vnet with a private IP, registered in Private DNS Zone. Callers accessing the resource will get the IP from the Private DNS and therefore there is no need for the public endpoint to still be available.


  for the resources in Azure.  that want access  meanwhile **Private Endpoint** is a dedicated connection for a sepcific resource. This means that you via **Private Endpoint** now can have control over who is accessing 


*Service Endpoint* is applied on subnet for a resource typ and that applies to all intstances of that type for all Azure customers, not just your instances. And connection still goes via the Pulbic IP Address and restrictions are made by Firewall rules, but with the big difference that it's using a private IP address to make it easy to filter the request and no need to know the public IP of the caller.

Let's take an example a Logic App running on ASE want's to connect to a Storage Account. We enable service endpoint for storage accounts and connect Storage Account A to the subnet. We can now access the storage account. Access will be done via the public ip/dns to the storage account and the restrictions on who has access to the storage account is done based on the calling Logic App's Private IP via firewall rule. 

![Service Endpoint scenario](https://my.revision.app/api/svg/wjc72bpXFR)


*Private Endpoint* is privately connect and done more inside the network. A Private Link is a resource that we can create in our subnet, it will then have it's own private link to the specific instance. A DNS record is needed to be created in your network (private DNS Zone) so you can find the private endpoint. Now traffic is routed to the Private Endpoint and privately to the instance. Not needing the Public endpoint, so we can disable Public Access for the instance.

Let's take the same example, this time 

![Private Endpoint scenario](https://my.revision.app/api/svg/wjc72bpXFR)




This will have impact on a few things, for instance using *Private Endpoint* requries a DNS entry in a Private DNS Zone in your network for the resources inside you network to find it (trafic is going only inside yur network)




When connecting a resource via *Service Endpoint* we need to do the followung:
* Enable the possibility on the subnet to connect to the particilar service type
* Connect the resource

A simple setup and very open, any resource of that type can now be connected to that subnet.
Traffic is routed out on the public network and the resource is verifying that the connection is comming from that particular subnet.

When connecting a resource via *Private Endpoint* we need to do alot more since Private Endpoint is a resource itself:
* Create a *Private Endpoint* resource for the resource
* Create a record in the private DNS for the specific domain for the *Private Endpoint*
* Connect the resource via 


#### Service Endpoint


#### Private Endpoint





To make this setup work, we need to configuration a couple of things for traffic to flow.

* VNet with at least 2 Subnets
* Each Subnet should have a Network Security Group (NSG)
    * To protect from unwanted traffic in and out of the subnet.
* App Service Environment, deployed in a dedicated subnet
* Logic App Standard, deployed on the App Service Environment
* Storage Account, deployed with private endpoint in to the other subnet.

The 2 subnets is divided as follows, *Subnet A* dedicated for the *App Service Environment* due to the need for it's own subnet. We then create a second subnet *Subnet B* so we can use it for adding private links or other services. It will look like this:

![Network overview Overview](https://my.revision.app/api/svg/wjc72bpXFR)

Now we can add the storage account and the private links. To lock down our storage account we need four private endpoints, one for each service (Blob, Files, Table, Queues). Private links require a private DNS record to be created inside the VNet. This is so that resources can find them. We can just use the built in suggestion and put the standard generated private DNS zone in a Production subscription (since we used hub-and-spoke network setup) attached to the VNet. Now DNS lookup will be correct inside the network.


![Network Overview](https://my.revision.app/api/svg/xjfrOFAdfK)

We also need to make sure that traffic can flow, all these endpoints are accessed over HTTPS so rules need to be added in the NSG's.
In the *NSG* on *Subnet A* we need to allow outbound HTTPS(443) and SMB(445) traffic, the rules below allows connecting to *Subnets* from *Subnet A*.(You can of course narrow this down by changing *destination* property)

![Outbound Rule](/assets/images/2023/march/outbundhttpsnsgrule.png)

The same goes for the NSG that is attached to *Subnet B* but inbound rule to allow HTTPS(443) and SMB(445) traffic. The rules below allow all *Subnets* to sen requests to *Subnet B* over HTTPS. (You can of course narrow this down by changing *source* property)

![Inbound Rule](/assets/images/2023/march/inboundhttpsnsgrule.png)

We can now deploy our *Logic App Standard* instance to our *App Service Environment*, this is done via picking the *App Service Environment* as the **region** when creating the *Logic App Standard* instance. It will then connect to the *Storage Account* as the image describes and the traffic will be allowed due to our rules in the NSG's.

![Full Overview](https://my.revision.app/api/svg/1yXN5QZ4Ut)

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


* [Integrate Azure services with virtual networks for network isolation](https://learn.microsoft.com/en-us/azure/virtual-network/vnet-integration-for-azure-services)

