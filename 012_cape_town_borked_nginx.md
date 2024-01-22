# "Cape Town": Borked Nginx

**Description:** There's an Nginx web server installed and managed by systemd. Running `curl -I 127.0.0.1:80` returns `curl: (7) Failed to connect to localhost port 80: Connection refused`, fix it so when you curl you get the default Nginx page.  

**Test:** `curl -Is 127.0.0.1:80|head -1` returns `HTTP/1.1 200 OK`  

---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin/agent/ && cat check.sh`  
```console
#!/usr/bin/bash
res=$(curl -Is 127.0.0.1:80|head -1)
res=$(echo $res|tr -d '\r')

if [[ "$res" = "HTTP/1.1 200 OK" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`curl -I 127.0.0.1:80`  
```console
curl: (7) Failed to connect to 127.0.0.1 port 80: Connection refused
```

`systemctl status nginx`  
```console
● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Mon 2024-01-22 18:52:22 UTC; 50s ago
    Process: 573 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
        CPU: 29ms

Jan 22 18:52:22 i-046269c6e9bdafe29 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
Jan 22 18:52:22 i-046269c6e9bdafe29 nginx[573]: nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/>
Jan 22 18:52:22 i-046269c6e9bdafe29 nginx[573]: nginx: configuration file /etc/nginx/nginx.conf test failed
Jan 22 18:52:22 i-046269c6e9bdafe29 systemd[1]: nginx.service: Control process exited, code=exited, status>
Jan 22 18:52:22 i-046269c6e9bdafe29 systemd[1]: nginx.service: Failed with result 'exit-code'.
Jan 22 18:52:22 i-046269c6e9bdafe29 systemd[1]: Failed to start The NGINX HTTP and reverse proxy server.
```


<details>

  <summary>journalctl -xe</summary>

```console
Jan 22 18:53:46 i-046269c6e9bdafe29 sudo[819]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Jan 22 18:53:46 i-046269c6e9bdafe29 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
░░ Subject: A start job for unit nginx.service has begun execution
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ A start job for unit nginx.service has begun execution.
░░ 
░░ The job identifier is 366.
Jan 22 18:53:46 i-046269c6e9bdafe29 nginx[822]: nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
Jan 22 18:53:46 i-046269c6e9bdafe29 nginx[822]: nginx: configuration file /etc/nginx/nginx.conf test failed
Jan 22 18:53:46 i-046269c6e9bdafe29 systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
░░ Subject: Unit process exited
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ An ExecStartPre= process belonging to unit nginx.service has exited.
░░ 
░░ The process' exit code is 'exited' and its exit status is 1.
Jan 22 18:53:46 i-046269c6e9bdafe29 systemd[1]: nginx.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ The unit nginx.service has entered the 'failed' state with result 'exit-code'.
Jan 22 18:53:46 i-046269c6e9bdafe29 systemd[1]: Failed to start The NGINX HTTP and reverse proxy server.
░░ Subject: A start job for unit nginx.service has failed
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░ 
░░ A start job for unit nginx.service has finished with a failure.
░░ 
░░ The job identifier is 366 and the job result is failed.
Jan 22 18:53:46 i-046269c6e9bdafe29 sudo[819]: pam_unix(sudo:session): session closed for user root
Jan 22 18:54:04 i-046269c6e9bdafe29 sudo[825]:    admin : TTY=pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/systemctl status nginx.service
Jan 22 18:54:04 i-046269c6e9bdafe29 sudo[825]: pam_unix(sudo:session): session opened for user root(uid=0) by (uid=1000)
Jan 22 18:54:04 i-046269c6e9bdafe29 sudo[825]: pam_unix(sudo:session): session closed for user root
```

</details>


#### 2. As we can see in system log query, there is a syntax error 
```console
Jan 22 18:53:46 i-046269c6e9bdafe29 nginx[822]: nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
```

<details>

  <summary>cat /etc/nginx/sites-enabled/default</summary>

```console
;
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```

</details>


```console
;
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
<...>
```


#### 3. Fix error - delete an unexpected sign `;` in 1st line
<details>

  <summary>cat /etc/nginx/sites-enabled/default</summary>

```console
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        # pass PHP scripts to FastCGI server
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
        #       fastcgi_pass unix:/run/php/php7.4-fpm.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#       listen 80;
#       listen [::]:80;
#
#       server_name example.com;
#
#       root /var/www/example.com;
#       index index.html;
#
#       location / {
#               try_files $uri $uri/ =404;
#       }
#}
```

</details>

```console
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
<...>
```


#### 4. Restart nginx daemon
`systemctl restart nginx && systemctl status nginx`  
```console
● nginx.service - The NGINX HTTP and reverse proxy server
     Loaded: loaded (/etc/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-01-22 18:56:00 UTC; 9s ago
    Process: 852 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 853 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 854 (nginx)
      Tasks: 2 (limit: 524)
     Memory: 2.4M
        CPU: 35ms
     CGroup: /system.slice/nginx.service
             ├─854 nginx: master process /usr/sbin/nginx
             └─855 nginx: worker process

Jan 22 18:56:00 i-046269c6e9bdafe29 systemd[1]: Starting The NGINX HTTP and reverse proxy server...
Jan 22 18:56:00 i-046269c6e9bdafe29 nginx[852]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 22 18:56:00 i-046269c6e9bdafe29 nginx[852]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 22 18:56:00 i-046269c6e9bdafe29 systemd[1]: Started The NGINX HTTP and reverse proxy server.
```


#### 5. Nginx service is started, but server is still not working
`curl -I 127.0.0.1:80`  
```console
HTTP/1.1 500 Internal Server Error
Server: nginx/1.18.0
Date: Mon, 22 Jan 2024 18:56:20 GMT
Content-Type: text/html
Content-Length: 177
Connection: close
```


#### 6. Let's check nginx.conf
<details>

  <summary>cat /etc/nginx/nginx.conf</summary>

```bash
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
```

</details>


`nginx -t`  
```console
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```


#### 7. Nginx.conf seems like OK, let's check nginx logs
`cd /var/log/nginx && ls -la`  
```console
total 234260
drwxr-xr-x 2 root     adm       4096 Sep 11  2022 .
drwxr-xr-x 9 root     root      4096 Jan 22 18:52 ..
-rw-r----- 1 www-data adm  239657447 Jan 22 18:57 access.log
-rw-r----- 1 www-data adm     209058 Jan 22 18:57 error.log
```

`tail error.log`  
```console
2022/09/11 17:02:07 [emerg] 6066#6066: unexpected ";" in /etc/nginx/sites-enabled/default:1
2022/09/11 17:07:03 [emerg] 6146#6146: unexpected ";" in /etc/nginx/sites-enabled/default:1
2024/01/22 18:52:22 [emerg] 573#573: unexpected ";" in /etc/nginx/sites-enabled/default:1
2024/01/22 18:53:46 [emerg] 822#822: unexpected ";" in /etc/nginx/sites-enabled/default:1
2024/01/22 18:56:00 [alert] 854#854: socketpair() failed while spawning "worker process" (24: Too many open files)
2024/01/22 18:56:00 [emerg] 855#855: eventfd() failed (24: Too many open files)
2024/01/22 18:56:00 [alert] 855#855: socketpair() failed (24: Too many open files)
2024/01/22 18:56:20 [crit] 855#855: *1 open() "/var/www/html/index.nginx-debian.html" failed (24: Too many open files), client: 127.0.0.1, server: _, request: "HEAD / HTTP/1.1", host: "127.0.0.1"
2024/01/22 18:57:22 [crit] 855#855: *2 open() "/var/www/html/index.nginx-debian.html" failed (24: Too many open files), client: 127.0.0.1, server: _, request: "HEAD / HTTP/1.1", host: "127.0.0.1"
2024/01/22 18:57:30 [crit] 855#855: *3 open() "/var/www/html/index.nginx-debian.html" failed (24: Too many open files), client: 127.0.0.1, server: _, request: "HEAD / HTTP/1.1", host: "127.0.0.1"
```


#### 8. There is an error "Too Many Open Files"
```console
open() "/var/www/html/index.nginx-debian.html" failed (24: Too many open files)
```

##### To find out the maximum number of files that each processe can open, we can use the ulimit command with the -n (open files) option
`sudo ulimit -n`  
```console
1024
```
##### It seems like OK

#### Let's find out the maximum number of files that NGINX processe can open
`cat /etc/systemd/system/nginx.service`  
```console
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
LimitNOFILE=10

[Install]
WantedBy=multi-user.target
```

##### Value `LimitNOFILE=10` is too low and not enough, let's encrese up to 1024
`vi /etc/systemd/system/nginx.service`  
```console
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
LimitNOFILE=1024

[Install]
WantedBy=multi-user.target
```

`systemctl daemon-reload && systemctl restart nginx`  

`curl -I 127.0.0.1:80`  
```console
HTTP/1.1 200 OK
Server: nginx/1.18.0
Date: Mon, 22 Jan 2024 19:06:46 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Sun, 11 Sep 2022 15:58:42 GMT
Connection: keep-alive
ETag: "631e05b2-264"
Accept-Ranges: bytes
```

`curl -Is 127.0.0.1:80|head -1`  
```console
HTTP/1.1 200 OK
```

`cd /home/admin/agent/ && ./check.sh`  
```console
OK
```
