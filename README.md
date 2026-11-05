# Digital Forensics Challenges: Data Leakage Case

**Case Number:** DATALEAK-2026-001  
**Evidence Number:** USB01  
**Examiner:** Christelle Janine M. Lurenana  
**Dataset:** [NIST CFReDS – Data Leakage Case](https://cfreds.nist.gov/)  
**Platform:** Ubuntu 22.04 LTS (Jammy Jellyfish)  

## Overview

This repository documents two consecutive phases of a digital forensics workflow performed on the NIST CFReDS Data Leakage Case:

1. **Challenge #1: Evidence Collection.** A raw DD disk image was inspected, converted to Expert Witness Format (E01), and verified to confirm data integrity.
2. **Challenge #2: Examination.** The verified E01 image was examined in Autopsy using ingest modules and file carving. A relevant file was identified and tagged, and an HTML report was generated.

The E01 image produced in Challenge #1 is the input evidence for Challenge #2.

## Tools Used

| Tool | Purpose |
|---|---|
| Ubuntu 22.04 LTS | Operating environment |
| libewf (ewf-tools) | E01 image acquisition and verification |
| The Sleuth Kit | Disk image and file system inspection |
| SHA-256 / MD5 | Cryptographic integrity verification |
| Autopsy | Digital forensic examination and reporting |
| PhotoRec Carver | File carving from unallocated space |

## Evidence Information

| Property | Value |
|---|---|
| Source Image | `cfreds_2015_data_leakage_rm#2.dd` |
| Source Size | 4,004,511,744 bytes |
| Partition Table | DOS / MBR |
| File System | FAT32 (starts at sector 128) |
| Sector Size | 512 bytes |
| Output Image | `rm2_evidence.E01` |
| Image Format | EnCase 6 (E01) |

---

# Challenge #1: Evidence Collection

## Objectives

- Inspect the provided raw DD disk image and identify its partition structure and file system.
- Convert the DD image into E01 format.
- Calculate and compare cryptographic hashes to establish data integrity.
- Verify the resulting forensic image before examination.

## Procedure

### 1. Evidence Inspection

The DD image was loaded into The Sleuth Kit utilities for preliminary inspection.

```bash
mmls "cfreds_2015_data_leakage_rm#2.dd"

fsstat -o 128 "cfreds_2015_data_leakage_rm#2.dd"

fls -o 128 -r "cfreds_2015_data_leakage_rm#2.dd"
```

**Findings:**

- Identified a DOS/MBR partition table.
- Detected a FAT32 file system beginning at sector 128.
- Identified directories and files within the partition.
- Observed deleted file entries during directory listing.

### 2. Original Evidence Hashing

The SHA-256 hash of the original DD image was calculated to establish a baseline for integrity comparison.

```bash
sha256sum "cfreds_2015_data_leakage_rm#2.dd" | tee original_hash.txt
```

**Original SHA-256:**

```text
d930afa23f8905e88a28041786edbe3791b5bf41b01468027ee27a19cd59d407
```

### 3. DD to E01 Conversion

The raw DD image was converted into EnCase 6 E01 format using `ewfacquire`.

```bash
ewfacquire \
  -f encase6 \
  -c best \
  -d sha256 \
  -m removable \
  -C "DATALEAK-2026-001" \
  -E "USB01" \
  -D "CFReDS Data Leakage Case" \
  -e "Christelle Janine M. Lurenana" \
  -N "Raw DD converted to E01 for Autopsy examination" \
  -t ../output/rm2_evidence \
  -l ../output/acquisition.log \
  "cfreds_2015_data_leakage_rm#2.dd"
```

The acquisition completed successfully. The SHA-256 digest calculated during acquisition matched the original DD image hash.

### 4. E01 Integrity Verification

The resulting E01 image was independently verified using `ewfverify`.

```bash
ewfverify ../output/rm2_evidence.E01
```

| Hash Type | Result |
|---|---|
| MD5 (stored) | `b4644902acab4583a1d0f9f1a08faa77` |
| MD5 (calculated) | `b4644902acab4583a1d0f9f1a08faa77` |
| Verification Status | **SUCCESS** |

## Challenge #1 Results

- Successfully inspected the original DD disk image.
- Identified the FAT32 partition and its directory structure.
- Converted the raw DD image into EnCase 6 E01 format.
- Confirmed the original and acquired data share the same SHA-256 digest.
- Successfully verified the E01 image using `ewfverify`.

## Challenge #1 Screenshots

| Screenshot | Description |
|---|---|
| `01_evidence_inspection.png` | Partition and file system inspection |
| `02_e01_acquisition.png` | E01 conversion and SHA-256 comparison |
| `03_e01_verification.png` | Successful E01 integrity verification |

---

# Challenge #2: Examination Phase

## Objectives

- Examine the verified E01 evidence image using Autopsy.
- Run ingest modules to identify potentially relevant files.
- Recover and identify a relevant carved file.
- Tag the identified evidence and generate an HTML forensic report.

## Procedure

### 1. Create a New Case

| Field | Value |
|---|---|
| Case Name | DataLeakageCase |
| Case Number | DATALEAK-2026-001 |
| Examiner | Christelle Janine M. Lurenana |

### 2. Add the Evidence Image

The verified E01 image from Challenge #1 was added as the data source.

- **Evidence File:** `rm2_evidence.E01`
- **Source:** `DF-Challenge-1/output/`

### 3. Configure and Run Ingest Modules

The following modules were enabled, as specified in the challenge instructions:

- Recent Activity
- Hash Lookup
- File Type Identification
- Extension Mismatch Detector
- Embedded File Extraction
- EXIF Parser
- Keyword Search
- Deleted Files

The PhotoRec Carver module was also run to recover files from unallocated space.

### 4. Review Ingest Results

Reviewed the Autopsy results, including deleted files, keyword hits, and carved files. A relevant file was identified under `$CarvedFiles`:

- **Recovered File:** `f0198240_secret_project_technical_review_3.doc`
- **Location:** `/img_rm2_evidence.E01/vol_vol2/$CarvedFiles/1/`

The filename and its location made it a relevant candidate for further examination in the data leakage case.

### 5. Tag the Evidence File

| Field | Value |
|---|---|
| Tag Name | Evidence - Data Leak |
| File | `f0198240_secret_project_technical_review_3.doc` |
| Location | `$CarvedFiles/1` |

**Tag Comment:**

> Recovered carved document named secret_project_technical_review_3.doc, identified during examination of the removable media image. Retained as a potentially relevant artifact for the data leakage investigation.

### 6. Generate the HTML Report

Report generated via **Tools → Generate Report → Results - HTML**, with *Tagged Results* included in the report configuration. The report contains the case information and the tagged evidence result.

## Challenge #2 Results

- Successfully loaded and examined the verified E01 image in Autopsy.
- Executed ingest analysis and file carving.
- Identified `f0198240_secret_project_technical_review_3.doc` under `$CarvedFiles`.
- Selected the recovered document as the relevant artifact for the investigation.
- Prepared the evidence tag and HTML report.

## Challenge #2 Screenshots

| Screenshot | Description |
|---|---|
| `01_ingest_modules.png` | Ingest module configuration |
| `02_carved_files.png` | Carved file under `$CarvedFiles` |
| `03_html_report.png` | Generated HTML report |

---

# Lessons Learned

- Preserving digital evidence, keeping case metadata consistent, and verifying image integrity with cryptographic hashes are essential before any examination begins.
- Linux-based forensic utilities are practical alternatives to Windows-based forensic tools.
- File carving can recover files that are not visible in the normal file-system directory structure.
- Deleted files and carved files are different forensic findings and should be examined separately.
- File names and metadata can help identify relevant artifacts, but file contents must also be examined before drawing conclusions.
- Evidence tagging and report generation help preserve and communicate examination findings.

# Repository Structure

```text
DF-Challenges/
├── README.md
├── DF-Challenge-1/
│   ├── evidence/
│   │   ├── cfreds_2015_data_leakage_rm#2.dd
│   │   └── original_hash.txt
│   ├── output/
│   │   ├── rm2_evidence.E01
│   │   └── acquisition.log
│   └── screenshots/
│       ├── 01_evidence_inspection.png
│       ├── 02_e01_acquisition.png
│       └── 03_e01_verification.png
└── DF-Challenge-2/
    ├── DataLeakageCase/          # Autopsy case files
    ├── report/                   # Generated HTML report
    └── screenshots/
        ├── 01_ingest_modules.png
        ├── 02_carved_files.png
        └── 03_html_report.png
```

> **Note:** The original raw disk image (`.dd`) and derived E01 evidence segments are omitted from this repository due to file size constraints.
