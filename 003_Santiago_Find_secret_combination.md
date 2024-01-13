# "Santiago": Find the secret combination

**Description**: Alice the spy has hidden a secret number combination, find it using these instructions:  
1) Find the number of lines with occurrences of the string `Alice` (case sensitive) in the *.txt files in the `/home/admin` directory.  
2) There's a file where Alice appears exactly once. In that file, in the line after that "Alice" occurrence there's a number.  

Write both numbers consecutively as one (no new line or spaces) to the solution file. For example if the first number from 1) is 11 and the second 22, you can do `echo -n 11 > /home/admin/solution; echo 22 >> /home/admin/solution` or `echo "1122" > /home/admin/solution`.  

Test: Running `md5sum /home/admin/solution` returns `d80e026d18a57b56bddf1d99a8a491f9` (just a way to verify the solution without giving it away).  

---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin`
`ls -la`
```console
total 2000
drwxr-xr-x 5 admin admin   4096 Nov 20  2022 .
drwxr-xr-x 3 root  root    4096 Aug 31  2022 ..
-rw------- 1 admin admin      0 Nov 21  2022 .bash_history
-rw-r--r-- 1 admin admin    220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin   3526 Aug  4  2021 .bashrc
-rw------- 1 admin admin     51 Nov 20  2022 .lesshst
drwxr-xr-x 3 admin admin   4096 Nov 20  2022 .local
-rw-r--r-- 1 admin admin    807 Aug  4  2021 .profile
drwx------ 2 admin admin   4096 Aug 31  2022 .ssh
-rw-r--r-- 1 admin admin 174313 Nov 20  2022 11-0.txt
-rw-r--r-- 1 admin admin 772181 Nov 20  2022 1342-0.txt
-rw-r--r-- 1 admin admin 607430 Nov 20  2022 1661-0.txt
-rw-r--r-- 1 admin admin 448819 Nov 20  2022 84-0.txt
drwxr-xr-x 2 admin admin   4096 Nov 21  2022 agent
-rw-r--r-- 1 admin admin      0 Nov 21  2022 solution
```

```bash
admin@i-068844a34e82f4dae:~$ grep -rwc "Alice" /home/admin/*.txt
/home/admin/11-0.txt:397
/home/admin/1342-0.txt:1
/home/admin/1661-0.txt:12
/home/admin/84-0.txt:0
```

```bash
admin@i-068844a34e82f4dae:~$ grep -rc "Alice" /home/admin/*.txt | awk -F ':' '{total += $2} END {print total}'
411
```

```bash
admin@i-068844a34e82f4dae:~$ grep -l "Alice" /home/admin/1342-0.txt | xargs awk '/Alice/ {getline; print $1}'
156
```

```bash
echo "411156" > /home/admin/solution
admin@i-068844a34e82f4dae:~$ cat /home/admin/solution
411156
admin@i-068844a34e82f4dae:~$ md5sum /home/admin/solution
d80e026d18a57b56bddf1d99a8a491f9  /home/admin/solution
```

#### [Issue #50](https://github.com/fduran/sadservers/issues/50)
