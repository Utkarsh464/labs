# VSFTPD 2.3.4 Backdoor Lab

## Objective

Document exploitation of the vsFTPd 2.3.4 backdoor vulnerability in an isolated Metasploitable 2 lab environment.

## Status

Writeup pending.

## Screenshots

| # | Evidence | Description |
| --- | --- | --- |
| 1 | [Zenmap service scan](screenshots/01-zenmap-service-scan.png) | Nmap/Zenmap service discovery against the Metasploitable 2 target. |
| 2 | [Metasploit module options](screenshots/02-metasploit-vsftpd-module-options.png) | Metasploit search result, module selection, and required options for `exploit/unix/ftp/vsftpd_234_backdoor`. |
| 3 | [Successful Meterpreter session](screenshots/03-vsftpd-forceexploit-meterpreter-session.png) | Forced exploit attempt resulting in a Meterpreter session. |
| 4 | [Root shell validation](screenshots/04-meterpreter-root-shell-validation.png) | Interactive shell validation with `whoami` and `uname -a`. |

## Notes

Detailed notes and command references will be added in later phases.
