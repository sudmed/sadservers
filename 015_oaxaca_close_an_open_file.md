# "Oaxaca": Close an Open File

**Description:** The file `/home/admin/somefile` is open for writing by some process. Close this file without killing the process.  

**Test:** `lsof /home/admin/somefile` returns nothing.


---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin/ && ls -la`  
```console
total 40
drwxr-xr-x 5 admin admin 4096 Nov 30  2022 .
drwxr-xr-x 3 root  root  4096 Aug 31  2022 ..
-rw------- 1 admin admin    0 Nov 30  2022 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3557 Nov 30  2022 .bashrc
drwxr-xr-x 3 admin admin 4096 Nov 29  2022 .local
-rw-r--r-- 1 admin admin  807 Nov 30  2022 .profile
-rw-r--r-- 1 admin admin   66 Nov 29  2022 .selected_editor
drwx------ 2 admin admin 4096 Aug 31  2022 .ssh
drwxr-xr-x 2 admin admin 4096 Nov 30  2022 agent
-rwxr-xr-x 1 admin admin   43 Nov 30  2022 openfile.sh
-rw-r--r-- 1 admin admin    0 Jan 25 05:51 somefile
```

`cat openfile.sh`  
```console
#!/bin/bash
exec 66> /home/admin/somefile
```

`cd ~/agent && cat check.sh `  
```console
#!/usr/bin/bash
res=$(lsof /home/admin/somefile)
res=$(echo $res|tr -d '\r')

if [[ "$res" = "" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```


#### 2. Identify the file descriptor corresponding to the file
`lsof /home/admin/somefile`  
```console
COMMAND PID  USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
bash    806 admin   77w   REG  259,1        0 272875 /home/admin/somefile
```


#### 3. Close the file descriptor 77
`eval "exec 77>&-"`  
```console
<no output>
```


#### 4. Validate the task
`lsof /home/admin/somefile`  
```console
<no output>
```

`cd ~/agent && ./check.sh `  
```console
OK
```
