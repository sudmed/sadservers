# "Lhasa": Easy Math

**Description**: There's a file `/home/admin/scores.txt` with two columns (imagine the first number is a counter and the second one is a test score for example).  

Find the average (more precisely; the arithmetic mean: sum of numbers divided by how many numbers are there) of the numbers in the second column (find the average score).  

Use exaclty two digits to the right of the decimal point. i. e., use exaclty two "decimal digits" without any rounding. Eg: if average = 21.349 , the solution is 21.34. If average = 33.1 , the solution is 33.10.  

Save the solution in the `/home/admin/solution` file, for example: `echo "123.45" > ~/solution`  

**Tip:** There's bc, Python3, Golang and sqlite3 installed in this VM.  

**Test:** `md5sum /home/admin/solution` returns `6d4832eb963012f6d8a71a60fac77168` solution.  

---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 40
drwxr-xr-x 5 admin admin 4096 Nov 21 02:26 .
drwxr-xr-x 3 root  root  4096 Sep 17 16:44 ..
drwx------ 3 admin admin 4096 Nov 19 14:56 .ansible
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Sep 17 16:44 .ssh
-rw-r--r-- 1 admin admin 1237 Nov 21 02:26 README.txt
drwxr-xr-x 2 admin root  4096 Nov 21 02:26 agent
-rw-r--r-- 1 admin admin  692 Nov 21 02:26 scores.txt
-rw-r--r-- 1 admin admin    0 Nov 21 02:26 solution
```


<details>

  <summary>cat /home/admin/scores.txt </summary>

```console
1 7.4
2 0.4
3 1.6
4 6.2
5 7.6
6 7.7
7 5.6
8 4.4
9 8.0
10 7.0
11 3.1
12 5.1
13 3.2
14 0.3
15 2.2
16 6.7
17 0.8
18 8.3
19 1.8
20 9.0
21 9.2
22 9.6
23 5.9
24 8.4
25 4.0
26 2.3
27 4.2
28 7.4
29 8.9
30 5.3
31 7.9
32 6.6
33 9.7
34 3.6
35 3.9
36 8.8
37 1.3
38 3.4
39 3.7
40 1.6
41 9.1
42 2.0
43 0.9
44 3.1
45 3.4
46 2.8
47 3.4
48 7.1
49 7.9
50 1.5
51 6.8
52 6.3
53 0.5
54 0.0
55 7.0
56 6.7
57 7.4
58 8.7
59 9.5
60 4.7
61 3.6
62 7.0
63 9.1
64 6.2
65 5.8
66 7.1
67 5.8
68 1.1
69 5.6
70 4.8
71 4.2
72 2.4
73 2.9
74 8.4
75 7.3
76 0.2
77 0.0
78 8.5
79 0.2
80 9.8
81 2.9
82 9.0
83 3.5
84 1.9
85 5.0
86 2.2
87 6.1
88 9.4
89 5.0
90 9.7
91 3.5
92 4.4
93 3.5
94 4.2
95 8.0
96 5.5
97 7.2
98 4.8
99 4.7
100 9.0
```

</details>


`cat ~/agent/check.sh`  
```bash
#!/usr/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)
res=$(md5sum /home/admin/solution |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "6d4832eb963012f6d8a71a60fac77168" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```


#### 2. Find the average
`awk '{ sum += $2 } END { printf "%.2f\n", sum/NR }' /home/admin/scores.txt`  
```console
5.20
```

`awk '{ sum += $2 } END { printf "%.2f\n", sum/NR }' /home/admin/scores.txt > /home/admin/solution`  
`cat solution `  
```console
5.20
```


#### 3. Validate the task
`md5sum /home/admin/solution`  
```console
6d4832eb963012f6d8a71a60fac77168  /home/admin/solution
```

`~/agent$ ./check.sh`  
```console
OK
```
