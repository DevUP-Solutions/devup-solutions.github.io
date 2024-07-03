---
layout: post
title: "Azure API Manageent and TLS 1.3 Compability"
description: TLS 1.3 is currently rolling out in Azure and API Management plays an vital role in the API landscape, so we need to understand what capabilities there are.
date: 2024-06-05 09:00:00 +0200
categories: Security Encryption
tags: [TLS, Security, Encryption]
author: "Mattias Lögdberg"
comments: true
---

# Understanding the Impact of TLS 1.3 on Azure API Management

API Management is a crucial part of the Azure Integration Services landscape, responsible for managing API calls and ensuring secure, efficient communication between services. As we move forward with technological advancements, it's important to understand the implications of upgrading TLS versions, particularly with the rollout of TLS 1.3.

As of today, July 3, 2024, here’s the current state of TLS 1.3 support in Azure API Management:

1. **Inbound TLS 1.3 Support:**
   - Since February 5, 2024, Azure API Management has automatically supported TLS 1.3 for inbound calls.
   - This support is evidenced by the enabled ciphers under Security -> "Protocols + ciphers":
     - TLS_AES_256_GCM_SHA384
     - TLS_AES_128_GCM_SHA256

2. **Enforcing TLS Versions:**
   - Azure API Management can enforce the use of TLS 1.2 and TLS 1.3 by disabling support for TLS 1.0 and TLS 1.1 under Protocols (Security -> "Protocols + ciphers").

3. **Outbound TLS Support:**
   - Currently, Azure API Management supports calling other services using TLS versions up to 1.2.

**In Summary:** Upgrading to TLS 1.3 is essential for maintaining a secure, efficient, and reliable digital presence. However, due to the current limitations in Azure API Management's outbound TLS support, we need to postpone the enforcement of TLS 1.3 for services being called by Azure API Management until further notice.

### How do I know if I have an issue?

If you suddenly receive **500 Internal Error** messages, this could be due to a TLS 1.3 enforcement issue with a service called by Azure API Management.

```
HTTP/1.1 500 Internal Server Error
content-length: 111
content-type: application/json
date: Wed, 03 Jul 2024 09:24:01 GMT
request-context: appId=cid-v1:8c873921-86a3-45d5-a560-1088bff1d583
vary: Origin
    
{
    "statusCode": 500,
    "message": "Internal server error",
    "activityId": "269679da-4d32-418e-be8d-db2e0e6f2dcb"
}
```

### Troubelshoot your trace
If you encounter this error in the forward request, ensure that the backend service is compatible with TLS 1.2.

```
forward-request (17.367 ms)
{
    "messages": [
        "Error occured while calling backend service.",
        "The request was aborted: Could not create SSL/TLS secure channel."
    ]
}
```


### If I need some assistance?
We at DevUP are experts in this area, and we use our service **Helium** to quickly scan the whole environment and get the TLS versions per resource, we know and will let you know when new versions of TLS is released.

Reach out, and we can show you how to get this information in minutes and continuously.