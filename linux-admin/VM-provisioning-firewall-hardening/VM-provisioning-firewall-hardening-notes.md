# Linux Server Setup: VM Provisioning, Remote Access, and Firewall Hardening

**Date:** September 11, 2026

**Tools used:** VirtualBox, Ubuntu Server (LTS), OpenSSH, ufw

**Environment:** Personal virtual machine, accessed remotely from my own laptop over my home network

### Objective

Provision a Linux server from scratch in a virtualized environment, configure it for secure remote administration, and apply baseline firewall hardening — the standard first steps taken when standing up any new production server.

### Steps taken

1. Installed VirtualBox and downloaded the latest Ubuntu Server LTS release.
2. Created a new virtual machine (2048 MB RAM, 20 GB disk) and configured its network adapter in **Bridged mode**, allowing the VM to obtain its own independent IP address on the home network rather than sitting behind NAT — a requirement for reaching it remotely as if it were a separate physical device.
3. Installed Ubuntu Server via its text-based installer, creating a non-root user account with sudo privileges during setup and explicitly installing the OpenSSH server package to enable remote access from the outset.
4. Logged in locally for the first time via the VM's console, and verified basic account privileges using `sudo whoami`, confirming the created account had administrative rights.
5. Identified the VM's assigned IP address using `ip addr`, needed to connect to it remotely.
6. From my laptop's terminal, connected to the VM over SSH using that IP address and the account credentials created during installation, establishing remote administrative access to the server for the first time.
7. Configured the `ufw` (Uncomplicated Firewall) service: explicitly allowed the OpenSSH service before enabling the firewall, then enabled `ufw` to enforce a default-deny posture on all other inbound traffic.
8. Verified the firewall configuration using `ufw status`, confirming SSH was allowed and all other inbound traffic was blocked by default.

### What I found

The completed setup resulted in a Linux server reachable only via SSH, with all other inbound ports blocked by a default-deny firewall policy — a secure-by-default baseline configuration. Critically, the OpenSSH allowance was configured **before** enabling the firewall; enabling `ufw` first without an explicit SSH allow rule would have blocked the very connection being used to administer the server, resulting in an immediate lockout with no other configured access path.

This lab also reinforced the practical difference between local (console) access and remote (SSH) access to the same machine — while both ultimately control the same underlying system, they represent genuinely separate access paths, which became directly relevant later when a preserved console session served as the recovery path during a separate SSH authentication issue (documented in a companion write-up).

### Screenshot(s)
VirtualBox settings/configuration

![VirtualBox VM settings showing Bridged Adapter network configuration](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/VM-provisioning-firewall-hardening/VMbox.png?raw=true)

Proof of admin privileges

![Successful `sudo whoami` output confirming admin privileges](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/VM-provisioning-firewall-hardening/Proof-of-admin.png?raw=true)

Successful SSH connection from laptop to VM

![Initial successful SSH connection from laptop to VM](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/VM-provisioning-firewall-hardening/Connecting-to-vmbox-from-ssh.png?raw=true)

Proof of active firewall

![`ufw status` output showing OpenSSH allowed, firewall active](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/VM-provisioning-firewall-hardening/Active-firewall.png?raw=true)

### Why it matters

This lab demonstrates the principle of **secure-by-default configuration**: rather than installing a server and later locking it down, the firewall was configured to deny all inbound traffic except the one explicitly required service, from the very first session. This default-deny approach is a foundational security control — it ensures that any future service accidentally left exposed (a misconfigured web server, an unintended open port) is blocked by default rather than silently reachable.

The ordering of firewall configuration (allow the required service, then enable enforcement) is itself a meaningful operational security lesson: firewall changes on a remote-access-only system carry real risk of self-lockout if applied out of sequence, reinforcing the value of deliberate, verified sequencing over rushing through hardening steps.

### Next steps

- Periodically re-run `ufw status` after installing any new service on this server to confirm no unintended ports are left open.
- Next step: apply the same bridged-network and SSH-first-access pattern to future lab VMs as a standard, repeatable provisioning baseline.
- Continue building on this server for subsequent labs (user/permission management, service management, log analysis) now that baseline remote access and firewall protection are in place.
