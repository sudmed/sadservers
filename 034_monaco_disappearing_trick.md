# "Monaco": Disappearing Trick

**Description:** There is a web server on `:5000` with a form. POSTing the correct form password into this web service will return a secret.  

Save this secret provided by the web page (not the password you sent to it) to `/home/admin/mysolution`, for example: `echo "SecretFromWebSite" > ~/mysolution`.  

**TIP:** a developer worked on the web server code in this VM, using the same 'admin' account.  

**Test:** `md5sum /home/admin/mysolution` returns `a250aa19f16dda6f9fcef286f035ec4b`.  


---

### Solution:
#### 1. Reconnaissance on the server
`history`  
```console
    1  2023-09-20T15:57:57 > /home/admin/.bash_history
    2  2023-09-20T15:58:02 exit
```

`ls -la`  
```console
total 44
drwxr-xr-x 7 admin admin 4096 Feb 13 12:22 .
drwxr-xr-x 3 root  root  4096 Sep 17 16:44 ..
drwx------ 3 admin admin 4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin   77 Feb 13 12:23 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Sep 20 15:56 .config
drwxr-xr-x 8 admin admin 4096 Sep 30 17:45 .git
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root  4096 Sep 30 17:45 agent
```

`cat agent/check.sh`  
```bash
#!/bin/bash

expected_checksum="a250aa19f16dda6f9fcef286f035ec4b"
actual_checksum=$(md5sum /home/admin/mysolution | awk '{print $1}')

if [[ "$actual_checksum" == "$expected_checksum" ]]; then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`cat .git/config`  
```console
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
```

`cat .git/index`  
```console
DIRCe^,e^&=

7+#
d"webserver_v1.pyTREE1 0
~ӵsS\Zc(m9VH`
             %s#a>Nm
```

`git status`  
```console
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    webserver_v1.py

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .bash_history
        .bash_logout
        .bashrc
        .config/
        .profile
        .ssh/
        agent/

no changes added to commit (use "git add" and/or "git commit -a")
```
##### HAHAHA! There is one deleted file, let's restore it

`git restore .`  
`ls -la`  
```console
total 48
drwxr-xr-x 7 admin admin 4096 Feb 13 12:39 .
drwxr-xr-x 3 root  root  4096 Sep 17 16:44 ..
drwx------ 3 admin admin 4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin 1605 Feb 13 12:39 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Sep 20 15:56 .config
drwxr-xr-x 8 admin admin 4096 Feb 13 12:39 .git
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root  4096 Sep 30 17:45 agent
-rwxr-xr-x 1 admin admin  712 Feb 13 12:39 webserver_v1.py
```

`cat webserver_v1.py`  
```console
from flask import Flask, request
import os

app = Flask(__name__)


@app.route('/', methods=['GET', 'POST'])
def home():
    if request.method == 'POST':
        entered_password = request.form.get('password')
        super_secret_password = os.environ.get('SUPERSECRETPASSWORD')

        if entered_password == super_secret_password:
            return 'Access granted!'
            # DO STUFF HERE

        return 'Access denied!'

    return '''
        <form method="POST">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password">
            <input type="submit" value="Submit">
        </form>
    '''


if __name__ == '__main__':
    app.run()
```

`curl localhost:5000`  
```console
        <form method="POST">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password">
            <input type="submit" value="Submit">
        </form>
```

`curl -v localhost:5000`  
```console
*   Trying 127.0.0.1:5000...
* Connected to localhost (127.0.0.1) port 5000 (#0)
> GET / HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: Werkzeug/3.0.0 Python/3.9.2
< Date: Tue, 13 Feb 2024 12:33:32 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 217
< Connection: close
< 

        <form method="POST">
            <label for="password">Password:</label>
            <input type="password" id="password" name="password">
            <input type="submit" value="Submit">
        </form>
* Closing connection 0
```

`curl -X POST -d "password=SUPERSECRETPASSWORD" localhost:5000`  
```console
Access denied!
```
##### We can see interesting construction in the `webserver_v1.py` file: `super_secret_password = os.environ.get('SUPERSECRETPASSWORD')`.  
##### This code snippet retrieves the value of an environment variable named 'SUPERSECRETPASSWORD' using the `os.environ.get()` method.
##### Let's try to get it from terminal

`echo $SUPERSECRETPASSWORD`  
```console
<empty>
```
##### Searching in `~/.bashrc` or `~/.profile` also failed. 
##### Let's inspect the file descriptor-based environment

`lsof -i :5000`  
```console
COMMAND PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python3 582 admin    3u  IPv4  11651      0t0  TCP localhost:5000 (LISTEN)
```

`cat  /proc/582/environ`  
```console
LANG=C.UTF-8PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binHOME=/home/adminLOGNAME=adminUSER=adminSHELL=/bin/bashINVOCATION_ID=b41d9ab7672e413ebeaf6a8c9761c5e6JOURNAL_STREAM=8:11434SUPERSECRETPASSWORD=bdFBkE4suaCy
```
##### Here you are! `SUPERSECRETPASSWORD=bdFBkE4suaCy`
##### Let's check the password

`curl -X POST -d "password=bdFBkE4suaCy" localhost:5000`  
```console
Access granted! Secret is QhyjuI98BBvf
```

`echo "QhyjuI98BBvf" > ~/mysolution`  


#### 2. Validate the task
`md5sum /home/admin/mysolution`  
```console
a250aa19f16dda6f9fcef286f035ec4b  /home/admin/mysolution
```

`./agent/check.sh`  
```console
OK
```
