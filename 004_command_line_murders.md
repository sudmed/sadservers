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

#### 2. Check hints
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

#### 3. Follow the hints
`head -n 20 people`  
```console
***************
To go to the street someone lives on, use the file
for that street name in the 'streets' subdirectory.
To knock on their door and investigate, read the line number
they live on from the file.  If a line looks like gibberish, you're at the wrong house.
***************
NAME    GENDER  AGE     ADDRESS
Alicia Fuentes  F       48      Walton Street, line 433
Jo-Ting Losev   F       46      Hemenway Street, line 390
Elena Edmonds   F       58      Elmwood Avenue, line 123
Naydene Cabral  F       46      Winthrop Street, line 454
Dato Rosengren  M       22      Mystic Street, line 477
Fernanda Serrano        F       37      Redlands Road, line 392
Emiliano Wenk   M       90      Paulding Street, line 490
Larry Lapin     M       71      Atwill Road, line 298
Jakub Gondos    M       61      Mitchell Street, line 187
Derek Kazanin   M       55      Tennis Road, line 440
Jens Tuimalealiifano    M       83      Rockwood Street, line 205
Nikola Kadhi    M       75      Glenville Avenue, line 226
```

`grep "CLUE" crimescene`  
```console
CLUE: Footage from an ATM security camera is blurry but shows that the perpetrator is a tall male, at least 6'.
CLUE: Found a wallet believed to belong to the killer: no ID, just loose change, and membership cards for Rotary_Club, Delta SkyMiles, the local library, and the Museum of Bash History. The cards are totally untraceable and have no name, for some reason.
CLUE: Questioned the barista at the local coffee shop. He said a woman left right before they heard the shots. The name on her latte was Annabel, she had blond spiky hair and a New Zealand accent.
```

`head -n 20 vehicles`  
```console
***************
Vehicle and owner information from the Terminal City Department of Motor Vehicles
***************
License Plate T3YUHF6
Make: Toyota
Color: Yellow
Owner: Jianbo Megannem
Height: 5'6"
Weight: 246 lbs

License Plate EZ21ECE
Make: BMW
Color: Gold
Owner: Norbert Feldwehr
Height: 5'3"
Weight: 205 lbs

License Plate CQN2TJE
Make: Mazda
```

`grep "Annabel" people`  
```console
Annabel Sun     F       26      Hart Place, line 40
Oluwasegun Annabel      M       37      Mattapan Street, line 173
Annabel Church  F       38      Buckingham Place, line 179
Annabel Fuglsang        M       40      Haley Street, line 176
```

`head -n 173 streets/Mattapan_Street | tail -n 1`  
```console
SEE INTERVIEW #9437737
```

`cat clmystery/mystery/interviews/interview-9437737`  
```console
Doesn't appear to be the witness from the cafe, who is female.
```

```console
grep "Honda" vehicles
Make: Honda

grep "Blue" vehicles
Color: Blue

grep "L337" vehicles
License Plate L337ZR9
License Plate L337P89
License Plate L337GX9
License Plate L337QE9
License Plate L337GB9
License Plate L337OI9
License Plate L337X19
License Plate L337539
License Plate L3373U9
License Plate L337369
License Plate L337DV9
License Plate L3375A9
License Plate L337WR9
```

`grep -A 5 "L337" vehicles`  
```console
License Plate L337ZR9
Make: Honda
Color: Red
Owner: Katie Park
Height: 6'2"
Weight: 189 lbs
--
License Plate L337P89
Make: Honda
Color: Teal
Owner: Mike Bostock
Height: 6'4"
Weight: 173 lbs
--
License Plate L337GX9
Make: Mazda
Color: Orange
Owner: John Keefe
Height: 6'4"
Weight: 185 lbs
--
License Plate L337QE9
Make: Honda
Color: Blue
Owner: Erika Owens
Height: 6'5"
Weight: 220 lbs
--
License Plate L337GB9
Make: Toyota
Color: Blue
Owner: Matt Waite
Height: 6'1"
Weight: 190 lbs
--
License Plate L337OI9
Make: Jaguar
Color: Blue
Owner: Brian Boyer
Height: 6'6"
Weight: 201 lbs
--
License Plate L337X19
Make: Fiat
Color: Blue
Owner: Al Shaw
Height: 6'5"
Weight: 240 lbs
--
License Plate L337539
Make: Honda
Color: Blue
Owner: Aron Pilhofer
Height: 5'3"
Weight: 198 lbs
--
License Plate L3373U9
Make: Ford
Color: Blue
Owner: Miranda Mulligan
Height: 6'6"
Weight: 156 lbs
--
License Plate L337369
Make: Honda
Color: Blue
Owner: Heather Billings
Height: 5'2"
Weight: 244 lbs
--
License Plate L337DV9
Make: Honda
Color: Blue
Owner: Joe Germuska
Height: 6'2"
Weight: 164 lbs
--
License Plate L3375A9
Make: Honda
Color: Blue
Owner: Jeremy Bowers
Height: 6'1"
Weight: 204 lbs
--
License Plate L337WR9
Make: Honda
Color: Blue
Owner: Jacqui Maher
Height: 6'2"
Weight: 130 lbs
```


##### 4. Narrowing the circle of suspects
The blue Honda, male and 6 feet tall are suitable only for `Jeremy Bowers` or `Joe Germuska`.  


##### 5. Let's check the membership in the clubs
`cat ~/clmystery/mystery/memberships/Rotary_Club ~/clmystery/mystery/memberships/Delta_SkyMiles ~/clmystery/mystery/memberships/Terminal_City_Library ~/clmystery/mystery/memberships/Museum_of_Bash_History | grep -c "Jeremy Bowers"`  
```console
3
```

`cat ~/clmystery/mystery/memberships/Rotary_Club ~/clmystery/mystery/memberships/Delta_SkyMiles ~/clmystery/mystery/memberships/Terminal_City_Library ~/clmystery/mystery/memberships/Museum_of_Bash_History | grep -c "Joe Germuska"`  
```console
4
```


#### 6. Name of the murderer
`echo "Joe Germuska" > ~/mysolution`  

`md5sum ~/mysolution`  
```console
9bba101c7369f49ca890ea96aa242dd5  /home/admin/mysolution
```


#### 7. Validate the task
`cat /home/admin/agent/check.sh`  
```console
#!/usr/bin/bash
res=$(md5sum /home/admin/mysolution |awk '{print $1}')
res=$(echo $res|tr -d '\r')

if [[ "$res" = "9bba101c7369f49ca890ea96aa242dd5" ]]
then
  echo -n "OK"
else
  echo -n "NO"
fi
```

`bash /home/admin/agent/check.sh`  
```console
OK
```
