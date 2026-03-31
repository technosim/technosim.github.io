---
layout: post
title: Optimize WiFi coverage for high interference environments
date: 2026-03-30
categories: Networking
---

Shared commercial or retail areas are a challenge for WiFi when sharing RF space with neighbouring tenants, customer mobile hotspots, and Bluetooth devices wall to wall. This interference falls into two categories:

- **Co-channel interference (CCI)**: other APs transmitting on the same channel
- **Adjacent/non-adjacent channel interference**: neighbouring APs bleeding into your spectrum

When raw throughput isn't the goal and coverage reliability is, noise floor management and channel discipline is key.

---

## Deployment and planning

There is huge amounts of info about AP placement planning, cell overlap targets, height, obstruction, ceiling vs wall mount, all this kind of thing. I don't intend to cover that here.  
Rather, I am sharing information that I have found helpful when addressing performance in wireless networks that had already been deployed, by someone else, many years ago.

### Commit to 5 GHz and 6 GHz

The 2.4 GHz band is essentially unusable in dense retail environments. Every neighbouring store, everyone is already there. 5 GHz offers far more non-overlapping channels, and 6 GHz is still largely clear in most venues.

### Plan channels manually

Automatic channel selection tends to perform poorly in high interference environments. DCS algorithms react to neighbour APs and can make changes that create new conflicts rather than resolve them.

Instead, do a quick WiFi scan with something like WiFi Analyzer or Ekahau Sidekick, map out what your neighbours are using, and assign channels manually around that. iOS has Apples "Airport Utility" which can do the job ok too.

On 5 GHz, prefer UNII-1 and UNII-3 (channels 36–48 and 149–165). Avoid DFS channels where possible, radar detection events trigger channel changes and cause brief outages. Leave as many channels unused as your plan allows to maximise channel reuse distance. 
This being said, I have found Aruba ARM (Adaptive Radio Management) to sometimes perform better than manual channel assignment. Experiment with both.

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

### Disable auto TX power, turn it down

Maybe counterintuitive but reducing AP transmit power can improve performance in high interference environments. Lower TX power means:

- Your AP hears fewer competing APs and defers to them less
- You shrink your BSS, reducing the number of foreign APs that treat your channel as busy
- Clients roam more aggressively to the nearest AP instead of clinging to a distant one

The trade-off is that you need sufficient AP density to maintain coverage. In a high interference environment, more lower-power APs is almost always better than fewer high-power ones.

### Configure Interference Immunity Level
An Aruba setting configured to improve performance in high-noise environments by making the radio less sensitive to non-802.11 interference, so for stabilising channel selection.
Check your vendor for a similar feature.

### Talk to your neighbours

Many small tenants are running a default-config D-Link or TP-Link router, transmitting at full power because nobody told them otherwise. Some might even have cranked the TX power intentionally, not realising it degrades performance for everyone including themselves.

It sounds unusual, but approaching a neighbouring and politely asking them to dial back their transmit power can genuinely help. Frame it as mutual benefit.

---

## Other general best practices

- Disable 2.4 GHz entirely if clients support 5/6 GHz, or at the very least **use band steering** to prefer 5/6Ghz
- Limit advertised SSIDs to three or less, every SSID adds beacon overhead across all APs
- Enable Multicast-to-Unicast conversion where supported, to reduce broadcast overhead and improve reliability for multicast traffic
