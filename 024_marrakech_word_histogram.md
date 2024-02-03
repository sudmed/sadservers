# "Marrakech": Word Histogram

**Description:** Find in the file `frankestein.txt` the second most frequent word and save in UPPER (capital) case in the `/home/admin/mysolution file`.  

A word is a string of characters separated by space or newlines or punctuation symbols `.,:;`. Disregard case ('The', 'the' and 'THE' is the same word) and for simplification consider the apostrophe as another character not as punctuation ("it's" would be a word, distinct from "it" and "is"). Also disregard plurals ("car" and "cars" are different words) and other word variations (don't do "stemming").  

We are providing a shorter `test.txt` file where the second most common word in upper case is "WORLD", so we could save this solution as: `echo "WORLD" > /home/admin/mysolution`.  

This problem can be done with a programming language (Python, Golang and sqlite3) or with common Linux utilities.  

**Test:** `echo "SOLUTION" | md5sum` returns `19bf32b8725ec794d434280902d78e18`.  

See `/home/admin/agent/check.sh` for the test that "Check My Solution" runs.  


---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 480
drwxr-xr-x 5 admin admin   4096 Dec 23 14:43 .
drwxr-xr-x 3 root  root    4096 Nov 22 01:04 ..
drwx------ 3 admin admin   4096 Nov 22 01:06 .ansible
-rw-r--r-- 1 admin admin    220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin   3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin    807 Mar 27  2022 .profile
drwx------ 2 admin admin   4096 Nov 22 01:04 .ssh
-rw-r--r-- 1 admin admin   1301 Dec 23 14:43 README.txt
drwxr-xr-x 2 admin admin   4096 Dec 23 14:42 agent
-rw-r--r-- 1 root  root  448642 Dec 23 14:42 frankestein.txt
-rw-r--r-- 1 root  root     177 Dec 23 14:42 test.txt
```

`cat /home/admin/agent/check.sh`  
```bash
#!/usr/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

res=$(md5sum /home/admin/mysolution |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "19bf32b8725ec794d434280902d78e18" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`cat test.txt`  
```text
A sad world. The new thing in the world is the occurrence of new things,
given the natural fallacy in the world.
This cannot be overstated. It's impossible to know these things.
```


<details>

  <summary>cat frankestein.txt</summary>

```text
The Project Gutenberg eBook of Frankenstein, by Mary Wollstonecraft Shelley

This eBook is for the use of anyone anywhere in the United States and
most other parts of the world at no cost and with almost no restrictions
whatsoever. You may copy it, give it away or re-use it under the terms
of the Project Gutenberg License included with this eBook or online at
www.gutenberg.org. If you are not located in the United States, you
will have to check the laws of the country where you are located before
using this eBook.

Title: Frankenstein
       or, The Modern Prometheus

Author: Mary Wollstonecraft Shelley

Release Date: October 31, 1993 [eBook #84]
[Most recently updated: December 2, 2022]

Language: English

Character set encoding: UTF-8

Produced by: Judith Boss, Christy Phillips, Lynn Hanninen and David Meltzer. HTML version by Al Haines.
Further corrections by Menno de Leeuw.

*** START OF THE PROJECT GUTENBERG EBOOK FRANKENSTEIN ***




Frankenstein;

or, the Modern Prometheus

by Mary Wollstonecraft (Godwin) Shelley


 CONTENTS

 Letter 1
 Letter 2
 Letter 3
 Letter 4
 Chapter 1
 Chapter 2
 Chapter 3
 Chapter 4
 Chapter 5
 Chapter 6
 Chapter 7
 Chapter 8
 Chapter 9
 Chapter 10
 Chapter 11
 Chapter 12
 Chapter 13
 Chapter 14
 Chapter 15
 Chapter 16
 Chapter 17
 Chapter 18
 Chapter 19
 Chapter 20
 Chapter 21
 Chapter 22
 Chapter 23
 Chapter 24
```

</details>


#### 2. Try on the second text
`cat test.txt | tr '[:upper:]' '[:lower:]' | tr -s '[:punct:]' ' ' | tr ' ' '\n' | grep -v '^$' | sort | uniq -c | sort -nr | awk 'NR==2{print toupper($2)}'`  
```console
WORLD
```
##### Fine, it works.


#### 3. 
`cat frankestein.txt | tr '[:upper:]' '[:lower:]' | tr -s '[:punct:]' ' ' | tr ' ' '\n' | grep -v '^$' | sort | uniq -c | sort -nr | awk 'NR==2{print toupper($2)}'`  
```console
AND
```

`echo "AND" | md5sum`  
```console
19bf32b8725ec794d434280902d78e18  -
```
##### Fine, it works.


#### 4. Create some script
`vi script.sh`  
```bash
input_file="frankestein.txt"
output_file="/home/admin/mysolution"

# Используем команды grep, tr, sort, uniq и awk для обработки файла
# сначала преобразуем текст в нижний регистр и разделим на слова,
# затем находим частоту каждого слова, сортируем их и извлекаем второе слово
# наконец, преобразуем его в верхний регистр и записываем в файл

cat $input_file | tr '[:upper:]' '[:lower:]' | tr -s '[:punct:]' ' ' | tr ' ' '\n' | grep -v '^$' | sort | uniq -c | sort -nr | awk 'NR==2{print toupper($2)}' > $output_file
```
`chmod +x script.sh`  
`./script.sh`  



#### 5. Validate the task
`./home/admin/agent/check.sh`  
```console
OK
```

