# "Karakorum": WTFIT – What The Fun Is This?

**Description:** There's a binary at `/home/admin/wtfit` that nobody knows how it works or what it does ("what the fun is this"). Someone remembers something about wtfit needing to communicate to a service in order to start. Run this wtfit program so it doesn't exit with an error, fixing or working around things that you need but are broken in this server. (Note that you can open more than one web "terminal").  

**Test:** Running `/home/admin/wtfit` returns `OK`.  

---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 6264
drwxr-xr-x 4 admin admin    4096 Sep 13  2022 .
drwxr-xr-x 3 root  root     4096 Aug 31  2022 ..
-rw------- 1 admin admin       5 Sep 13  2022 .bash_history
-rw-r--r-- 1 admin admin     220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin    3526 Aug  4  2021 .bashrc
-rw-r--r-- 1 admin admin     807 Aug  4  2021 .profile
drwx------ 2 admin admin    4096 Aug 31  2022 .ssh
drwxr-xr-x 2 admin admin    4096 Sep 13  2022 agent
-rw-r--r-- 1 admin admin 6381234 Sep 13  2022 wtfit
```

`cat agent/check.sh`  
```bash
#!/usr/bin/bash
res=$(/home/admin/wtfit)
res=$(echo $res|tr -d '\r')

if [[ "$res" = "OK." ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`sudo -i && /home/admin/wtfit`  
```console
-bash: /home/admin/wtfit: Permission denied
```

`strace ./wtfit`  
```console
execve("./wtfit", ["./wtfit"], 0x7ffe030395b0 /* 17 vars */) = -1 EACCES (Permission denied)
strace: exec: Permission denied
+++ exited with 1 +++
```
##### There is no permition to execute the file, let's add this permition.


#### 2. Assign execute permission
`chmod +x wtfit`  
```console
-bash: /usr/bin/chmod: Permission denied
```

`ls -la /usr/bin/chmod`  
```console
-rw-r--r-- 1 root root 64448 Sep 24  2020 /usr/bin/chmod
```
##### There is no permition to execute chmod, let's do chmod without /usr/bin/chmod.


#### 3. Chmod without chmod
`perl -e 'chmod 0755, "wtfit"'`  

##### And try again...

`/home/admin/wtfit`  
```console
ERROR: can't open config file
```

`strace ./wtfit`  
```console
<...>
fcntl(2, F_GETFL)                       = 0x8002 (flags O_RDWR|O_LARGEFILE)
openat(AT_FDCWD, "/home/admin/wtfitconfig.conf", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
write(1, "ERROR: can't open config file\n", 30ERROR: can't open config file
) = 30
exit_group(1)                           = ?
+++ exited with 1 +++
```

##### There is an error: can't open config file. Let's create file `/home/admin/wtfitconfig.conf`


#### 4. Make file
`touch /home/admin/wtfitconfig.conf`  
```console
```

##### And try again...

`/home/admin/wtfit`  
```console
ERROR: can't connect to server
```

`strace ./wtfit`  
```console
<...>
close(3)                                = 0
futex(0xc000032548, FUTEX_WAKE_PRIVATE, 1) = 1
socket(AF_INET, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, IPPROTO_IP) = 3
connect(3, {sa_family=AF_INET, sin_port=htons(7777), sin_addr=inet_addr("127.0.0.1")}, 16) = -1 EINPROGRESS (Operation now in progress)
epoll_ctl(4, EPOLL_CTL_ADD, 3, {EPOLLIN|EPOLLOUT|EPOLLRDHUP|EPOLLET, {u32=2749411048, u64=140233431625448}}) = 0
write(6, "\0", 1)                       = 1
futex(0xc000032548, FUTEX_WAKE_PRIVATE, 1) = 1
getsockopt(3, SOL_SOCKET, SO_ERROR, [ECONNREFUSED], [4]) = 0
futex(0xc000032548, FUTEX_WAKE_PRIVATE, 1) = 1
epoll_ctl(4, EPOLL_CTL_DEL, 3, 0xc0000b5324) = 0
close(3)                                = 0
futex(0xc000032548, FUTEX_WAKE_PRIVATE, 1) = 1
write(1, "ERROR: can't connect to server\n", 31ERROR: can't connect to server
) = 31
exit_group(1)                           = ?
+++ exited with 1 +++
```

##### There is an error: can't connect to server 127.0.0.1:7777. And what about web server?


#### 5. Try NGINX
`systemctl status nginx`  
```console
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:nginx(8)
```

`systemctl start nginx && systemctl status nginx`  
```console
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2024-02-06 13:10:04 UTC; 12ms ago
       Docs: man:nginx(8)
    Process: 938 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 939 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 940 (nginx)
      Tasks: 3 (limit: 524)
     Memory: 3.1M
        CPU: 33ms
     CGroup: /system.slice/nginx.service
             ├─940 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─941 nginx: worker process
             └─942 nginx: worker process

Feb 06 13:10:04 i-02e5598634668cce3 systemd[1]: Starting A high performance web server and a reverse proxy server...
Feb 06 13:10:04 i-02e5598634668cce3 systemd[1]: Started A high performance web server and a reverse proxy server.
```

`vi /etc/nginx/sites-available/default`  
```console
<...>
# Default server configuration
#
server {
        listen 7777 default_server;
<...>
```

`nginx -t && nginx -s reload`  
```console
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
2024/02/06 13:14:53 [notice] 1018416#1018416: signal process started
```

#### 5. Validate the task
`cd /home/admin/agent/ && ./check.sh`  
```console
OK
```

`/home/admin/wtfit`  
```console
OK.
```


---

## AUTHOR'S SOLUTION
`python3 -m http.server 7777`  
```console
Serving HTTP on 0.0.0.0 port 7777 (http://0.0.0.0:7777/) ...
127.0.0.1 - - [06/Feb/2024 13:06:22] "GET / HTTP/1.1" 200 -
```
