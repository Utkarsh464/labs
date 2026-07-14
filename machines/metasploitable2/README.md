# Metasploitable 2

My lab target — an intentionally vulnerable Ubuntu 8.04 VM exposing 12+ services on a single host.

## Enumeration

Nmap `-A` against `192.168.122.229`: FTP (vsftpd 2.3.4), SSH (OpenSSH 4.7p1), Telnet, SMTP (Postfix), DNS (BIND 9.4.2), HTTP (Apache 2.2.8), SMB (Samba), bindshell 1524, MySQL, PostgreSQL.

## Vulnerabilities Found

| Service | Lab | Vulnerability |
|---|---|---|
| vsftpd 2.3.4 | [vsFTPd Backdoor](vsftpd-2.3.4-backdoor/) | CVE-2011-2523 — supply-chain backdoor |
| HTTP (DVWA) | [DVWA Brute Force](dvwa-brute-force/) | Weak credentials, no rate-limiting |

## What I Learned

- The same IP that gave root through the FTP backdoor also served default credentials on the web app. One host, multiple independent exploitation paths.
- Filtering 12+ services down to the one relevant finding is the skill that matters — it carries to any target, not just this VM.
- Old CVEs follow the same workflow as modern ones: recon, version match, payload configure, troubleshoot, validate.

## Labs

| Lab | Service | Vulnerability | Status |
|---|---|---|---|
| [vsFTPd 2.3.4 Backdoor](vsftpd-2.3.4-backdoor/) | FTP | CVE-2011-2523 | Complete |
| [DVWA Brute Force](dvwa-brute-force/) | HTTP | Weak credentials | Complete |

## Safety Boundary

These labs were conducted in an isolated virtual network against an intentionally vulnerable machine. Do not reproduce these techniques against systems you do not own or lack explicit authorization to test.
