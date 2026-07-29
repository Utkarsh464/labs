# Metasploitable 2

My lab target — an intentionally vulnerable Ubuntu 8.04 VM exposing a dozen+ services on a single host.

## Enumeration

Nmap `-A` against `192.168.122.229`:

| Port | Service | Version |
|---|---|---|
| 21/tcp | FTP | vsftpd 2.3.4 |
| 22/tcp | SSH | OpenSSH 4.7p1 Debian 8ubuntu1 |
| 23/tcp | Telnet | Linux telnetd |
| 25/tcp | SMTP | Postfix smtpd |
| 53/tcp | DNS | ISC BIND 9.4.2 |
| 80/tcp | HTTP | Apache httpd 2.2.8 (Ubuntu) DAV/2 + PHP/5.2.4-2ubuntu5.10 |
| 139/tcp, 445/tcp | SMB | Samba smbd |
| 1524/tcp | bindshell | Metasploitable root shell |
| 3306/tcp | MySQL | MySQL 5.0.51a |
| 5432/tcp | PostgreSQL | PostgreSQL DB 8.3.x |

## Labs

| Lab | Service | Vulnerability | Status |
|---|---|---|---|
| [vsFTPd 2.3.4 Backdoor](vsftpd-2.3.4-backdoor/) | FTP | CVE-2011-2523 | Complete |
| [SSH Login Brute-Force](ssh-login-bruteforce-msfadmin/) | SSH | Default credentials | Complete |
| [Telnet Login Brute-Force](telnet-login-bruteforce-msfadmin/) | Telnet | Default credentials | Complete |
| [PHP CGI Arg Injection](php-cgi-arg-injection-www-data/) | HTTP (Apache/PHP) | CVE-2012-1823 | Complete |

## What I Learned

- The same IP that gave root through the FTP backdoor also exposed default SSH credentials. One host, multiple independent exploitation paths.
- Filtering 12+ services down to the one relevant finding is the skill that matters — it carries to any target, not just this VM.
- Old CVEs follow the same workflow as modern ones: recon, version match, payload configure, troubleshoot, validate.
- SSH credential brute-forcing with `ssh_login` finds weak passwords quickly. Metasploitable 2's `msfadmin:msfadmin` is a reminder that default credentials are the easiest vulnerability to find and fix.
- Metasploit's built-in SSH client handles legacy key exchange algorithms that modern OpenSSH clients reject, making it more reliable against old targets.
- Telnet brute-force with `telnet_login` uses a cartesian product strategy (`USER_FILE` × `PASS_FILE`), different from `ssh_login`'s paired approach.
- The same `msfadmin:msfadmin` credential works across SSH, Telnet, and system login — credential reuse multiplies the impact of a single weak password.
- Version fingerprinting (`http_version`) provides precisely the detail needed to match a CVE — Apache version alone was not enough; PHP version was the critical signal.
- CVE-2012-1823 is a configuration injection, not a memory corruption bug. The entire exploitation chain from query string to shell is built on how PHP parses CGI arguments.
- The `php/meterpreter/reverse_tcp` payload does not include a full Meterpreter command set — dropping to `shell` is the expected way to run system commands.

## Safety Boundary

These labs were conducted in an isolated virtual network against an intentionally vulnerable machine. Do not reproduce these techniques against systems you do not own or lack explicit authorization to test.
