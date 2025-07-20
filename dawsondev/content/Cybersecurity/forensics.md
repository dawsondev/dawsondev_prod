+++
title = 'Digital Forensics'
date = "2025-05-05"
+++


# Advanced Digital Forensics Guide (Disk Imaging to File Carving)

Digital forensic investigations require a methodical approach and expertise in various tools and techniques. This guide provides a comprehensive, step-by-step walkthrough of advanced disk forensics – from acquiring a forensically sound image of a “cold” (powered-off) system, through detailed file system analysis (FAT32 and ext3/ext4), to recovering deleted data via file carving. All procedures emphasize open-source tools on Linux. Use this as a detailed checklist and reference for conducting thorough forensic analysis. Most of what I know is based on research during my masters thesis (and some tryhackme rooms), although I personally only extensively used certain technologies practically (like bulkextractor) while most I just reviewed as part of research for the thesis.

## Basic Digital Forensics workflow

**NIST Standards** (National institute of Science and Technology) are the guidelines I used in my masters thesis since it provides a strong theoretical background to all relevant workflows in digital forensics. The general process starts with collection and ends with reporting. The following image (based on special publication 800-86 NIST) shows the steps that are taken in virtually all forms of digital forensics in some variation or another. For further reading on this, I suggest reviewing the NIST publications on digital forensics.

![NIST](/images/NIST%20copy2.png)


## Disk Imaging and Acquisition

**Disk Imaging** is the process of creating a bit-for-bit copy of a storage device, preserving *all* data (including deleted files and slack space). The goal is to obtain a **forensically sound** image that exactly replicates the original drive, so analysis can proceed without risking alteration of the original evidence. Adhering to best practices during acquisition is critical:

* **Pre-Imaging Preparation:** Always use a hardware write-blocker or enable write-blocking mode to prevent any writes to the source drive. Document the device state and configure your imaging system to avoid auto-mounting or modifying the source. Ensure you have sufficient storage for the image and that your destination is error-free (consider using `fdisk -l` or `dmesg` to verify the target drive status).

* **Imaging Process:** Utilize reliable open-source tools to create the image. Common options include the Linux `dd` utility (for raw images), enhanced versions like `dcfldd`/`dc3dd`, or GUI tools like **Guymager** (which supports raw `dd`, EWF (E01), and AFF formats). For example, using `dd` you might run:
  `sudo dd if=/dev/sdX of=/media/evidence/case1_disk.img bs=4M conv=fsync`
  This reads from the source `/dev/sdX` and writes a raw image file. Guymager provides a user-friendly interface and can calculate hashes on-the-fly. It’s multi-threaded for speed and can directly produce E01 or AFF images with integrated compression.

* **Hashing and Verification:** Calculate cryptographic hash values (e.g., SHA-256 or MD5) of the source *and* the image to verify integrity. This step ensures the image is an exact replica. Many imaging tools can generate hashes during acquisition; otherwise use commands like `md5sum` or `sha256sum` on the raw image file. The hash of the image **must** match the original drive’s hash for the image to be considered a true, unaltered copy. Log the hash values and store them with your case notes.

* **Post-Imaging Handling:** Secure the original drive (preserve it as evidence) and conduct all analysis on the image copy. Store the image file in a safe location, and document the acquisition process thoroughly (including time, tool, version, operator, and hashes). Maintain chain-of-custody paperwork for the original media and evidence images – this is crucial in professional settings to ensure admissibility of digital evidence.

**Tip – Whole Disk vs Partition Images:** If you image an entire disk (including all partitions), you will need to identify where each partition starts to analyze file systems individually. Use `fdisk -l` or the Sleuth Kit’s `mmls` to list partition table details. For instance, if the first partition starts at byte offset 32256, you can mount it directly from the image with: `sudo mount -o loop,ro,offset=32256 case1_disk.img /mnt/evidence`. Always use the `ro` (read-only) option when mounting forensic images to prevent any writes to the mounted file system.

### Objectives

- Create exact bit-by-bit images of target storage media.
- Verify image integrity using cryptographic hashes.
- Ensure no modification of original evidence.

### Checklist

1. **Preparation**
   - Use a write-blocker (hardware or software).
   - Disable automount features (e.g., `udisksctl`, `autofs`).
   - Identify device: `sudo fdisk -l`

2. **Imaging Tools**
   - **dd**:
     ```bash
     sudo dd if=/dev/sdX of=case1.img bs=4M conv=fsync,noerror status=progress
     ```
   - **dc3dd**:
     ```bash
     sudo dc3dd if=/dev/sdX of=case1.img hash=sha256 log=case1.log
     ```
   - **Guymager** (GUI-based): [https://guymager.sourceforge.io/](https://guymager.sourceforge.io/)

3. **Hashing**
   - After imaging:
     ```bash
     sha256sum case1.img > case1.img.sha256
     ```

4. **Preservation**
   - Store original media securely.
   - Use only copies for analysis.
   - Maintain logs: tool used, timestamps, device details, hashes.


## Cold System Forensics Methodology

Forensic analysis typically distinguishes between **live** and **cold** (or “dead”) analysis. **Cold System Forensics** refers to examining a system that is powered off (dormant) or removed from its normal operating environment. This is the classic approach of “pull the plug and analyze”:

* **Definition:** Cold forensics involves analysis of an entire system in its powered-off state, often in cases of incident response after a breach or malware infection. Unlike live forensics, which gathers volatile data (RAM, active processes) from a running system, cold forensics prioritizes data persistence and integrity by working on static copies of disks.

* **Advantages:** Since the system is off, data is not changing during acquisition – this avoids the risk of malware countermeasures or data being overwritten in memory. It also ensures a **forensically sterile** environment for analysis; investigators can apply tools without running processes that could modify timestamps or files. Cold forensics is ideal for legal evidence preservation and when *post-mortem* analysis is needed.

* **Challenges:** The trade-off is loss of volatile information (RAM content, network connections, etc.). Additionally, encrypted drives or certain anti-forensic measures might lock data once power is removed. Investigators must be prepared that only on-disk artifacts are available in cold analysis. In practice, many DFIR teams will perform a live memory capture *prior* to powering down, if possible, then proceed with cold imaging of disks.

* **Process Overview:**

  1. **Secure the System:** If a machine is found powered on but compromised, assess quickly if pulling power is safe (to avoid destructive malware triggers). Ideally, disconnect from network and then shut down to preserve disk state (or use a controlled shutdown if necessary to flush disk writes). In many cases, simply unplugging (for PCs) or removing the battery (for devices) is done to maintain the exact state on disk (albeit with some risk of minor file system journal replay when mounting, which can be mitigated by mounting read-only).
  2. **Acquire Disk Images:** Follow the imaging steps above for each storage device (internal drives, external USBs, etc.). Do not boot the system into its OS; instead, remove the drive or boot to a forensic boot CD/USB (like a Linux-based forensic distro) to image. This ensures no further data is written.
  3. **Verify & Preserve:** Hash and document each image. Store originals securely. All analysis moving forward is done on the image copies.
  4. **Analyze in Isolation:** Load the images in forensic software on a separate analysis machine. Because the subject system’s OS is not running, any analysis (file extraction, malware scanning, etc.) happens in a controlled environment, often using specialized forensic tools or sandboxes as needed.

Cold system forensics focuses on the *offline* examination of data. It covers tasks such as identifying differences from live analysis (e.g., no running processes, only disk artifacts), dealing with data *at rest*, and understanding data **volatility** (what data might have been lost by powering off, such as RAM-only encryption keys). Professionals also weigh legal considerations like ensuring a proper chain of custody and documenting each step exhaustively.

## Open-Source Forensic Tools (Linux Focus)

A variety of open-source tools are available to support each phase of the forensic process. Below is a categorized list of commonly used tools and their purposes:

* **Forensic Suites / Distributions:**

  * **SIFT Workstation:** A Linux (Ubuntu-based) distribution preloaded with forensic tools. SIFT supports raw (dd), E01 (Expert Witness), and AFF image formats and includes utilities for file carving, timeline analysis, registry examination, and more. It’s a ready-to-use toolkit maintained by SANS.
  * **Autopsy & Sleuth Kit:** Autopsy is a user-friendly GUI forensic platform backed by The Sleuth Kit (TSK) libraries. It allows analysis of disk images (Windows, Linux, or macOS file systems), recovery of deleted files, web artifacts, keyword search, and timeline visualization. Autopsy is often used by law enforcement and has modules for hash filtering (ignoring known good or flagging known bad files) and data carving (integrating tools like PhotoRec).

* **Disk Imaging Tools:**

  * **dd / dc3dd:** Versatile command-line tools to create raw disk images. `dc3dd` is an enhanced version of `dd` that can compute hashes during imaging and split images into segments.
  * **Guymager:** A fast, multi-threaded imaging GUI for Linux. It automates disk acquisition with options for format (raw, E01, AFF), calculates MD5/SHA hashes on the fly, and logs all metadata. Guymager’s interface is intuitive (great for those who prefer GUI) and it’s completely open-source.
  * **ewfacquire (Libewf):** A command-line tool to create EWF (E01) forensic images using the libewf library. It allows compression and notes metadata like case numbers, examiner name, etc. Useful if E01 is desired output in a pure Linux environment.
  * **ddrescue:** A data recovery imaging tool. If the drive has physical errors, `ddrescue` will attempt to repeatedly read bad sectors and create an image, logging unreadable sections. This is vital for forensic imaging of damaged media (to maximize data retrieval).

* **Filesystem Analysis Tools:**

  * **The Sleuth Kit (TSK):** A collection of command-line tools for file system analysis on disk images. Key tools include:
    **mmls** (partition table examiner), **fsstat** (file system superblock and layout info), **fls** (list files and directories, including deleted entries), **istat** (display details of a specific inode or FAT entry), **icat** (extract file content by inode), and **blkls** (extract raw unallocated space from an image). These allow in-depth analysis of FAT, NTFS, ext2/3/4, etc., in a forensically sound manner (read-only). For example, `fls -r -d image.dd` will recursively list deleted files in an image (if supported).
  * **Debugfs (for ext filesystems):** A low-level utility to interact with ext2/3/4 file systems. Investigators can use `debugfs` to list deleted inodes (`lsdel`), inspect inode structures (`istat` or `stat <inode>`), and even dump content of a given inode to a file. *Note:* On ext3/4, due to journaling, debugfs might not recover deleted content directly (as data blocks are often cleared on deletion), but it’s still useful for metadata analysis.
  * **ext4magic:** A specialized tool for recovering deleted files on ext3/ext4 by leveraging the filesystem’s journal. Since ext4 typically wipes or reinitializes inode data upon deletion, ext4magic reconstructs files using journal records of recent deletes. With a saved journal (`debugfs -R "dump <8> journal.bin" image.dd` since the journal is inode 8 on ext filesystems), `ext4magic` can restore files that were deleted, especially if journal entries haven’t been overwritten. For instance:
    `ext4magic /dev/sdX -j journal.bin -d ./recovered -f \"/path/to/deleted/file\"`
    would attempt to recover a specific file by name. Or use `-m` to recover *all* deleted files to an output folder.
  * **Mount (Read-Only Loopback):** Mounting the image (or a specific partition offset) as a read-only file system on Linux can be convenient for browsing intact files and using native OS tools. For example: `sudo mount -o loop,ro,offset=<bytes> image.dd /mnt/image`. Ensure the `ro` flag to prevent any write-backs. This method will not reveal deleted files (those not in the active file system tree), but it’s useful for quickly accessing visible data or using OS-specific analysis tools on the mounted content.

* **Search and Triage Tools:**

  * **bulk\_extractor:** Scans a disk image (particularly unallocated space) for patterns like email addresses, credit card numbers, URLs, and other artifacts. It’s useful for quickly identifying pockets of interesting data (e.g., finding if a credit card number is present in slack space, or parsing out all URLs from unallocated space).
  * **grep / strings:** Standard UNIX tools can be surprisingly powerful. Investigators often run `strings image.dd` piped to search for keywords or IOC patterns (e.g., email addresses, suspicious URLs, known malware signatures). This can help find evidence in free space or file slack that isn’t easily accessible via file system metadata. Combine with `grep -a -R` on a mounted image or using `fdisk`/`xxd` for targeted hex search. (The `-a` flag in grep treats binary data as text, allowing searching of raw bytes.)

* **Memory (if applicable):** (Although this guide focuses on disk forensics, note there are open-source memory forensics tools like **Volatility** or **Rekall**. In a “cold” forensic scenario, memory analysis is skipped, but if you perform live capture prior to shutdown, these tools would be used on the RAM image.)

Each tool above plays a role. Mastery of a toolkit like SIFT or TSK provides flexibility: for example, you might list files and identify a suspicious deleted executable with `fls`, then use `icat` to recover it for malware analysis, and finally run `photorec` or `foremost` to carve out any other remnants from unallocated space.

## File System Analysis – FAT32

**FAT32** is a legacy file system still common on USB drives and memory cards due to its simplicity and cross-platform support. Forensic analysis of FAT32 requires understanding its structure: boot sector, FAT tables, and directory entries. Key considerations include how files are allocated in clusters and how deletions are handled.

* **Structure Overview:** A FAT32 volume starts with a Volume Boot Record (VBR) at sector 0, containing vital parameters (bytes per sector, sectors per cluster, FAT count, etc.). Investigators often examine the VBR to determine cluster size, since knowing the cluster size is essential for interpreting addresses and slack space. For example, if Bytes/Sector = 512 and Sectors/Cluster = 2, then each cluster is 1024 bytes. Following the VBR, FAT32 has two copies of the File Allocation Table (for redundancy) and a **root directory** (which in FAT32 is a normal directory located in a data cluster, unlike FAT16 where root was a fixed area).

* **File Allocation Table (FAT):** The FAT is essentially a linked list structure for clusters. Each file or directory is allocated clusters, and the FAT entry for each cluster either points to the next cluster in the file or marks end-of-file (with a special `0xFFFFFFF` value). Investigators can inspect the FAT to trace a file’s cluster chain. For example, a FAT entry value of `0x00000006` at index 5 means cluster 5 is followed by cluster 6 in that file’s chain. An `EOF` marker (e.g., `0xFFFFFF0F` as shown for cluster 66) indicates the last cluster of a file. Modern forensic tools (TSK’s `fsstat` or `fatcat` utility) can display FAT allocation details, but manual hex inspection of the FAT region is also possible for validation.

* **Directory Entries and Metadata:** FAT32 directories are arrays of 32-byte entries for each file or subdirectory. Each entry (for normal 8.3 filenames) contains the file name, attributes, timestamps (created, modified, accessed dates), starting cluster number, and file size. Long File Names (LFN) are stored in special chained entries marked with attribute 0x0F; these precede the short 8.3 entry. A crucial aspect for forensics: when a file is **deleted** in FAT, the directory entry’s first byte of the filename is replaced with `0xE5` as a tombstone marker. The remaining directory entry still holds the file’s starting cluster and the length (in bytes) of the file. However, the FAT entries for that file’s clusters are *set to 0* (marked free) upon deletion. This means the chain of clusters is lost unless the file was contiguous and you can infer the chain from the file size.

* **Recovering Deleted Files (FAT32):** If a FAT32 file was deleted, forensic recovery involves the following steps:

  1. **Identify the Deleted Entry:** Scan the directory for entries starting with `0xE5`. Tools like TSK’s `fls -d` will list deleted filenames in a directory. For example, a file `Secret Deal.docx` might appear as `<Deleted> Secret Deal.docx` in `fls` output, confirming its presence.
  2. **Record Metadata:** Open the directory entry (e.g., with a hex editor or using `istat` on the file’s meta address in TSK) and note the starting cluster and file size. The short filename entry stores the “Low” 16 bits of the start cluster at offset 0x1A, and the “High” 16 bits at 0x14 (usually zero in FAT32 unless the file starts beyond cluster 0xFFFF). Also note the size at 0x1C (4 bytes). For instance, the entry might indicate Start Cluster = 67 and File Size = 90,692 bytes.
  3. **Compute Needed Clusters:** Determine how many clusters the file spanned. Clusters needed = ceil(FileSize / ClusterSize). In our example, 90,692 bytes / 1024 bytes per cluster ≈ 88.57, so 89 clusters.
  4. **Extract Clusters:** Because the FAT chain was cleared on deletion, you assume the file was contiguous (which is often the case for smaller files on FAT). Using a disk editor or tool, read cluster 67 through cluster 67+88 and save that data. In practice, you can do this with a hex editor (mark a block of the file size from the start cluster) or via `dd` by calculating sector offset. FTK Imager and some tools allow selecting a starting sector and length to export directly.
     *Example:* If cluster 67 corresponds to logical sector 0x10400 (as in the Sleuthifer example), you would extract 89 clusters (89 \* 2 sectors each, since 2 sectors per cluster) from that point. `dd if=image.dd of=recovered.docx skip=<start_sector> count=<sectors_to_read>` can be used, or TSK’s `icat` if you reconstruct a TSK-style sector range.
  5. **Verify File Integrity:** Open the carved file (if it’s a known format, see if it opens properly). Check the file header (magic number) to ensure it matches the expected type (e.g., DOCX files should start with PK\x03\x04 since they are ZIP archives). If the file was fragmented (non-contiguous), manual recovery is much harder – you would need to find subsequent fragments by content signature or analyze the FAT (if some parts of the chain remain) which is an expert-level task.

**Slack Space in FAT32:** Because FAT32 allocates in clusters, any unused bytes at the end of the last cluster of a file is **file slack**. This slack space may contain residual data from previous files that occupied that cluster or just remain as latent data. Forensically, slack can hold fragments of deleted files. Investigators should examine slack space by extracting entire clusters of files (many forensic tools can include slack in file exports) or by carving unallocated space including slack. As an example, if a 3 KB file resides in a 4 KB cluster, \~1 KB of slack could contain old data. Slack analysis can reveal hidden or remnants of data that are not in any allocated file.

### Procedure

1. **Determine Offset and Cluster Size**
   ```bash
   mmls case1.img
   fsstat -f fat case1.img
   ```

2. **List Files (Including Deleted)**
   ```bash
   fls -r -p -f fat -o <offset> case1.img
   fls -r -d -f fat -o <offset> case1.img
   ```

3. **Recover Files**
   ```bash
   icat -f fat -o <offset> case1.img <inode>
   ```

4. **Slack Space Analysis**
   - Use carving tools or manual inspection with `xxd`, `strings`.

## File System Analysis – Ext3/Ext4

Modern Linux systems typically use ext3 or ext4 file systems. These are more complex than FAT and include features like journaling, inode-based allocation, and advanced timestamps. From a forensic perspective, ext3/4 present both opportunities (rich metadata) and challenges (aggressive deletion behavior due to journaling).

* **Structure Overview:** ext3/ext4 divides the disk into block groups. Important components include the **Superblock** (containing overall FS info), **Group Descriptors**, **Inode Table**, **Data Blocks**, and the **Journal** (for ext3/4 with journaling enabled). Each file or directory is described by an **inode** (which contains metadata and pointers to data blocks). Filenames are stored in directory files that map names to inode numbers.

* **Inodes and Metadata:** Each inode holds metadata like user/group ownership, permissions, and timestamps (ext4 inodes have creation time, modification (mtime), access (atime), change (ctime), and deletion time if the file is deleted, plus extended attributes). It also contains pointers to data blocks: ext uses direct, single-indirect, double-indirect, etc., or ext4 may use extents (a more compact way to represent contiguous runs of blocks). Key forensic note: *ext file systems do not use a central index of all files (like NTFS’s MFT or FAT’s table); instead, you discover files by traversing directory inodes.* Deleted files might still have their inode and data blocks on disk until reallocated.

* **Deletion in ext3/ext4:** When a file is deleted on ext3 or ext4, the OS will remove the directory entry and mark the inode as free. For ext3 and ext4 (with default settings), the inode’s data blocks pointers are also wiped (cleared), effectively unlinking the data. This is in contrast to ext2 (non-journaling) where the inode pointers *might remain intact* until reused, allowing easier recovery with tools like debugfs. Therefore, basic undelete is difficult on ext3/4 because the mapping of file to blocks is lost immediately on deletion. However, the actual data blocks are not zeroed – they are just unreferenced. Recovery then relies on indirect methods like scanning the journal or carving.

* **Journal Analysis:** ext3/4 journal (inode 8) logs metadata updates (and optionally data, if journal=data or ordered mode for ext3). Deleted file metadata might be found in the journal before it was committed. Advanced forensics can involve parsing the journal to find recently deleted filenames, their inode numbers, and even data block references that were overwritten. TSK provides `jls` to list journal transactions and `jcat` to view journal contents. This is quite advanced, but in cases where a crucial file was deleted, examining the journal can recover information such as the file’s inode number or even content (if journaled). The **ext4magic** tool essentially automates this: it extracts the journal and hunts for deleted entries to restore files.

* **Recovery Techniques:** There are a few approaches to recover deleted data on ext:

  * **extundelete / ext4magic:** These tools use the journal and any remaining inode data to reconstruct deleted files. If the deletion was recent and the filesystem hasn’t reused the blocks, these can be very effective. For ext4, having a journal is actually required for `extundelete` to work properly. Usage is straightforward, e.g., `extundelete --restore-all image.img --output-dir recovered/`. Always work on a copy of the image when using such tools.
  * **Sleuth Kit (ISTAT/ICAT):** If you know a file’s inode (from directory evidence or logs), you can use `istat` to inspect the inode. In ext, if the file is deleted, `istat` might show it as unallocated but still list the block addresses (this works on ext2, but on ext4 usually the block list is zeroed out on deletion). If block addresses are present, you can manually extract them with `icat` using the inode number. If they are not present, you fall back to carving.
  * **File Carving:** As discussed in the next section, carving will search the entire ext volume for file signatures. This does not rely on metadata at all, which is beneficial if both directory entry and inode references are gone.

* **Timestamps and Forensic Timelines:** Ext file systems offer detailed timestamps. Notably, ext4 records creation time (`crtime`) in the inode (something ext3 and many other Linux FS didn’t). Investigators can leverage these for timeline analysis. For example, using `fls` with `-p` (to get full paths) and then feeding into `mactime` (a Sleuth Kit tool) can generate a timeline of file activity. Understanding ext timestamps:

  * **atime** (access time) – often disabled or not up-to-date on modern systems for performance, but if enabled, shows last read access.
  * **mtime** (modification time) – when file content last changed.
  * **ctime** (inode change time) – when metadata last changed (permissions, or also updated on file content change as well).
  * **crtime** (creation, ext4 only) – when file was created.
  * **dtime** (deletion time in ext4 inode) – when file was deleted (if recorded before inode reuse).
    These can help investigators correlate events. For example, a suspicious file might have an mtime far in the past but a crtime more recent, indicating it might have been planted and backdated – or simply recovered from backup.

In summary, ext3/4 analysis often requires specialized tools (ext4magic, debugfs) and an understanding that *quick undelete is not trivial*. The presence of journaling means ext filesystems are less forgiving for simple undelete, but the journal itself becomes an artifact to analyze. Always safely work on a copy of the image, and consider mounting the image read-only on a Linux system to browse non-deleted files (ext4 is fully supported by Linux kernel, so a loopback mount can be useful for intact files).

### Steps

1. **List File System Layout**
   ```bash
   fsstat -f linux-ext3 case1.img
   ```

2. **List and Recover Files**
   ```bash
   fls -r -p -f ext4 -o <offset> case1.img
   icat -f ext4 -o <offset> case1.img <inode>
   istat -f ext4 -o <offset> case1.img <inode>
   ```

3. **Deleted File Recovery**
   - Attempt with `ext4magic`:
     ```bash
     ext4magic /dev/loopX -r -d ./recovered/
     ```

4. **Journal Review**
   ```bash
   jls -f ext4 -o <offset> case1.img
   jcat -f ext4 -o <offset> case1.img <block>
   ```


## File Carving and Data Recovery Techniques

Even after analyzing file system metadata, a large portion of the evidence might reside in **unallocated space** or **slack space** – places where deleted or hidden data lurk without file system pointers. **File carving** is the technique of scanning raw bytes of an image to find files or fragments based on content signatures, rather than metadata. It is indispensable for recovering deleted files when metadata is lost or to retrieve fragments from corrupted file systems.

**Key Concepts of File Carving:**

* **File Signatures (Headers/Footers):** Many file types have identifiable byte patterns at their start (and sometimes end). For example, JPEG images start with `FF D8 FF E0` (or `FF D8 FF E1` for some JPEG subtypes), PDFs start with `%PDF`, PNGs with the ASCII for `.PNG` after an 8-byte header, etc. These signatures (also called “magic numbers”) allow carving tools to locate potential file beginnings. Some file types also have an end marker (e.g., PDFs end with `%%EOF`). Carving tools use these to determine where a file ends, or they rely on file size fields and subsequent headers to infer boundaries. A good reference for file signatures is Gary Kessler’s File Signatures table.

* **No Metadata Dependency:** Carving operates without file system metadata. This means it can recover files even if the file table or inode references are gone (for instance, after a format or when working with a raw unpartitioned space). However, because it ignores metadata, carving has *no inherent understanding of fragmentation* – if a file’s data is not contiguous, basic carving may recover only the first fragment. Advanced carving algorithms attempt to deal with fragmentation by recognizing patterns or using knowledge of file format structures (e.g., carving a large video split into pieces).

* **When to Use Carving:** Typical use cases include:

  * Recovering files from **unallocated space** (after deletion or formatting).
  * Extracting files from a **corrupted file system** that cannot be mounted.
  * **Incident response triage:** carving memory dumps or pagefiles for useful artifacts.
  * Searching for **embedded files** in compound formats or slack space (e.g., a hidden JPEG embedded in the slack of a larger file).

**Open-Source Carving Tools:**

* **PhotoRec:** Part of the TestDisk suite (by CGSecurity), PhotoRec is a powerful carving tool that supports hundreds of file formats. It’s menu-driven in a terminal interface. PhotoRec ignores the file system and sequentially scans the media for known headers. It will retrieve files to an output folder, sorting them by type. PhotoRec is known for being very effective and simple: one can point it at an image and let it carve everything, or select specific file families to recover. *Usage:* `photorec image.dd` (then follow prompts to choose output, file types, etc.). Keep in mind PhotoRec will carve *everything* it recognizes, which can result in thousands of files – so use its file-type selection to narrow scope if needed.

* **Foremost:** A classic file carving tool (developed by the Air Force OSI). Foremost uses a configuration file (default `/etc/foremost.conf`) that defines signatures for various file types. It reads the image and looks for those signatures, then writes out recovered files to an organized directory structure. Foremost can be directed to carve only certain types via command-line options (e.g., `-t jpg,pdf`). It is straightforward and good for targeted carving. *Usage example:*
  `foremost -i case1_disk.img -o output_folder -t jpg,png,pdf`
  This would scan for JPEG, PNG, and PDF files only, saving them in `output_folder`. Foremost processes the entire image from start to finish looking for headers; when a header is found, it continues until a footer is found or a max file size is reached. Results are stored in subfolders by file type.

* **Scalpel:** A successor to Foremost, Scalpel is another open-source carver that focuses on performance improvements and more flexible configuration. It also uses a config file (often `/etc/scalpel/scalpel.conf`) where you uncomment or add signatures to enable carving of those types. Scalpel performs two-pass carving: first building a list of fragment offsets, second pass writing out files (which makes it faster on large images by minimizing disk I/O). It’s particularly useful for large datasets or when Foremost/PhotoRec are too slow. *Usage:* similar to Foremost:
  `scalpel case1_disk.img -o output_dir` (assuming config is edited for desired file types).
  Scalpel’s default config includes many common file types; you enable the ones you want by removing comments. It can carve very quickly and is a good alternative if Foremost is slow.

* **Bulk Extractor:** While not a traditional file carver that reassembles files, Bulk Extractor deserves mention. It scans images and outputs *features*—like email addresses, phone numbers, credit card numbers, URLs, and other structured data. It’s invaluable for quickly identifying interesting data in unallocated space or within carved files. For example, Bulk Extractor can find a snippet of text from an email in slack space even if the actual email file was deleted. It won’t give you the whole email in order, but it will pull out pieces of evidence (which can lead you to carve the right files or search deeper).

I used BE extensively in my masters thesis project because it is a scalable solution for high volumes of unknown data before engaging with any further advanced (multi-modal) methods like employing machine learning and neural network computer vision models. In my experience, it performs surprisingly well at identifying "low-hanging fruit" in terms of PII (personal identifiable information) like telephone numbers, credit card numbers, urls, domains etc. It is also therefore quite useful for any phishing data dumps which tend to contain high amounts of PII compared to other cybersecurity artifacts (say, Malware codefiles etc.). With some additional tweaks it is possible to render visualizations of the histograms that BE supports based on found data artifacts and even use some of the outputs as inputs for a further pipeline stage (for example, a machine learning model.)


* **Manual Carving (Hex Editors / Binwalk):** In advanced scenarios, an expert might manually carve data using a hex editor or a tool like `binwalk`. For instance, if you suspect a particular file type not handled by automated tools, you can search for its header in a hex editor and attempt to extract it by eyeballing where it ends. **Binwalk** is useful for carving files from firmware images or unorthodox disk images; it can scan for embedded file signatures (like finding an image inside a binary blob). **Hex editors** (like `hexedit` or GUI ones like Bless or wxHexEditor) let you inspect raw data and select segments to export. This requires deep knowledge of file formats and is time-consuming, but it can recover data that automated tools might miss or misidentify. (For example, carving a partially overwritten file where you manually piece together fragments.)

### Commands

- **PhotoRec (Interactive):**
   ```bash
   photorec case1.img
   ```

- **Foremost:**
   ```bash
   foremost -i case1.img -o output/ -t jpg,pdf,docx
   ```

- **Scalpel:**
   - Edit `/etc/scalpel/scalpel.conf`
   ```bash
   scalpel case1.img -o output/
   ```

- **bulk_extractor:**
   ```bash
   bulk_extractor -o bulk_output/ -x all case1.img
   ```

**Carving Best Practices:** Always carve *copies* of data or at least ensure you are not altering the original image. Carving can be I/O intensive; using a fast SSD for your workspace can speed up the process. After carving, verify the carved files if possible – check file hashes against known values (for known malware or known documents), or at least open files to confirm they are valid (especially for files of interest). Note that carved files from unallocated space might be incomplete or corrupted (if parts were overwritten). Treat carved results as potential evidence that may need further validation.

Also, be aware of **false positives**: carving tools might extract random byte sequences that match a header by chance but are not real files. This is why verifying headers and content is important. Many tools include some checks (for instance, photorec might validate that a carved JPEG can be opened), but the examiner should still validate critical evidence.

**Slack & Unallocated Space Recovery:** As a final note, to comprehensively analyze a disk, you should not only rely on file system metadata. Professional investigators inspect both **allocated space** (the live file system) and **unallocated** (free space). Slack space and unallocated clusters often contain remnants of prior files. Techniques include carving (as above), keyword searching, and hashing known fragments. For example, if you have a hash of a contraband image and the file was deleted, you could hash all fixed-size blocks of the disk to see if any block matches a portion of that hash (a technique sometimes called piecewise hashing or similarity hashing with tools like ssdeep). In general, *recovering data from slack and unallocated space involves using forensic tools, file carving, hash analysis, and manual reconstruction of file fragments*. Always work on the forensic image (never on the original disk), and ensure any recovered data is handled with the same evidence integrity as the original (document where it was recovered from, how, and maintain hashes for the recovered files as well).

## Conclusion and Further Resources

In this advanced guide, we walked through the entire process of cold forensic analysis on a storage device – from creating a verifiable image, through examining file system artifacts in FAT32 and ext4, to carving out data from the “empty” spaces of the disk. The emphasis was on open-source tools and Linux methodologies, which offer powerful capabilities at little to no cost, backed by a strong community of practitioners and developers.

For deeper dives into specific topics, consider the following resources:

* **DFIR Community Resources:** The *SANS Investigative Forensic Toolkit (SIFT) Workstation* comes with extensive documentation and cheat-sheets for using its tools. Likewise, the Sleuth Kit/Autopsy website provides training materials on using the suite’s features (timeline analysis, web artifacts, etc.).
* **File System Internals:** For FAT and NTFS, *Brian Carrier’s book* "File System Forensic Analysis" is an excellent reference. It covers FAT, NTFS, ext2, UFS, etc., in extreme detail. Knowing the internals will make you much more effective when dealing with corrupted or partial data. Also, consider reading academic papers or forensic blog posts on ext4 (e.g., research by Fairbanks on ext4 forensics) for the latest findings.
* **Practice Scenarios:** TryHackMe and similar platforms have rooms like “Intro to Cold System Forensics” and “File Carving” which provide practical exercises. These can be useful to apply the knowledge hands-on in a guided environment.
* **Communities and Updates:** Digital forensics is an evolving field. Stay updated through forums like **Forensic Focus**, the **DFIR community on Twitter**, and attend workshops or training (DFIR Summit, etc.). Open-source tools get updated, and new ones emerge – for instance, projects like **KAPE** (for triage) or **Magnet’s community editions** might interest you for specific tasks, although they may not be fully open-source.

By following a structured approach and leveraging the powerful open-source tools mentioned, an investigator can preserve and analyze data from a suspect drive thoroughly and professionally. Always remember to document each step, preserve original evidence, and when in doubt, practice on sample data to refine your techniques. Digital forensics requires patience and rigor – but with these advanced techniques, you’ll be well-equipped to tackle complex investigations.

**Sources:** The information and techniques above were compiled from industry best practices and authoritative sources, including forensic tool documentation and expert write-ups. These references (and those linked throughout this guide) offer further reading and validation for the methods described. Always cross-reference procedures with official documentation and ensure your tools are updated to the latest versions for best results.

## References and Tools

- Sleuth Kit / Autopsy: [https://www.sleuthkit.org/autopsy/](https://www.sleuthkit.org/autopsy/)
- Guymager: [https://guymager.sourceforge.io/](https://guymager.sourceforge.io/)
- EXT4Magic: [https://github.com/bitcurator/ext4magic](https://github.com/bitcurator/ext4magic)
- SIFT Workstation: [https://digital-forensics.sans.org/community/downloads](https://digital-forensics.sans.org/community/downloads)
- Forensics Wiki: [https://forensics.wiki](https://forensics.wiki)
- File Signatures DB: [https://www.garykessler.net/library/file_sigs.html](https://www.garykessler.net/library/file_sigs.html)

---