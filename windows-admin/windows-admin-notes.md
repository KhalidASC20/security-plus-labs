# Building a Small Active Directory Domain and Practicing Help-Desk Tasks

**Date:** October 16, 2026

**Tools used:** VirtualBox, Windows Server [2025 / 2022 - verify] Evaluation (Desktop Experience), Active Directory Domain Services, DNS, Group Policy Management, PowerShell

**Environment:** One virtual domain controller (`DC01`) on a private VirtualBox NAT Network (`ADLab`, 10.0.2.0/24), domain `lab.local`. All accounts and passwords are lab-only.

### Objective

Build a small Active Directory domain from scratch and practice the tasks an IT help desk or MSP technician does every day: organizing users and groups, resetting passwords, handling an account lockout, offboarding a departing employee, and enforcing a security setting with Group Policy. The theme is a small law firm, which is the kind of client many MSPs serve.

I built it mainly to get hands-on experience creating my own Active Directory, since it's central to the roles I'm applying for.

---

### Steps taken

**Part 1: Network and virtual machine**
1. Created a VirtualBox **NAT Network** (`ADLab`, 10.0.2.0/24) instead of plain NAT, so that a future client VM could reach the domain controller and still have internet access.
2. Created the VM `DC01`: 4 GB RAM, 2 CPUs, 60 GB disk, attached to `ADLab`. Skipped unattended install so I controlled the setup.

**Part 2: Install Windows Server**
3. Installed Windows Server Standard Evaluation with **Desktop Experience** (the edition without it has no graphical interface), set the Administrator password, and installed VirtualBox Guest Additions. Took a snapshot of the fresh install so I could roll back.

**Part 3: Prepare the server**
4. A domain controller needs a permanent name and address, because every other computer finds it by those. Set a **static IP** (10.0.2.10, gateway 10.0.2.1) and set the server's DNS to itself (127.0.0.1), then renamed the computer to `DC01`:
```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.2.10 -PrefixLength 24 -DefaultGateway 10.0.2.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
Rename-Computer -NewName DC01 -Restart
```

**Part 4: Create the domain**
5. Installed the **Active Directory Domain Services** role and promoted the server to a domain controller, creating a new forest called `lab.local` (DNS installed with it). Set a DSRM recovery password.
6. After the reboot, signed in as `LAB\Administrator` and confirmed with `Get-ADDomain` and `nslookup lab.local`.

**Part 5: Structure: OUs, groups, users**
7. Built an Organizational Unit (OU) tree under a parent `LawFirm` OU, with child OUs for **Staff**, **Groups**, **Computers**, and **Disabled Users** [plus Interns / any others you created - verify].
8. Created three **security groups** (`Attorneys`, `Paralegals`, `Office-Admin`) and users `lisa`, `bart`, and `maggie`, each placed in a group. Every user was set to **change password at next logon**, as in real onboarding.
9. Created another user (`homer`) with PowerShell to practice doing the same tasks from the command line:
```powershell
New-ADUser -Name "Homer Simpson" -SamAccountName homer `
  -Path "OU=Staff,OU=LawFirm,DC=lab,DC=local" `
  -AccountPassword (Read-Host -AsSecureString "Temp password") `
  -ChangePasswordAtLogon $true -Enabled $true
Add-ADGroupMember -Identity Paralegals -Members homer
```
Permissions are meant to go to **groups**, and people go into groups, so access is never granted one person at a time (least privilege).

**Part 6: Help-desk tasks**
10. **Password reset:** reset `bart`'s password and forced a change at next logon.
11. **Account lockout policy:** in the Default Domain Policy, set the account lockout threshold to **5 invalid attempts**. (Password and lockout policies only take effect at the domain level.)
12. **Triggered a lockout** by repeating a failed login against the server:
```
net use \\DC01\IPC$ /user:LAB\bart WrongPassword1
```
After **6 failed attempts**, `Get-ADUser bart -Properties LockedOut` returned `True`. Unlocked the account with `Unlock-ADAccount bart`, and `LockedOut` returned `False`.
13. **Offboarded `maggie`** the way it's done in production: **disabled** the account (not deleted), removed her from her group, moved her to the `Disabled Users` OU, and added a note in the Description field.
```powershell
Disable-ADAccount maggie
Remove-ADGroupMember -Identity Office-Admin -Members maggie -Confirm:$false
Get-ADUser maggie | Move-ADObject -TargetPath "OU=Disabled Users,OU=LawFirm,DC=lab,DC=local"
```

**Part 7: Group Policy**
14. Created and linked a GPO named **"Staff - Screen Lock 10 min"** to the Staff OU. It enables a password-protected screen saver with a 600-second (10 minute) timeout, a common compliance requirement for law firms.
15. Verified it [with Group Policy Modeling / gpresult - verify which you used].

---

### What I found

**1. Group Policy applies by OU, not by person.** The screen-lock GPO is linked to Staff only, so it applies to users in that OU. The domain Administrator isn't in Staff, so it isn't affected. This is why the OU structure matters: it decides who gets which policy.

**2. The lockout worked as designed.** The account locked after repeated failed attempts (6 attempts against a threshold of 5), and it was clearly visible in `LockedOut = True`. That is the same diagnosis a help-desk tech does for a "my account is locked" ticket: confirm it's locked, find out why (a mistyped password, a phone with an old saved password, or an attack), then unlock. Repeated lockouts across many accounts could mean a password-guessing attack, which is worth escalating.

**3. Disable first, delete later.** Deleting an account is permanent and can break the audit trail and access to the person's files. Disabling, removing group memberships, and moving to a separate OU takes away access immediately and keeps it reversible.

**4. A domain controller depends on DNS and a stable address.** That's why it has a static IP and points DNS at itself, and why a client joining the domain later must point its DNS at 10.0.2.10.

---


### Screenshot(s)
[1. NAT Network `ADLab` showing 10.0.2.0/24]
[2. DC01 VM summary: 4 GB RAM, 2 CPUs, 60 GB disk, attached to ADLab]
[3. Server Manager dashboard after first login]
[4. `ipconfig /all`: static IP 10.0.2.10, DNS 127.0.0.1, host name DC01]
[5. `Get-ADDomain` output showing lab.local]
[6. Active Directory Users and Computers with the LawFirm OU tree expanded]
[7. Users in the Staff OU and one user's "Member Of" tab]
[8. Default Domain Policy: account lockout threshold = 5]
[9. The failed `net use` attempts in PowerShell]
[10. bart locked out: `LockedOut = True`]
[11. After `Unlock-ADAccount`: `LockedOut = False`]
[12. maggie disabled and in the Disabled Users OU, with group membership removed]
[13. Group Policy Management: "Staff - Screen Lock 10 min" linked to the Staff OU]
[14. GPO settings (screen saver timeout 600 seconds) and the Modeling report or `gpresult` result]
[Optional: the troubleshooting screens (red ISO icon, grayed-out roles) with the fix]

---

### Why it matters (the security angle)
Active Directory is the identity system behind a large share of small and mid-size offices, and many security controls depend on it being set up correctly.

- **Least privilege:** access is given to groups, and people are added to or removed from groups. This makes onboarding quick, offboarding clean, and access easy to audit.
- **Consistent policy:** a Group Policy applies a security setting (like a screen lock) to a whole OU at once, so it doesn't depend on each user doing it themselves.
- **Account lockout:** limits password-guessing attacks, but it also creates help-desk tickets, so you need to be able to tell a user who mistyped a password from the start of an attack.
- **Safe offboarding:** a departing employee's access should end immediately, without destroying records.
- **Naming:** I used `lab.local` for convenience. Companies normally use a subdomain of a domain they own (for example `ad.firmname.com`).

### How this connects to the cloud
Many small offices now use Microsoft 365, **Entra ID**, and **Intune** instead of, or along with, on-site AD. The concepts carry over directly:

| This lab | Cloud equivalent |
|---|---|
| Active Directory | Entra ID |
| Users and Computers (`dsa.msc`) | Entra / M365 admin center |
| Group Policy (GPOs) | Intune configuration and compliance policies |
| Domain-joined computer | Entra-joined device |
| OUs | Groups and administrative units |

### What I learned
- Creating an Active Directory domain was less daunting than I expected, and the concepts (domain, OU, group, policy) were easy to understand once I built them.
- I understand why least privilege and Group Policy matter: permissions through groups and policies by OU keep things consistent and auditable, which is much safer than one-off changes per person.
- A broken piece is easier to find when you know what each piece depends on. A domain controller depends on a static IP and DNS, and a group picker is a search box, not a list. Several of my problems came from not knowing that, not from something actually being broken.
- Troubleshooting is part of the skill: I documented what went wrong and how I fixed it, as I would in a ticket.

### Limitations and next steps
- This was a **single-server** lab. I did not join a client computer to the domain (the optional Windows 11 client). Next step: add a Windows 11 Enterprise evaluation VM, point its DNS at the domain controller, join it to `lab.local`, and confirm the screen-lock GPO applies when `bart` signs in, using `gpresult /r`.
- Try **delegating control** so a help-desk group can reset passwords only in the Staff OU (least privilege for IT staff).
- Try a **fine-grained password policy** for a different group, and look at logging for lockout events (Event ID 4740).
- Explore how on-site AD syncs to Entra ID with **Entra Connect**.
