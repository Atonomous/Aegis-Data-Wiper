# Aegis Data Wiper (Flash Storage Optimized)

## Objective
Aegis Data Wiper is an advanced, military-grade secure data shredding utility optimized for flash storage (USB drives/SSDs). It bypasses standard filesystem limitations by providing both targeted file shredding and full free-space over-allocation wiping to ensure data non-recoverability against physical chip-off analysis and FTL wear-leveling algorithms.

---

## 🚀 Download & Installation (Standalone Application)

You do not need to install Python to use this software. 

1. Download the latest standalone Windows Executable (`Aegis_Data_Wiper.exe`).
2. Double-click `Aegis_Data_Wiper.exe` to run the application. No installation is required.

---

## 1. THE SYSTEM-LEVEL LOGIC (High-Level/Prose Mode)

### Mechanics of File Deletion on Flash Drives
When a standard OS deletes a file, it only removes the file's index entry from the filesystem table (e.g., MFT in NTFS, FAT). The actual data blocks remain intact on the physical storage until the OS needs to write new data to those logical sectors. On magnetic hard disk drives (HDDs), simply overwriting the logical sector guarantees the physical platter is overwritten. However, modern flash storage (SSDs, USB drives) uses complex abstraction layers.

### The Flash Translation Layer (FTL) Conflict
Flash memory has a limited number of write/erase cycles. To maximize the drive's lifespan, the controller employs a **Flash Translation Layer (FTL)** to implement **Wear-Leveling**.
- **The Issue:** When software requests to overwrite logical block "A", the FTL rarely overwrites physical cell "A". Instead, it writes the new data to a fresh, less-used physical cell "B", updates the translation table to point logical block "A" to physical cell "B", and marks cell "A" as stale (to be erased later during Garbage Collection).
- **The Consequence:** Traditional file shredders that overwrite an existing file directly often fail on flash storage. The original data remains on the physical flash chip, hidden from the OS, and can be retrieved by physical chip-off analysis or factory-level diagnostic commands.

### Bypassing Hardware Limitations: The "Over-Allocation" Strategy
To guarantee data destruction on flash media, the most reliable strategy available to user-space software is **Free-Space Wiping** (or over-allocation). 
1. **Targeted File Deletion:** The target file is overwritten and deleted, though the FTL may keep ghosts.
2. **Free-Space Filler:** The utility generates massive "dummy" files that consume **100% of the drive's logical free space**.
3. **FTL Exhaustion:** By forcing the drive to write to every single available logical block, the FTL runs out of fresh cells for wear-leveling. It is forced to trigger garbage collection, erase the stale blocks (which hold the ghost data), and reuse them.
4. **Result:** Once the drive is full, all unallocated physical flash cells have been demonstrably overwritten, destroying remnants of deleted files.

### Performance vs. Security Trade-offs
A single **Random Data Pass** is used for optimal security on flash storage. This method writes cryptographically random bytes, which is optimal for flash media as it defeats internal deduplication/compression engines on modern SSDs while minimizing excessive wear compared to multi-pass standards like DoD 5220.22-M, which are largely irrelevant or even harmful to flash storage lifespan without added security benefits.

---
## 2. THE INTERACTIVE GUI IMPLEMENTATION

### Key Features
- **Modern Dark-Mode UI:** Clean, military-grade interface providing visual clarity and preventing accidental catastrophic clicks.
- **Threaded Execution & Live Console:** Keeps the UI responsive while streaming real-time terminal output of the wiping process.
- **OS-Level Buffering Bypass:** Uses `os.fsync` to ensure data is written directly to the physical storage cells, bypassing RAM cache.
- **Graceful Disk Full Handling:** Catches `OSError` (No space left on device) to safely finalize the free-space wiping process.
- **Metadata Obfuscation:** Renames files randomly before final deletion to destroy MFT/filesystem name records.
