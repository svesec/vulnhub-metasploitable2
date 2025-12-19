# VulnHub – Metasploitable2 (Lab)

## Table of Contents
- General Info  
- Objectives  
- Enumeration  
- Exploitation  
- Initial Shell  
- Privilege Escalation  
- Post-Exploitation Proof  
- Cleanup  
- Key Takeaways  
- Artifacts (masked)  
- Status

## General Info
**Machine:** Metasploitable2 (lab)  
**Platform:** VulnHub / local VM image  
**Author:** SveSec  
**Scope:** non-destructive lab testing — only masked artifacts and screenshots are published  
**Status:** PARTIAL — Work in progress (Part 1: Recon & SMB evidence). Not a final report.

## Objectives
- Produce reproducible, masked reconnaissance outputs for review.  
- Demonstrate SMB findings (shares and permissions) with minimal, professional artifacts.  
- Prioritize low-noise vectors (FTP, SMB, NFS) and prepare safe next steps for lab exploitation.

## Enumeration
Multiple services discovered: FTP (vsftpd 2.3.4), SSH, Telnet, HTTP/Apache, Tomcat/AJP, RPC/NFS, SMB (smbd 3.x), databases (MySQL/Postgres), VNC. Key observation: SMB guest/account mapping and a `tmp` share reported as **READ and WRITE** — high priority for initial, low-noise collection and validation.

![Figure 1 — Nmap summary](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/01_nmap_summary.png)  
*Figure 1 — high-level Nmap service summary (masked).*

![Figure 2 — Nmap services (masked)](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/02_nmap_services.png)  
*Figure 2 — detailed Nmap services excerpt (masked). Shows RPC/NFS, Samba, Apache versions used for prioritization.*

SMB discovery shows available shares and guest-mapping behavior. `tmp` is highlighted by smbmap output as READ/WRITE in this environment; this makes it the primary artifact collection target for Part 1.

![Figure 3 — smbclient shares listing (masked)](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/03_smb_shares_terminal.png)  
*Figure 3 — smbclient listing of shares (masked).*

![Figure 4 — smbmap tmp permissions (masked)](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/04_smbmap_tmp.png)  
*Figure 4 — smbmap output showing `tmp` with READ and WRITE (masked).*

## Exploitation
**Plan (lab-only, non-destructive, reversible):**  
- **Priority 1:** Validate anonymous FTP / vsftpd behavior (read-only checks, no destructive actions).  
- **Priority 2:** Enumerate SMB `tmp` contents (read and identify candidate artifacts for offline analysis). Upload tests only if fully authorized, reversible and logged.  
- **Priority 3:** Check NFS exports for readable/writable mounts to support local analysis for escalation vectors.  
- **Priority 4:** Enumerate Tomcat/HTTP endpoints for manager pages or upload opportunities — secondary, only after SMB/NFS checks.

![Figure 5 — tmp share listing (masked)](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/05_smb_tmp_ls.png)  
*Figure 5 — example listing of `tmp` share (masked).*

## Commands used (examples — include in report as fenced code block)
    # basic discovery / enumeration (run in lab)
    sudo nmap -sC -sV -p- <TARGET_IP>

    # SMB discovery & listing (anonymous)
    smbclient -L //<TARGET_IP> -N
    smbmap -H <TARGET_IP>

    # list tmp share contents (anonymous)
    smbclient //<TARGET_IP>/tmp -N -c 'ls'

    # full listing/allinfo (if needed, for local analysis only)
    smbclient //<TARGET_IP>/tmp -N -c 'allinfo'

## Initial Shell
No interactive shell is documented in this Part 1. Any shell proof will be covered in Part 2 if obtained under lab rules; public proofs will be minimal and redacted (example: `id`, small sanitized directory listings).

## Privilege Escalation
Planned steps (post-foothold and validated):  
- Local enumeration for SUID binaries and writable configuration files.  
- Inspect mounted exports and recovered config files for credentials or keys.  
- Escalation steps will be documented only after verification; redact any secrets in public artifacts.

## Post-Exploitation Proof
Not included in Part 1. When applicable, proofs will be minimal, redacted, and aligned with lab rules (no full flags or sensitive dumps in public).

## Cleanup
- Any temporary uploads or test artifacts must be removed and cleanup steps logged.  
- Raw, unmasked outputs are retained locally under `evidence/raw/` and are **not** committed to the repo.

## Key Takeaways
- High-value, low-effort vectors: anonymous FTP and SMB `tmp` (READ/WRITE).  
- Next immediate task: validate `tmp` contents offline, extract candidate files locally for analysis (do not publish originals). Prepare Part 2 with safe exploitation steps and concise findings.

## Artifacts (masked)
Files are in `docs/` (relative links):
- `full_scan_masked.txt`  
- `smb_shares_metasploitable_masked.txt`  
- `smbclient_tmp_metasploitable_masked.txt`  
- `smbmap_metasploitable_masked.txt`  *(confirm tmp = READ, WRITE before Part 2)*  
- `smbclient_tmp_allinfo_5043_metasploitable_masked.txt`

## Status
Part 1 complete — Reconnaissance and SMB evidence collected.

## Part 2 — Exploitation, Initial Shell & Privilege Escalation

### Exploitation — distccd Remote Command Execution

Service enumeration confirmed an exposed distccd service on TCP/3632. The service was identified as distccd v1 ((GNU) 4.2.4), a known vulnerable version allowing unauthenticated remote command execution.

The exploitation was performed using Metasploit’s official module to ensure protocol-correct interaction and full reproducibility.

- Service: distccd v1
- Port: 3632/tcp
- Vulnerability: Unauthenticated Remote Command Execution
- Exploit module: exploit/unix/misc/distcc_exec
- Payload: cmd/unix/reverse

Successful exploitation resulted in a reverse command shell as a low-privileged system user.

Figure 6 — Initial shell via distccd (masked)  
![Initial distccd shell](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/07_distcc_initial_shell.png)

---

### Initial Shell — Proof of Access

The obtained shell was validated using minimal proof commands to confirm execution context and target identity. The shell runs under a low-privileged account, which is expected behavior for this service.

Validated commands:
- whoami
- id
- hostname

Results confirm controlled command execution on the target system without destructive actions.

---

### Privilege Escalation — SUID Binary Abuse (nmap)

Local enumeration of SUID binaries revealed several executables with elevated privileges. Among them, `/usr/bin/nmap` was identified as a high-confidence privilege escalation vector due to its legacy interactive mode and SUID configuration.

Technique:
- Vector: SUID binary abuse
- Binary: /usr/bin/nmap
- Method: GTFOBins — interactive mode

The interactive mode of nmap allows execution of system commands that inherit the effective UID of the binary. Since nmap is SUID-root in this environment, invoking a shell from interactive mode yields root privileges.

Figure 7 — Root shell via SUID nmap abuse (masked)  
![Root via nmap SUID](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/08_nmap_suid_root.png)

The resulting shell demonstrates effective UID 0 (root), confirming full privilege escalation.

---

### Additional Finding — SSH Key Material via NFS (Non-exploitable)

During offline analysis of exposed NFS data, SSH key material was identified under the msfadmin home directory. While the presence of private keys represents a security misconfiguration, SSH key-based authentication was not exploitable in the current server configuration.

This finding is documented as an informational issue rather than an exploitation path.

Figure 8 — Exposed SSH key material via NFS (masked)  
![SSH key listing](https://raw.githubusercontent.com/svesec/vulnhub-metasploitable2/main/assets/screenshots/06_foothold_ssh_key_listing.png)
---

### Post-Exploitation Proof

Post-exploitation actions were limited strictly to identity validation commands to confirm privilege level and system ownership. No persistence mechanisms, data exfiltration, or configuration changes were performed.

Validated as root:
- whoami → root
- id → euid=0(root)
- hostname → metasploitable

---

### Cleanup

No files were modified on the target system. No persistence or scheduled tasks were created. All testing was conducted in-memory or via ephemeral shells. The lab environment can be safely reset without residual impact.

---

### Status
- Initial access achieved via distccd RCE
- Privilege escalation achieved via SUID nmap
- Proof artifacts collected and masked
