# "Jakarta": it's always DNS.

**Description:** Can't `ping google.com`. It returns `ping: google.com: Name or service not known`. Expected is being able to resolve the hostname. (Note: currently the VMs can't ping outside so there's no automated check for the solution).  

**Test:** `ping google.com` should return something like `PING google.com (172.217.2.46) 56(84) bytes of data`.  


---

### Solution:
#### 1. Reconnaissance on the server
`ping google.com`  
```console
ping: google.com: Name or service not known
```

`cat resolv.conf`  
```console
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search us-east-2.compute.internal
```

`ip a`  
```console
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 0a:f0:03:4c:36:1d brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet 172.31.34.187/20 metric 100 brd 172.31.47.255 scope global dynamic ens5
       valid_lft 3316sec preferred_lft 3316sec
    inet6 fe80::8f0:3ff:fe4c:361d/64 scope link 
       valid_lft forever preferred_lft forever
```

`systemctl status systemd-resolved`  
```console
● systemd-resolved.service - Network Name Resolution
     Loaded: loaded (/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-02-03 14:51:43 UTC; 6min ago
       Docs: man:systemd-resolved.service(8)
             man:org.freedesktop.resolve1(5)
             https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
             https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
   Main PID: 436 (systemd-resolve)
     Status: "Processing requests..."
      Tasks: 1 (limit: 521)
     Memory: 8.1M
        CPU: 119ms
     CGroup: /system.slice/systemd-resolved.service
             └─436 /lib/systemd/systemd-resolved

Feb 03 14:51:43 ip-172-31-42-233 systemd[1]: Starting Network Name Resolution...
Feb 03 14:51:43 ip-172-31-42-233 systemd-resolved[436]: Positive Trust Anchors:
Feb 03 14:51:43 ip-172-31-42-233 systemd-resolved[436]: . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc683457104237c7f8ec8d
Feb 03 14:51:43 ip-172-31-42-233 systemd-resolved[436]: Negative trust anchors: home.arpa 10.in-addr.arpa 16.172.in-addr.arpa 17.172.in-addr.arpa 18.172.in-addr.arpa 19.172.in-addr.arpa 20.172.in-addr.arpa 21.172.i>
Feb 03 14:51:43 ip-172-31-42-233 systemd-resolved[436]: Using system hostname 'ip-172-31-42-233'.
Feb 03 14:51:43 ip-172-31-42-233 systemd[1]: Started Network Name Resolution.
```

`sudo systemctl restart systemd-resolved`  
```console
sudo: unable to resolve host ip-172-31-42-233: Name or service not known
```

`hostname`  
```console
ip-172-31-42-233
```

`cat /etc/hosts`  
```console
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

`cat /etc/nsswitch.conf`  
```console
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd
group:          files systemd
shadow:         files
gshadow:        files

hosts:          files
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```
##### Here you are, the error has been found! The line "hosts: files" means that only the local file /etc/hosts will be used for host name resolution. Let's add DNS service.


#### 2. Fix nsswitch.conf
`sudo vi /etc/nsswitch.conf`  
```console
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd
group:          files systemd
shadow:         files
gshadow:        files

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```


#### 5. Validate the task
`ping google.com`  
```console
PING google.com (142.250.190.78) 56(84) bytes of data.
```
