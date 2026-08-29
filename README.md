# Security Labs

[![License](https://img.shields.io/github/license/Utkarsh464/labs)](LICENSE)
[![Labs](https://img.shields.io/badge/labs-11-blue)](#lab-index)
[![Target](https://img.shields.io/badge/target-Metasploitable%202%20%7C%20WebGoat-critical)](machines/metasploitable2/)

Penetration testing labs conducted in an isolated virtual network. Every lab documents the full workflow — recon, exploitation, troubleshooting, and remediation — against intentionally vulnerable targets.

> All activity was performed in an isolated lab environment for educational purposes.

---

## Lab Index

| Lab                                                                                             | Target                        | Technique                                                  | Status   |
| ----------------------------------------------------------------------------------------------- | ----------------------------- | ---------------------------------------------------------- | -------- |
| [vsFTPd 2.3.4 Backdoor](machines/metasploitable2/vsftpd-2.3.4-backdoor/README.md)               | Metasploitable 2              | Metasploit — CVE-2011-2523                                 | Complete |
| [SSH Login Brute-Force](machines/metasploitable2/ssh-login-bruteforce-msfadmin/README.md)       | Metasploitable 2              | Metasploit — ssh_login auxiliary                           | Complete |
| [Telnet Login Brute-Force](machines/metasploitable2/telnet-login-bruteforce-msfadmin/README.md) | Metasploitable 2              | Metasploit — telnet_login auxiliary                        | Complete |
| [PHP CGI Argument Injection](machines/metasploitable2/php-cgi-arg-injection-www-data/README.md) | Metasploitable 2              | Metasploit — CVE-2012-1823 (www-data)                      | Complete |
| [Samba usermap_script Execution](machines/metasploitable2/samba-usermap-script-root/README.md)  | Metasploitable 2              | Metasploit — CVE-2007-2447 (root)                          | Complete |
| [UnrealIRCd Backdoor](machines/metasploitable2/unrealircd-backdoor-root/README.md)              | Metasploitable 2              | Metasploit — CVE-2010-2075 (root)                          | Complete |
| [DVWA Brute Force](web-apps/dvwa/dvwa-brute-force/README.md)                                    | DVWA on Metasploitable 2      | Hydra — HTTP form brute-force                              | Complete |
| [DVWA Reflected XSS](web-apps/dvwa/dvwa-reflected-xss/README.md)                                | DVWA on Metasploitable 2      | Reflected XSS — blacklist bypass                           | Complete |
| [WebGoat SSRF](web-apps/webgoat/webgoat-ssrf/README.md)                                         | WebGoat (Docker)              | Burp Suite — SSRF parameter tampering                      | Complete |
| [dir-brute Directory Enumeration](web-apps/dir-brute/README.md)                                 | Local test server             | Custom Python brute-forcer — status filtering, save, crawl | Complete |
| [Redeemer](machines/redeemer/README.md)                                                         | HTB Redeemer (Starting Point) | Redis unauthenticated access — flag in keystore            | Complete |

---

## Methodology

| Phase                 | Approach                                                                                                                                                       |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Reconnaissance**    | Nmap service discovery, port scanning, OS fingerprinting. Identify every open port and banner before selecting a target.                                       |
| **Enumeration**       | Version-to-CVE mapping. Research service versions, review exploit behavior, identify prerequisites — LHOST reachability, session cookies, module quirks.       |
| **Exploitation**      | Configure and run the exploit. When it breaks — and it will — troubleshoot methodically: verify the network path, check module options, read the error output. |
| **Post-Exploitation** | Validate access level (`whoami`, `id`, `uname -a`), navigate the filesystem, capture evidence.                                                                 |
| **Remediation**       | Map findings to actionable controls: version upgrades, configuration hardening, network segmentation, detection rules.                                         |

---

## Key Takeaways

- **Wrong `LHOST` is the most common failure.** Reverse payloads fail silently if the target can't reach your listener. Always verify the network path before running an exploit.
- **Session cookies are everything in web app testing.** Without injecting the `PHPSESSID`, every Hydra brute-force attempt hits a login redirect — valid credentials or not.
- **Blacklist filtering does not solve XSS.** Removing only lowercase `<script>` is trivial to bypass with uppercase `<SCRIPT>`.
- **`ForceExploit` bypasses a safety check, not a technical barrier.** Only use it after confirming the target is vulnerable through other means.
- **Wordlist size is a time budget.** 18.6 million entries at 2,188 tries/min is ~6 days. Start small, confirm syntax, then scale.
- **Metasploit handles legacy protocol compatibility better than CLI tools.** Modern OpenSSH rejects `ssh-rsa` and `ssh-dss`, but Metasploit's built-in client connects without issue.
- **`ssh_login` and `telnet_login` use different brute-force strategies.** The former uses paired `USERPASS_FILE` entries; the latter tries every combination of `USER_FILE` × `PASS_FILE`. Wordlist strategy must match the module.
- **Credential reuse magnifies impact.** The same `msfadmin:msfadmin` works across SSH, Telnet, and system console — one weak password compromises multiple services.
- **Unvalidated URL parameters enable SSRF.** Replacing a filename with an external URL or internal metadata endpoint gives the attacker control of the server's request direction.

---

## Lab Topology

| Node                      | Role         | Address               |
| ------------------------- | ------------ | --------------------- |
| Linux pentest workstation | Attacker     | `192.168.122.1`       |
| Metasploitable 2          | Target       | `192.168.122.229`     |
| WebGoat (Docker)          | Target       | `192.168.122.84:8080` |
| Virtual network           | Isolated NAT | `192.168.122.0/24`    |

**Tools:** Nmap, Zenmap, Metasploit Framework, Hydra, Burp Suite, Netcat, Firefox

---

## Repository Layout

```text
labs/
├── machines/
│   └── metasploitable2/
│       ├── README.md
│       ├── vsftpd-2.3.4-backdoor/
│       ├── ssh-login-bruteforce-msfadmin/
│       ├── telnet-login-bruteforce-msfadmin/
│       ├── php-cgi-arg-injection-www-data/
│       ├── samba-usermap-script-root/
│       └── unrealircd-backdoor-root/
├── web-apps/
│   ├── dvwa/
│   │   ├── dvwa-brute-force/
│   │   └── dvwa-reflected-xss/
│   ├── webgoat/
│   │   └── webgoat-ssrf/
│   └── dir-brute/
├── .gitignore
├── LICENSE
└── README.md
```

---

## Legal Notice

The content in this repository is for authorized lab use only. Do not run these techniques against systems you do not own or do not have explicit permission to test.
