# "Lisbon": etcd SSL cert troubles

**Description:** There's an etcd server running on `https://localhost:2379`, get the value for the key "foo", ie `etcdctl get foo` or `curl https://localhost:2379/v2/keys/foo`.  

**Test:** `etcdctl get foo` returns `bar`.  


---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin/ && ls -la`  
```console
total 40
drwxr-xr-x 5 admin admin 4096 Jan  1  2023 .
drwxr-xr-x 3 root  root  4096 Aug 31  2022 ..
-rw------- 1 admin admin    5 Jan  3  2024 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3629 Jan  1  2023 .bashrc
drwxr-xr-x 3 admin admin 4096 Jan  1  2023 .local
-rw-r--r-- 1 admin admin  807 Jan  1  2023 .profile
-rw-r--r-- 1 admin admin   66 Jan  1  2023 .selected_editor
drwx------ 2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x 2 admin admin 4096 Jan  1  2023 agent
```

`cd agent/ && ls -la`  
```console
total 11144
drwxr-xr-x 2 admin admin     4096 Jan  1  2023 .
drwxr-xr-x 5 admin admin     4096 Jan  1  2023 ..
-rwxr-xr-x 1 admin admin      212 Jan  1  2023 check.sh
-rwxr-xr-x 1 admin admin 11397096 Aug 31  2022 sadagent
-rw-r--r-- 1 admin admin        0 Jan  1  2023 sadagent.txt
```

`cd ~/agent && cat check.sh `  
```console
#!/usr/bin/bash
export GODEBUG=x509ignoreCN=0
export ETCDCTL_ENDPOINT=https://localhost:2379
res=$(etcdctl get foo)
res=$(echo $res|tr -d '\r')

if [[ "$res" = "bar" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`curl https://localhost:2379/v2/keys/foo`  
```console
curl: (60) SSL certificate problem: certificate has expired
More details here: https://curl.se/docs/sslcerts.html
curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

`etcdctl get foo`  
```console
Error:  client: etcd cluster is unavailable or misconfigured; error #0: x509: certificate has expired or is not yet valid: current time 2025-01-27T06:55:29Z is after 2023-01-30T00:02:48Z
error #0: x509: certificate has expired or is not yet valid: current time 2025-01-27T06:55:29Z is after 2023-01-30T00:02:48Z
```

`systemctl status nginx`  
```console
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-01-27 06:54:02 UTC; 1 years 0 months ago
       Docs: man:nginx(8)
    Process: 569 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 603 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 615 (nginx)
      Tasks: 3 (limit: 521)
     Memory: 12.9M
        CPU: 76ms
     CGroup: /system.slice/nginx.service
             ├─615 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             ├─617 nginx: worker process
             └─618 nginx: worker process

Jan 27 06:54:01 i-068c30f995dc52707 systemd[1]: Starting A high performance web server and a reverse proxy server...
Jan 27 06:54:02 i-068c30f995dc52707 systemd[1]: Started A high performance web server and a reverse proxy server.
```

`cat /etc/nginx/sites-enabled/default `  
```console
<...>
        ssl_certificate /etc/ssl/certs/localhost.crt;
        ssl_certificate_key //etc/ssl/certs/localhost.key;
<...>
```

`date`  
```console
Mon Jan 27 08:27:11 UTC 2025
```

`systemctl status chrony`  
```console
● chrony.service - chrony, an NTP client/server
     Loaded: loaded (/lib/systemd/system/chrony.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:chronyd(8)
             man:chronyc(1)
             man:chrony.conf(5)
```


#### 2. Fix date and time
`sudo -i && systemctl start chrony`  
`date`  
```console
Sat Jan 27 08:27:43 UTC 2024
```


``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

``  
```console
```

