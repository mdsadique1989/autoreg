# Setup Guide (Windows)

Two parts: **A. one-time setup** (about 15 minutes) and **B. every time you register** (about 10 minutes plus the registration itself).

Words used here:
- **CSV** = a plain text file with values separated by commas. Notepad can open it.
- **ZIP** = one compressed file that Google Drive makes when you download a folder.
- **`.bat` file** = a small file that runs commands when you double-click it. You do not need to understand what is inside.

---

## A. One-time setup

### A1. Install Python

1. Open <https://www.python.org/downloads/> and download Python **3.9 or newer** for Windows.
2. Run the installer. On the first screen **tick "Add python.exe to PATH"**, then click *Install Now*.
3. To check: press the Windows key, type `cmd`, press Enter, type `py --version`, press Enter. You should see something like `Python 3.12.x`.

If typing `python` opens the Microsoft Store, ignore that. Use the python.org installer above.

### A2. Put the two files in a folder

Create a folder such as `C:\bseb_registration` and put these two files in it, side by side:

- `run.bat`
- `bseb_class9_registration.py`

Nothing else is required. Avoid folders inside OneDrive.

### A3. Nothing to run: the tool sets itself up

The first time you pick an option that needs the tool (for example **1**), `run.bat` shows `First-time setup` and prepares everything by itself. It needs the **internet** and takes a few minutes, once. You will see `Creating the private Python environment...`, then `Installing the two small packages...`, then `Setup finished.`, and your option continues.

If it says **"Python was not found"**, redo A1 and tick the PATH box, then double-click `run.bat` again. You can also press **S** in the menu any time to repeat the setup.

You never need to repeat Part A.

---

### A4. Low-end PC? (4 GB RAM)

- You do **not** need Excel, pandas or any spreadsheet program. The tool reads the Google files itself.
- It installs only two small packages (about 41 MB) and uses about 60 to 70 MB of memory while running.
- Chrome is the heavy part. Close other programs and browser tabs, use only the Chrome window opened by `run.bat` option C, and restart the PC before a long session.

---

## B. Every time you register students

### Step 1. Download your admission folder from Google Drive

1. Open Google Drive in your browser (the Google account that owns your admission folder).
2. Find the folder that holds your sheet and all the photos (the one named like *Admission Upgrade...*).
3. **Right-click the folder, choose Download.**
4. Wait. Google prepares a ZIP and your browser saves it in **Downloads**. A name like `Admission Upgrade-20261003T081500Z-001.zip`.

This only copies your data. Nothing in your live sheet or folder changes, and your web app keeps working as usual.

**The sheet.** The tool needs the sheet data too. It looks in three places and uses the copy with the **newest applications** (it tells you which one it used and which it ignored):

1. the sheet inside the ZIP, if Google put one there;
2. a CSV or Excel file in your Downloads folder, for example the file your web app exports (named like `uhsk-admissions-1792977870616.csv`);
3. nothing else is needed.

So if your web app has a download button for the sheet, press it now as well, and leave the file in Downloads. Download it **after** the folder if you want to be sure it is the newest.

Very large folders may be split by Google into several ZIPs. This tool reads one ZIP at a time and cannot combine them yet; if that happens, ask for help.

### Step 2. Open Chrome and log in to BSEB

1. Close every Chrome window.
2. Double-click `run.bat`, type **C** and press Enter. A new Chrome window opens (no bookmarks; that is normal).
3. In that window go to <https://www.biharboardonline.org/>, open the registration portal and **log in yourself** (password, captcha).
4. Leave this Chrome window open the whole time.

### Step 3. Check the data

1. Double-click `run.bat`. A menu appears.
2. Type **1** and press Enter.

You will see something like:

```
Using Drive download: Admission Upgrade-20261026T081500Z-001.zip  (0.1 hours old)
Loading data...
Sheet used:
  USED     uhsk-admissions-1792977870616.csv (in Downloads): 199 rows, newest application 23-09-2026 04:49
Rows in the sheet by class: IX 79 | X 48 | XI 71 | XII 1   -> only Class IX is used
Your sheet has no roll numbers. The tool numbers the Class IX students 1, 2, 3 ... in the order
they applied (oldest first) so you can point at a student. Use this "No." in menu option 2.
Note: your sheet has no email column, so the email field on the BSEB form is left empty.
Class IX rows: 79   valid: 0   blocked: 79
  [BLOCKED] row 5 VIVEK KUMAR
            - IFSC "PUNBO103320" invalid (4 letters, a ZERO, ...) (did you mean PUNB0103320?)
  ...
  70 more student(s) are blocked only because their marks / town-city / language are not typed yet.

79 student(s) need identification marks, town/city and/or language.
```

It is normal that everyone is blocked the first time, because the marks, city and (for Muslim students) language are not typed yet. The tool has now created `extra_info.csv` for you.

Things with a different reason are listed by name, for example a bank code with the letter O instead of the digit 0. The message often suggests the fix (`did you mean ...?`). Correct such a value in your sheet, download again, and check again.

### Step 4. Type the identification marks, city and language

You do **not** need Excel. Choose the easy way:

**Way 1 (easiest): type them in the black window.**

1. In the menu type **8** and press Enter.
2. First it may ask `Town/City for ALL ... students`. Type the town once (for example `KANTI`) and press Enter. Press Enter on an empty line to type it for each student instead.
3. Then for each student it shows the number, name and father, and asks:

```
[1/79] No. 20   MD SABBIR ALAM   (father: MOHAMMAD KHUSHNOOR)
  Identification mark 1 (example: MOLE ON LEFT CHEEK): 
  Identification mark 2 (example: SCAR ON RIGHT HAND): 
  Language: type U for Urdu (+NLH) or H for Hindi (+Sanskrit): 
```

The **Language** question appears only for Muslim students, whose language cannot be guessed. For everybody else Hindi + Sanskrit is assumed; if one of them takes Urdu, type `URDU` in the `language` column of `extra_info.csv` (menu 9).

4. Type in English letters and press Enter. Press Enter on an empty line to skip a student. Type **q** to stop; everything you typed is saved. You can come back later and choose 8 again; students already done are not asked again.

**Way 2: edit the file in Notepad.**

1. In the menu type **9**. `extra_info.csv` opens in Notepad.
2. Each student is one line. After the student's father's name there are seven slots separated by commas: `idMark1,idMark2,city,language,alreadyRegistered,ignore,distinct`. Fill the first three, and `language` where needed:

```
UHSK2610018609,20,Md Sabbir Alam,Mohammad Khushnoor,MOLE ON CHEEK,SCAR ON HAND,KANTI,URDU,,,
```


3. Do not remove the commas. If the text itself contains a comma, put it in quotes: `"KANTI, MUZAFFARPUR"`. Save (Ctrl + S) and close Notepad.

**If you have Excel, LibreOffice or Google Sheets**, you can open the same file there. Save it again as CSV.

The last three columns (`alreadyRegistered`, `ignore`, `distinct`) are optional switches (Way 2 only). Type `YES` in **alreadyRegistered** if you already registered that student on BSEB by hand or with an older tool. Type `YES` in **ignore** to skip an application (for example a second application of the same student). Type `YES` in **distinct** when two students have the same name, father and mother but are different people (see "Two students with identical names" below).

What you type is remembered. Next time only the new students need filling.

### Step 5. Check again

Menu, type **1**. Fix anything still listed as BLOCKED (see "What the messages mean" below). Repeat until the students you want show as valid.

### Step 6. Register ONE student first (important)

1. Menu, type **2**, then type the **No.** (or roll number) of a valid student. Menu 1 or 7 shows the numbers.
2. Watch Chrome fill the form by itself.
3. At the preview, **check every field, the photo and the signature**. In the black window type **y** and Enter to save.
4. Open the portal's declaration list and confirm the student is there.

### Step 7. Register everyone

- Menu **3**: registers all valid students. Fast.
- Menu **4**: same, but it pauses for your approval at every preview. Safer.

Before anything is sent you see a summary like this and must type **YES**:

```
==============================================================
READY TO REGISTER
  Data from              : Admission Upgrade-....zip (0.3 hours old)
  Class IX students      : 12
  already registered     : 5   (skipped, never sent twice)
  blocked (not sent)     : 2
  language               : Hindi+Sanskrit 4 (4 assumed), Urdu+NLH 1
  to register now        : 5   no.: 20, 22, 29, 31, 33
==============================================================
Type YES to continue:
```

Check the source and age of the ZIP, the numbers, and the **language line**: the number marked *assumed* are students set to Hindi + Sanskrit because the sheet has no language. If any of them takes Urdu, answer anything but YES, fix `extra_info.csv`, and run again. Anything other than YES cancels and sends nothing.

It stops at the first problem so you can look at Chrome. Fix the cause and run the same option again; students already saved are skipped automatically.

Once you trust it, press **N** in the menu. The line `Photo mode now:` changes from `temporary files` to `memory only`: photos are then sent from memory without even temporary files. Press N again to switch back. (The choice lasts until you close the window.)

### Step 8. When you finish

1. **Delete the ZIP from Downloads** and empty the Recycle Bin. It contains everyone's Aadhaar, bank details and photos.
2. Close the special Chrome window.

---

## Next day: new students

Repeat Steps 1, 2, 3, 4 (only the new students), 5 and 7. Students already registered are skipped.

---

## Many classes, many downloads: your questions answered

Your sheet holds Class IX, X, XI and XII and grows every day. Every download is a complete copy of everything.

**Which ZIP is used when there are several?**
Only the newest ZIP in Downloads that contains your admission pictures (or sheet). It is named on screen with its age. Other, older admission ZIPs are listed as **IGNORED**. They are never read. Delete old ZIPs after each use.

**Which sheet is used when there are several?**
The tool compares the copies it finds (the sheet inside the ZIP, and CSV or Excel files in Downloads) and uses the one with the **newest application**. It prints every copy it found, with its number of rows and its newest application date, marked `USED` or `ignored`. So an outdated sheet is never used by mistake. If you want to force one file, the program accepts `--file`.

**How must the pictures be named in the ZIP?**
Your web app names them `<applicationId>_photo.jpg` and `<applicationId>_signature.jpg`. Other names that contain the application ID also work (for example `UHSK..._sign.png`, `photo_UHSK....jpeg`, or pictures inside sub-folders), as long as a name says whether it is the photo or the signature. If a picture cannot be found the message shows how the pictures in your ZIP are named, so you can see what is different.

**What if I use an old ZIP by mistake?**
You are warned when it is more than a day old. It cannot register anyone twice, because the log remembers every saved student for good. The remaining danger is *old data*: if you corrected a mistake in the sheet after that ZIP was made, the old ZIP still has the mistake. So always download fresh before registering.

**Can the same student be registered twice?**
Several protections work together:

| Situation | What happens |
|---|---|
| You run again with a newer ZIP | Students already in the log are skipped (matched by application ID, and by name + father + mother + date of birth) |
| The student applied twice (two application IDs) | Both applications are **blocked** with a message naming them. Type `YES` in the `ignore` column of the one you do not want; the other registers once |
| Spelling or spacing differs between the two applications | Still detected (case, dots and spaces are ignored; same father and the same DOB or mother) |
| Two real students with different names but the same parents and DOB (twins) | Treated as different people |
| The student was registered on BSEB by hand or with an older tool | Type `YES` in `alreadyRegistered` in `extra_info.csv`. The tool never sends them. The portal's declaration list is also read (all pages) |
| The same student appears twice in one run | The second is skipped |
| You delete the log by accident | The portal list check is the only remaining protection; use `alreadyRegistered` for everyone already done |

**Two students with identical name, father and mother: what happens?**
The tool cannot tell by itself whether two applications with the same name, father and mother are one student who applied twice, or two different people who happen to share the names (common with frequent village names). So it never guesses. The rule is the same in the ZIP method and in the Colab method, because both run the same registration program and everything is tied to each application's own ID, pictures and extra_info row.

| Situation | What the tool does | What you do |
|---|---|---|
| Both applications are in the same download | **Both are blocked**, nothing is sent. The message names both applications | Different students: type `YES` in the `distinct` column for them. One student who applied twice: type `YES` in the `ignore` column of the extra application |
| After `distinct` | Both are registered, each with his own pictures, marks and bank details | Nothing |
| One was registered earlier, the other appears in a later download | The new one is **blocked** (it looks like a possible duplicate of the registered one) and nothing is sent for it | Same choice as above. A registered student is never sent again |
| Same names **and the same date of birth**, new application ID | Treated as the **same student**: not sent again. The run says so | If it is really a different person (for example twins given the same name), type `YES` in `distinct` for both |
| The other student was registered on BSEB by hand, and is not in the sheet | The BSEB declaration list shows these names, but your log does not explain it. The new student is **skipped** with a message `[SKIPPED] ... may already be registered` | Different person: type `YES` in `distinct`. Same person: nothing to do |
| The BSEB list shows these names once and your own log explains it | Not a problem: the new student is **not** blocked by the list | Nothing |
| The pictures | Found by each application's own ID, so identical names can never mix up pictures | Nothing |

"Different date of birth" alone does not unblock a pair, because a typing mistake in the date would also look like that. The first time it happens you answer once with `distinct`, and the answer is remembered in `extra_info.csv`.

**Can a Class X student be registered as Class IX?**
The tool uses only rows whose class column says Class IX (`Class IX`, `IX`, `9`, `9th` all count). Class X, XI and XII rows are never used.
- Every run prints the number of rows per class, so you can see what was included: `IX 12 | X 20 | XI 31 | XII 25`.
- A class value the tool does not understand (for example `Class 9A` or empty) is listed as a warning and is **not** registered.
- If the sheet has no class column at all, the tool **stops**; it will not guess.
- A Class IX row with a stream filled in (Science, Arts) is blocked, because streams exist only for Class XI/XII.
- Ten students entered under the wrong class by the student themselves cannot be detected by any tool. Look at the class breakdown and the summary before you type YES.

**What if a student's data changes after I registered them?**
The registered record on BSEB is not updated. Correct it on the BSEB portal yourself.

---

## Checking progress

| Menu | Shows |
|---|---|
| **7** | A table of every student: REGISTERED (with time), PENDING (valid, not yet done) or BLOCKED (with the reason) |
| **5** | How many are registered and the last 5 |
| **6** | Opens the folder `C:\Users\<you>\bseb_registration` |

Files in that folder:

| File | Contents |
|---|---|
| `class9_registration_log.tsv` | One line per registered student. A line is written only after BSEB confirms the save |
| `extra_info.csv` | The marks and city you typed |
| `validation_report.csv` | Every student as OK or ERROR with reasons |
| `status_report.csv` | The table from menu 7. Opens in Notepad (or Excel if you have it) |

Important:
- **Do not delete the log.** It is how the tool knows who is done.
- If an error happens at the **Save** step, open the portal's declaration list and check whether that student is already there **before** running again.
- The log is only on this computer. Menu 7 does not check the BSEB portal; it reads your data and the log.
- Only successes are recorded. If something goes wrong, take a screenshot of the black window so the error can be understood.

---

## What the messages mean

| Message | What to do |
|---|---|
| `Town/City missing`, `Identification marks missing` | Type them in `extra_info.csv` (menu 8) |
| `STOPPED: the sheet has no class column` | The class column (`admClass`) is missing or renamed to something unknown. Fix the header in the first row of the sheet, download again |
| `Unrecognised class "..."` | Those rows are not registered. Correct the class value in the sheet if they are Class IX |
| `Language (Urdu or Hindi) is not in the sheet` | Muslim students need the language typed: menu 8 (answer U or H) or the `language` column of `extra_info.csv` |
| `photo ... not found in the downloaded folder (pictures in this ZIP are named like: ...)` | The picture of that student is not in the ZIP, or is named differently. Look at the example names in the message; download the Drive folder again |
| `Sheet used:` lists several copies | Normal. The newest copy is used. Delete old CSV and ZIP files from Downloads to keep it simple |
| `Possible duplicate application of the same student` | The tool cannot tell whether it is one student who applied twice or two different people with the same names. One student twice: type YES in the `ignore` column of the extra one. Different people: type YES in the `distinct` column of each |
| `[SKIPPED] ... may already be registered` | The BSEB list already shows the same name, father and mother, and your own log does not explain it. Different person: type YES in the `distinct` column. Same person: nothing to do |
| `application(s) are the same student (same names and date of birth) as one already registered` | Information only. These are repeat applications of a student who is already registered; nothing is sent again |
| `Stream "..." is filled in` | Streams exist only for Class XI/XII. Check the student's class in the sheet |
| `Other (older) admission ZIPs found and IGNORED` | Just information. Delete old ZIPs from Downloads |
| `Aadhaar must be 12 digits` | Correct the number in that student's row in your Google Sheet, then download the folder again |
| `Bank account: ... scientific notation` | The number was damaged by the sheet. Retype the full number in the sheet as plain text, download again |
| `IFSC ... invalid` | Wrong code (often a letter O instead of the digit 0). Correct it in the sheet |
| `Email domain looks like a typo` | Correct the email in the sheet |
| `DOB ... is today/in the future` | Correct the date of birth in the sheet |
| `UHSK..._photo.jpg not found in the downloaded folder` | The photo is missing in Drive. Ask the student to upload it again, then download again |
| `... is not a JPG/PNG picture` | The uploaded file is not a normal picture. Ask for a new one |
| `Aadhaar shared by: ...` / `Bank account shared by: ...` | Two students have the same number; one is a typing mistake |

Correcting one cell in your sheet does not disturb your web app. If you prefer not to touch the sheet, ask the student to apply again.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| "Could not find the Google Drive download" | Do Step 1 again. Check the ZIP is in Downloads. If Downloads is in another place, see `advanced/ADVANCED.md` (`BSEB_ZIP_DIR`) |
| "This download is more than a day old" | Download the folder again to include new students |
| "The zip has no sheet inside" or "does not contain the admission sheet" | Open your sheet in Google Sheets and choose File, Download, Microsoft Excel (.xlsx) or Comma-separated values (.csv, from the tab with the students). Wait until it is in Downloads, then run again. The tool reads both; you do not need Excel installed |
| `cannot connect to 127.0.0.1:9222` | Chrome was not started with option **C**, or another Chrome was open. Close all Chrome windows and do Step 2 again |
| `The program file bseb_class9_registration.py is not in this folder` | Put `run.bat` and `bseb_class9_registration.py` in the same folder |
| `Python was not found on this PC` | Install Python from python.org and tick **Add python.exe to PATH**, then double-click `run.bat` again |
| `Something went wrong during setup` | Check the internet connection and press **S** to try again. If it fails again, take a photo of the message |
| "Could not read extra_info.csv" | The file is damaged or its first line was changed. Fix it, or delete the file and type the values again with menu 8. The tool never overwrites a file it cannot read |
| "Could not update extra_info.csv ... open in another program" | Close Notepad (or whatever has it open) and run again |
| `LookupError: No option "..." for caste. Available: [...]` | The portal uses a different label. Note the available labels and ask for the setting to be updated |
| `Preview did not open` followed by red field errors | BSEB rejected a value. Fix it and run again |
| `NoSuchElementException` on a field | BSEB changed its page. A field name in the program needs updating |
| Photo or signature rejected by BSEB | The portal has size limits. Ask for `IMAGE_MAX_KB` to be set |
| Black window closes immediately | Open `run.bat` by double-click, not from inside a zip. Extract the toolkit folder first |

---

## Checklist

- [ ] Python installed with PATH ticked; `run.bat` and the `.py` file in one folder (the first use sets itself up)
- [ ] Admission folder downloaded as ZIP from Google Drive
- [ ] `run.bat` option **C** opened Chrome, logged in to BSEB
- [ ] Menu 1 run; marks and city typed (menu 8) and saved
- [ ] Menu 2 on one student, approved, confirmed on the portal
- [ ] Menu 3 or 4 for the rest
- [ ] ZIP deleted from Downloads afterwards

For optional older methods (online access, Colab) see `advanced/ADVANCED.md`. You do not need them.
