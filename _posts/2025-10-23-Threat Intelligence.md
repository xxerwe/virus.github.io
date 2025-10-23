---
title: "Oman Cybersecurity Olympiad-Threat Intelligence"
date: 2025-10-22 14:30:00 +0400
categories: [write-up's, Oman Cybersecurity Olympiad]
tags: [write-up, Oman Cybersecurity Olympiad]
---

![Desktop View](/assets/lib/Threat intelligence.png){: .normal }

This short write-up documents how I triaged an external IP repeatedly contacted by a corporate endpoint during unusual hours, determined whether it belongs to a legitimate service, and extracted the three flags requested by the challenge.

# TL;DR (Flags)
```
Flag 1 — Geolocation: 'United States'

Flag 2 — Associated domain/vendor: 'amazon.com'

Flag 3 — Most reported threat type (abuse reports): 'phishing'
```
Objective

Given the suspicious IP 35.89.44.32, assess ownership, reputation, and network context to decide whether the traffic is likely legitimate (e.g., mail relay / SaaS endpoint) or potential C2/exfiltration, and provide three specific answers: geolocation, associated domain, and most reported threat type.

Toolkit
```
WHOIS / RDAP / BGP ASN lookups

Passive DNS / reverse DNS (when available)

Abuse reputation portals (categories only; not needed to quote exact counts)

tshark / Wireshark on the provided Network.pcap (contextual cross-check)
```
# Methodology (Step by Step)
1) Ownership and Network Context

Run WHOIS/RDAP to map the netblock/ASN and operator:
```
whois 35.89.44.32 | egrep -i 'OrgName|netname|CIDR|Country|OrgAbuseEmail'
whois -h whois.cymru.com " -v 35.89.44.32"
```

Result: Address is hosted in AWS infrastructure (Amazon ASN), geolocated to the `United States` ⇒ Flag 1.

(Optional) Check reverse DNS / passive DNS to spot service hints:
```
dig +short -x 35.89.44.32
```

Even if rDNS isn’t set, the ASN and allocation confirm Amazon as the infrastructure owner ⇒ Flag 2 as `amazon.com`.

2) Reputation & Abuse Categories

Check abuse-tracking portals and reputation feeds for category trends (not necessarily specific incidents). Cloud mail/relay endpoints often accumulate reports due to high message volume from varied tenants. The dominant category seen for similar IPs is typically phishing or spam.
Outcome: Most common observed label: `phishing` ⇒ Flag 3.


# Difficulty & Notes


Difficulty: Easy ↔ Medium.

Gotchas: Don’t rely on a single source. Correlate ownership (WHOIS/ASN), category trend from abuse feeds, and PCAP evidence (ports/SNI) before concluding.

Answer format: Keep flags exactly as the challenge expects.

Final Flags
```
Geolocation: 'United States'

Associated domain/vendor: 'amazon.com'

Most reported threat type: 'phishing'
```
