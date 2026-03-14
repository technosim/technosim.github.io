---
layout: post
title: Aruba and FortiNet Multicast Routing with PIM
date: 2026-03-13
categories: Networking
---

Intra-vlan multicast is simple, just let default IGMP configuration do the work. But, having the traffic pass through a Firewall and Switch to be routed with Protocol Independant Multicast (PIM) introduced more of a challenge.
While it is still pretty straight forward with PIM, what I found is a limitation in FortiOS versions below 7.6.

## Documentation 

### FortiNet
[FortiGate 6.4 Configuring Multicast Forwarding](https://docs.fortinet.com/document/fortigate/6.4.0/administration-guide/968606/configuring-multicast-forwarding)

[FortiGate 7.6.4 Multicast Routing and PIM Support](https://docs.fortinet.com/document/fortigate/7.6.4/administration-guide/896987/multicast-routing-and-pim-support)

[Troubleshooting Multicast](https://community.fortinet.com/t5/FortiGate/Troubleshooting-Tip-Troubleshooting-issues-with-multicast/ta-p/272244)

[Multicast Routing on FortiGate](https://www.fortinetguru.com/tag/multicast-routing-on-fortigate/)

### Aruba
[AOS-CX Multicast Guide PDF](https://arubanetworking.hpe.com/techdocs/AOS-CX/10.13/PDF/multicast_6200-6300-6400-8xxx-9300-10000.pdf)

[AOS-S Multicast Guide PDF](https://arubanetworking.hpe.com/techdocs/AOS-Switch/16.09/Aruba%203810M%265400R%20Multicast%20and%20Routing%20Guide%20for%20ArubaOS-Switch%2016.09.pdf)

[2930F/M Multicast Guide 16.09 PDF](https://arubanetworking.hpe.com/techdocs/AOS-Switch/16.09/Aruba%202930F%26M%20Multicast%20and%20Routing%20Guide%20for%20AOS-S%20Switch%2016.09.pdf)

[2930F/M Multicast Guide 16.10 PDF](https://arubanetworking.hpe.com/techdocs/AOS-Switch/16.10/Aruba%202930M%26F%20Multicast%20and%20Routing%20Guide%20for%20AOS-S%20Switch%2016.10.pdf)

### IETF

https://datatracker.ietf.org/doc/html/rfc4601

https://datatracker.ietf.org/doc/html/rfc3973

https://www.rfc-editor.org/rfc/rfc2365

# My Setup
Using VLC I stream to UDP 238.1.2.3:1234 from server 192.168.100.1 to receiver 10.10.100.1 via routed multi-vendor network.

![UDP Stream Setup](images/2026-03-14-Aruba-FortiNet-Multicast-Routing/1-multicast-setup.jpg)

**From server, kill and restart stream:**
```
pgrep vlc
pkill vlc
vlc -vvv sample.mp4 --sout udp:238.1.2.3:1234 --ttl 12
```

For TTL in GUI: Generated stream output string section, look for `dst=238.1.2.3:1234`
Manually prepend TTL parameter so it looks like `{access=udp{ttl=12},mux=ts,dst=238.1.2.3:1234}`

**From receiver**
VLC > Media > Network stream udp://@238.1.2.3:1234 

## Breaking it down:

1. You need a multicast policy on the FortiGate 
2. You need to enable pim sm on all L3 interfaces in the path
3. And enable pim on all L3 hops in the path
4. Need IGMP on L2 facing the receiver so they can join the group
5. Run a stream and use show commands to verify end to end stream connectivity
6. You may need unicast connectivity for PIM reverse path forwarding calulculation RPF - I found not required in my testing.

## Multicast Notes:
- The first 24 bits of a multicast MAC address always start with 01:00:5E
- There are 3 versions of IGMP. Backwards compatible.
- The switch facing the receiver needs IGMP. Default version v3 on Aruba CX
- with PIM-SM you need to designate an RP but can be dynamic
- Seems default behavior for Multicast stream is with **TTL=1**, this needs to be changed to support multihop with PIM.

## Show commands
### CX & AOS-S

```
show ip pim neighbor
show ip igmp statistics
show ip igmp
show ip pim interface
show ip mroute
show ip igmp config
```

### FortiGate
```
get router info multicast pim sparse-mode rp-mapping
get router info multicast igmp groups
get router info multicast pim sparse-mode next-hop
get router info multicast pim sparse-mode interface

diagnose ip multicast mfc-add
diagnose ip multicast mfc-del
get router info multicast igmp groups
get router info multicast igmp groups-detail
get router info multicast table
get router info multicast table-count
get router info multicast pim sparse-mode bsr-info
get router info multicast pim sparse-mode rp-mapping
get router info multicast pim sparse-mode next-hop
get router info multicast pim sparse-mode table
execute mrouter clear multicast-routes
execute mrouter clear sparse-mode-bsr
execute mrouter clear sparse-routes
execute mrouter clear statistics

diagnose debug disable
diagnose debug reset
diagnose ip router pim-sm all enable
diagnose ip router pim-sm level info
diagnose debug enable
#wait
diagnose debug disable
diagnose debug reset


```

## Configuration
### FortiGate

```
config router multicast
    set multicast-routing enable
    config pim-sm-global
        config rp-address
            edit 1
                set ip-address 10.10.105.254 # set static RP as switch
            next
        end
    end
    config interface
        edit "TRANS" # vlan facing switch
            set pim-mode sparse-mode
        next
        edit "internal1" # router interface facing stream server
            set pim-mode sparse-mode
        next
    end
end

config firewall multicast-policy
    edit 1
        set name "Multicast"
        set srcintf "ZoneSender"
        set dstintf "ZoneReceiver"
        set srcaddr "StreamServer"
        set dstaddr "238.1.2.3"
        set protocol 17 # UDP
        set start-port 1234
        set end-port 1234
    next
end
config system settings
    set multicast-forward enable
    set multicast-ttl-notchange enable
end
```

## AOS-S Aruba



```
ip igmp
exit

ip multicast routing

router pim
   enable
   rp-address 10.10.105.254 224.0.0.0 240.0.0.0
   exit

vlan 105
   name "FortiGate Transit"
   untagged 2
   ip address 10.10.105.254 255.255.255.248
   ip pim-sparse
      ip-addr any
      exit
   exit

	
vlan 100 # Is IGMP Querier
   name "Receiver-vlan"
   untagged 1
   ip address 10.10.100.254 255.255.255.0
   ip igmp
   ip pim-sparse
   exit

```

## AOS-CX


```
router pim
    enable
    rp-address 10.10.105.254


interface vlan 105
    ip pim-sparse enable

vlan 100
    ip igmp snooping enable
    ip igmp snooping version 2

interface vlan 100
    ip pim-sparse enable
    ip igmp enable
    ip igmp version 2

```

> **NOTE**
> Verify IGMP configuration on any layer 2 switches down stream.

# Test, Validate and Diag

Confirm there is a PIM neighbourship on the switch:
`show ip pim neighbor`

**When stream is running:**

Confirm the FortiGate has a route for 238.1.2.3
`get router info multicast table`

Confirm the switch has a route for 238.1.2.3
`show ip pim mroute`

Check the switch has received IGMP Join from receiver
`show ip igmp`

For further troubleshooting on the FortiGate, use Network > Diagnostics > Debug Flow.

You need an IGMP Querier - on AOS-S this is done by simply configuring an IP address on the vlan.
If no IP, the switch does not participate in the querier election. I believe this is not the same for most switch vendors, instead an ip address configured on the vlan is required i.e SVI.


`show ip igmp vlan 100 config`

shows port mode 'auto', test with override to forward:
`vlan 100 ip igmp forward 1`
`show ip igmp vlan 100 group 238.1.2.3`

Test with proxy IGMP to the FW
`igmp-proxy-domain some.domain 10.10.105.250 238.1.2.3 238.1.2.3`

Static group floods all multicast traffic to every port on the vlan, test with:
`vlan 100`
    `ip igmp static-group 238.1.2.3`


`show ip igmp statistics`

`ip arp-mcast-replies` Enables acceptance of multicast MAC addresses in the IP multicast address range in ARP requests and replies.

Validate next hop on the firewall:
`get router info multicast pim sparse-mode next-hop`

# The catch with FortiOS

- FortiOS Documentation 7.2 "Multicast routing and PIM support" - no mention of VRF.

- Same in 7.4 Documentation.

- But in 7.6:
>PIM supports all VRFs and is aware of IPv4 multicast routing and forwarding over a single overlay, enhancing network scalability and flexibility **compared to the previous VRF 0-only support.**

**FortiOS <7.6 only supports PIM in VRF 0.**
I found that since I was using VRF-1 for routing this stream, it would fail because the PIM process only runs under VRF-0, it had the incorrect next hop, using the incorrect routing table. Even if both interfaces are in VRF-1 it will still operate under VRF-0 without clear indication.

> **NOTE**
> At the time of writing this, **FortiManager** does not support FortiOS 7.6. If you are using FortiManager, your only option is to flip VRFs so that the stream is running under VRF-0 routing table.

Multicast route leaking between vrf? Not supported, unicast only.

### igmp, rp and bsr candidate
Not required since we have static RP. 
IGMP also should not be required on the firewall but seems like it is enforced
vlan 100 ip igmp forward 1 # also not required