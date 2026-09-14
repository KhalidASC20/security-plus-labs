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
## Network Security and Traffic Analysis
### 1. [Router Default Credential Discovery and Remediation](network-security/local-network-recon-2/Default-Router-Admin-Notes)

**Date:** Aug 7, 2026

Discovered my home router was still running factory-default admin credentials. Documents the specific threat vectors this exposed (router takeover, DNS hijacking, botnet recruitment) and the remediation steps applied.

### 2. [Observing Plaintext HTTP vs. Encrypted HTTPS Traffic](network-security/local-network-packet-capture-1/packet-capture-notes.md)

**Date:** Aug 2026

Used Wireshark to directly compare unencrypted HTTP traffic against TLS-encrypted HTTPS traffic on the same type of request, demonstrating why encryption in transit matters using real captured evidence rather than just theory.

### 3. [Identifying Third-Party Tracking Domains via TLS SNI Analysis](network-security/local-network-packet-capture-2/packet-capture-notes-2.md)

**Date:** Aug 8, 2026

Used the TLS Client Hello's SNI field to identify distinct encrypted conversations without decryption, uncovering multiple third-party ad-tech/tracking domains silently contacted by a single university webpage load.

### 4. [Decrypting Own TLS Traffic with SSLKEYLOGFILE and HTTP/2 Analysis](network-security/local-network-packet-capture-3/decrypting-TLS-traffic-notes.md)

**Date:** Aug 9, 2026

Decrypted my own browser's TLS sessions using the SSLKEYLOGFILE technique, then compared raw decrypted TLS stream output against parsed HTTP/2 stream output to demonstrate the difference between decryption and protocol-level readability.

### 5. [Capturing Unprotected HTTP Login Credentials](network-security/local-network-packet-capture-4/http-password-capture-notes.md)

**Date:** Aug 11, 2026

Located a live, publicly accessible login page still using plain HTTP, and captured the full plaintext credential submission via packet analysis using fake/dummy credentials — a real-world demonstration of credential exposure risk over unencrypted channels.

## Remote Systems Administration

### 1. [Remote Desktop Management](remote-support/remote-desktop-management-1/remote-desktop-management-notes.md)

**Date:** Aug 29, 2026

Remotely accessed my desktop via Anydesk from my laptop to simulate an IT ticket. Disabled a printing function remotely to create a fault and then went through trouble shooting steps to restart the printing adapter remotely.

### 2. [Remote Desktop App Installation](remote-support/remote-software-installation/remote-software-installation-notes.md)

**Date:** Aug 29, 2026

Installed third-party software (7-Zip) on a remote endpoint via AnyDesk, encountering and navigating Windows' Secure Desktop UAC isolation — a security boundary that blocked the remote tool from directly interacting with the elevation prompt, requiring physical confirmation at the endpoint to complete. 

## Linux Administration

### 1. [Linux Server Setup: VM Provisioning, Remote Access, and Firewall Hardening](linux-admin/VM-provisioning-firewall-hardening/VM-provisioning-firewall-hardening-notes.md)

**Date:** Sep 11, 2026

Provisioned an Ubuntu Server VM from scratch, configured remote access via SSH, and applied a default-deny firewall policy using ufw — establishing a secure baseline server configuration and demonstrating correct sequencing to avoid self-lockout during remote firewall setup.

### 2. [Troubleshooting a Broken SSH Key Authentication Setup (Multi-Layered Root Cause)](linux-admin/VM-SSH-troubleshooting/VM-SSH-Troubleshooting-notes.md)

**Date:** Sep 11, 2026

Diagnosed and resolved a chained SSH authentication failure spanning three separate root causes — a key pair generated in the wrong location, a silently overriding configuration file, and a missing Windows utility — recovering access using a preserved out-of-band console session rather than reverting the hardening in progress.

### 3. [Linux User, Group, and Permission Management](linux-admin/user-permission-management/user-permission-management-notes.md)

**Date:** Sep 13, 2026

Implemented multi-user account management, shared directory permissions, and access control lists (ACLs) on a Linux server, then applied the least-privilege principle in practice by granting a user a single, narrowly scoped sudo capability via visudo — verified by confirming the user could perform the one authorized action while being correctly denied access to a sensitive system file.

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
