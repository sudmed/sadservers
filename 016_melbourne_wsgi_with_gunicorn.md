# "Melbourne": WSGI with Gunicorn

**Description:** There is a Python WSGI web application file at `/home/admin/wsgi.py`, the purpose of which is to serve the string "Hello, world!". This file is served by a Gunicorn server which is fronted by an nginx server (both servers managed by systemd). So the flow of an HTTP request is: Web Client (curl) -> Nginx -> Gunicorn -> wsgi.py . The objective is to be able to curl the localhost (on default port :80) and get back "Hello, world!", using the current setup.  

**Test:** `curl -s http://localhost` returns `Hello, world!` (serving the wsgi.py file via Gunicorn and Nginx).  

---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin/ && ls -la`  
```console
total 40
drwxr-xr-x 6 admin admin 4096 Dec 23  2022 .
drwxr-xr-x 3 root  root  4096 Aug 31  2022 ..
-rw------- 1 admin admin    0 Dec 23  2022 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Dec 22  2022 .local
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x 2 admin admin 4096 Dec 23  2022 __pycache__
drwxr-xr-x 2 admin admin 4096 Dec 23  2022 agent
-rw-r--r-- 1 admin admin  162 Dec 23  2022 wsgi.py
```

`cd ~/agent && cat check.sh`  
```console
#!/usr/bin/bash
res=$(curl -s --unix-socket /run/gunicorn.sock http://localhost)
res=$(echo $res|tr -d '\r')

if [[ "$res" != "Hello, world!" ]]
then
  echo -n "NO"
  exit
fi

res=$(curl -s http://localhost)
res=$(echo $res|tr -d '\r')
if [[ "$res" = "Hello, world!" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`curl -s http://localhost`  
```console
<no output>
```

`systemctl list-units | grep gunicorn`  
```console
  gunicorn.service                                                         loaded active running   gunicorn daemon
  gunicorn.socket                                                          loaded active running   gunicorn socket
```

`systemctl status gunicorn`  
```console
● gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2024-01-25 20:02:21 UTC; 3min 44s ago
TriggeredBy: ● gunicorn.socket
   Main PID: 616 (gunicorn)
      Tasks: 2 (limit: 524)
     Memory: 17.1M
        CPU: 240ms
     CGroup: /system.slice/gunicorn.service
             ├─616 /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
             └─679 /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi

Jan 25 20:02:21 i-03a8f571125757354 systemd[1]: Started gunicorn daemon.
Jan 25 20:02:21 i-03a8f571125757354 gunicorn[616]: [2024-01-25 20:02:21 +0000] [616] [INFO] Starting gunicorn 20.1.0
Jan 25 20:02:21 i-03a8f571125757354 gunicorn[616]: [2024-01-25 20:02:21 +0000] [616] [INFO] Listening at: unix:/run/gunicorn.sock (616)
Jan 25 20:02:21 i-03a8f571125757354 gunicorn[616]: [2024-01-25 20:02:21 +0000] [616] [INFO] Using worker: sync
Jan 25 20:02:21 i-03a8f571125757354 gunicorn[679]: [2024-01-25 20:02:21 +0000] [679] [INFO] Booting worker with pid: 679
```

`systemctl status nginx`  
```console
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:nginx(8)
```

### NGINX service is down


#### 2. Start NGINX daemon
`sudo systemctl start nginx`  
```console
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; disabled; vendor preset: enabled)
     Active: active (running) since Thu 2024-01-25 20:06:57 UTC; 4s ago
       Docs: man:nginx(8)
    Process: 908 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 909 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 910 (nginx)
      Tasks: 3 (limit: 524)
     Memory: 11.1M
        CPU: 35ms
     CGroup: /system.slice/nginx.service
             ├─910 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─911 nginx: worker process
             └─912 nginx: worker process

Jan 25 20:06:57 i-03a8f571125757354 systemd[1]: Starting A high performance web server and a reverse proxy server...
Jan 25 20:06:57 i-03a8f571125757354 systemd[1]: Started A high performance web server and a reverse proxy server.
```

`curl -s http://localhost`  
```html
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>
```


#### 3. Check NGINX config
`cat /etc/nginx/sites-enabled/default`  
```console
server {
    listen 80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.socket;
    }
}
```


#### 4. Check if that proxy_pass is true
`ps auxf|grep gunicorn`  
```console
admin        924  0.0  0.1   5268   712 pts/0    S<+  20:09   0:00      \_ grep gunicorn
admin        616  0.0  4.7  27480 22024 ?        Ss   20:02   0:00 /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
admin        679  0.0  4.1  27560 19360 ?        S    20:02   0:00  \_ /usr/bin/python3 /usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
```

`cat /etc/systemd/system/gunicorn.service`  
```console
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=admin
Group=admin
WorkingDirectory=/home/admin
ExecStart=/usr/local/bin/gunicorn \
          --bind unix:/run/gunicorn.sock \
          wsgi
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### No, it's not true. The correct one is `unix:/run/gunicorn.sock`


#### 5. Change proxy_pass in NGINX config
`sudo vi /etc/nginx/sites-enabled/default`  
```console
server {
    listen 80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

`sudo nginx -t && sudo nginx -s reload`  
```console
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

`curl -s http://localhost`  
```console
<no output>
```

`curl -I http://localhost`  
```console
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Thu, 25 Jan 2024 20:12:39 GMT
Content-Type: text/html
Content-Length: 0
Connection: keep-alive
```

`curl --unix-socket /run/gunicorn.sock http://localhost`  
```console
<no output>
```

`curl -I --unix-socket /run/gunicorn.sock http://localhost`  
```console
HTTP/1.1 200 OK
Server: gunicorn
Date: Thu, 25 Jan 2024 20:13:27 GMT
Connection: close
Content-Type: text/html
Content-Length: 0
```


#### 6. Let's check Gunicorn files
`cat wsgi.py`  
```console
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'), ('Content-Length', '0'), ])
    return [b'Hello, world!']
```

### The `Content-Length` value is `0`, that's why the response body is empty


#### 7. Encrease the value of `Content-Length`
`vi wsgi.py`  
```console
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'), ('Content-Length', '100'), ])
    return [b'Hello, world!']
```

`sudo systemctl restart gunicorn`  


#### 8. Validate the task
`curl -s http://localhost`  
```console
Hello, world!
```

`cd ~/agent && ./check.sh`  
```console
OK
```
