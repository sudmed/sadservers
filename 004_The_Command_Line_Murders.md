# The Command Line Murders

**Description**: This is the [Command Line Murders](https://github.com/veltman/clmystery) with a small twist as in the solution is different.  

Enter the name of the murderer in the file `/home/admin/mysolution`, for example `echo "John Smith" > ~/mysolution`.  
Hints are at the base of the `/home/admin/clmystery` directory. Enjoy the investigation!  

Test: `md5sum ~/mysolution` returns `9bba101c7369f49ca890ea96aa242dd5`.  
(You can always see `/home/admin/agent/check.sh` to see how the solution is evaluated).  

---

### Solution:
#### 1. Reconnaissance on the server
`cd /home/admin/ && ls -la`  
```console
total 44
drwxr-xr-x 7 admin admin 4096 Oct  3 21:46 .
drwxr-xr-x 3 root  root  4096 Sep 17 16:44 ..
drwx------ 3 admin admin 4096 Sep 20 15:52 .ansible
-rw------- 1 admin admin   57 Sep 20 15:58 .bash_history
-rw-r--r-- 1 admin admin  220 Aug  4  2021 .bash_logout
-rw-r--r-- 1 admin admin 3526 Aug  4  2021 .bashrc
drwxr-xr-x 3 admin admin 4096 Sep 20 15:56 .config
-rw-r--r-- 1 admin admin  807 Aug  4  2021 .profile
drwx------ 2 admin admin 4096 Sep 17 16:44 .ssh
drwxr-xr-x 2 admin root  4096 Oct  3 21:46 agent
drwxr-xr-x 4 admin admin 4096 Oct  3 21:46 clmystery
-rw-r--r-- 1 admin admin    0 Oct  3 21:46 mysolution
```

`cd /home/admin/clmystery && ls -la`  
```console
total 264
drwxr-xr-x 4 admin admin   4096 Oct  3 21:46 .
drwxr-xr-x 7 admin admin   4096 Oct  3 21:46 ..
drwxr-xr-x 8 admin admin   4096 Oct  3 21:46 .git
-rw-r--r-- 1 admin admin   1073 Oct  3 21:46 LICENSE.md
-rw-r--r-- 1 admin admin   2668 Oct  3 21:46 README.md
-rw-r--r-- 1 admin admin  13844 Oct  3 21:46 cheatsheet.md
-rw-r--r-- 1 admin admin 189042 Oct  3 21:46 cheatsheet.pdf
-rw-r--r-- 1 admin admin    146 Oct  3 21:46 hint1
-rw-r--r-- 1 admin admin     88 Oct  3 21:46 hint2
-rw-r--r-- 1 admin admin    187 Oct  3 21:46 hint3
-rw-r--r-- 1 admin admin    183 Oct  3 21:46 hint4
-rw-r--r-- 1 admin admin    280 Oct  3 21:46 hint5
-rw-r--r-- 1 admin admin    272 Oct  3 21:46 hint6
-rw-r--r-- 1 admin admin    329 Oct  3 21:46 hint7
-rw-r--r-- 1 admin admin    473 Oct  3 21:46 hint8
-rw-r--r-- 1 admin admin   2559 Oct  3 21:46 instructions
drwxr-xr-x 5 admin admin   4096 Oct  3 21:46 mystery
```

`cat instructions`
```console
.OOOOOOOOOOOOOOO @@                                   @@ OOOOOOOOOOOOOOOO.
OOOOOOOOOOOOOOOO @@                                    @@ OOOOOOOOOOOOOOOO
OOOOOOOOOO'''''' @@                                    @@ ```````OOOOOOOOO
OOOOO'' aaa@@@@@@@@@@@@@@@@@@@@"""                   """""""""@@aaaa `OOOO
OOOOO,""""@@@@@@@@@@@@@@""""                                     a@"" OOOA
OOOOOOOOOoooooo,                                            |OOoooooOOOOOS
OOOOOOOOOOOOOOOOo,                                          |OOOOOOOOOOOOC
OOOOOOOOOOOOOOOOOO                                         ,|OOOOOOOOOOOOI
OOOOOOOOOOOOOOOOOO @          THE                          |OOOOOOOOOOOOOI
OOOOOOOOOOOOOOOOO'@           COMMAND                      OOOOOOOOOOOOOOb
OOOOOOOOOOOOOOO'a'            LINE                         |OOOOOOOOOOOOOy
OOOOOOOOOOOOOO''              MURDERS                      aa`OOOOOOOOOOOP
OOOOOOOOOOOOOOb,..                                          `@aa``OOOOOOOh
OOOOOOOOOOOOOOOOOOo                                           `@@@aa OOOOo
OOOOOOOOOOOOOOOOOOO|                                             @@@ OOOOe
OOOOOOOOOOOOOOOOOOO@                               aaaaaaa       @@',OOOOn
OOOOOOOOOOOOOOOOOOO@                        aaa@@@@@@@@""        @@ OOOOOi
OOOOOOOOOO~~ aaaaaa"a                 aaa@@@@@@@@@@""            @@ OOOOOx
OOOOOO aaaa@"""""""" ""            @@@@@@@@@@@@""               @@@|`OOOO'
OOOOOOOo`@@a                  aa@@  @@@@@@@""         a@        @@@@ OOOO9
OOOOOOO'  `@@a               @@a@@   @@""           a@@   a     |@@@ OOOO3
`OOOO'       `@    aa@@       aaa"""          @a        a@     a@@@',OOOO'

There's been a murder in Terminal City, and TCPD needs your help.
To figure out whodunit, go to the 'mystery' subdirectory and start working from there.
You'll want to start by collecting all the clues at the crime scene (the 'crimescene' file).
The officers on the scene are pretty meticulous, so they've written down EVERYTHING in their officer reports.
Fortunately the sergeant went through and marked the real clues with the word "CLUE" in all caps.
If you get stuck, open one of the hint files (from the CL, type 'cat hint1', 'cat hint2', etc.).
To check your answer or find out the solution, open the file 'solution' (from the CL, type 'cat solution').
To get started on how to use the command line, open cheatsheet.md or cheatsheet.pdf (from the command line, you can type 'nano cheatsheet.md').
Don't use a text editor to view any files except these instructions, the cheatsheet, and hints.
```

`cat hint1`  
```console
Try poking around what's in a file by using the 'head' command:
  head -n 20 people
This will show you the first 20 lines of the 'people' file.
```

`cat hint2`  
```console
Try using grep to search for the clues in the crimescene file:
        grep "CLUE" crimescene
```

`cat hint3`  
```console
In order to track down our potential witness, we need to figure out where she lives.
Try using 'head' on some of the files like 'people' and 'vehicles' and see where we might find that.
```

`cat hint4`  
```console
To find all the Annabels' addresses, use the 'people' file:
        grep "Annabel" people
Notice that not all of the results are worth investigating.  Remember what we know about Annabel.
```

`cat hint5`  
```console
"Interview" the two possible witnesses by reading the correct line from the streets they live on:
        head -n 173 streets/Mattapan_Street | tail -n 1
This will give you just line 173 of Mattapan street, because it will take first 173 lines, and then take
the last line from those.
```

`cat hint6`  
```console
To find a matching license plate, or a matching car, you can use grep on the 'vehicles' file:
        grep "Honda" vehicles
        grep "Blue" vehicles
        grep "L337" vehicles
This doesn't give us anything useful - why not? Try using 'head' on the file to investigate its structure.
```

`cat hint7`  
```console
In order to actually get information about vehicles that might match our description,
we need to get multiple lines AROUND each match.  We can use the -A, -B, or -C option with grep:
        grep -A 5 "L337" mystery/vehicles
This will match the license plates that contain "L337" and, for each match, show us the five lines AFTER it.
```

`cat hint8`  
```console
To see who was a member of several different groups, you can combine their membership lists into one and search against that.
        cat Fitness_Galaxy AAA United_MileagePlus | grep "John Smith"
If you only want to see the number of matches, you can use grep's -c option (the c must be lowercase):
        cat Fitness_Galaxy AAA United_MileagePlus | grep -c "John Smith"
Or you can pipe the result to 'wc -l':
        cat Fitness_Galaxy AAA United_MileagePlus | grep "John Smith" | wc -l
```
