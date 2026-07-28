# Metasploitable 2

My lab target — an intentionally vulnerable Ubuntu 8.04 VM exposing 12+ services on a single host.

## Enumeration

Nmap `-A` against `192.168.122.229`: FTP (vsftpd 2.3.4), SSH (OpenSSH 4.7p1), Telnet, SMTP (Postfix), DNS (BIND 9.4.2), HTTP (Apache 2.2.8), SMB (Samba), bindshell 1524, MySQL, PostgreSQL.

## Vulnerabilities Found

| Service | Lab | Vulnerability |
|---|---|---|
| vsftpd 2.3.4 | [vsFTPd Backdoor](vsftpd-2.3.4-backdoor/) | CVE-2011-2523 — supply-chain backdoor |
| SSH | [SSH Login Brute-Force](ssh-login-bruteforce-msfadmin/) | Default credentials — msfadmin:msfadmin |
| HTTP (DVWA) | [DVWA Brute Force](../../web-apps/dvwa/dvwa-brute-force/) | Weak credentials, no rate-limiting |
| HTTP (DVWA) | [DVWA Reflected XSS](../../web-apps/dvwa/dvwa-reflected-xss/) | Reflected XSS, unsafe output handling |

## What I Learned

- The same IP that gave root through the FTP backdoor also served default credentials on the web app. One host, multiple independent exploitation paths.
- DVWA's security levels are useful for comparing weak filtering with proper output encoding in the same application flow.
- Filtering 12+ services down to the one relevant finding is the skill that matters — it carries to any target, not just this VM.
- Old CVEs follow the same workflow as modern ones: recon, version match, payload configure, troubleshoot, validate.
- SSH credential brute-forcing with `ssh_login` finds weak passwords quickly. Metasploitable 2's `msfadmin:msfadmin` is a reminder that default credentials are the easiest vulnerability to find and fix.
- Metasploit's built-in SSH client handles legacy key exchange algorithms that modern OpenSSH clients reject, making it more reliable against old targets.

## Labs

| Lab | Service | Vulnerability | Status |
|---|---|---|---|
| [vsFTPd 2.3.4 Backdoor](vsftpd-2.3.4-backdoor/) | FTP | CVE-2011-2523 | Complete |
| [SSH Login Brute-Force](ssh-login-bruteforce-msfadmin/) | SSH | Default credentials | Complete |
| [DVWA Brute Force](../../web-apps/dvwa/dvwa-brute-force/) | HTTP | Weak credentials | Complete |
| [DVWA Reflected XSS](../../web-apps/dvwa/dvwa-reflected-xss/) | HTTP | Reflected XSS | Complete |

## Safety Boundary

These labs were conducted in an isolated virtual network against an intentionally vulnerable machine. Do not reproduce these techniques against systems you do not own or lack explicit authorization to test.
