# BSEB Class IX Registration Helper (Windows)

Fills the BSEB registration form for your Class IX students using the data your admission web app already collects.

- **Your live Google Sheet and Drive folder are never touched.** You only *download* a copy.
- **Nothing is made public.**
- **You stay in control.** You log in to BSEB yourself, and you can approve every preview before it saves.
- **Photos and signatures are not kept on your computer.** They are read from the downloaded ZIP and sent to the form.

## Your sheet format

The tool understands the **current sheet** (one row per applicant with these columns: `applicationId, submittedAt, admClass, stream, pen, apaar, aadhar, eshiksha, studentName, dob, gender, category, religion, motherName, fatherName, aadharOwner, guardianAadhar, mobile, address, pincode, distance, cwsn, income, bloodGroup, height, weight, prevUdise, bankAccount, ifsc, bankName, accHolder, accRelation, photoUrl, signatureUrl`) and the older one. It may come as a **CSV file** downloaded from your web app, as an Excel file, or as the sheet inside the Drive folder ZIP; the tool picks the copy with the **newest applications**.

What the current sheet does **not** contain, and how the tool copes:

| Missing in the sheet | What the tool does |
|---|---|
| Roll number | Numbers the Class IX students **1, 2, 3 ...** in the order they applied (shown as "No."). Use that number in menu option 2 |
| Language (Hindi or Urdu) | Takes it from `extra_info.csv`. If empty: Hindi + Sanskrit is assumed, **except Muslim students, who must be typed (URDU or HINDI)** because they are the ones who may take Urdu. Every run says how many are assumed, and the final summary shows the split |
| Email | Leaves the email field on the BSEB form empty, and says so once. If BSEB insists on an email you will see it at the first preview |
| Identification marks, town/city | Typed once (menu 8), see below |
| Pictures as file IDs | Pictures come out of the ZIP by application ID; the online mode reads the ID out of `photoUrl` / `signatureUrl` |

## The whole process

| Step | What you do | Where |
|---|---|---|
| 1 | Right-click your admission folder, choose **Download** (the pictures). If your web app gives you a CSV file, download it too | Google Drive / web app |
| 2 | Double-click `run.bat`, press **C**, log in to BSEB in the Chrome window that opens | Your PC |
| 3 | In the same menu choose **1** (check the data). It creates `extra_info.csv` | Your PC |
| 4 | Choose **8**, type the identification marks and city in the black window | Your PC |
| 5 | Choose **2** for one student (you approve), then **3** for everyone | Your PC |

Next day, when new students have applied: do steps 1, 2, 3 and 5 again. Students already done are skipped.

First time only: install Python (free). The first time you choose an option in `run.bat` it sets itself up by itself; that takes a few minutes and needs internet.

## What is in this folder

| File | What it is |
|---|---|
| `run.bat` | **The only file you open.** A menu that sets the tool up, opens Chrome, and does everything. It needs no other `.bat` file |
| `bseb_class9_registration.py` | The program itself. Keep it in the same folder as `run.bat` |
| `requirements.txt` | Just a list of the two small parts the tool uses (for reference; not needed by `run.bat`) |
| `setup_guide.md` | Step-by-step instructions with what you will see |
| `architecture.md` | How it works inside |
| `advanced/` | Optional older methods (online access, Colab). Not needed |

## Why `extra_info.csv`? (and no Excel is needed)

The BSEB form asks for two identification marks and the town/city. Your sheet does not collect them. So the tool asks you once and remembers.

**You do not need Excel or any spreadsheet program.** Two easy ways:

1. **Type them in the black window** (`run.bat` option **8**). The tool shows each student and asks for the two marks; the town/city can be typed once for everyone.
2. **Edit the file in Notepad** (`run.bat` option **9**). `extra_info.csv` is a plain text file. If you ever have Excel, LibreOffice or Google Sheets, it opens there too.

The file has one line per Class IX student:

| Column | You type | Purpose |
|---|---|---|
| `idMark1`, `idMark2`, `city` | the values | Needed by the BSEB form. Remembered for the next run |
| `alreadyRegistered` | `YES` | Student was registered on BSEB outside this tool, so never send again |
| `ignore` | `YES` | Skip this application (for example a second application of the same student) |
| `distinct` | `YES` | This is a **different person** who only happens to have the same name, father and mother as another student. Register both |

Your live sheet is not changed.

## Works on a low-end PC (Windows 10, 4 GB RAM)

| | Measured |
|---|---|
| Memory used by this program | about **60 to 70 MB** (60 MB for a school-sized sheet; 71 MB for a very large test with 3,000 sheet rows and 600 students' photos) |
| Disk space for what it installs | about **41 MB** (two small packages: `selenium`, `openpyxl`) |
| Time to check a school-sized download | about 1 second |
| pandas, Excel, LibreOffice, Pillow | **Not used or needed** |

The heavy part of any registration is the Chrome window that shows the BSEB portal, not this tool. Tips for 4 GB: close other programs and browser tabs, use only the one Chrome window opened by `run.bat` option C, close the Google Drive tab after downloading, and restart the PC before a long session. The numbers above were measured on a test machine, not on your PC.

## Keeping track of who is registered

- `run.bat` option **7**: one table showing every student as REGISTERED, PENDING or BLOCKED (with the reason).
- `run.bat` option **5**: how many are registered and the last 5.
- The log file `C:\Users\<you>\bseb_registration\class9_registration_log.tsv` has one line per registered student (date and time, application ID, roll, name, parents, date of birth).

## Checks done automatically before anything is typed on the portal

Aadhaar 12 digits, APAAR 12 digits, mobile 10 digits, pincode 6 digits, IFSC format, bank account digits (catches numbers damaged like `1.0332E+15`), date of birth, email typos (`gamil.com`), duplicate Aadhaar / bank account / APAAR, photo and signature present and really JPG or PNG, identification marks and city present. A student with a problem is skipped and the reason is shown; the others continue.

## Many classes, many downloads: how mistakes are prevented

Your sheet holds Class IX, X, XI and XII and grows every day. Each download is a full copy, so the tool protects against wrong-class and double registration in layers:

| Risk | Protection |
|---|---|
| A Class X/XI/XII student registered as Class IX | Only rows whose class column says Class IX are used. If the class column cannot be found the tool **stops** instead of guessing. Unrecognised class values are listed and never registered. Every run prints how many rows each class has. A Class IX row that has a stream (Science/Arts) is blocked as suspicious |
| Same student registered twice | Log of every saved student (by application ID, and by names + date of birth); the portal's declaration list is read (all pages); the same student applying twice is detected and blocked; one run never sends the same student twice; you can mark students `alreadyRegistered` |
| Two different students with identical name, father and mother | The tool cannot know they are different people, so it **blocks both and asks you once**: type `YES` in the `distinct` column for them (or `ignore` for the extra application if it is one student who applied twice). After that both are registered, each with his own pictures. Works the same in the ZIP and Colab methods |
| Old ZIP used by mistake | Only the newest ZIP is used and its age is shown; older admission ZIPs are listed as ignored; a warning appears after 24 hours. Even an old ZIP cannot register someone twice, because the log remembers everyone |
| Registering without looking | Before a bulk run you see a summary (data source, counts, roll numbers) and must type YES |

Details and answers to common questions are in `setup_guide.md`.

## Good to know

- Only the **Class IX** form is automated. Rows for other classes are ignored.
- The ZIP has everyone's data (Aadhaar, bank accounts, photos). **Delete the ZIP from Downloads when you finish** and empty the Recycle Bin.
- The ZIP is a copy at one moment. Students who apply later are not in it, so download again for new students. The tool warns if the ZIP is more than a day old.
- If BSEB changes its web page, a field name may need updating (see troubleshooting in `setup_guide.md`).
- The program was tested with a simulated browser, simulated Google files and the batch files run under a Windows emulator. It has not been run on your PC or the live BSEB portal, so always **test one student with approval first** (`run.bat` option 2).
