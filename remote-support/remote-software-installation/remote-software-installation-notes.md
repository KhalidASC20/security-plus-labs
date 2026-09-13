# Remote Software Installation and UAC Secure Desktop Elevation (7-Zip)

**Date:** August 2026
**Tools used:** AnyDesk, Windows UAC
**Environment:** My own laptop (technician role) remotely connected to my own desktop (client role), simulating a help desk support session

### Objective
Simulate a remote IT support scenario requiring installation of third-party software on a remote endpoint, specifically to observe and successfully navigate a genuine UAC elevation prompt — as a direct comparison to the Print Spooler exercise, where no prompt appeared due to a Microsoft auto-elevation exception.

### Steps taken
1. Connected to the desktop from my laptop via an active AnyDesk session.
2. Opened a browser on the desktop (through the remote session) and downloaded the 7-Zip installer from the official site.
3. Ran the downloaded installer.
4. The installer triggered a UAC consent prompt on the desktop, running in Secure Desktop mode.
5. AnyDesk displayed a message indicating it could not automatically interact with the prompt, showing "awaiting authentication — please wait for the remote user to accept the UAC dialog," and the session became temporarily unresponsive to remote input.
6. Physically accessed the desktop directly and clicked "Yes" on the UAC prompt.
7. This action reset/reconnected the AnyDesk session automatically.
8. Reconnected remotely and confirmed 7-Zip had installed successfully.

### What I found
Unlike the Print Spooler service restart (which required no UAC prompt due to a Microsoft auto-elevation exception for signed system tools), installing third-party software triggered a full UAC consent prompt as expected, since installer executables are not on that same trusted allowlist and must explicitly request elevation.

More significantly, the UAC prompt ran in Windows' **Secure Desktop** mode — an isolated rendering context specifically designed to prevent other software, including remote-control applications, from programmatically interacting with or automatically dismissing elevation prompts. AnyDesk's free version was unable to click through this prompt remotely and instead required the action to be completed directly at the physical machine, after which the session reconnected on its own.

### Screenshot(s)

AnyDesk session showing "awaiting authentication" message
![AnyDesk session showing "awaiting authentication" message](https://github.com/KhalidASC20/security-plus-labs/blob/main/remote-software-installation/anydesk-authorization-for-remote-access.png?raw=true)
UAC consent prompt for the 7-Zip installer
![UAC consent prompt for the 7-Zip installer](https://github.com/KhalidASC20/security-plus-labs/blob/main/remote-software-installation/anydesk-UAC-pompt.jpeg?raw=true)
7-Zip successfully installed, confirmed post-reconnection
![7-Zip successfully installed, confirmed post-reconnection](https://github.com/KhalidASC20/security-plus-labs/blob/main/remote-software-installation/7-zip-running.png?raw=true) 

### Why it matters
This is a direct, hands-on demonstration of a real Windows security control working as intended: Secure Desktop mode exists specifically to prevent malware or unauthorized remote software from silently self-elevating privileges by scripting past a UAC prompt. By experiencing this restriction firsthand — being blocked from remotely accepting the prompt through standard AnyDesk functionality — this exercise validated that the isolation boundary functions even against a legitimate, authorized remote-support tool being used for an authorized purpose.

This also reflects a genuine operational consideration in real IT/help desk work: not all remote support scenarios can be fully resolved without either physical presence or specific tooling/configuration (e.g., enterprise remote-support platforms sometimes have specific permissions or agents configured to interact with Secure Desktop, unlike a default free-tier consumer tool).

### Next steps
- Research what specific configuration or tooling (e.g., paid AnyDesk tiers, alternative remote support software, or specific Windows Group Policy settings) would allow authorized remote interaction with Secure Desktop UAC prompts, and the security trade-offs of enabling that capability.
- Next step: document the full contrast between this exercise and the Print Spooler exercise as a single comparative note — same underlying mechanism (UAC), two different behaviors depending on whether the triggering action is on Microsoft's auto-elevation allowlist.

### Related exam concepts
Domain 3 (Security Architecture — privilege management, Secure Desktop isolation as a security boundary), Domain 4 (Security Operations — remote endpoint management, real-world help desk constraints), Domain 2 (Threats, Vulnerabilities & Mitigations — understanding why Secure Desktop specifically defends against privilege escalation via remote/scripted input). Reflects a genuine, common real-world help desk situation: a remote fix that cannot be fully completed without user/physical interaction, and knowing how to recognize and communicate that limitation.
