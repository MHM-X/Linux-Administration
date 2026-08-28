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
cat /home/admin/clmystery/instructions
# or:
vim /home/admin/clmystery/instructions
```

It shows:

There's been a murder in Terminal City, and TCPD needs your help.

To figure out whodunit, go to the 'mystery' subdirectory and start working from there.

You'll want to start by collecting all the clues at the crime scene (the 'crimescene' file).

The officers on the scene are pretty meticulous, so they've written down EVERYTHING in their officer reports.

Fortunately the sergeant went through and marked the real clues with the word "CLUE" in all caps.

If you get stuck, open one of the hint files (from the CL, type 'cat hint1', 'cat hint2', etc.).

To check your answer or find out the solution, open the file 'solution' (from the CL, type 'cat solution').

To get started on how to use the command line, open cheatsheet.md or cheatsheet.pdf (from the command line, you can type 'nano cheatsheet.md').

Don't use a text editor to view any files except these instructions, the cheatsheet, and hints.


### Step 2: following the first clue

```bash
grep CLUE crimescene
```
<img width="1241" height="191" alt="image" src="https://github.com/user-attachments/assets/b9780ee4-67cd-4edc-ac92-2c307fcb6f2b" />

CLUE: Footage from an ATM security camera is blurry but shows that the perpetrator is a tall male, at least 6'.
CLUE: Found a wallet believed to belong to the killer: no ID, just loose change, and membership cards for Rotary_Club, Delta SkyMiles, the local library, and the Museum of Bash History. The cards are totally untraceable and have no name, for some reason.
CLUE: Questioned the barista at the local coffee shop. He said a woman left right before they heard the shots. The name on her latte was Annabel, she had blond spiky hair and a New Zealand accent.

### Step 2: following the Second clue "A female called Annabel"
```bash

admin@ip-10-1-13-57:~/clmystery/mystery$ grep Annabel people
```

Annabel Sun     F       26      Hart Place, line 40
Oluwasegun Annabel      M       37      Mattapan Street, line 173
Annabel Church  F       38      Buckingham Place, line 179
Annabel Fuglsang        M       40      Haley Street, line 176

### Step 3: Searching in the streets
```bash

#head -n 20 people

#This will show you the first 20 lines of the 'people' file.

admin@ip-10-1-13-49:~/clmystery/mystery$ head -n 20 people
```

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

### Step 4: Searching in the streets for each name
```bash
admin@ip-10-1-13-24:~/clmystery/mystery/streets$ head -n 173 Mattapan_Street | tail -n 1
```
SEE INTERVIEW #9437737
```bash

admin@ip-10-1-13-24:~/clmystery/mystery/interviews$ cat interview-9437737
```
finally:
Doesn't appear to be the witness from the cafe, who is female.

Now try for the onother name:
```bash
admin@ip-10-1-13-150:~/clmystery/mystery$ head -n 179 streets/Buckingham_Place | tail -n 1
```

shows:
SEE INTERVIEW #699607
then:
```bash
admin@ip-10-1-13-150:~/clmystery/mystery$ cat interviews/interview-699607
```
Interviewed Ms. Church at 2:04 pm.  Witness stated that she did not see anyone she could identify as the shooter, that she ran away as soon as the shots were fired.

However, she reports seeing the car that fled the scene.  Describes it as a blue Honda, with a license plate that starts with "L337" and ends with "9"

### Step 5: Now we had another clue to go with it "blue Honda, with a license plate that starts with "L337" and ends with "9"" and we know from a previous clue that its tall is 6 or more.

```bash
admin@ip-10-1-13-150:~/clmystery/mystery$ grep -n L337 vehicles
# to view all of the matches and their line number.
```
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

```bash
admin@ip-10-1-13-47:~/clmystery/mystery$ tail -n +13564 vehicles | head -n 7
# to view for each line the 7 lines under it so we could identify if it's matches with our criterias.
```
License Plate L337GB9
Make: Toyota
Color: Blue
Owner: Matt Waite
Height: 6'1"
Weight: 190 lbs
tail: error writing 'standard output': Broken pipe

Or: we could use 
```bash
grep -A 5 "L337" mystery/vehicles
```
to print all the matched lines and the 5 lines after them for each.


### Step 6: Try for each name if it's the right one via the command:
```bash
md5sum ~/mysolution
```


the wright name was: "oe Germuska".
```bash
admin@ip-10-1-13-179:~/clmystery$ echo "Joe Germuska" > ~/mysolution
admin@ip-10-1-13-179:~/clmystery$ md5sum ~/mysolution
```
9bba101c7369f49ca890ea96aa242dd5  /home/admin/mysolution
