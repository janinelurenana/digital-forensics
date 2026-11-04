# Digital Forensics Challenge #1 — Evidence Collection

**Platform:** Ubuntu 22.04 LTS (Jammy Jellyfish)  
**Case Number:** DATALEAK-2026-001  
**Evidence Number:** USB01  
**Examiner:** Christelle Janine M. Lurenana  
**Dataset:** NIST CFReDS – Data Leakage Case

## Overview

This challenge demonstrates the evidence collection phase of digital forensics. Using Linux-based forensic utilities, an existing raw DD disk image from the NIST CFReDS Data Leakage Case was inspected, converted into the Expert Witness Format (E01), and verified to ensure data integrity before proceeding to the examination phase.

## Objectives

- Inspect the provided raw DD disk image and identify its partition structure and file system.
- Convert the DD image into E01 format.
- Calculate and compare cryptographic hashes to establish data integrity.
- Verify the resulting forensic image before examination.

## Tools Used

- **Ubuntu 22.04 LTS** – Operating environment.
- **libewf (ewf-tools)** – E01 image acquisition and verification.
- **The Sleuth Kit** – Disk image and file system inspection.
- **SHA-256 / MD5** – Cryptographic integrity verification.

## Evidence Information

| Property | Value |
|---|---|
| Source Image | `cfreds_2015_data_leakage_rm#2.dd` |
| Source Size | 4,004,511,744 bytes |
| Partition Table | DOS / MBR |
| File System | FAT32 |
| Sector Size | 512 bytes |
| Output Image | `rm2_evidence.E01` |
| Image Format | EnCase 6 (E01) |

## Procedure

### 1. Evidence Inspection

The provided DD image was loaded into The Sleuth Kit utilities for preliminary inspection.

**Commands:**

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

**Verification results:**

| Hash Type | Result |
|---|---|
| MD5 (stored) | `b4644902acab4583a1d0f9f1a08faa77` |
| MD5 (calculated) | `b4644902acab4583a1d0f9f1a08faa77` |
| Verification Status | SUCCESS |

## Results

- Successfully inspected the original DD disk image.
- Identified the FAT32 partition and its directory structure.
- Converted the raw DD image into EnCase 6 E01 format.
- Confirmed that the original and acquired data shared the same SHA-256 digest.
- Successfully verified the E01 image using `ewfverify`.

The resulting forensic image is ready to be used as the input evidence for the examination phase in Autopsy.

## Lessons Learned

This activity demonstrated the importance of preserving digital evidence, maintaining consistent case metadata, and verifying image integrity through cryptographic hashing. It also provided practical experience using Linux-based forensic utilities as alternatives to Windows-based forensic tools.

## Screenshots

Store the following screenshots in the `screenshots/` directory:

| Screenshot | Description |
|---|---|
| `01_evidence_inspection.png` | Partition and file system inspection |
| `02_e01_acquisition.png` | E01 conversion and SHA-256 comparison |
| `03_e01_verification.png` | Successful E01 integrity verification |

## Directory Structure

```text
DF-Challenge-1/
├── evidence/
│   ├── cfreds_2015_data_leakage_rm#2.dd
│   └── original_hash.txt
├── output/
│   ├── rm2_evidence.E01
│   └── acquisition.log
└── screenshots/
    ├── 01_evidence_inspection.png
    ├── 02_e01_acquisition.png
    └── 03_e01_verification.png
```

**Note:** Original raw disk image (.dd) and derived E01 evidence segments omitted due to file size constraints.