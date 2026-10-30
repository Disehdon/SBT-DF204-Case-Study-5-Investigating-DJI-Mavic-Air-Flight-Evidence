# SBT-DF204-Case-Study-5-Investigating-DJI-Mavic-Air-Flight-Evidence
# DJI Mavic Air Flight Evidence Investigation
**International Cybersecurity and Digital Forensics Academy (ICDFA)**
**School of Basic Vocational Training (SVT)**

![Course](https://img.shields.io/badge/Course-SBT--DF204-red)
![Case Study](https://img.shields.io/badge/Case%20Study-05%20DJI%20Mavic%20Air-blue)
![Tools](https://img.shields.io/badge/Tools-ExifTool%20%7C%20MediaInfo%20%7C%20SQLite%20%7C%20Kali%20Linux-informational)
![Status](https://img.shields.io/badge/Status-Complete-darkgreen)

---

### Author Information & Copyright

| Field | Details |
| :--- | :--- |
| **Student Name** | Ibrahim Diseh Garba |
| **Student ID** | 2025/FWSD/11521 |
| **Programme** | Fellowship in Web Application Security & Digital Forensics |
| **Course** | SBT-DF204 — Computer Forensics Case Studies |
| **Instructor** | Aminu Idris, AMCPN |
| **Date** | 12 October, 2026 |
| **Case Reference** | SBT-DF204-C5 |

---
## Executive Summary

This repository contains the forensic artefacts, methodology, and final report for an investigation into DJI Mavic Air flight evidence recovered from an Android logical acquisition of the `dji.go.v4` application. 

> **Primary Finding:** The extraction holds DJI GO 4 artefacts relating to a flight on **2018-06-19**. The aircraft is identified from plaintext strings in the record header as a **Mavic Air** (Aircraft Serial: `0K1DF313BD4PR1`, Battery Serial: `0K4AEBQA3400DT`). A cached video of **234.833 seconds** (3 min 54.8 s) is present, with the `.info` sidecar giving its recording end time as **2018-06-19 18:55:41 UTC**. 
> 
> **Limitation:** The bodies of the main TXT flight record and the DAT records are **AES-encrypted**, meaning no plaintext flight fields (altitude, GPS, speed) could be extracted. Additionally, two supplementary images (`df048_internal_microSD.001` and `DF048.E01`) could not be examined due to rate-limited Dropbox links.

---

## 🔍 Key Findings & Artefacts

Four distinct artefact families were identified and analysed within `Android_Logical/sdcard/DJI/dji.go.v4/`:

1.  **Main Flight Record (TXT):** `DJIFlightRecord_2018-06-19_[14-50-34].txt`. The body is encrypted. Plaintext strings in the header confirmed the drone model (`Mavic Air`) and serial numbers.
2.  **Cached Video (MP4):** `2018_06_19_14_50_39.mp4`. The MP4 container (H.264, 1280x720) has no embedded timestamps. However, the `.info` sidecar declares the recording end time as `Tue Jun 19 14:55:41 EDT 2018` (converted to `18:55:41 UTC`).
3.  **Detailed Flight Records (DAT):** `FLY057.DAT` and `FLY058.DAT`. Headers indicate `DJI_LOG_V3` format. The bodies are AES-encrypted. Filesystem mtimes suggest two separate power cycles occurring 3 minutes 48 seconds apart.
4.  **Cached Pictures (JPEG):** 6 x `thumb_*.jpg`. These are historical thumbnails. While their filesystem mtimes are `2018-06-19`, their filenames encode epoch timestamps dating back to **April 2018**, indicating they are not from the flight in question.

---

## Evidence Handling & Integrity

### Evidence Source
*   **Dataset:** Android logical extraction (`Android_Logical/` - 406 MB, 375 files).
*   **Missing Sources:** `df048_internal_microSD.001` (Raw image) and `DF048.E01` (EWF image) were unavailable due to HTTP 429/403 errors (rate-limiting) from the provided Dropbox links.

### Integrity Verification
*   A complete SHA-256 manifest of all 375 extracted files is stored at `~/Mavic_air/reports/android_tree_hashes.txt`.
*   **Sample Hashes:**
    *   TXT Record: `0a9f1816d52a98f840f6e67c08a14473ae5a0a2e75e9484d5de66368c4fd4c4a`
    *   MP4 Video: `2ae2a4f186803ec592b0af6284634e592c5079395e99f7df3edd583f8e72ab00`
    *   FLY058.DAT: `fdfccf0c5b1d90d189c19032bb34e151914f42fba42c7994d0492cf327ade07a`

---

### Tools Used
The following tools were utilised during the forensic analysis (all running in a Kali Linux Rolling environment):

| Tool | Version | Purpose |
| :--- | :--- | :--- |
| **tree / stat / ls** | system default | Directory enumeration and timestamp analysis |
| **strings / xxd** | system default | Plaintext extraction from encrypted binaries |
| **file** | system default | File-type identification |
| **exiftool** | 13.55 | MP4 container metadata extraction |
| **mediainfo** | 26.05 | Independent MP4 container verification |
| **sha256sum / md5sum** | system default | Integrity verification (hashing) |
| **ewfinfo / ewfverify** | 20140816 | Expert Witness Format verification (installed but not applied due to missing image) |
| **losetup / lsblk** | system default | Read-only loop device tools (available but not applied) |

---

## Reconstructed Timeline (UTC)

| Stage | UTC Timestamp | Evidence ID | Observed Fact |
| :--- | :--- | :--- | :--- |
| **Device State** | 2018-06-19 11:47:50 | E04 | DJI GO 4 cache refreshed (6 thumbnails written). |
| **Device State** | 2018-06-19 11:55:40 | E01 | Main TXT record written (mtime). |
| **Device State** | 2018-06-19 11:55:42 | E02 | MP4 video written (mtime). |
| **Power Cycle 1** | 2018-06-19 11:58:24 | E03 | FLY057.DAT written (mtime). |
| **Power Cycle 2** | 2018-06-19 12:02:12 | E03 | FLY058.DAT written (mtime). |
| **Video End** | 2018-06-19 18:55:41 | E02 | `.info` sidecar declares recording end. |

---

## Challenges & Limitations

1.  **Encrypted Record Bodies:** The main TXT and DAT files are AES-encrypted. Offline tools could not parse the bodies, preventing the extraction of GPS coordinates, altitude, speed, and exact motor-on/off times.
2.  **Time-Zone Anomaly:** There is a **7-hour discrepancy** between the `.info` sidecar header (`18:55:41 UTC`) and the filesystem mtime of the MP4 (`11:55:42 UTC`). The evidence cannot definitively resolve whether this was caused by device clock drift, a timezone change, or extraction artefacts.
3.  **Missing Sources (Q7/Q8):** The raw microSD image and EWF image could not be downloaded (Dropbox rate-limited the links). Consequently, Questions 7 and 8 in the assessment brief are marked as "Not Performed."
4.  **Cache Discrepancy:** The cached thumbnails have filesystem mtimes of `2018-06-19` but filename epochs dating to `2018-04-13/14`. This is a known artefact of DJI GO 4's cache refreshing mechanism, but it highlights the danger of relying solely on mtimes.
5.  **Attribution Gap:** The evidence proves the *device* and *aircraft* were present and active, but it **cannot identify the human operator** holding the controller.

---

##  Recommendations

### For the Investigation
*   **Obtain Missing Evidence:** Bypass Dropbox rate-limits by requesting the raw `df048_internal_microSD.001` and `DF048.E01` files through an official secure evidence transfer method (e.g., direct SFTP or physical media).
*   **Seek Decryption Tools:** Utilise specialised DJI forensic tools (e.g., DatCon, CsvView) or collaborate with DJI to decrypt the `DJI_LOG_V3` DAT files to extract the flight telemetry (GPS, altitude, waypoints).
*   **Correlate with External Logs:** Cross-reference the `18:55:41 UTC` end time with external sources (e.g., mobile network provider logs, CCTV footage of the launch site in Raleigh, NC) to resolve the timezone anomaly.
*   **Seize the Physical Drone:** If the physical Mavic Air is available, extract the internal flash memory to recover higher-resolution logs or deleted flight records.

---

### For Device Security (Organisational)
*   **App Whitelisting:** If this were a corporate device, implement MDM policies to restrict the installation of unauthorised drone applications.
*   **Data-at-Rest Encryption:** Ensure that if drones are used for corporate purposes, the internal storage and mobile app databases are encrypted and logs are offloaded to secure servers immediately after flight.

---

##  Conclusion

The forensic examination successfully identified the aircraft (Mavic Air), its serial numbers, and the date of the flight (`2018-06-19`). The `.info` sidecar provided the most reliable timestamp for the recording's end (`18:55:41 UTC`).

However, the investigation was significantly hampered by the AES encryption of the TXT and DAT records, rendering the core flight telemetry unreadable. Furthermore, the 7-hour time-zone discrepancy between the filesystem and the sidecar prevents the establishment of a definitive flight start time. 

**Final Assessment:** While the technical evidence establishes the presence and operation of the drone on the specific date with high confidence (95%), it cannot prove **who** was operating the drone, nor does it provide the **flight route** or **altitude**. The missing microSD and EWF images would likely have bridged this gap.

---

Copyright & Usage
© 2026 Ibrahim Diseh Garba. All Rights Reserved.

This repository contains academic coursework and a forensic report submitted as part of the Fellowship in Web Application Security & Digital Forensics at the International Cybersecurity and Digital Forensics Academy (ICDFA).

