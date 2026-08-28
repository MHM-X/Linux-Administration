# Lab 03: The Command Line Murders

## Description
This is the [Command Line Murders](https://github.com/veltman/clmystery) with a small twist as in the solution is different

Enter the name of the murderer in the file /home/admin/mysolution, for example echo "John Smith" > ~/mysolution

Hints are at the base of the /home/admin/clmystery directory. Enjoy the investigation!

🔗 **Lab Link:** [SadServers - "Saint John"](https://sadservers.com/scenario/command-line-murders)

<br>

## 🪜 Steps

### Step 1: first we need to view the hints via:

```bash
cat /home/admin/clmystery
# or:
vim /home/admin/clmystery
```

### Step 2:

```bash
cat /home/admin/clmystery/instructions

There's been a murder in Terminal City, and TCPD needs your help.

To figure out whodunit, go to the 'mystery' subdirectory and start working from there.

You'll want to start by collecting all the clues at the crime scene (the 'crimescene' file).

The officers on the scene are pretty meticulous, so they've written down EVERYTHING in their officer reports.

Fortunately the sergeant went through and marked the real clues with the word "CLUE" in all caps.

If you get stuck, open one of the hint files (from the CL, type 'cat hint1', 'cat hint2', etc.).

To check your answer or find out the solution, open the file 'solution' (from the CL, type 'cat solution').

To get started on how to use the command line, open cheatsheet.md or cheatsheet.pdf (from the command line, you can type 'nano cheatsheet.md').

Don't use a text editor to view any files except these instructions, the cheatsheet, and hints.

grep CLUE crimescene

<img width="1241" height="191" alt="image" src="https://github.com/user-attachments/assets/b9780ee4-67cd-4edc-ac92-2c307fcb6f2b" />

CLUE: Footage from an ATM security camera is blurry but shows that the perpetrator is a tall male, at least 6'.
CLUE: Found a wallet believed to belong to the killer: no ID, just loose change, and membership cards for Rotary_Club, Delta SkyMiles, the local library, and the Museum of Bash History. The cards are totally untraceable and have no name, for some reason.
CLUE: Questioned the barista at the local coffee shop. He said a woman left right before they heard the shots. The name on her latte was Annabel, she had blond spiky hair and a New Zealand accent.

admin@ip-10-1-13-57:~/clmystery/mystery$ grep Annabel people
Annabel Sun     F       26      Hart Place, line 40
Oluwasegun Annabel      M       37      Mattapan Street, line 173
Annabel Church  F       38      Buckingham Place, line 179
Annabel Fuglsang        M       40      Haley Street, line 176

head -n 20 people

This will show you the first 20 lines of the 'people' file.
admin@ip-10-1-13-49:~/clmystery$ head -n 20 people
head: cannot open 'people' for reading: No such file or directory
admin@ip-10-1-13-49:~/clmystery$ cd mystery
admin@ip-10-1-13-49:~/clmystery/mystery$ head -n 20 people
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
admin@ip-10-1-13-49:~/clmystery/mystery$


admin@ip-10-1-13-24:~/clmystery/mystery/streets$ head -n 173 Mattapan_Street | tail -n 1
SEE INTERVIEW #9437737
admin@ip-10-1-13-24:~/clmystery/mystery/streets$

admin@ip-10-1-13-24:~/clmystery/mystery/interviews$ cat interview-9437737
Doesn't appear to be the witness from the cafe, who is female.

admin@ip-10-1-13-150:~/clmystery/mystery$ head -n 179 streets/Buckingham_Place | tail -n 1
SEE INTERVIEW #699607
admin@ip-10-1-13-150:~/clmystery/mystery$ cat interviews/interview-699607
Interviewed Ms. Church at 2:04 pm.  Witness stated that she did not see anyone she could identify as the shooter, that she ran away as soon as the shots were fired.

However, she reports seeing the car that fled the scene.  Describes it as a blue Honda, with a license plate that starts with "L337" and ends with "9"
admin@ip-10-1-13-150:~/clmystery/mystery$

admin@ip-10-1-13-150:~/clmystery/mystery$ grep L337 vehicles
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
admin@ip-10-1-13-150:~/clmystery/mystery$


License Plate L337ZR9
admin@ip-10-1-13-150:~/clmystery/mystery$ sed -n '866,871p' vehicles
License Plate L337ZR9
Make: Honda
Color: Red
Owner: Katie Park
Height: 6'2"
Weight: 189 lbs
admin@ip-10-1-13-150:~/clmystery/mystery$ head -n 2168 vehicles | tail -n 6
Color: Orange
Owner: Deni Iovu
Height: 5'0"
Weight: 150 lbs




----------------------
admin@ip-10-1-13-150:~/clmystery/mystery$ head -n 13564 vehicles | tail -n 6
Color: Blue
Owner: Dieter Schooling
Height: 6'2"
Weight: 226 lbs

License Plate L337GB9

admin@ip-10-1-13-150:~/clmystery/mystery$ echo "Dieter Schooling" > ~/mysolution

-------------------------


admin@ip-10-1-13-150:~/clmystery/mystery$ grep -n 'L337.*9$' vehicles
866:License Plate L337ZR9
2168:License Plate L337P89
5458:License Plate L337GX9
7166:License Plate L337QE9
13564:License Plate L337GB9
13655:License Plate L337OI9
15923:License Plate L337X19
17743:License Plate L337539
22083:License Plate L3373U9
23826:License Plate L337369
24834:License Plate L337DV9
31827:License Plate L3375A9
34312:License Plate L337WR9
admin@ip-10-1-13-150:~/clmystery/mystery$


------------------------
هذا الامر الصح الي لازم استخ\مه 
admin@ip-10-1-13-47:~/clmystery/mystery$ tail -n +13564 vehicles | head -n 7
License Plate L337GB9
Make: Toyota
Color: Blue
Owner: Matt Waite
Height: 6'1"
Weight: 190 lbs

tail: error writing 'standard output': Broken pipe
admin@ip-10-1-13-47:~/clmystery/mystery$
-------------------------------
grep -A 5 "L337" mystery/vehicles

admin@ip-10-1-13-179:~/clmystery$ echo "Joe Germuska" > ~/mysolution
admin@ip-10-1-13-179:~/clmystery$ md5sum ~/mysolution
9bba101c7369f49ca890ea96aa242dd5  /home/admin/mysolution
admin@ip-10-1-13-179:~/clmystery$ 
```

### Step 3:
```bash

```
