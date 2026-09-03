# Security+ Hands-On Labs

A collection of hands-on network and security exercises completed while studying for the CompTIA Security+ (SY0-701) certification. Each write-up documents a real finding, tool, or technique — not just a tutorial followed step by step.

**Certification status:** CompTIA Security+ (SY0-701) — Exam Scheduled 09/04/26.

## Skills demonstrated across these labs
- Network reconnaissance and port scanning (Nmap)
- Packet capture and protocol analysis (Wireshark)
- Identifying unencrypted vs. encrypted traffic, and the security implications of each
- TLS session decryption and protocol-layer analysis (SSLKEYLOGFILE, HTTP/2 dissection)
- Home network hardening (default credential remediation, attack surface reduction)
- Third-party risk identification via traffic metadata analysis
- Clear technical write-ups translating raw findings into security impact and remediation

---

## Lab Index
### 1. [Router Default Credential Discovery and Remediation](local-network-recon-2/Default-Router-Admin-Notes)
**Date:** Aug 7, 2026
Discovered my home router was still running factory-default admin credentials. Documents the specific threat vectors this exposed (router takeover, DNS hijacking, botnet recruitment) and the remediation steps applied.

### 2. [Observing Plaintext HTTP vs. Encrypted HTTPS Traffic](local-network-packet-capture-1/packet-capture-notes.md)
**Date:** Aug 2026
Used Wireshark to directly compare unencrypted HTTP traffic against TLS-encrypted HTTPS traffic on the same type of request, demonstrating why encryption in transit matters using real captured evidence rather than just theory.

### 3. [Identifying Third-Party Tracking Domains via TLS SNI Analysis](local-network-packet-capture-2/packet-capture-notes-2)
**Date:** Aug 8, 2026
Used the TLS Client Hello's SNI field to identify distinct encrypted conversations without decryption, uncovering multiple third-party ad-tech/tracking domains silently contacted by a single university webpage load.

### 4. [Decrypting Own TLS Traffic with SSLKEYLOGFILE and HTTP/2 Analysis](local-network-packet-capture-3/decrypting-TLS-traffic-notes)
**Date:** Aug 9, 2026
Decrypted my own browser's TLS sessions using the SSLKEYLOGFILE technique, then compared raw decrypted TLS stream output against parsed HTTP/2 stream output to demonstrate the difference between decryption and protocol-level readability.

### 5. [Capturing Unprotected HTTP Login Credentials](local-network-packet-capture-4/http-password-capture-notes)
**Date:** Aug 11, 2026
Located a live, publicly accessible login page still using plain HTTP, and captured the full plaintext credential submission via packet analysis using fake/dummy credentials — a real-world demonstration of credential exposure risk over unencrypted channels.

### 6. [Remote Desktop Management](remote-desktop-management-1/remote-desktop-management-notes.md)
**Date:** Aug 29, 2026
Remotely accessed my desktop via Anydesk from my laptop to simulate an IT ticket. Disabled a printing function remotely to create a fault and then went through trouble shooting steps to restart the printing adapter remotely.

---

## Additional recon (informal findings, write-ups in progress)
A few early exploratory scans that i did, but reflect the same skill set:
### 0. [Pinging Smart TV and discovering an unauthenticated local API](smart-tv-recon-1/smart-tv-recon-notes).
**Date:** Aug 7, 2026
- Identified an exposed, unauthenticated local API (embedded Mongoose httpd server) on a smart TV via Nmap, and fingerprinted it as a Netflix companion-app endpoint through passive banner-grabbing.
OTHERS:
- Identified an iPhone on the local network via its lockdownd service (port 62078) and researched the associated pairing-record security model.
- Practiced vendor/device fingerprinting via MAC address OUI lookups on unidentified network devices.

## Tools used across this repo
Nmap, Wireshark, Google Chrome DevTools/SSLKEYLOGFILE, PowerShell, Python (for local test server setup)

## About this repo
These labs were completed independently, on my own home network and personally owned devices, alongside structured Security+ exam preparation. Each write-up follows a consistent format: objective, steps taken, findings, security impact, and remediation/next steps — modeled after how real vulnerability and incident findings are documented in industry.




