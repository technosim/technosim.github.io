---
layout: post
title: Aruba AOS-CX Policy Based Routing (PBR)
date: 2025-12-24
categories: Networking
---

Ensure that all traffic originating from `10.199.0.0/16` subnet will be sent to a firewall as the next hop, rather than through the default route. 

Since a standard route will only match based on destination, we classify the source traffic, create an action, create a policy to apply the action to the classified traffic, then apply the policy to the VLAN interface.

---

## 1. Classify the source traffic

Use format `class ip <name-of-class>`. I put "class" explicitly in the name to keep it clearer. I am also matching for traffic destined to `10.0.0.0/8`.

```text
class ip tempsite-class
match ip 10.199.0.0/16 10.0.0.0/8 count
exit
```

## 2. Create an action

Our action is going to be simple: set the next hop, but there are other actions. Same as with class, I put "action" explicitly in the name for simplicity.

```text
pbr-action-list tempsite-action
    nexthop 10.20.30.2
exit
```

## 3. Create the policy

This step applies the routing policy to the classified traffic from step 1 and forwards it according to the policy defined in step 2, instead of the default routing table.

```text
policy tempsite-pbr
    1 class ip tempsite-class action pbr tempsite-action
exit
```

## 4. Apply it to the VLAN interface

We use "routed-in" to specify that we want to match on traffic coming into the VLAN interface.

```text
int vlan 199
    apply policy tempsite-pbr routed-in
```

## 5. Verify the configuration

You can verify from the switch, or simply do a traceroute from inside the subnet, where you would see 10.20.30.2 as the next hop after the switch.

```text
show policy hitcounts tempsite-pbr
```

---

**Sources:**

https://arubanetworking.hpe.com/techdocs/AOS-CX/10.09/HTML/ip_route_4100i-6000-6100-6200/Content/Chp_PBR/pol-bas-rou-pbr.htm

https://community.arubanetworks.com/discussion/aruba-cx-6300m-policy-based-routing