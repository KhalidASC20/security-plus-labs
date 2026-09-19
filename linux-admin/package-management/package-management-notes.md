# Linux Package Management and Custom systemd Service Deployment

**Date:** September 19, 2026

**Tools used:** Ubuntu Server, apt, systemd, nginx, ufw

**Environment:** Personal Ubuntu Server VM, administered remotely via SSH from my own laptop

### Objective

Install, configure, and manage software packages and services on a Linux server, and design and deploy a custom systemd service from scratch — extending prior server-hardening and access-control work into ongoing operational service management.

### Steps taken

1. Updated the package index and installed nginx using `apt`, then verified the service was active via `systemctl status`.
2. Attempted to reach the installed web server from a browser on my laptop and encountered a connection timeout despite the service reporting as running.
3. Diagnosed the issue by checking `ufw status`, confirming only SSH was explicitly allowed and port 80 (HTTP) was being blocked by the default-deny firewall policy established in Lab 1.
4. Resolved the issue by explicitly allowing the nginx HTTP profile (`sudo ufw allow 'Nginx HTTP'`), then confirmed successful access to the default nginx page from my laptop's browser.
5. Practiced clean package removal using `apt remove` and `apt autoremove` on a smaller test package, confirming dependency cleanup.
6. Encountered a separate, unrelated issue when running `apt update`: repository validation errors indicating release files were "not valid yet." Diagnosed this as a system clock synchronization problem rather than a repository or network issue, based on the specific error phrasing.
7. Resolved the clock issue using `timedatectl set-ntp true` and confirmed synchronization via `timedatectl` before re-running `apt update` and `apt install` successfully.
8. Wrote a custom Bash script that logs a periodic "heartbeat" entry to a log file, and created a corresponding systemd unit file to run it as a managed service.
9. Reloaded the systemd daemon, started and enabled the custom service, and confirmed it was actively logging output as expected.
10. Rebooted the server and reconnected via SSH to verify the custom service resumed automatically and continued logging without manual intervention, confirming persistence was correctly configured.

### What I found

The nginx connectivity issue was not a service failure — nginx was genuinely running — but a firewall configuration gap, since Lab 1's default-deny policy only accounted for SSH and had no rule for a service installed afterward. This reinforced a practical operational pattern: every new service added to a hardened system requires its own explicit firewall consideration; hardening is not a one-time setup step but an ongoing part of service deployment.

The `apt update` failure was a good example of a misleading-looking error resolving to a root cause outside the area initially suspected. The specific phrasing "not valid yet," combined with a stated time duration until validity, pointed to clock skew rather than a package or repository problem — recognizing that phrasing pattern allowed for a fast, targeted diagnosis rather than troubleshooting the wrong layer of the system.

Writing and deploying the custom systemd service demonstrated that service management extends beyond operating pre-built software — the same `systemctl` lifecycle (start, stop, enable, status) applies identically to a script I authored myself, and `enable` combined with `Restart=always` in the unit file was what genuinely ensured persistence across a full reboot, not merely a successful manual start.

### Screenshot(s)

[Browser showing nginx connection timeout before the firewall fix]
[`ufw status` showing only SSH allowed]
[Successful nginx default page after allowing the HTTP rule]
[`timedatectl` output showing synchronized clock]
[`systemctl status heartbeat` showing the custom service active]
[Heartbeat log entries continuing after a full reboot]

### Why it matters 

This lab reinforced that a default-deny firewall posture (established in Lab 1) requires deliberate, ongoing maintenance as new services are introduced — a hardened baseline can still silently block legitimate functionality if firewall rules aren't updated alongside new deployments, which is a realistic operational trade-off between security and availability that real system administrators manage continuously, not just once at setup.

The clock synchronization issue also has a genuine security dimension beyond just package installation: accurate system time is a prerequisite for validating cryptographic signatures (including the repository release files that failed here), and is equally critical for reliable log timestamps, certificate validity checks, and time-based authentication tokens — a skewed clock can silently undermine multiple unrelated security mechanisms at once.

Building a custom systemd service also reinforced the least-privilege thinking from Lab 2: the unit file explicitly specifies which user the service runs as, a decision with direct security implications, since a service misconfigured to run as root when it doesn't need to would unnecessarily broaden the impact of any bug or compromise within that script.

### Next steps

- When deploying any new service going forward, treat updating firewall rules as a required step of the deployment itself, not an afterthought triggered by a connectivity failure.
- Periodically verify system clock synchronization on any server, especially one that may have been powered off for extended periods, before troubleshooting other symptoms that could stem from clock skew.
- Next step: extend the custom systemd service to write logs to a centralized/rotated location rather than a single flat file, and explore `journalctl`-based log review for services managed this way.
