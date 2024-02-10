# "Pokhara": SSH and other sshenanigans

**Description:** A user `client` was added to the server, as well as their SSH public key.  

The objective is to be able to SSH locally (there's only one server) as this user `client` using their ssh keys. This is, if as root you change to this user `sudo su; su client`, you should be able to login with ssh: `ssh localhost`.  

**Test: As user admin: `sudo -u client ssh client@localhost 'pwd'` returns `/home/client`.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la /home/admin/`  
```console
total 32
drwxr-xr-x 5 admin admin 4096 Feb  6  2023 .
drwxr-xr-x 4 root  root  4096 Feb  5  2023 ..
-rw------- 1 admin admin    0 Feb  6  2023 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Feb  6  2023 .local
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x 2 admin admin 4096 Feb  6  2023 agent
```

`cat /home/admin/agent/check.sh`  
```bash
#!/usr/bin/bash
res=$(sudo -u client ssh client@localhost 'pwd')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "/home/client" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`ls -la /home/client/`  
```console
total 32
drwxr-xr-x 4 client client 4096 Feb  5  2023 .
drwxr-xr-x 4 root   root   4096 Feb  5  2023 ..
-rw------- 1 client client  475 Feb  6  2023 .bash_history
-rw-r--r-- 1 client client  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 client client 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 client client 4096 Feb  5  2023 .local
-rw-r--r-- 1 client client  807 Aug  4  2021 .profile
drwx------ 2 client client 4096 Feb  6  2023 .ssh
```

`ls -la /home/client/.ssh/`  
```console
total 24
drwx------ 2 client client 4096 Feb  6  2023 .
drwxr-xr-x 4 client client 4096 Feb  5  2023 ..
-rw------- 1 client client  569 Feb  6  2023 authorized_keys
-rw-r--r-- 1 client client 2610 Feb  5  2023 id_rsa
-rw-r--r-- 1 client client  569 Feb  5  2023 id_rsa.pub
-rw-r--r-- 1 client client  222 Feb  5  2023 known_hosts
```

`cat /home/client/.ssh/id_rsa.pub`  
```console
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDQopf1DsoscAMwsWJL1V1F1GHom1+qwkp0QlXnfWP+bKwkXZfTj+IIvtEZ3ECyIQa2bMEFdME9aU67vJoF4X3KSvQQijxr9ng5fWhVxvCauYdVi4UU1NtvHtw7RUj+gIIwcqmOg2/wwhRb7zjN53arsaJu0P77VOhwTRYI0fcUX6iE+W6wNvKRBvnZShSla5pCk0x7Ox2wptYfTnbIOx6+me7g9fkctakm1yeujsRbQjJHwkfdLfYhMJkT4wibLvJSooB5WIe62ioCcFbiHEywMrrdKH8oCy8i8wD28S5vh6FTZiZxBX6nL7HModXQI9RV6mZ9TE/ZovWYCk3Cp0675JoWEM94C53S+5eVtZTj4l2ZsYwmc8WaJiullLYdWEYi2jtmnHxsnFQ/YZ/Tf9zndRBVUPKuG84mGzJ5Fs28w0u5SiNeHdS0OOHvAcGnuoweKigt01JRJdt8DZO+N8jCEM/jo1TST131sy1TLyylY6zBe6ID/uoyWkVLc5/iFhM= client@somehost
```

`cat /home/client/.ssh/authorized_keys`  
```console
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDQopf1DsoscAMwsWJL1V1F1GHom1+qwkp0QlXnfWP+bKwkXZfTj+IIvtEZ3ECyIQa2bMEFdME9aU67vJoF4X3KSvQQijxr9ng5fWhVxvCauYdVi4UU1NtvHtw7RUj+gIIwcqmOg2/wwhRb7zjN53arsaJu0P77VOhwTRYI0fcUX6iE+W6wNvKRBvnZShSla5pCk0x7Ox2wptYfTnbIOx6+me7g9fkctakm1yeujsRbQjJHwkfdLfYhMJkT4wibLvJSooB5WIe62ioCcFbiHEywMrrdKH8oCy8i8wD28S5vh6FTZiZxBX6nL7HModXQI9RV6mZ9TE/ZovWYCk3Cp0675JoWEM94C53S+5eVtZTj4l2ZsYwmc8WaJiullLYdWEYi2jtmnHxsnFQ/YZ/Tf9zndRBVUPKuG84mGzJ5Fs28w0u5SiNeHdS0OOHvAcGnuoweKigt01JRJdt8DZO+N8jCEM/jo1TST131sy1TLyylY6zBe6ID/uoyWkVLc5/iFhM= client@somehost
```

`su client`  
```console
Your account has expired; please contact your system administrator.
su: Authentication failure
```
##### Error: account has expired. Let's check it

`cat /etc/passwd`  
```console
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
messagebus:x:101:101::/nonexistent:/usr/sbin/nologin
uuidd:x:102:102::/run/uuidd:/usr/sbin/nologin
tcpdump:x:103:103::/nonexistent:/usr/sbin/nologin
systemd-network:x:104:105:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:105:106:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
_chrony:x:106:112:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
sshd:x:107:65534::/run/sshd:/usr/sbin/nologin
systemd-timesync:x:999:999:systemd Time Synchronization:/:/usr/sbin/nologin
systemd-coredump:x:998:998:systemd Core Dumper:/:/usr/sbin/nologin
admin:x:1000:1000:Debian:/home/admin:/bin/bash
client:x:1001:1001::/home/client:/usr/sbin/nologin
```
##### As we can see, for user client `nologin` shell is set. Let's fix it

`vi /etc/passwd`  
```console
<...>
client:x:1001:1001::/home/client:/bin/bash
```

`cat /etc/shadow`  
```console
root:*:19115:0:99999:7:::
daemon:*:19115:0:99999:7:::
bin:*:19115:0:99999:7:::
sys:*:19115:0:99999:7:::
sync:*:19115:0:99999:7:::
games:*:19115:0:99999:7:::
man:*:19115:0:99999:7:::
lp:*:19115:0:99999:7:::
mail:*:19115:0:99999:7:::
news:*:19115:0:99999:7:::
uucp:*:19115:0:99999:7:::
proxy:*:19115:0:99999:7:::
www-data:*:19115:0:99999:7:::
backup:*:19115:0:99999:7:::
list:*:19115:0:99999:7:::
irc:*:19115:0:99999:7:::
gnats:*:19115:0:99999:7:::
nobody:*:19115:0:99999:7:::
_apt:*:19115:0:99999:7:::
messagebus:*:19115:0:99999:7:::
uuidd:*:19115:0:99999:7:::
tcpdump:*:19115:0:99999:7:::
systemd-network:*:19115:0:99999:7:::
systemd-resolve:*:19115:0:99999:7:::
_chrony:*:19115:0:99999:7:::
sshd:*:19115:0:99999:7:::
systemd-timesync:!*:19235::::::
systemd-coredump:!*:19235::::::
admin:!:19235:0:99999:7:::
client:!:19393:0:90:7::0:
```

`chage -l  client`  
```console
Last password change                                    : Feb 05, 2023
Password expires                                        : May 06, 2023
Password inactive                                       : never
Account expires                                         : Jan 01, 1970
Minimum number of days between password change          : 0
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 7
```
##### Account expires: Jan 01, 1970. Let's fix it

`usermod -e 2025-01-01 client`  
`chage -M 99999 client`  
`chage -l  client`  
```console
Last password change                                    : Feb 05, 2023
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Jan 01, 2025
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

`su client`  
```console
su: failed to execute /bin/bash: Resource temporarily unavailable
```
##### The error message `su: failed to execute /bin/bash: Resource temporarily unavailable` typically indicates that the system is unable to allocate the necessary resources to execute the /bin/bash command with the su command. Let's check limits

`ulimit -n`  
```console
1024
```

`ulimit -u`  
```console
1748
```

`cat /etc/security/limits.conf`  
<details>

  <summary>output</summary>

```console
# /etc/security/limits.conf
#
#Each line describes a limit for a user in the form:
#
#<domain>        <type>  <item>  <value>
#
#Where:
#<domain> can be:
#        - a user name
#        - a group name, with @group syntax
#        - the wildcard *, for default entry
#        - the wildcard %, can be also used with %group syntax,
#                 for maxlogin limit
#        - NOTE: group and wildcard limits are not applied to root.
#          To apply a limit to the root user, <domain> must be
#          the literal username root.
#
#<type> can have the two values:
#        - "soft" for enforcing the soft limits
#        - "hard" for enforcing hard limits
#
#<item> can be one of the following:
#        - core - limits the core file size (KB)
#        - data - max data size (KB)
#        - fsize - maximum filesize (KB)
#        - memlock - max locked-in-memory address space (KB)
#        - nofile - max number of open file descriptors
#        - rss - max resident set size (KB)
#        - stack - max stack size (KB)
#        - cpu - max CPU time (MIN)
#        - nproc - max number of processes
#        - as - address space limit (KB)
#        - maxlogins - max number of logins for this user
#        - maxsyslogins - max number of logins on the system
#        - priority - the priority to run user process with
#        - locks - max number of file locks the user can hold
#        - sigpending - max number of pending signals
#        - msgqueue - max memory used by POSIX message queues (bytes)
#        - nice - max nice priority allowed to raise to values: [-20, 19]
#        - rtprio - max realtime priority
#        - chroot - change root to directory (Debian-specific)
#
#<domain>      <type>  <item>         <value>
#

#*               soft    core            0
#root            hard    core            100000
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#ftp             -       chroot          /ftp
#@student        -       maxlogins       4
client          hard    nproc           0
# End of file
```

</details>

```console
<...>
client          hard    nproc           0
<...>
```
##### As we can see, the user `client` is limited to zero processes. Let's fix it

`vi /etc/security/limits.conf`  
```console
<...>
client          hard    nproc           20
<...>
```

`su client`  
```console
client@i-073e4be234951cc28:/root$
```
##### OK, it works!

`history`  
```console
    1  nano ~/.ssh/id_rsa.pub 
    2  cat  ~/.ssh/id_rsa.pub 
    3  exit
    4  touch ~/.ssh/authorized_keys
    5  ls -l ~/.ssh/authorized_keys
    6  chmod 600 ~/.ssh/authorized_keys
    7  ls -l ~/.ssh/authorized_keys
    8  nano ~/.ssh/authorized_keys
    9  ssh localhost
   10  ssh client@localhost 'pwd' 
   11  exit
   12  ssh client@localhost 'pwd'
   13  exit
   14  cat ~/.ssh/id_rsa.pub
   15  cat ~/.ssh/authorized_keys
   16  nano ~/.ssh/authorized_keys
   17  cat ~/.ssh/authorized_keys
   18  cat ~/.ssh/id_rsa.pub
   19  md5sum ~/.ssh/id_rsa.pub
   20  md5sum ~/.ssh/authorized_keys 
   21  whoami 
   22  exit
```

`ls -la ~/.ssh/`  
```console
total 24
drwx------ 2 client client 4096 Feb 10 10:52 .
drwxr-xr-x 4 client client 4096 Feb  5  2023 ..
-rw------- 1 client client  569 Feb 10 11:08 authorized_keys
-rw-r--r-- 1 client client 2610 Feb  5  2023 id_rsa
-rw-r--r-- 1 client client  569 Feb  5  2023 id_rsa.pub
-rw-r--r-- 1 root   root      0 Feb 10 10:52 known_hosts
```
##### We can see wrong permitions on private key file `id_rsa`. Let's fix it

`chmod 600 id_rsa`  


`ssh client@localhost 'pwd'`  
```console
client@localhost: Permission denied (publickey).
```
##### The problem is not related to the ssh keys. Let's check sshd config

`exit && sudo -i && cd /etc/ssh/ && ls -la`  
```console
total 628
drwxr-xr-x  4 root root   4096 Feb 10 10:47 .
drwxr-xr-x 76 root root   4096 Feb 10 11:16 ..
-rw-r--r--  1 root root 577771 Mar 13  2021 moduli
-rw-r--r--  1 root root   1650 Mar 13  2021 ssh_config
drwxr-xr-x  2 root root   4096 Mar 13  2021 ssh_config.d
-rw-------  1 root root   1393 Feb 10 10:47 ssh_host_dsa_key
-rw-r--r--  1 root root    614 Feb 10 10:47 ssh_host_dsa_key.pub
-rw-------  1 root root    513 Feb 10 10:47 ssh_host_ecdsa_key
-rw-r--r--  1 root root    186 Feb 10 10:47 ssh_host_ecdsa_key.pub
-rw-------  1 root root    419 Feb 10 10:47 ssh_host_ed25519_key
-rw-r--r--  1 root root    106 Feb 10 10:47 ssh_host_ed25519_key.pub
-rw-------  1 root root   2610 Feb 10 10:47 ssh_host_rsa_key
-rw-r--r--  1 root root    578 Feb 10 10:47 ssh_host_rsa_key.pub
-rw-r--r--  1 root root   3311 May  3  2022 sshd_config
drwxr-xr-x  2 root root   4096 Feb  5  2023 sshd_config.d
-rw-r--r--  1 root root   3287 Aug 31  2022 sshd_config.ucf-dist
```

`cat ssh_config`  
```console
# This is the ssh client system-wide configuration file.  See
# ssh_config(5) for more information.  This file provides defaults for
# users, and the values can be changed in per-user configuration files
# or on the command line.

# Configuration data is parsed as follows:
#  1. command line options
#  2. user-specific file
#  3. system-wide file
# Any configuration value is only changed the first time it is set.
# Thus, host-specific definitions should be at the beginning of the
# configuration file, and defaults at the end.

# Site-wide defaults for some commonly used options.  For a comprehensive
# list of available options, their meanings and defaults, please see the
# ssh_config(5) man page.

Include /etc/ssh/ssh_config.d/*.conf

Host *
#   ForwardAgent no
#   ForwardX11 no
#   ForwardX11Trusted yes
#   PasswordAuthentication yes
#   HostbasedAuthentication no
#   GSSAPIAuthentication no
#   GSSAPIDelegateCredentials no
#   GSSAPIKeyExchange no
#   GSSAPITrustDNS no
#   BatchMode no
#   CheckHostIP yes
#   AddressFamily any
#   ConnectTimeout 0
#   StrictHostKeyChecking ask
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   IdentityFile ~/.ssh/id_ecdsa
#   IdentityFile ~/.ssh/id_ed25519
#   Port 22
#   Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
#   MACs hmac-md5,hmac-sha1,umac-64@openssh.com
#   EscapeChar ~
#   Tunnel no
#   TunnelDevice any:any
#   PermitLocalCommand no
#   VisualHostKey no
#   ProxyCommand ssh -q -W %h:%p gateway.example.com
#   RekeyLimit 1G 1h
#   UserKnownHostsFile ~/.ssh/known_hosts.d/%k
    SendEnv LANG LC_*
    HashKnownHosts yes
    GSSAPIAuthentication yes
```
##### All is OK here. 

`ls -la /etc/ssh/sshd_config.d/`  
```console
total 12
drwxr-xr-x 2 root root 4096 Feb  5  2023 .
drwxr-xr-x 4 root root 4096 Feb 10 11:28 ..
-rw-r--r-- 1 root root   17 Feb  6  2023 sad.conf
```

`cat /etc/ssh/sshd_config.d/sad.conf`  
```console
DenyUsers client
```
##### This is a clocker! Let's fix it

`rm /etc/ssh/sshd_config.d/sad.conf`  
`systemctl restart ssh`  
`systemctl status ssh`  
```console
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-02-10 11:33:58 UTC; 6s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 918 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 919 (sshd)
      Tasks: 1 (limit: 524)
     Memory: 1.0M
        CPU: 22ms
     CGroup: /system.slice/ssh.service
             └─919 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

Feb 10 11:33:58 i-0f5fab67d7a19f92d systemd[1]: Starting OpenBSD Secure Shell server...
Feb 10 11:33:58 i-0f5fab67d7a19f92d sshd[919]: Server listening on 0.0.0.0 port 22.
Feb 10 11:33:58 i-0f5fab67d7a19f92d sshd[919]: Server listening on :: port 22.
Feb 10 11:33:58 i-0f5fab67d7a19f92d systemd[1]: Started OpenBSD Secure Shell server.
```


#### 2. Validate the task
`sudo -u client ssh client@localhost 'pwd'`  
```console
/home/client
```

`bash /home/admin/agent/check.sh`  
```console
OK
```
