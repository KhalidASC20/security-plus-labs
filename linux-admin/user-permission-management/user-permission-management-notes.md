# Linux User, Group, and Permission Management

**Date:** September 13, 2026

**Tools used:** Ubuntu Server, OpenSSH, ACL utilities (setfacl/getfacl), visudo

**Environment:** Personal Ubuntu Server VM, administered remotely via SSH from my own laptop

### Objective

Implement multi-user account management, shared directory permissions, access control lists (ACLs), and least-privilege administrative access on a Linux server — core day-to-day responsibilities of a Linux system administrator.

### Steps taken

1. Created multiple user accounts (`lisa`, `maggie`, `bart`) and a dedicated group (`team-simpson`) using `adduser` and `groupadd`.
2. Added users to the group using `usermod -aG`, deliberately using the append flag rather than `-G` alone to avoid overwriting existing group memberships.
3. Created a shared directory (`/srv/project-files`), assigned group ownership with `chown root:team-simpson`, and applied `chmod 770` to grant full access to the owner and group while denying access entirely to all other users.
4. Verified the permission boundary directly by switching to `lisa`'s account (`su - lisa`) and confirming successful file creation inside the shared directory, consistent with her group membership.
5. Used `setfacl` to grant a third user, `bart`, read and execute access to the same directory without adding him to `team-simpson` or altering the directory's base permissions — demonstrating access control lists as a tool for handling exceptions outside the standard owner/group/other permission model.
6. Verified the ACL using `getfacl`, confirming the additional user-specific permission entry was applied correctly alongside the base permissions.
7. Used `visudo` to safely create a scoped sudo rule in `/etc/sudoers.d/`, granting `maggie` permission to restart the SSH service specifically, with no password prompt, and no broader administrative access.
8. Verified the scoped privilege by logging in as `maggie` and successfully restarting the SSH service, confirmed via the updated "Active since" timestamp in `systemctl status ssh` — while confirming the service restart did not interrupt my own already-established SSH session.
9. Confirmed the privilege boundary held by attempting an unauthorized action as `maggie` (`sudo cat /etc/shadow`), which was correctly denied.

### What I found

Standard Unix permissions (owner/group/other via `chmod`) proved sufficient for the primary shared-directory use case, but access control lists were necessary to handle a legitimate exception — granting one additional user narrower access without restructuring group membership or loosening the directory's base permissions. This reflects a common real-world tension in permission design: balancing simplicity (fewer groups, broader rules) against precision (exact access per user).

The sudo scoping exercise directly demonstrated the least-privilege principle in practice rather than just conceptually: `maggie` was able to perform exactly one administrative action and was correctly denied everything else, including a sensitive file (`/etc/shadow`) that a fully-privileged sudo user could access. This is a meaningfully different (and more secure) outcome than the common but weaker practice of adding a user to the `sudo` group wholesale.

Restarting the SSH service while connected via SSH also clarified an important operational distinction: the service *listener* was restarted, but my already-established session was unaffected, since existing connections are handled independently of the listener process accepting new ones.

### Screenshot(s)
Successfully added two new users (Lisa and Maggie) to a group (team-simpson)

![`groups lisa` and `groups maggie` output confirming group membership](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/user-permission-management/added-users-to-group.png?raw=true)

Created read and write priviliges for users within group team-simpson

![lisa touch file within group](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/user-permission-management/proof-of-group-privelege.png?raw=true)

Created a user outside of the group (bart) who cannot access files controlled by group team-simpson

![bart can't access project-files](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/user-permission-management/bart-edit-denial.png?raw=true)

Created an Access Control List allowing Lisa to restart SSH without needing a password

![`getfacl /srv/project-files` output showing bart's ACL entry](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/user-permission-management/lisa-acl-restart-nopasswrd.png?raw=true)

Using Lisa's account I was able to restart SSH services with no password

![maggie successfully restarting the SSH service with no password prompt](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/user-permission-management/lisa-restarting-ssh.png?raw=true)

Lisa's account rightfully does not have access to any other file or service

![maggie denied access to /etc/shadow](https://github.com/KhalidASC20/security-plus-labs/blob/main/linux-admin/user-permission-management/Lisa-shadow-denial.png?raw=true)

### Why it matters

This lab is a direct, hands-on application of the **least-privilege principle** — a concept covered extensively in Security+ study but rarely practiced hands-on at that stage. Rather than granting broad administrative access for convenience, each user in this lab was granted exactly the access their role required: full access for group members collaborating on shared files, narrow read-only access for an individual exception, and a single, specific administrative capability for a user who needed it — nothing more.

The use of `visudo` specifically (rather than editing sudoers files directly) also reinforces a real operational safety practice: validating configuration syntax before it takes effect, preventing a typo from locking out administrative access entirely — a direct, lower-risk parallel to the SSH configuration lockout encountered in the previous lab.

### Next steps

- Apply the same scoped-sudo pattern to any future service accounts on this server, rather than defaulting to full sudo access for convenience.
- Periodically audit `/etc/sudoers.d/` and group memberships as the number of users grows, to ensure access still matches actual need.
- Next step: move into service and package management, applying these same access-control principles to services running on the server.
