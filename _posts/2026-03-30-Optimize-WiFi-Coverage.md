---
layout: post
title: Optimize WiFi coverage for high interference environments
date: 2026-03-30
categories: Networking
---
# Optimize WiFi coverage for high interference environments

Guidance for environments where there is a large amount of interference, for example shopping centres sharing their space with neighbouring shops and customers with mobile hotspot and bluetooth. This type of interference falls into two categories:
- co-channel interference (CCI) - other APs on the same channel
- Adjacent/non-adjactent channel interferences - other APs bleeding into your space

When raw throughput is not the goal but simply best coverage in a high intereference environment, noise floor management and channel discipline is key.

## Deployment and planning considerations

### Plan for high density deployment, 5 GHz / 6GHz
The 2.4 GHz band is a write-off in a high interefence envrionment, every neighbouring store, every customer hotspot, every Bluetooth device is there.
- 5GHz has far more non-overlapping channels
- 6GHz is almost empty in most spaces right now

### Careful channel planning - manual, not auto
Dynamic channel selection can perform poorly in high interefence environments because they react to neighbour APs and make changes that create new conflicts.
	• Manually assign channels, mapping out what your neighbours are using first (a quick Wi-Fi scan with something like WiFi Analyzer or Ekahau Sidekick)
	• On 5 GHz, prefer the UNII-1 and UNII-3 bands (36-48, 149-165) avoid the DFS channels if you can, as radar detection events cause channel changes and brief outages
Leave as many channels unused in your plan as possible for channel reuse distance



## Configuration considerations
### 802.11k/r/v 
If your clients support it (most modern devices do):
	• 802.11k: neighbour reports so clients know which AP to roam to before they need to
	• 802.11r: fast roaming handoff with minimal authentication delay
	• 802.11v: BSS Transition Management, lets the AP tell a sticky client to move along
These keep clients on the best AP at all times, important when coverage quality varies across a large space.

### Channel bands
Disable wide channel bands, use 20 or 40Mhz only

###  Higher TX power is not always better
This one surprises people. In high interference environments, turning down your AP transmit power actually improves performance. Higher TX power can mean more noise. When lower:
	• Your AP hears fewer competing APs that it then has to defer to
	• You shrink your BSS, reducing the number of foreign APs that consider your channel "busy"
	• Clients roam more aggressively to the nearest AP rather than holding onto a distant one
The tradeoff is you need enough AP density to maintain coverage

### Confront the interferes
Many small stores or neighbouring networks will have a simple d-link or tp-link all in one wifi router with default settings. Meaning default power. Or maybe theyre just savvy enough to know how to turn up the TX power to max, but don't understand that this can be worse for them and for us. If you have neighbours like this, considder approaching and asking them to turn down their power. Explain it will improve performance for everyone. Sounds weird but might be required in some scenarios.

### Best practices and suggestions
- Consider disabling 2.4ghz range
- Minimise the amount of SSIDs being advertised, try to keep to 3 or less.
- Consider enabling Multicast Transmission Optimisation, can reduce latency and packet loss by converting multicast frames into higher-rate unicast streams or by optimizing broadcast data rates.

# ################
# ################ AI formatted:

# Optimising Wi-Fi coverage in high interference environments

Shared commercial or retail areas are a challenge for WiFi when sharing RF space with neighbouring tenants, customer mobile hotspots, and Bluetooth devices wall to wall. This interference falls into two categories:

- **Co-channel interference (CCI)**: other APs transmitting on the same channel
- **Adjacent/non-adjacent channel interference**: neighbouring APs bleeding into your spectrum

When raw throughput isn't the goal and coverage reliability is, noise floor management and channel discipline is key.

---

## Deployment and planning

### Commit to 5 GHz and 6 GHz

The 2.4 GHz band is essentially unusable in dense retail environments. Every neighbouring store, everyone is already there. 5 GHz offers far more non-overlapping channels, and 6 GHz is still largely clear in most venues.

### Plan channels manually

Automatic channel selection tends to perform poorly in high interference environments. DCS algorithms react to neighbour APs and can make changes that create new conflicts rather than resolve them.

Instead, do a quick WiFi scan with something like WiFi Analyzer or Ekahau Sidekick, map out what your neighbours are using, and assign channels manually around that. iOS has Apples "Airport Utility" which can do the job ok too.

On 5 GHz, prefer UNII-1 and UNII-3 (channels 36–48 and 149–165). Avoid DFS channels where possible, radar detection events trigger channel changes and cause brief outages. Leave as many channels unused as your plan allows to maximise channel reuse distance.

---

## Configuration

### 802.11k/r/v

Most modern clients support these. Enable all three if your infrastructure allows it.

- **802.11k**: Neighbour reports let clients know which AP to roam to before they need to
- **802.11r**: Fast roaming with minimal re-auth delay
- **802.11v**: BSS Transition Management lets the AP nudge sticky clients off when a better option is available

In a large space where coverage quality varies, keeping clients on the best AP at all times will make a difference.

### Channel width

Use 20 or 40 MHz. Avoid 80 MHz and wider, channel bonding increases your vulnerability to interference.

### Turn down TX power

Maybe counterintuitive but reducing AP transmit power can improve performance in high interference environments. Lower TX power means:

- Your AP hears fewer competing APs and defers to them less
- You shrink your BSS, reducing the number of foreign APs that treat your channel as busy
- Clients roam more aggressively to the nearest AP instead of clinging to a distant one

The trade-off is that you need sufficient AP density to maintain coverage. In a high interference environment, more lower-power APs is almost always better than fewer high-power ones.

### Talk to your neighbours

Many small tenants are running a default-config D-Link or TP-Link router, transmitting at full power because nobody told them otherwise. Some might even have cranked the TX power intentionally, not realising it degrades performance for everyone including themselves.

It sounds unusual, but approaching a neighbouring store and politely asking them to dial back their transmit power can genuinely help. Frame it as mutual benefit.

---

## General best practices

- Disable 2.4 GHz entirely if clients support 5/6 GHz
- Limit advertised SSIDs to three or less, every SSID adds beacon overhead across all APs
- Enable Multicast-to-Unicast conversion where supported, to reduce broadcast overhead and improve reliability for multicast traffic
