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

`systemctl status etcd`  
```console
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/lib/systemd/system/etcd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-01-27 08:30:06 UTC; 2s ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 919 (etcd)
      Tasks: 7 (limit: 521)
     Memory: 5.1M
        CPU: 108ms
     CGroup: /system.slice/etcd.service
             └─919 /usr/bin/etcd --cert-file /etc/ssl/certs/localhost.crt --key-file /etc/ssl/certs/localhost.key --advertise-client-urls=https://localhost:2379 --listen-client-urls=https://localhost:2379

Jan 27 08:30:04 i-09d929399be4dc1ae etcd[919]: enabled capabilities for version 3.3
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: 8e9e05c52164694d is starting a new election at term 19
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: 8e9e05c52164694d became candidate at term 20
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: 8e9e05c52164694d received MsgVoteResp from 8e9e05c52164694d at term 20
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: 8e9e05c52164694d became leader at term 20
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: raft.node: 8e9e05c52164694d elected leader 8e9e05c52164694d at term 20
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: published {Name:ip-10-0-0-138 ClientURLs:[https://localhost:2379]} to cluster cdf818194e3a8c32
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: ready to serve client requests
Jan 27 08:30:06 i-09d929399be4dc1ae systemd[1]: Started etcd - highly-available key value store.
Jan 27 08:30:06 i-09d929399be4dc1ae etcd[919]: serving client requests on 127.0.0.1:2379
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

`etcdctl get foo`  
```console
Error:  client: response is invalid json. The endpoint is probably not valid etcd cluster endpoint.
```


#### 3. Let's check URL with flags --verbose and --insecure
`curl -vk https://localhost:2379/v2/keys/foo`  
```console
*   Trying 127.0.0.1:2379...
* Connected to localhost (127.0.0.1) port 2379 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=localhost
*  start date: Dec 31 00:02:48 2022 GMT
*  expire date: Jan 30 00:02:48 2023 GMT
*  issuer: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=localhost
*  SSL certificate verify result: certificate has expired (10), continuing anyway.
> GET /v2/keys/foo HTTP/1.1
> Host: localhost:2379
> User-Agent: curl/7.74.0
> Accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< Server: nginx/1.18.0
< Date: Sat, 27 Jan 2024 12:06:42 GMT
< Content-Type: text/html
< Content-Length: 153
< Connection: keep-alive
< 
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>
* Connection #0 to host localhost left intact
```


#### 4. Let's see NGINX logs
`tail /var/log/nginx/access.log`  
```console
127.0.0.1 - - [27/Jan/2025:12:04:20 +0000] "PRI * HTTP/2.0" 400 157 "-" "-"
127.0.0.1 - - [27/Jan/2025:12:04:46 +0000] "PRI * HTTP/2.0" 400 157 "-" "-"
127.0.0.1 - - [27/Jan/2024:12:05:25 +0000] "PRI * HTTP/2.0" 400 157 "-" "-"
127.0.0.1 - - [27/Jan/2024:12:06:19 +0000] "GET /v2/members HTTP/1.1" 400 255 "-" "Go-http-client/1.1"
127.0.0.1 - - [27/Jan/2024:12:06:19 +0000] "GET /v2/keys/foo?quorum=false&recursive=false&sorted=false HTTP/1.1" 400 255 "-" "Go-http-client/1.1"
127.0.0.1 - - [27/Jan/2024:12:06:40 +0000] "PRI * HTTP/2.0" 400 157 "-" "-"
127.0.0.1 - - [27/Jan/2024:12:06:42 +0000] "GET /v2/keys/foo HTTP/1.1" 404 153 "-" "curl/7.74.0"
```


#### 5. Let's see processes and ports
`netstat -putana | grep nginx`  
```console
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      639/nginx: master p
```

`netstat -putana | grep 2379`  
```console
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      566/etcd
```

`ps aux | grep etcd`  
```console
etcd         566  0.8  5.1 11729040 23880 ?      Ssl  12:03   0:02 /usr/bin/etcd --cert-file /etc/ssl/certs/localhost.crt --key-file /etc/ssl/certs/localhost.key --advertise-client-urls=https://localhost:2379 --listen-client-urls=https://localhost:2379
root         926  0.0  0.1   5268   704 pts/0    R<+  12:07   0:00 grep etcd
```


#### 6. Let's see iptables rules
`iptables -L -v -n`  
```console
Chain INPUT (policy ACCEPT 1061 packets, 115K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 918 packets, 484K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0
```

`iptables -S`  
```console
-P INPUT ACCEPT
-P FORWARD DROP
-P OUTPUT ACCEPT
-N DOCKER
-N DOCKER-ISOLATION-STAGE-1
-N DOCKER-ISOLATION-STAGE-2
-N DOCKER-USER
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
```

`iptables -t nat -S`  
```console
-P PREROUTING ACCEPT
-P INPUT ACCEPT
-P OUTPUT ACCEPT
-P POSTROUTING ACCEPT
-N DOCKER
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT -o lo -p tcp -m tcp --dport 2379 -j REDIRECT --to-ports 443
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A DOCKER -i docker0 -j RETURN
```


#### 7. Delete suspicious rule with redirect
`iptables -t nat -L --line-numbers`  
```console
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source               destination         
1    DOCKER     all  --  anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination         
1    REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:2379 redir ports 443
2    DOCKER     all  --  anywhere            !ip-127-0-0-0.us-east-2.compute.internal/8  ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
num  target     prot opt source               destination         
1    MASQUERADE  all  --  ip-172-17-0-0.us-east-2.compute.internal/16  anywhere            

Chain DOCKER (2 references)
num  target     prot opt source               destination         
1    RETURN     all  --  anywhere             anywhere
```

`iptables -t nat -D OUTPUT 1`  
`iptables -t nat -L -v -n --line-numbers`  
```console
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1       16   960 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           

Chain DOCKER (2 references)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0
```


#### 8. Now we can see some meaningful output
`curl -vk https://localhost:2379/v2/keys/foo`  
```console
*   Trying 127.0.0.1:2379...
* Connected to localhost (127.0.0.1) port 2379 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=localhost
*  start date: Dec 31 00:02:48 2022 GMT
*  expire date: Jan 30 00:02:48 2023 GMT
*  issuer: C=AU; ST=Some-State; O=Internet Widgits Pty Ltd; CN=localhost
*  SSL certificate verify result: certificate has expired (10), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55a20d53ead0)
> GET /v2/keys/foo HTTP/2
> Host: localhost:2379
> user-agent: curl/7.74.0
> accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 250)!
< HTTP/2 200 
< content-type: application/json
< x-etcd-cluster-id: cdf818194e3a8c32
< x-etcd-index: 21
< x-raft-index: 39
< x-raft-term: 19
< content-length: 88
< date: Sat, 27 Jan 2024 12:11:16 GMT
< 
{"action":"get","node":{"key":"/foo","value":"bar","modifiedIndex":4,"createdIndex":4}}
* Connection #0 to host localhost left intact
```

`curl -k https://localhost:2379/v2/keys/foo`  
```console
{"action":"get","node":{"key":"/foo","value":"bar","modifiedIndex":4,"createdIndex":4}}
```

`curl https://localhost:2379/v2/keys/foo`  
```console
curl: (60) SSL certificate problem: certificate has expired
More details here: https://curl.se/docs/sslcerts.html
curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```


#### 9. Fix SSL certificate problem
##### 9.1. Generate new self signed certificate
`openssl genrsa -out cert.key 2048`  
```console
Generating RSA private key, 2048 bit long modulus (2 primes)
.............................+++++
.....................+++++
e is 65537 (0x010001)
```

`openssl req -new -key cert.key -out cert.csr`  
```console
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:RU
State or Province Name (full name) [Some-State]:Moscow
Locality Name (eg, city) []:Moscow
Organization Name (eg, company) [Internet Widgits Pty Ltd]:OOO ROGA & KOPYTA
Organizational Unit Name (eg, section) []:DEVOPS DEP
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:root@localhost
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

`openssl x509 -req -days 3650 -in cert.csr -signkey cert.key -out cert.crt`  
```console
Signature ok
subject=C = RU, ST = Moscow, L = Moscow, O = OOO ROGA & KOPYTA, OU = DEVOPS DEP, CN = localhost, emailAddress = root@localhost
Getting Private key
```

##### 9.2. Check new certificates and fix permissions
`ls -la`  
```console
total 48
drwx------  5 root root 4096 Jan 27 12:13 .
drwxr-xr-x 18 root root 4096 Jan 27 12:03 ..
-rw-------  1 root root    5 Jan  3 01:18 .bash_history
-rw-r--r--  1 root root  648 Jan  1  2026 .bashrc
drwx------  3 root root 4096 Aug 31  2022 .config
drwxr-xr-x  3 root root 4096 Aug 31  2022 .local
-rw-r--r--  1 root root   23 Dec 31  2022 .profile
-rw-r--r--  1 root root   66 Dec 31  2022 .selected_editor
drwx------  2 root root 4096 Aug 31  2022 .ssh
-rw-r--r--  1 root root 1338 Jan 27 12:13 cert.crt
-rw-r--r--  1 root root 1066 Jan 27 12:13 cert.csr
-rw-------  1 root root 1675 Jan 27 12:11 cert.key
```

`chmod a+r cert.key`  
`ls -la`  
```console
total 48
drwx------  5 root root 4096 Jan 27 12:13 .
drwxr-xr-x 18 root root 4096 Jan 27 12:03 ..
-rw-------  1 root root    5 Jan  3 01:18 .bash_history
-rw-r--r--  1 root root  648 Jan  1  2026 .bashrc
drwx------  3 root root 4096 Aug 31  2022 .config
drwxr-xr-x  3 root root 4096 Aug 31  2022 .local
-rw-r--r--  1 root root   23 Dec 31  2022 .profile
-rw-r--r--  1 root root   66 Dec 31  2022 .selected_editor
drwx------  2 root root 4096 Aug 31  2022 .ssh
-rw-r--r--  1 root root 1338 Jan 27 12:13 cert.crt
-rw-r--r--  1 root root 1066 Jan 27 12:13 cert.csr
-rw-r--r--  1 root root 1675 Jan 27 12:11 cert.key
```


##### 9.3. Copy new certificates to destination directory
`ls -ltr /etc/ssl/certs/`  
```console
-rw-r--r-- 1 root root   1704 Dec 31  2022  localhost.key
-rw-r--r-- 1 root root   1302 Dec 31  2022  localhost.crt
lrwxrwxrwx 1 root root     40 Dec 31  2022  localhost.pem -> /usr/share/ca-certificates/localhost.crt
lrwxrwxrwx 1 root root     13 Dec 31  2022  bfb60d90.0 -> localhost.crt
-rw-r--r-- 1 root root 201615 Dec 31  2022  ca-certificates.crt
```

`cp cert.crt /etc/ssl/certs/localhost.crt`  
`cp cert.key /etc/ssl/certs/localhost.key`  


##### 9.4. Restart ETCD daemon
`systemctl restart etcd2.service`  
`systemctl status etcd2.service`  
```console
● etcd.service - etcd - highly-available key value store
     Loaded: loaded (/lib/systemd/system/etcd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-01-27 12:31:20 UTC; 9s ago
       Docs: https://etcd.io/docs
             man:etcd
   Main PID: 927 (etcd)
      Tasks: 8 (limit: 521)
     Memory: 4.9M
        CPU: 139ms
     CGroup: /system.slice/etcd.service
             └─927 /usr/bin/etcd --cert-file /etc/ssl/certs/localhost.crt --key-file /etc/ssl/certs/localhost.key --advertise-client-urls=https://localhost:2379 --listen-client-urls=https://localhost:2379

Jan 27 12:31:19 i-069899b7a1eff8778 etcd[927]: enabled capabilities for version 3.3
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: 8e9e05c52164694d is starting a new election at term 19
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: 8e9e05c52164694d became candidate at term 20
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: 8e9e05c52164694d received MsgVoteResp from 8e9e05c52164694d at term 20
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: 8e9e05c52164694d became leader at term 20
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: raft.node: 8e9e05c52164694d elected leader 8e9e05c52164694d at term 20
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: published {Name:ip-10-0-0-138 ClientURLs:[https://localhost:2379]} to cluster cdf818194e3a8c32
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: ready to serve client requests
Jan 27 12:31:20 i-069899b7a1eff8778 systemd[1]: Started etcd - highly-available key value store.
Jan 27 12:31:20 i-069899b7a1eff8778 etcd[927]: serving client requests on 127.0.0.1:2379
```

`curl -vk https://localhost:2379/v2/keys/foo`  
```console
*   Trying 127.0.0.1:2379...
* Connected to localhost (127.0.0.1) port 2379 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=RU; ST=Moscow; L=Moscow; O=Internet Widgits Pty Ltd; CN=localhost
*  start date: Jan 27 12:30:32 2024 GMT
*  expire date: Jan 24 12:30:32 2034 GMT
*  issuer: C=RU; ST=Moscow; L=Moscow; O=Internet Widgits Pty Ltd; CN=localhost
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x558ab0466ad0)
> GET /v2/keys/foo HTTP/2
> Host: localhost:2379
> user-agent: curl/7.74.0
> accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS == 250)!
< HTTP/2 200 
< content-type: application/json
< x-etcd-cluster-id: cdf818194e3a8c32
< x-etcd-index: 22
< x-raft-index: 41
< x-raft-term: 20
< content-length: 88
< date: Sat, 27 Jan 2024 12:32:27 GMT
< 
{"action":"get","node":{"key":"/foo","value":"bar","modifiedIndex":4,"createdIndex":4}}
* Connection #0 to host localhost left intact
```

`curl -k https://localhost:2379/v2/keys/foo`  
```console
{"action":"get","node":{"key":"/foo","value":"bar","modifiedIndex":4,"createdIndex":4}}
```


#### 10. Validate the task
`etcdctl get foo`  
```console
bar
```

`cd ~/agent && ./check.sh`  
```console
OK
```

