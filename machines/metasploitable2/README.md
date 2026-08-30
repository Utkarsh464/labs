# Metasploitable 2

My lab target — an intentionally vulnerable Ubuntu 8.04 VM exposing a dozen+ services on a single host.

## Enumeration

Nmap `-A` against `192.168.122.229`:

| Port             | Service    | Version                                                   |
| ---------------- | ---------- | --------------------------------------------------------- |
| 21/tcp           | FTP        | vsftpd 2.3.4                                              |
| 22/tcp           | SSH        | OpenSSH 4.7p1 Debian 8ubuntu1                             |
| 23/tcp           | Telnet     | Linux telnetd                                             |
| 25/tcp           | SMTP       | Postfix smtpd                                             |
| 53/tcp           | DNS        | ISC BIND 9.4.2                                            |
| 80/tcp           | HTTP       | Apache httpd 2.2.8 (Ubuntu) DAV/2 + PHP/5.2.4-2ubuntu5.10 |
| 111/tcp          | RPC        | rpcbind 2                                                 |
| 139/tcp, 445/tcp | SMB        | Samba 3.0.20-Debian                                       |
| 512/tcp          | exec       | rexec (?)                                                 |
| 513/tcp          | login      | rlogin                                                    |
| 514/tcp          | shell      | rsh (?)                                                   |
| 1099/tcp         | Java RMI   | GNU Classpath grmiregistry                                |
| 1524/tcp         | bindshell  | Metasploitable root shell                                 |
| 2049/tcp         | NFS        | 2-4                                                       |
| 2121/tcp         | FTP        | ProFTPD 1.3.1                                             |
| 3306/tcp         | MySQL      | MySQL 5.0.51a-3ubuntu5                                    |
| 5432/tcp         | PostgreSQL | PostgreSQL DB 8.3.0–8.3.7                                 |
| 5900/tcp         | VNC        | VNC (protocol 3.3)                                        |
| 6000/tcp         | X11        | (access denied)                                           |
| 6667/tcp         | IRC        | UnrealIRCd                                                |
| 8009/tcp         | AJP        | Apache Jserv (Protocol v1.3)                              |
| 8180/tcp         | HTTP       | Apache Tomcat/Coyote JSP engine 1.1                       |

## Labs

| Lab                                                           | Service           | Vulnerability                    | Status   |
| ------------------------------------------------------------- | ----------------- | -------------------------------- | -------- |
| [vsFTPd 2.3.4 Backdoor](vsftpd-2.3.4-backdoor/)               | FTP               | CVE-2011-2523                    | Complete |
| [SSH Login Brute-Force](ssh-login-bruteforce-msfadmin/)       | SSH               | Default credentials              | Complete |
| [Telnet Login Brute-Force](telnet-login-bruteforce-msfadmin/) | Telnet            | Default credentials              | Complete |
| [PHP CGI Arg Injection](php-cgi-arg-injection-www-data/)      | HTTP (Apache/PHP) | CVE-2012-1823                    | Complete |
| [Samba Usermap Script](samba-usermap-script-root/)            | SMB               | CVE-2007-2447                    | Complete |
| [UnrealIRCd Backdoor](unrealircd-backdoor-root/)              | IRC               | CVE-2010-2075                    | Complete |
| [SMTP User Enumeration](smtp-user-enumeration/)               | SMTP              | Metasploit — smtp_enum auxiliary | Complete |

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
- SMB version alone (Samba 3.0.20-Debian) was enough to identify CVE-2007-2447 — the `smb-os-discovery` Nmap script provides the exact version string needed.
- The Samba usermap_script exploit requires no authentication; any anonymous SMB connection can trigger code execution.
- Having multiple root-level exploit paths on one host (vsftpd, Samba, UnrealIRCd) reinforces that defense-in-depth is necessary — patching one service is not enough.
- The UnrealIRCd backdoor (CVE-2010-2075) is a supply-chain attack, same class as vsftpd 2.3.4 — both came from compromised source archives, not code flaws.
- The `check` command in Metasploit is valuable: it registered a test IRC user and verified the backdoor was present before sending the payload.
- Network interface selection matters — `192.168.122.1` (virbr0) was on the same VM bridge, while `192.168.29.176` (wlo1) was on a NAT'd host network.

## Safety Boundary

These labs were conducted in an isolated virtual network against an intentionally vulnerable machine. Do not reproduce these techniques against systems you do not own or lack explicit authorization to test.
