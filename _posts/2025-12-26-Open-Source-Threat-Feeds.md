---
layout: post
title: Dynamically Block Bad Actors with Open-Source Threat Feeds
date: 2025-12-26
categories: Security
---
Open-source threat feeds are dynamically updated, public lists of bad actors or threats.  
Among these threat feeds, you can find lists of public IP addresses with a poor reputation, typically identified automatically using honeypots and other techniques. Common sources include:


- **CINS Army** - https://cinsscore.com/list/ci-badguys.txt  
- **Abuse.ch**  
- **FireHOL** - https://iplists.firehol.org/
- **threatfeeds.io** - https://threatfeeds.io/


## Threat Feed Categories

Threat feeds are typically categorised by behaviour, such as brute-force attackers or botnet command-and-control (C2) servers. It’s important to understand that feeds also fall somewhere along a classification scale:

- **Highly aggressive:** Includes emerging threats but comes with a higher risk of false positives  
- **High stability:** Fewer false positives, while still providing protection by blocking well-known bad actors


## Why Use Open-Source Threat Feeds?

It’s pretty standard for network security vendors to charge a subscription for dynamically updated security intelligence, such as public IP reputation data and malware hashes for file signatures. Open-source threat feeds are a great solution for small business networks, homelab environments, or even as part of an enterprise-grade security stack on a budget. They allow you to gain basic dynamic protection without paying for a subscription.



## Common Use Cases

- Use open-source threat feeds in your DMZ policy to deny all known bad actors inbound from the internet.  
- Use open-source threat feeds to block hosts inside your network from establishing outbound connections to C2 servers.



## Configuring an Open-Source Threat Feed on FortiGate

1. Navigate to **Security Fabric > External Connectors > Create New**.  
2. Scroll down to the **Threat Feeds** section and select the type of feed you are adding:
   - IP Address  
   - Domain Name  
   - Malware Hash  


![FortiGate Screenshot ThreatFeed](/images/2025-12-26-Open-Source-Threat-Feeds/fortigate1.png)

3. Place the threat feed inside a policy. In the source or destination field, select the feed object you just created.  
4. Set the action to **DENY**.

That’s it - you now have a dynamically updated list of bad actors.

![FortiGate Screenshot Policy](/images/2025-12-26-Open-Source-Threat-Feeds/fortigate2.png)

## Other Ideas

- Set up alerts to be notified when a host may be compromised (for example, communicating with a C2 server).  
- Create your own threat list and store it centrally, then reference it from all firewalls in your network. This allows you to block an IP address everywhere without logging into each firewall individually.



## Consumer and SMB Router Support

Many consumer grade routers support similar features, TP-Link routers include an IP Group feature, which doesn’t update dynamically but does support a one-time import. You could build custom automation around this if needed.

(Speaking of TP-Link, did you know they have online emulators, ha.. [TP-Link Emulators](https://www.tp-link.com/us/support/emulator/))

ASUS and Netgear routers offer similar features but with a different name.



## Conclusion

Open-source threat feeds provide a simple, cost-effective way to add meaningful threat intelligence and dynamic protection to almost any network.
