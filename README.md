# File System-Based Digital Forensics: Deleted File Recovery (NTFS)

## Overview

You are a newly hired **Junior Digital Forensic Analyst at Sentinel Security Solutions**, a cybersecurity firm specializing in **incident response and forensic investigations**.

Your first assignment is to investigate a **suspected data deletion on a corporate drive**.

### Investigation Objective

Recover a deleted **JPEG image** from a forensic disk image, analyze its **NTFS metadata**, and verify the **integrity of the recovered evidence**.

This investigation uses:

- Command-line forensic tools
- NTFS file system metadata
- Manual byte offset calculations
- Python-based automated file carving
- MD5 hashing for forensic verification

---

# Learning Objectives

By the end of this investigation, you will be able to:

- Analyze **NTFS file system metadata**
- Identify **deleted files using forensic tools**
- Extract **file metadata from the Master File Table (MFT)**
- Calculate **byte offsets for manual carving**
- Recover deleted files using **CLI tools and Python**
- Verify file integrity using **MD5 hashing**

---

# Tools and Resources

## Tools

### Forensic Virtual Machine
All tools and files are preloaded in the VM.

**Download Link:**
https://drive.google.com/file/d/1vgRgNsmWfugncEfx7ncbHsSy6FHOMI7z/view

---

### Disk Image File
- `forensic_image.dd`
- A forensic copy of an **NTFS disk**

---

### Sleuth Kit CLI Tools

Used for forensic file system analysis.

| Tool | Purpose |
|-----|------|
| `fls` | List files (including deleted files) |
| `fsstat` | Display file system metadata |
| `istat` | Display detailed metadata about a file |
| `icat` | Extract files from disk images |

---

### Hex Editor

Used to manually verify file headers and inspect raw disk data.

Examples:

- **HxD**
- **xxd**

---

### Python 3

Used to run an automated **file carving script** to extract files using calculated byte offsets.

---

## Resources

### Digital Forensics Lab Guide

Make a copy of the document:

https://docs.google.com/document/d/1znfatFau0N6pg_LPBfcX_P9Cp4qxoD8CFatdib4KwZ4/copy

---

### Python File Carving Script

Preloaded in the VM:

```
~/Forensics/file_carver.py
```

---

# Investigation Instructions

---

# Task 1: Set Up the Virtual Environment

## Step 1: Launch the Ubuntu VM

1. Open **VirtualBox**
2. Start the provided **Ubuntu VM**

Login credentials:

```
Username: student
Password: student
```

---

### Navigate to Forensic Materials

```bash
cd Forensics
```

---

## Step 2: Verify Installed Tools

Check if forensic tools are installed.

```bash
fls -v
fsstat -v
```

If tools are missing:

```bash
sudo apt update && sudo apt install sleuthkit -y
```

Verify again:

```bash
fls -v forensic_image.dd
fsstat -v forensic_image.dd
```

---

# Task 2: Verify the File System & Gather Metadata

---

## Step 1: Identify File System Type

Mount the disk image in **read-only mode**.

```bash
sudo losetup -Pf forensic_image.dd
```

Check file system information.

```bash
sudo fsstat /dev/loop0
```

If the file system type cannot be determined:

```bash
file forensic_image.dd
```

Expected result:

```
File System Type: NTFS
```

---

## Step 2: Gather File System Metadata

Display file system metadata.

```bash
fsstat forensic_image.dd
```

### Key Information

| Metadata | Value |
|--------|------|
| File System | NTFS |
| Cluster Size | 4096 bytes |
| First Cluster of MFT | 4 |
| MFT Mirror | 6399 |
| MFT Entry Size | 1024 bytes |
| Index Record Size | 4096 bytes |
| Root Directory | 5 |
| Sector Size | 512 bytes |
| Total Clusters | 0 - 12798 |
| Total Sectors | 0 - 102398 |

---

# Task 3: Locate Deleted File Information

The **Master File Table (MFT)** stores metadata for every file in an NTFS system.

Even if a file is deleted, its metadata often remains.

---

## Step 1: List Files in the Disk Image

```bash
sudo fls -r forensic_image.dd
```

Look for deleted files marked:

```
-/r *
```

Example:

```
-/r * 64-128-2: IMG_3046.jpg
```

Meaning:

- File was **deleted**
- MFT record still exists
- File may be recoverable

---

## Deleted File Identified

```
IMG_3046.jpg
MFT Entry: 64-128-2
```

Additional orphaned files may appear:

```
-/r * 16: OrphanFile-16
-/r * 17: OrphanFile-17
-/r * 18: OrphanFile-18
```

---

# Step 2: Extract File Metadata

Analyze the file using its **MFT entry number**.

```bash
sudo istat forensic_image.dd 64-128-2
```

---

### Metadata Findings

| Attribute | Value |
|------|------|
| File Name | IMG_3046.jpg |
| Created | 2025-03-06 10:33:21 |
| File Size | 10,626,782 bytes |
| Allocated Size | 10,629,120 bytes |
| Storage Clusters | 6912 - 9506 |

---

### Why Metadata Matters

Metadata helps investigators determine:

- When a file was **created**
- When it was **modified**
- Where it was **stored**
- Whether it has been **tampered with**

---

# Byte Offset Calculation for File Carving

To recover the file manually, calculate the byte offsets.

---

## Start of File (SOF)

```
Cluster Number × Cluster Size
```

```
6912 × 4096 = 28,311,552
```

Convert to hexadecimal:

```bash
printf "%x\n" 28311552
```

Result:

```
1B00000
```

---

## End of File (EOF)

```
SOF + File Size
```

```
28311552 + 10626782 = 38938334
```

Convert to hexadecimal:

```
25226DE
```

---

# Task 4: Recover Deleted Files

---

## Method 1: CLI Recovery Using icat

```bash
sudo icat forensic_image.dd 64-128-2 > recovered.jpg
```

Verify the file:

```bash
file recovered.jpg
```

---

## Method 2: Automated File Carving Using Python

Run the carving script.

```bash
python3 ~/Forensics/file_carver.py
```

Enter the following:

```
File name: forensic_image.dd
Start Offset: 1B00000
End Offset: 25226DE
Output file name: carved.jpg
```

---

# Task 5: Verify File Integrity

Even if the file opens, forensic verification is required.

---

## Generate MD5 Hashes

```bash
md5sum recovered.jpg carved.jpg
```

Expected output:

| File | MD5 Hash |
|----|----|
| Original | bfc40fd6b8adf2e7745d833de36b4bc0 |
| Recovered | bfc40fd6b8adf2e7745d833de36b4bc0 |
| Carved | bfc40fd6b8adf2e7745d833de36b4bc0 |

If hashes match, the recovery was **successful**.

---

# Troubleshooting

| Challenge | Solution |
|---|---|
| File appears corrupted | Ensure SOF and EOF offsets are correct |
| Wrong file type recovered | Verify file signature |
| icat extraction fails | Use manual carving or hex editor |

---

# Decision Points

During forensic investigations, analysts must decide:

- When to use **manual carving vs automated carving**
- How **file system structures affect recovery**
- Why **file signatures matter for carving automation**

---

# Summary

During this investigation you:

- Set up a **forensic Ubuntu VM**
- Analyzed an **NTFS disk image**
- Located **deleted files using fls**
- Extracted metadata using **istat**
- Recovered files using **icat**
- Performed **manual byte offset calculations**
- Carved files using a **Python script**
- Verified integrity using **MD5 hashing**

These techniques are fundamental for **real-world digital forensic investigations**.

---

# Forensic Investigation Overview

## Case Summary

The objective of this investigation was to recover a deleted image file from an **NTFS forensic disk image** and validate its integrity.

The analysis confirmed:

- The deleted file metadata remained in the **Master File Table**
- The file data was still present in disk clusters
- Successful recovery was possible using forensic tools

---

# Reflection and Forensic Insights

### Why Metadata Analysis Matters

Metadata allows investigators to:

- Verify file authenticity
- Detect potential tampering
- Reconstruct forensic timelines

---

### Importance of Timestamps

Timestamps help determine:

- When files were created
- When they were accessed
- When they were modified

This information is critical for **incident response investigations**.

---

### Benefits of Automated File Carving

Automated carving:

- Reduces **human error**
- Speeds up investigations
- Enables **large-scale evidence recovery**

---

# Next Steps

In the next lab you will:

- Recover **multiple file types**
- Compare **NTFS vs FAT32 recovery**
- Simulate **real-world forensic investigation scenarios**

---
