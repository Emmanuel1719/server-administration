# Hybrid Windows/Linux Server Deployment & Hardening

A hybrid cloud/on-premises server infrastructure project — deploying, configuring and securing both a Windows Server and a Linux server to host web applications handling sensitive client data, as part of my Server Administration and Security module.

## Scenario

Acting as sysadmin for a fictional startup ("Highway-2005"), I was responsible for standing up two server environments to support web-based applications with sensitive client information — prioritising security, reliability, and realistic infrastructure choices over a purely theoretical setup.

## Architecture

- **Linux server** — Ubuntu Server 20.04 LTS, hosted on **Google Cloud Platform**, chosen for cost-effective, flexible, remotely-accessible hosting suited to lightweight web apps
- **Windows server** — Windows Server 2019, deployed locally via **VMware Workstation**, chosen to reflect how many organisations still keep internal-only roles (DNS, file sharing) on-premises rather than in the cloud

This hybrid split was a deliberate design decision to reflect how infrastructure is actually built in the real world — not everything moves to the cloud, and not everything stays on-prem.

## What I Built

**Windows Server:**
- DNS server role with a custom internal zone (`highway-2005.local`), A records for `www`/`mail`, and a CNAME alias for `ftp`
- File sharing with NTFS permissions (admins: full control, standard users: read/write) — tested from a separate client machine on the virtual network
- IIS for static web hosting, plus XAMPP running alongside it (reconfigured to a different port to avoid conflicts) for dynamic PHP/MySQL content
- Windows Defender Firewall locked down to only HTTP, HTTPS and RDP
- Local Users and Groups audited to ensure only the right accounts had admin rights

**Linux Server (GCP):**
- Full LAMP stack (Apache, MySQL, PHP) via `apt`, serving from `/var/www/html`
- A MySQL database with a dedicated low-privilege application user (`SELECT`/`INSERT` only) rather than using root, to limit blast radius if credentials were ever compromised
- Ownership of deployed PHP files correctly set to `www-data` to avoid permission issues
- **UFW** firewall restricting traffic to SSH, HTTP and HTTPS only
- SSH hardened by moving off the default port (22 → 2222) and disabling direct root login via `sshd_config`

**Backup & Recovery:**
- Windows: a PowerShell script using `Compress-Archive` to produce date-stamped backups of the shared folder
- Linux: a Bash script using `tar` and `mysqldump` to back up the web directory and database, scheduled via `cron`

## Key Decisions & Trade-offs

- Chose a **low-privilege database user** over root/admin access for the web app — a small change with a real security payoff if the app were ever compromised
- Prioritised **SSH hardening** (non-default port, no root login) as a cheap, high-impact defence against brute-force attempts
- Was honest about limitations rather than glossing over them: HTTPS/SSL wasn't implemented on the Windows side due to time constraints, and neither environment was load-tested under real-world traffic since both were virtualised

## What I'd Do Differently at Production Scale

- Automate the setup with **Ansible or Terraform** instead of manual configuration, for consistency and repeatability
- Add **centralised logging/monitoring** (ELK stack or Graylog)
- Enable **HTTPS everywhere** with proper certificates (e.g. Let's Encrypt)
- Add **offsite/cloud-based backup redundancy** rather than local-only backups
- Consider additional Linux hardening (Fail2Ban, AppArmor) and MFA for SSH access

## Tools Used

Windows Server 2019, Ubuntu Server 20.04 LTS, IIS, Apache2, XAMPP, MySQL, PowerShell, Bash, UFW, SSH, Google Cloud Platform, VMware Workstation

## Note

This was a university coursework project against virtualised/cloud lab environments for a fictional company, not a real production deployment.
