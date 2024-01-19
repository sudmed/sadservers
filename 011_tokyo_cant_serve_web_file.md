# "Tokyo": can't serve web file

**Description:** There's a web server serving a file `/var/www/html/index.html` with content "hello sadserver" but when we try to check it locally with an HTTP client like `curl 127.0.0.1:80`, nothing is returned. This scenario is not about the particular web server configuration and you only need to have general knowledge about how web servers work.  

**Test:** `curl 127.0.0.1:80` should return: `hello sadserver`.  

---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 72
drwxr-xr-x  19 root root  4096 Jan 19 08:27 .
drwxr-xr-x  19 root root  4096 Jan 19 08:27 ..
lrwxrwxrwx   1 root root     7 Jun  9  2022 bin -> usr/bin
drwxr-xr-x   4 root root  4096 Jun  9  2022 boot
drwxr-xr-x  16 root root  3260 Jan 19 08:27 dev
drwxr-xr-x 100 root root  4096 Jan 19 08:27 etc
drwxr-xr-x   3 root root  4096 Aug  1  2022 home
lrwxrwxrwx   1 root root     7 Jun  9  2022 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Jun  9  2022 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Jun  9  2022 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Jun  9  2022 libx32 -> usr/libx32
drwx------   2 root root 16384 Jun  9  2022 lost+found
drwxr-xr-x   2 root root  4096 Jun  9  2022 media
drwxr-xr-x   2 root root  4096 Jun  9  2022 mnt
drwxr-xr-x   2 root root  4096 Jun  9  2022 opt
dr-xr-xr-x 178 root root     0 Jan 19 08:26 proc
drwx------   6 root root  4096 Aug 31  2022 root
drwxr-xr-x  28 root root   840 Jan 19 08:27 run
lrwxrwxrwx   1 root root     8 Jun  9  2022 sbin -> usr/sbin
drwxr-xr-x   8 root root  4096 Jun  9  2022 snap
drwxr-xr-x   2 root root  4096 Jun  9  2022 srv
dr-xr-xr-x  13 root root     0 Jan 19 08:26 sys
drwxrwxrwt  14 root root  4096 Jan 19 08:27 tmp
drwxr-xr-x  14 root root  4096 Jun  9  2022 usr
drwxr-xr-x  14 root root  4096 Aug  1  2022 var
```

`cat /home/ubuntu/agent/check.sh`  
```console
#!/usr/bin/bash
res=$(curl -s -m 2 127.0.0.1:80)
res=$(echo $res|tr -d '\r')

if [[ "$res" = "hello sadserver" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`iptables -L -n -v --line-numbers`  
```console
Chain INPUT (policy ACCEPT 704 packets, 65974 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 829 packets, 434K bytes)
num   pkts bytes target     prot opt in     out     source               destination         
```

`curl 127.0.0.1:80`  
```console
<no answer/nothing is returned>
```


#### 2. Problem seems like iptables's `DROP` rule blocks input traffic. Let's delete this rule
`iptables -D INPUT 1`  

`curl 127.0.0.1:80`  
```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
<hr>
<address>Apache/2.4.52 (Ubuntu) Server at 127.0.0.1 Port 80</address>
</body></html>
```


#### 3. As we can see there is No permission error to access on index file. Let's check it
`cd /var/www/html/ && ls -la`  
```console
total 12
drwxr-xr-x 2 root root 4096 Aug  1  2022 .
drwxr-xr-x 3 root root 4096 Aug  1  2022 ..
-rw------- 1 root root   16 Aug  1  2022 index.html
```


#### 4. The problem is the absent read permission. Let's chmod it
`chmod +r index.html`  

`ls -la`  
```console
total 12
drwxr-xr-x 2 root root 4096 Aug  1  2022 .
drwxr-xr-x 3 root root 4096 Aug  1  2022 ..
-rw-r--r-- 1 root root   16 Aug  1  2022 index.html
```


#### 5. Validate the task
`curl 127.0.0.1:80`  
```console
hello sadserver
```

`cd /home/ubuntu/agent/ && ./check.sh`  
```console
OK
```
