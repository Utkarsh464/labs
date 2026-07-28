# SSH Login Brute-Force — msfadmin Credential Discovery

This lab documents SSH credential brute-forcing against Metasploitable 2 using Metasploit's `ssh_login` auxiliary module. The activity was performed only against an intentionally vulnerable virtual machine for educational purposes.

## Objective

Identify valid SSH credentials on Metasploitable 2 using a password wordlist, establish a session, and validate user-level access.

## Lab Environment

### Attacker Machine

- Linux penetration testing workstation
- Tools used: Metasploit Framework
- Attacker IP: `192.168.122.1`

### Target Machine

- Target: Metasploitable 2
- Target IP: `192.168.122.229`
- Vulnerable service: SSH (`OpenSSH 4.7p1 Debian 8ubuntu1`)
- SSH port: `22/tcp`
- Discovered credentials: `msfadmin:msfadmin`

### Network

- Isolated virtual lab network: `192.168.122.0/24`
- No internet-facing systems were tested.

## Screenshots

![Metasploit ssh_login module with successful credential discovery](screenshots/01-msfconsole-ssh-login-success.png)

*Metasploit console showing `ssh_login` auxiliary module configuration, successful `msfadmin:msfadmin` credential discovery, and post-exploitation `whoami` validation.*

## Enumeration

### Nmap Service Scan

Previous scans against the target identified SSH on port `22/tcp`:

| Port | Service | Version |
| --- | --- | --- |
| `22/tcp` | SSH | OpenSSH 4.7p1 Debian 8ubuntu1 |

### SSH Version

The Metasploit `ssh_version` scanner confirmed the OpenSSH version, which is relevant for identifying any SSH-specific vulnerabilities. In this lab, the exploitation path was credential brute-forcing rather than an SSH protocol vulnerability.

## Vulnerability Identification

The target runs `OpenSSH 4.7p1` with default credentials still enabled. Metasploitable 2 ships with the `msfadmin` user account set to the password `msfadmin`. This is a documented default credential for this VM.

The vulnerability is not in the SSH protocol software — OpenSSH 4.7p1 on its own is not the flaw. The weakness is credential management: a predictable username and weak password left unchanged in a deployed system.

## Exploitation

### Module Selection

The `ssh_login` auxiliary module performs credential brute-forcing against SSH:

```text
search ssh_login
```

Result:

```text
0  auxiliary/scanner/ssh/ssh_login  normal  No  SSH Login Check Scanner
```

Selected with:

```text
use 0
```

### Module Options

The module was configured with:

```text
set RHOSTS 192.168.122.229
set USERPASS_FILE /tmp/mysql_wordlist.txt
```

The `USERPASS_FILE` format is space-separated username and password pairs, one pair per line. The wordlist was populated with common Metasploitable 2 credentials.

### Credential Discovery

The module immediately found valid credentials:

```text
[+] Success: 'msfadmin:msfadmin'
```

The success output included the full `id` and `uname -a` output, confirming both the credential validity and the target identity.

### Session Interaction

A session was opened but the first attempt to interact via direct SSH failed:

```text
ssh msfadmin@192.168.122.229
Unable to negotiate with 192.168.122.229 port 22: no matching host key type found.
Their offer: ssh-rsa, ssh-dss
```

This is a client-side compatibility issue — the modern OpenSSH client disabled `ssh-rsa` and `ssh-dss` by default due to cryptographic weaknesses, while the old Metasploitable 2 server only offers those algorithms. The Metasploit session bypasses this because it uses its own SSH implementation.

The session was accessed through Metasploit:

```text
sessions -i 2
```

## Post Exploitation

### User Validation

Inside the session, the current user was confirmed:

```text
whoami
msfadmin
```

### Privilege Level

The `msfadmin` user has `sudo` privileges on Metasploitable 2 and belongs to the `admin` group, providing a path to privilege escalation.

## Lessons Learned

- Default credentials on Metasploitable 2 are well-documented and trivially discoverable with any wordlist containing `msfadmin:msfadmin`.
- The `ssh_login` auxiliary module is a fast, reliable way to test credential strength across SSH services.
- Modern OpenSSH clients may fail to connect to legacy servers due to disabled host key algorithms. Metasploit's built-in SSH client handles compatibility transparently.
- A successful SSH session at the user level is a foothold — privilege escalation is the next phase.
- Wordlist quality matters more than wordlist size. A small targeted list covering known defaults is often more effective than a massive generic list.

## Remediation

| Control | Action |
| --- | --- |
| Change default credentials | Replace all default usernames and passwords before deploying any system. |
| Disable password authentication | Use SSH key-based authentication and disable password-based login. |
| Account audit | Review all user accounts and remove or disable unused accounts. |
| Fail2ban / rate-limiting | Deploy SSH rate-limiting to slow brute-force attempts. |
| Network segmentation | Restrict SSH access to authorized management networks only. |

## References

- [Metasploit Documentation: ssh_login](https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html)
- [Metasploitable 2 Documentation](https://docs.rapid7.com/metasploit/metasploitable-2/)
