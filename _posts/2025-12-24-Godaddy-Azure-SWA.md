---
layout: post
title: Configure a GoDaddy custom domain for Azure Static Web App
date: 2025-12-24
categories: Cloud
---

In this post we are going to outline the setup required to use a GoDaddy domain on Azure Static Web App (SWA).

The standard process to add a custom domain to an Azure SWA is straightforward. However, GoDaddy has limitations with naked domains and ALIAS records, which makes this slightly less obvious.

---

## Step 1

From the SWA Resource in Azure, under **Settings**, select:

**Custom domains > Add > Custom domain on other DNS**

---

## Step 2

Enter your custom domain using the **www** prefix.

> If you attempt to use a naked domain such as `technosim.me`, Azure will suggest:
>
> - Type: CNAME  
> - Host: `@`
>
> GoDaddy will reject this with:
> **“Record data is invalid.”**

---

## Step 3

Do not use a naked domain.  
Use `www.technosim.me` instead.

![Azure Portal Screenshot](/images/2025-12-24-Godaddy-Azure-SWA/godaddy1.png)

---

## Step 4

From your GoDaddy portal:

- DNS > Add New Record
- Type: **CNAME**
- Name: **www**
- Value: your Azure SWA default URL  
  (e.g. `lively-island-0ad256600.6.azurestaticapps.net`)

After DNS propagation, your site will load via the `www` domain.

![GoDaddy DNS Screenshot](/images/2025-12-24-Godaddy-Azure-SWA/godaddy2.png)

---

## Step 5 - Naked domain forwarding

In GoDaddy:

- DNS > Forwarding
- Add Forwarding under **Domain**
- Target: `https://www.technosim.me`
- Type: **Permanent**

This forwards the naked domain to `www`.

![Forwarding Screenshot](/images/2025-12-24-Godaddy-Azure-SWA/godaddy3.png)
