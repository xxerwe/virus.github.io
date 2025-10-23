---
title: "Oman Cybersecurity Olympiad-The hunter"
date: 2025-10-22 14:30:00 +0400
categories: [write-up's, Oman Cybersecurity Olympiad]
tags: [write-up, Oman Cybersecurity Olympiad]
---

![Desktop View](/assets/lib/The hunter.png){: .normal }


This write-up covers how I identified the infected host’s operating system and tied network authentication to a specific user account using the provided PCAP from an Active Directory environment.

# TL;DR (Flags)
```
Flag 1 — OS string: `Windows NT 10.0`

Flag 2 — User (Kerberos): `rene.mccollum`
```
Objective

Analyze an AD network capture to (1) fingerprint the infected Windows host’s OS version string, and (2) extract the user account authenticating from that host via Kerberos.

# Toolkit
```
Wireshark / tshark

Knowledge of Windows protocols (SMB/NTLMSSP, Kerberos, LDAP, DNS)
```
# Methodology (Step by Step)
1) Identify the OS from Protocol Fingerprints

The most reliable quick win is NTLMSSP metadata in SMB/related traffic. Windows endpoints often leak a canonical “Windows NT X.Y” string.

Pull NTLMSSP version hints
```
tshark -r Network.pcap -Y ntlmssp -T fields -e ntlmssp.version | sort -u
```
Detailed dump to catch explicit OS strings / build numbers
```
tshark -r Network.pcap -Y ntlmssp -V 2>/dev/null | egrep -i "NTLMSSP|Version|Windows NT|Build" | head -n 200
```

Outcome: The canonical string appears as `Windows NT 10.0` → Flag 1

Tip: In some captures you can also pivot on DHCP options, HTTP user-agents, or LDAP binds; but NTLMSSP is typically the cleanest for the exact phrasing the challenge wants.

1) Extract the Authenticating User via Kerberos

Next, enumerate Kerberos exchanges between the infected client and the Domain Controller. Kerberos AS-REQ/AS-REP/TGS-REQ/TGS-REP carry the client principal.

Show Kerberos frames with key identity fields
```
tshark -r Network.pcap -Y kerberos -T fields \
  -e frame.time -e ip.src -e ip.dst -e kerberos.CNameString -e kerberos.sname
```
Unique client names seen in tickets
```
tshark -r Network.pcap -Y "kerberos.CNameString" -T fields -e kerberos.CNameString | sort -u
```

Outcome: Client principal resolves to `rene.mccollum` → Flag 2

Assessment

OS: Windows 10 lineage confirmed by NTLMSSP string.

User: Kerberos CNameString links the infected Windows client to rene.mccollum.

Difficulty & Notes

Difficulty: Easy ↔ Medium.

Gotchas: Ensure your answer matches the exact string format required (Windows NT 10.0, not “Windows 10”). For the username, keep the dot separator and case exactly as captured.

# Final Flags

# OS: 'Windows NT 10.0'

# User: 'rene.mccollum'
