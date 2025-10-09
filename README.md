VulnHub – Metasploitable2 (Lab)
Table of Contents
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
- Figures (inline)
- Status

General Info
Machine: Metasploitable2 (lab)
Platform: VulnHub / local VM image
Author: SveSec
Scope: non-destructive lab testing — publishing only masked artifacts and screenshots
Status: PARTIAL — Work in progress (this file is Part 1: Recon & SMB evidence). This is not a final report.

Objectives
- Produce reproducible, masked reconnaissance outputs for review.
- Demonstrate SMB findings (shares and permissions) with minimal, professional artifacts.
- Prioritize low-noise vectors (FTP, SMB, NFS) and prepare safe next steps for lab exploitation.

Enumeration
Multiple services discovered: FTP (vsftpd 2.3.4), SSH, Telnet, HTTP/Apache, Tomcat/AJP, RPC/NFS, SMB (smbd 3.x), databases (MySQL/Postgres), VNC.
Key observation: SMB guest/account mapping and a `tmp` share that is reported as READ and WRITE — high priority for initial, low-noise collection and validation.

![Figure 1 — Nmap summary](../assets/screenshots/01_nmap_summary.png)
*Figure 1 — high-level service summary (masked).*

SMB discovery shows available shares and guest-mapping behavior. `tmp` is highlighted by smbmap output as READ/WRITE in this environment; this makes it the primary artifact collection target for Part 1.

![Figure 2 — smbclient shares listing (masked)](../assets/screenshots/03_smb_shares_terminal.png)
*Figure 2 — smbclient listing of shares (masked).*

![Figure 3 — smbmap tmp permissions (masked)](../assets/screenshots/04_smbmap_tmp.png)
*Figure 3 — smbmap output showing `tmp` with READ and WRITE (masked).*

Exploitation
Plan (lab-only, non-destructive, reversible):
- Priority 1: Validate anonymous FTP and vsftpd behavior (read-only checks, no destructive actions).
- Priority 2: Enumerate SMB `tmp` contents (read and identify candidate artifacts for offline analysis). Upload tests only if fully authorized, reversible and logged.
- Priority 3: Check NFS exports for readable/writable mounts to support local analysis for escalation vectors.
- Priority 4: Enumerate Tomcat/HTTP endpoints for manager pages or upload opportunities — treat as secondary and only after SMB/NFS checks.

![Figure 4 — tmp share listing (masked)](../assets/screenshots/05_smb_tmp_ls.png)
*Figure 4 — example listing of tmp share (masked).*

Initial Shell
No interactive shell is documented in this Part 1. Any shell proof will be covered in Part 2 if obtained under lab rules; public proofs will be minimal and redacted (example: `id`, small sanitized directory listings).

Privilege Escalation
Planned steps (post-foothold and validated):
- Local enumeration for SUID binaries and writable configuration files.
- Inspect mounted exports and recovered config files for credentials or keys.
- Only document escalation steps after verification; redact any secrets in public artifacts.

Post-Exploitation Proof
Not included in Part 1. When applicable, proofs will be minimal, redacted, and aligned with lab rules (no full flags or sensitive dumps in public).

Cleanup
- Any temporary uploads or test artifacts must be removed and cleanup steps logged.
- Raw, unmasked outputs are retained only locally under `evidence/raw/` and are not committed to the repository.

Key Takeaways
- High-value, low-effort vectors: anonymous FTP and SMB `tmp` (READ/WRITE).  
- Next immediate task: validate `tmp` contents offline, extract candidate files locally for analysis (do not publish originals). Prepare Part 2 with safe exploitation steps and concise findings.

Artifacts (masked) — relative links (files exist under docs/)
- full_scan_masked.txt
- smb_shares_metasploitable_masked.txt
- smbclient_tmp_metasploitable_masked.txt
- smbmap_metasploitable_masked.txt
- smbclient_tmp_allinfo_5043_metasploitable_masked.txt

Figures (inline, adjacent to related text above)
- ../assets/screenshots/01_nmap_summary.png  — Nmap summary (Figure 1)
- ../assets/screenshots/03_smb_shares_terminal.png — smbclient shares (Figure 2)
- ../assets/screenshots/04_smbmap_tmp.png — smbmap (Figure 3)
- ../assets/screenshots/05_smb_tmp_ls.png — tmp listing (Figure 4)

Reviewer Notes
- This is a partial, work-in-progress document. Only masked artifacts and the images above are published. Do not publish unmasked outputs.
- Confirm that `smbmap_metasploitable_masked.txt` shows `tmp` as READ and WRITE before proceeding to Part 2.
- If you prefer a different visual layout (alternate: paragraph → image → paragraph → image consistently), specify which sections to alternate and I will produce the final Markdown with exact paragraph-image pairing.

Status
- Ready to be saved as docs/REPORT_PART1_Recon_and_SMB.md. This file intentionally omits procedural masking commands; masking was completed offline and masked copies are in docs/.
- After your confirmation I will commit this as Part 1 (single file) and then prepare Part 2 when `tmp` is validated.

End of Part 1 — Recon & SMB evidence (partial)

