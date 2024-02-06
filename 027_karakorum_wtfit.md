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
```console
<no output>
```

`/home/admin/wtfit`  
```console
ERROR: can't open config file
```

`strace ./wtfit`  
```console
<...>
openat(AT_FDCWD, "/home/admin/wtfitconfig.conf", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
write(1, "ERROR: can't open config file\n", 30ERROR: can't open config file
```
