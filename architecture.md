# Architecture

## 1. Overview

```
 Google Drive (your admission folder: sheet + all photos)       <- never modified
        │  you: right-click > Download
        ▼
 Downloads\Admission Upgrade-....zip      (a copy; you delete it afterwards)
        │  read in memory, nothing extracted
        ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │ bseb_class9_registration.py   (runs on your Windows PC)          │
 │                                                                  │
 │  ZipSource ─► sheet (xlsx or csv) ─► rows ─► + extra_info.csv   │
 │      │                                    │                      │
 │      │                               build_student()             │
 │      │                               validate + cross_check      │
 │      │                                    │                      │
 │      └─► photo / signature bytes ◄── ZipAssets (by applicationId)│
 │                                           │                      │
 │              log (skip done) ──► register_student() ──► Chrome   │
 └───────────────────────────────────────────┬──────────────────────┘
                                             ▼
                  reg28.biharboardonline.org  (you are logged in, port 9222)
```

Everything runs locally. No Google login, key, public link or cloud service is involved.

## 2. Design decisions

| Decision | Reason |
|---|---|
| **Use a Drive folder download (ZIP)** | The simplest way to get data and photos without touching the live sheet, making anything public, or setting up cloud access. Drive converts a Google Sheet inside the folder to Excel when you download |
| **Find photos by `applicationId` in the file name** | Your web app already names files `<applicationId>_photo.jpg` and `<applicationId>_signature.jpg`. No Drive file IDs are needed |
| **Read the ZIP in memory** | Photos are never extracted to a folder, which was the original requirement |
| **`extra_info.csv` for marks and city** | The form needs them, the sheet does not have them, and the live sheet must not change. A plain CSV (not Excel format) so it needs no spreadsheet program: it is filled in with `--fill` or Notepad |
| **Read the sheet in streaming (read-only) mode** | Cuts memory roughly in half on large sheets; important on a 4 GB PC |
| **Only two dependencies** | `selenium` and `openpyxl`. No pandas, no Pillow, no Google libraries unless an advanced mode is used (they are imported only then) |
| **Validate before filling** | Fixing a spreadsheet row is cheap; fixing a submitted government record is not |
| **Attach to your own Chrome (port 9222)** | Password and captcha stay with you |
| **Log written only after BSEB confirms the save** | A crash can never mark a student as done |
| **Whole-word dropdown matching** | `BC` must never match `EBC`, `MALE` never `FEMALE` |

## 2a. Sheet formats and how they are mapped

The tool matches column names without caring about case, spaces, `_` or `-`, and accepts several names for each field (`ALIASES`). Both the older and the current sheet work.

| Field | Older sheet | Current sheet | Handling |
|---|---|---|---|
| Picture IDs | `photoFileId`, `signatureFileId` | `photoUrl`, `signatureUrl` (links like `.../thumbnail?id=<ID>&sz=w1000`) | ID read from the link (`id=`, `/d/<ID>`, or a plain ID). In ZIP mode pictures are found by applicationId instead |
| Roll number | `rollNo` | **absent** | If the sheet has no roll column at all, students are numbered 1, 2, 3 ... by `submittedAt` (oldest first, ties by applicationId); shown as "No." |
| Language | `langGroup` (`URDU / NLH`, `HINDI / SANSKRIT`) | **absent** | `language` from `extra_info.csv`; if empty: HINDI + SANSKRIT assumed, except `religion = Muslim`, which is blocked until URDU or HINDI is typed |
| Email | `email` | **absent** | One note; no per-student warning; email field left empty |
| Submission time | `submittedAt` as `30-09-2026 06:11` | `submittedAt` as `2026-09-23T04:49:00.000Z` | Both parsed; used for numbering and to choose the newest sheet |
| `updatedAt`, `otr`, subject columns, device and location columns | present | absent | Not used |

Numbers are only labels for pointing at a student on screen. Who is registered is decided by `applicationId` and the name key, never by the number, so numbers shifting (for example a student's class corrected later) cannot cause a wrong or double registration.

Identity rules (names are compared ignoring case, spaces and dots):

| Where | Key | Why |
|---|---|---|
| Registration log | applicationId, and (name, father, mother, **date of birth**) | The log has the date of birth, so same names with a different date of birth are treated as a different person |
| BSEB declaration list | (name, father, mother) counted per list; no date of birth is shown | A name-only match is not proof. A student is skipped only when the portal lists these names **more often than our own log explains** |
| Duplicate applications inside the sheet | same name + father, and the same date of birth or the same mother | Cannot distinguish "one student, two applications" from "two students with the same names", so both are blocked until the user answers (`ignore` or `distinct`) |
| Students marked `distinct` | applicationId only | The user declared them different people; name-based checks (duplicate rule, log names, portal list, in-run guard) are bypassed for them. Aadhaar, APAAR and bank account duplicates are still errors |

## 3. Where data and pictures come from

| Mode | When | Data | Pictures |
|---|---|---|---|
| **ZIP (normal)** | No `BSEB_SHEET_ID`, no `--file` | Sheet inside the ZIP, or a separately downloaded `.xlsx` or `.csv` in Downloads | Inside the ZIP, found by applicationId |
| ZIP + file | `--zip` and `--file` | The `.xlsx` or `.csv` you name | ZIP |
| Online (advanced) | `BSEB_SHEET_ID` set, or `--service-account`, or `--file` alone | Google Sheets export | Google Drive by file ID |

### Finding the ZIP

1. Look in `BSEB_ZIP_DIR` (if set), `Downloads`, `OneDrive\Downloads`, `Desktop`.
2. Take the 15 newest `.zip` files.
3. The newest ZIP that has the admission sheet **or** pictures named `<applicationId>_photo|signature...` wins. A ZIP with other pictures is used only if nothing better exists.
4. Unrelated or damaged ZIPs are skipped. Other usable ZIPs are printed as IGNORED.
5. The name and age of the ZIP are printed. A ZIP older than 24 hours triggers a warning.

### Reading the ZIP (`ZipSource`)

- Pictures are found by applicationId. Exact names `<applicationId>_photo|signature[ (n)].jpg|jpeg|png` match first. Otherwise, once the sheet is known, any picture whose name contains the application ID as a whole token matches; the name must contain `sign`/`sig` (signature) or `photo`/`pic`/`passport`/`portrait`/`face`/`image`/`img` (photo). If a student has two pictures and one is clearly the other kind, the unnamed one takes the remaining role. An ID that is the beginning of a longer ID never matches the longer one's pictures.
- If a picture is missing, the error shows three example names found in the ZIP.
- If the same picture appears twice (a re-upload), the entry with the newest timestamp is used.
- `__MACOSX`, `._*` and `~$*` entries are ignored.
- Each picture is checked for a real JPEG or PNG header before use, so an error page or other file saved with a `.jpg` name is rejected.
- The data tab is found by scoring each tab on how many known form columns it has, so extra tabs do not matter.
- The sheet may be `.xlsx` (read in streaming mode) or `.csv` (UTF-8, Windows-1252, `,` `;` or tab separated).

### Choosing between several copies of the sheet

Candidates: the sheet inside the chosen ZIP (`.xlsx` or `.csv`), and up to five `.xlsx` / `.csv` files in Downloads that look like the admission sheet (enough known columns). Each is read and its newest `submittedAt` is found. The candidate with the newest application wins; on a tie a file in Downloads beats the one inside the ZIP. Every candidate is printed with its row count and newest application date, marked USED or ignored. `--file` overrides the choice. Both are read by the program itself; Excel is never required.

## 4. Run sequence

```
1. Find the ZIP (or use --zip)                      print name and age; list ignored older ZIPs
2. Load the sheet                                   ZIP, loose .xlsx, --file, or online
3. Find the class column; STOP if there is none     never guess the class
4. Print rows per class; warn about unrecognised class values
5. Keep Class IX rows only
6. Merge extra_info.csv                            idMark1, idMark2, city; read alreadyRegistered / ignore
7. Build + validate every student                   errors and warnings (includes the stream check)
8. Check pictures exist and are real images         added to errors
9. Cross-check duplicates                           Aadhaar / APAAR / bank = error, email = warning,
                                                    same student twice = error for both
10. Write validation_report.csv, update extra_info.csv
11. Classify: registered (log / portal / marked) | ignored | valid | blocked
12. --status: print the table and stop;  --dry-run: stop after the report
13. Open Chrome, read the portal's declaration list (all pages)
14. Print READY TO REGISTER summary, ask for YES    (not with --yes or --only-roll)
15. Register one student at a time; stop at the first portal or browser error;
    skip anyone already sent in this run
```

## 5. `extra_info.csv`

A plain text file, UTF-8 with a BOM, comma separated, one line per Class IX student. It needs no spreadsheet program.

| Rule | Detail |
|---|---|
| Created | Automatically by every run except `--status` |
| Columns | `applicationId, rollNo, studentName, fatherName, idMark1, idMark2, city, language, alreadyRegistered, ignore, distinct` (`rollNo` holds the tool's No. when the sheet has no roll column) |
| Ways to fill | `--fill` (run.bat 8) asks in the black window: town/city once for everyone, then two marks per student, saving after each student; or edit in Notepad (run.bat 9); or any spreadsheet program |
| `alreadyRegistered` = YES | Student done outside the tool: never sent, shown as REGISTERED (marked by you), takes no part in duplicate checks |
| `ignore` = YES | Application skipped (for example a second application of the same student): never sent, shown as IGNORED, takes no part in duplicate checks |
| `distinct` = YES | A different person who shares name, father and mother with someone else: still registered, but exempt from every name-based duplicate rule |
| Matching | By `applicationId` |
| Preserved | What you typed is kept across runs. Entries for students missing from a newer download are kept at the bottom, never deleted |
| Priority | If the Google Sheet itself has `idMark1`/`idMark2`/`city`/`language` columns, those values win |
| Tolerated when read | UTF-8 with or without BOM, Windows-1252 (Excel "CSV (Comma delimited)"), `;` or tab separators, spaces around values, quoted values containing commas, short lines (trailing commas deleted), `yes`/`YES`/`Y`/`1`/`TRUE`/`DONE` |
| Damaged or unreadable | The tool warns and does **not** overwrite it; `--fill` also refuses |
| Written safely | To a temporary file, then renamed over the old one; if another program has the file open, a warning is shown and the run continues |

`--fill` only asks about students who are still missing marks or a city, and never about students already registered or ignored. `q` or Enter-on-empty ends or skips without losing anything typed.

## 6. Registering one student

| Step | Action | If it fails |
|---|---|---|
| 1 | Read photo and signature bytes from the ZIP | Stops before the portal is touched |
| 2 | Open `/add-student`; choose REGULAR if the category dialog shows | Error, stop |
| 3 | Fill text fields, set DOB through React's value tracker, choose dropdowns | `LookupError` lists the options the portal really offers |
| 4 | Attach images: temporary files (default) or in-memory `DataTransfer` (`--no-disk`) | Stops if file inputs are missing |
| 5 | Click Preview; wait up to 10 s for the modal | Prints the portal's red field errors |
| 6 | With `--review`, wait for `y`; click Save; wait for redirect to `declaration-list` | Not logged unless the redirect happened |

Dropdown choice: exact text, then "starts with a whole word", then "contains a whole word". With no match it stops and lists the portal's real options rather than guessing.

## 6a. Validation rules

| Field | Rule |
|---|---|
| Names | Required; warning on non-English characters |
| DOB | `DD/MM/YYYY` (also `YYYY-MM-DD`, `DD-MM-YYYY`); before today; warning if age outside 11 to 19 |
| Gender, category | MALE / FEMALE / TRANSGENDER; GENERAL / EBC / BC / SC / ST |
| Language group | URDU → MIL Urdu + SIL Hindi (NLH); HINDI → MIL Hindi + SIL Sanskrit |
| Aadhaar, APAAR | Empty (Aadhaar: declaration ticked) or exactly 12 digits |
| Mobile | 10 digits starting 6 to 9 |
| Email | Valid shape; typo domains blocked |
| Pincode | 6 digits |
| Bank account | 9 to 18 digits; scientific notation or truncated numbers blocked |
| IFSC | `^[A-Z]{4}0[A-Z0-9]{6}$` |
| ID marks, city | Required (from `extra_info.csv`) |
| Photo, signature | Present in the ZIP and a real JPG/PNG |

## 7. Tracking and duplicates

| File (`C:\Users\<you>\bseb_registration\`) | Purpose |
|---|---|
| `class9_registration_log.tsv` | Append-only: timestamp, `REGISTERED`, applicationId, roll, name, father, mother, DOB. Written only after BSEB redirects to the declaration list |
| `extra_info.csv` | Marks and city |
| `validation_report.csv` | Each student OK/ERROR with reasons (UTF-8 with BOM for Excel) |
| `status_report.csv` | The `--status` table |

A student is skipped if their applicationId is in the log, or their name + father + mother is in the log or on the first page of the portal's declaration list.

`--status` logic, in order: this application is in the log → REGISTERED; marked `alreadyRegistered` → REGISTERED (marked by you); marked `ignore` → IGNORED; same names **and date of birth** as an entry in the log under another application (and not marked `distinct`) → REGISTERED (as another application of the same student); errors → BLOCKED; otherwise PENDING. It does not open Chrome or check the portal.

If the portal saves a student but the redirect takes more than 15 seconds, the script stops without logging that student. Check the declaration list before re-running.

## 7a. Safety against wrong-class and double registration

**Class**
- Class values are normalised (`Class IX`, `ix`, `9`, `9th`, `Ninth` → IX; X, XI, XII are recognised so they can be excluded). Anything else is "unrecognised": reported and not registered.
- A row counts as Class IX only if the sheet **has** a class column (`admClass`, `Class`, `Admission Class`, ...). If none exists the run stops. An earlier version kept every row when the column was missing, which would have sent Class X to the Class IX form.
- The per-class row counts are printed on every run.
- A non-empty stream (other than `-`, `NA`, `NONE`, ...) on a Class IX row is a blocking error.

**Double registration (layers)**
1. Log by application ID, and by name + father + mother + date of birth.
2. Portal declaration list read across all pages (next-page button; capped at 300 pages; stops if the page does not change), counted per (name, father, mother). A student is skipped only if the portal shows these names more often than the log accounts for; the user can override with `distinct`.
3. Duplicate applications inside the sheet: same normalised name and father, and the same DOB or the same mother. Both rows are blocked until the user marks the extra one `ignore` (one student, two applications) or each `distinct` (different people).
4. Aadhaar / APAAR / bank account shared between rows is a blocking error.
5. `alreadyRegistered` for students done outside the tool.
6. In-run guard: after each save the application ID and the (names + date of birth) key are added to the done-set, and anyone matching is skipped, unless marked `distinct`.
7. Final summary and YES before bulk registration.

**Old versus new downloads**
Only the newest ZIP containing the sheet is used; other admission ZIPs are listed as ignored; ZIPs over 24 hours old trigger a warning. Because the log lives outside the ZIP, switching between ZIPs can never cause a second registration. It can only mean stale data.

Limits: wrong class typed by the student, the same person under a different name and different parents, and registrations made elsewhere and not marked are not detectable. The portal name-matching and paging could not be tested against the live portal.

## 8. Windows files

| File | Role |
|---|---|
`run.bat` is one self-contained file. It needs no other `.bat` file (an earlier design used separate setup, Chrome and config files, which failed when they were missing).

| Part of `run.bat` | Role |
|---|---|
| Start-up | Switches to its own folder; checks that `bseb_class9_registration.py` is next to it (clear message if not); calls an **optional** `config.bat` only if it exists |
| `:needvenv` / `:setup` | Run automatically the first time an option needs the tool (or with **S**). Finds Python by actually running `py --version`, then `python --version` (so the Microsoft Store shortcut is not mistaken for Python), creates the private `venv` folder, installs `selenium` and `openpyxl` (quoted version specs, since `>` is a redirect character in cmd). Clear messages for missing Python or a failed install |
| **C** | Finds Chrome (`Program Files`, `Program Files (x86)`, `LocalAppData`), starts it with `--remote-debugging-port=9222` and a separate profile folder, opens the BSEB site |
| **N** | Switches `--no-disk` (photos from memory only) on and off for the session |
| Menu | Menu 1 check, 2 one student, 3 all (the program shows a summary and asks YES), 4 all with approval, 5 count, 6 open folder, 7 status, 8 type marks and city in the window (`--fill`), 9 open `extra_info.csv` in Notepad |

## 9. What the tool reads, writes, exposes

| Item | Action |
|---|---|
| Your live Google Sheet and Drive folder | Not accessed at all (you download a copy yourself) |
| Your web app / Apps Script | Not affected |
| The ZIP in Downloads | Read only. Nothing is extracted |
| `C:\Users\<you>\bseb_registration\` | Log, report, status and `extra_info.csv` are written here |
| Temporary files | Only in default mode (a temp folder removed after each student). None with `--no-disk` |
| Public sharing | None |

## 10. Testing and its limits

| Part | How it was tested | Result |
|---|---|---|
| Registration script and notebook | 160 automated checks (101 for the tool using your real CSV as the main fixture, 29 for identical names and the portal list, and 30 for the notebook) with a simulated Chrome and simulated Drive and Sheets: validation of real problem rows, ZIP search, newest-copy choice, fake pictures, missing signature, no ZIP, damaged or locked `extra_info.csv`, review mode, failures, re-runs, Hindi text, CSV and Excel-style variants of `extra_info.csv`, the typed `--fill` mode, the sheet as CSV, class recognition and class-column loss, duplicate applications, `alreadyRegistered` / `ignore`, the YES confirmation, old versus new ZIPs, portal list paging | All pass |
| `.bat` files | Run in a Windows emulator (Wine) with stand-ins that record the commands they launch | All menu paths work |
| Optional Colab notebook | Simulated Drive and Sheets | Pass |

Not tested: a real Windows PC, your real Google download (the assumption that Drive puts the sheet in the ZIP as Excel is handled by a fallback), the live BSEB portal. Parts of `run.bat` option 5 use PowerShell, which the emulator lacks. Always test one student with approval first.

## 10a. Resource use (measured on a test machine, not on your PC)

| Case | Peak memory of the program | Time |
|---|---|---|
| School-sized: 300 sheet rows, 80 students with photos | about 60 MB | under 1 s |
| Very large test: 3,000 sheet rows, 600 students' photos (133 MB ZIP) | about 71 MB | about 3 s |
| Same large sheet read as CSV | about 77 MB | about 1 s |
| Earlier version without streaming reads (same large test) | about 129 MB | |

Python and Selenium alone account for about 54 MB, so CSV and Excel-format sheets end up almost the same. Pictures are read from the ZIP one at a time. Installed size of `selenium` and `openpyxl` with their dependencies: about 41 MB. The Chrome window used for the portal is far heavier than the tool and is the part that matters on a 4 GB PC.

## 11. Extension points

| Change | Where |
|---|---|
| Portal field names changed | `register_student()` |
| Default district, marital status | constants at the top of the script |
| Dropdown label mismatch | `CATEGORY_MAP`, `GENDER_MAP` |
| Picture size limits | `IMAGE_MAX_KB` (needs `pip install Pillow`, which is not installed by default) |
| Other class | New student model and a `register_*` function; the ZIP, validation, log and extra-info parts are reusable |
| Several split ZIPs | Extend `ZipSource` to take a list of ZIP files |
