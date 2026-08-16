---
layout: post
title: Configure HA Load Balancing in Microsoft Global Secure Access ZTNA - Private Access
date: 2026-08-16
categories: Cloud Networking
---

To enable load balancing for two connector appliances in Microsoft Global Secure Access (GSA), simply deploy both connectors and use the GSA portal in Microsoft Entra admin to add them to the same Connector Group. GSA will automatically start load balancing traffic for application assigned to that group.  

However, the default behaviour for `trafficRoutingMethod` is `random`, where new requests for GSA Private Access resources are distributed across the connectors.  

Some applications may require `sessionPersistance`, also known as session affinity.  This configuration is applied per-Enterprise application. This configuration consistently routes every request from a given user and device to the same connector within a group for the duration of a session.

If you are experiencing problems with TLS session stability when using applications via GSA, test with `sessionPersistance`.

Microsofts documentation only provides raw API syntax to change this setting, they are not helpful enough to provide explicit steps. 

> I found that with FortiGate appliances and default GSA load balancing configuration, web GUI would authenticate but then revert back to the login screen immediately.

**Resources used:**

[Microsoft Learn: GSA Connector load balancing](https://learn.microsoft.com/en-us/entra/global-secure-access/concept-connector-groups#connector-load-balancing)
[Microsoft Learn: Configure traffic routing method for the GSA app](https://learn.microsoft.com/en-us/entra/global-secure-access/how-to-configure-per-app-access#configure-traffic-routing-for-the-app)  

---

1. Retrieve the `{appRegistrationObjectId}` for the Application in question. This value is in **Entra admin center** under **Entra ID** > **App registrations** > Open the **GSA Application** in question and copy the Object ID from the Overview page.
> You need to use the correct ID value, this is not the same as "Application ID" or even "Object ID" under Enterprise Application view. To avoid confusion, use these steps above specifically.

2. Log in to https://developer.microsoft.com/en-us/graph/graph-explorer
3. The default value for HTTP request method (drop down menu) should be `GET`. To verify the current load balancing configuration before we change it, set the drop down to `GET` if not already and in the url field, enter `https://graph.microsoft.com/beta/applications/{appRegistrationObjectId}?$select=onPremisesPublishing`

> Replace {appRegistrationObjectId} in the url above with your ObjectId retrieved from step 1.

  Click **Run query**


4. Check the reponse preview at the bottom of the page to verify the current configuration:
Under `onPremisesPublishing`, you see key value `trafficRoutingMethod : "none",`
> "none" means it is not configured and therefor default value (random).

5. Set the HTTP request method to `PATCH`. Remove `?$select=onPremisesPublishing` from the url, and in the **Request body**, enter:
```
{
    "onPremisesPublishing": {
        "trafficRoutingMethod": "sessionPersistence"
    }
}
```

  Click **Run query**

The query will take a couple of seconds and will update the trafficRoutingMethod to sessionPersistance.  

In my experience changing the setting, I did **not** notice any **session drops or disruption to user connections**.  

I suggest you perform the `GET` query again as above to confirm the setting has now changed.  

You can revert by simply replacing `sessionPersistance` with `random` and run again.
  
