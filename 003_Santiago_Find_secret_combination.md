## "Santiago": Find the secret combination

Description: Alice the spy has hidden a secret number combination, find it using these instructions:  
1) Find the number of lines with occurrences of the string `Alice` (case sensitive) in the *.txt files in the `/home/admin` directory.  
2) There's a file where Alice appears exactly once. In that file, in the line after that "Alice" occurrence there's a number.  
Write both numbers consecutively as one (no new line or spaces) to the solution file. For example if the first number from 1) is 11 and the second 22, you can do `echo -n 11 > /home/admin/solution; echo 22 >> /home/admin/solution` or `echo "1122" > /home/admin/solution`.  
  
Test: Running `md5sum /home/admin/solution` returns `d80e026d18a57b56bddf1d99a8a491f9` (just a way to verify the solution without giving it away).  

```bash
admin@i-068844a34e82f4dae:~$ grep -rwc "Alice" /home/admin/*.txt
/home/admin/11-0.txt:397
/home/admin/1342-0.txt:1
/home/admin/1661-0.txt:12
/home/admin/84-0.txt:0
```

```bash
admin@i-068844a34e82f4dae:~$ grep -rwc "Alice" /home/admin/*.txt | awk -F ':' '{total += $2} END {print total}'
410
```

```bash
admin@i-068844a34e82f4dae:~$ grep -l "Alice" /home/admin/1342-0.txt | xargs awk '/Alice/ {getline; print $1}'
156
```

```bash
echo "410156" > /home/admin/solution
admin@i-068844a34e82f4dae:~$ cat /home/admin/solution
410156
admin@i-068844a34e82f4dae:~$ md5sum /home/admin/solution
d80e026d18a57b56bddf1d99a8a491f9  /home/admin/solution
```
