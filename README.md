# Metasploit Labs

[![License](https://img.shields.io/github/license/Utkarsh464/metasploit-labs)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Linux-blue)](#lab-environment)
[![Tools](https://img.shields.io/badge/tools-Metasploit%20%7C%20Nmap%20%7C%20Netcat-informational)](#tools)
[![Lab](https://img.shields.io/badge/lab-Metasploitable%202-critical)](labs/metasploitable2/)

Hands-on penetration testing labs using Metasploit, Nmap and intentionally vulnerable machines.

This repository is a portfolio-style record of controlled security labs. Each lab focuses on enumeration, vulnerability analysis, exploitation, troubleshooting, post-exploitation validation, and clear reporting.

> All activity documented here was performed in an isolated, intentionally vulnerable lab environment for educational purposes.

## Lab Index

| Lab | Target | Technique | Status |
| --- | --- | --- | --- |
| [vsFTPd 2.3.4 Backdoor](labs/metasploitable2/vsftpd-2.3.4-backdoor/README.md) | Metasploitable 2 | Metasploit exploitation and Meterpreter validation | Complete |

## Featured Lab

### vsFTPd 2.3.4 Backdoor

This lab identifies `vsftpd 2.3.4` on Metasploitable 2, maps it to CVE-2011-2523, troubleshoots failed callback behavior, and validates a successful root shell through Meterpreter.

Key artifacts:

- [Full lab writeup](labs/metasploitable2/vsftpd-2.3.4-backdoor/README.md)
- [Command reference](labs/metasploitable2/vsftpd-2.3.4-backdoor/notes/commands.md)
- [Metasploit setup commands](labs/metasploitable2/vsftpd-2.3.4-backdoor/commands/metasploit-vsftpd-setup.rc)
- [Screenshots directory](labs/metasploitable2/vsftpd-2.3.4-backdoor/screenshots/)

Screenshot evidence:

| Evidence | Description |
| --- | --- |
| [Zenmap service scan](labs/metasploitable2/vsftpd-2.3.4-backdoor/screenshots/01-zenmap-service-scan.png) | Service discovery showing `vsftpd 2.3.4` on `21/tcp`. |
| [Metasploit module options](labs/metasploitable2/vsftpd-2.3.4-backdoor/screenshots/02-metasploit-vsftpd-module-options.png) | Module search, module selection, and required options. |
| [Meterpreter session](labs/metasploitable2/vsftpd-2.3.4-backdoor/screenshots/03-vsftpd-forceexploit-meterpreter-session.png) | Successful exploit run after troubleshooting. |
| [Root shell validation](labs/metasploitable2/vsftpd-2.3.4-backdoor/screenshots/04-meterpreter-root-shell-validation.png) | `whoami` and `uname -a` validation from an interactive shell. |

## Repository Layout

```text
metasploit-labs/
|-- labs/
|   `-- metasploitable2/
|       `-- vsftpd-2.3.4-backdoor/
|           |-- README.md
|           |-- commands/
|           |-- notes/
|           `-- screenshots/
|-- .gitignore
|-- LICENSE
`-- README.md
```

## Lab Environment

- Attacker machine: Linux penetration testing workstation
- Target machine: Metasploitable 2
- Network: isolated virtual network
- Tools: Nmap, Zenmap, Metasploit Framework, Meterpreter, Netcat

## Skills Demonstrated

- Network and service enumeration
- Version-based vulnerability identification
- Metasploit module selection and configuration
- Troubleshooting failed reverse payload callbacks
- Understanding `RHOSTS`, `LHOST`, bind listeners, and reverse payload behavior
- Post-exploitation validation with filesystem access and shell commands
- Professional lab documentation and git workflow

## Tools

- [Metasploit Framework](https://www.metasploit.com/)
- [Nmap](https://nmap.org/)
- [Netcat](https://nc110.sourceforge.io/)
- [Metasploitable 2](https://docs.rapid7.com/metasploit/metasploitable-2/)

## Legal Notice

The content in this repository is for authorized lab use only. Do not run these techniques against systems you do not own or do not have explicit permission to test.
