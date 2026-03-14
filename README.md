# MetaRecoverX

**Recovery of Deleted Data and Associated Metadata from XFS and Btrfs Filesystems**

MetaRecoverX is a Linux digital forensics and data recovery tool designed to recover deleted files and reconstruct their associated metadata from XFS and Btrfs filesystems. It scans low-level filesystem structures — including inodes, allocation groups, metadata logs, and filesystem trees — to accurately restore lost data along with critical metadata such as filenames, full directory paths, timestamps (atime/mtime/ctime), UID/GID, permissions, and file extents.

The tool runs **fully offline** in Ubuntu / Linux TTY environments, making it suitable for digital forensics, incident response, and system administration workflows.

---

## Project Information

- **Institute:** Swami Keshvanand Institute of Technology, Management & Gramothan (SKIT), Jaipur
- **Department:** Information Technology
- **Academic Session:** 2025–2026
- **Team Name:** Tech-Aizen

### Team Members

| Member | Roll No. | Role |
|---|---|---|
| Mayank Tak | 22ESKIT093 | Lead Developer — XFS module, recovery engine, reporting |
| Lucky Panchal | 22ESKIT088 | Btrfs module, testing, and integration |
| Milan Kumar | 22ESKIT094 | Metadata reconstruction pipeline, QA, and tooling |

### Faculty Mentors

- Mr. Jagendra Singh Chaudhary — Assistant Professor (II)
- Ms. Richa Rawal — Assistant Professor (I), Lab Coordinator

---

## Key Features

- Recovery of deleted files from **XFS** filesystems
- Recovery of deleted files from **Btrfs** filesystems
- **Full directory path reconstruction** — rebuilds multi-level paths (e.g. `/home/user/docs/file.txt`)
- **Timestamp extraction** — atime, mtime, ctime from on-disk inode structures
- **Metadata reconstruction** — filenames, UID/GID, permissions, file extents
- **Extent-based file recovery** — reassemble files from disk extents and verify integrity
- **Disk image analysis** — full pipeline on raw disk images (`.img`)
- **USB / pendrive forensic acquisition** — image USB drives with `dd`, hash with SHA-256
- **Filesystem auto-detection** — uses `blkid` to determine XFS vs Btrfs
- **SHA-256 hashing** for evidence integrity verification
- **Progress indicator** — real-time percentage display for large image scanning
- **Human-readable reports** — formatted forensic recovery reports (`report.txt`)
- **Automated test suite** — 29-test validation suite
- **Interactive TTY menu** — key-driven interface for all operations
- Support for **large disk images** (>100 GB) using `uint64_t` offsets
- **Fully offline execution** — no internet, no external libraries
- **CSV comma escaping** — proper RFC 4180 handling for filenames with special characters

---

## Supported Filesystems and Structures

### XFS

The XFS scanner detects and parses:

- XFS superblock (magic `XFSB`)
- Allocation groups (AGs)
- Inode structures (big-endian, including deleted inodes)
- Directory entries (`xfs_dir2_data_entry` — v4 and v5)
- B+Trees and metadata journals/logs
- Extent records (128-bit packed format for freed extents)

Recovered metadata: inode, filename, **full directory path**, size, uid, gid, mode, **atime, mtime, ctime**, extents.

### Btrfs

The Btrfs scanner parses:

- Superblock (at offset 0x10000, magic `_BHRfS_M`)
- Leaf node traversal with fsid validation
- INODE_ITEM (type 1) — size, uid, gid, mode, **timestamps**
- INODE_REF (type 12) — filename + **parent directory inode**
- DIR_ITEM (type 84) / DIR_INDEX (type 96) — child inode + **parent tracking**
- FILE_EXTENT_ITEM (type 108) — physical extent locations

Recovered metadata: inode, filename, **full directory path**, size, uid, gid, mode, **atime, mtime, ctime**, extents.

---

## Technology Stack

| Technology | Version | Purpose |
|---|---|---|
| C++17 | GCC 9+ | Core binaries — filesystem parsing, low-level recovery |
| Bash | System | Orchestration scripts (build, pipeline, demo, USB, reports) |
| POSIX APIs | — | Low-level I/O and system calls |
| Linux coreutils | — | `dd`, `stat`, `sha256sum`, `readlink`/`realpath`, `blkid`, `lsblk` |
| CMake (optional) | 3.10+ | Build system (falls back to direct `g++` if missing) |

**No external libraries, Python, Rust, or internet downloads are required.**

---

## Architecture and Modules

```
┌──────────────────────────────────────────────────────────────────┐
│                      MetaRecoverX Pipeline                      │
├────────────┬───────────┬──────────────┬───────────┬─────────────┤
│ 1. Acquire │ 2. Scan   │ 3. Recon-    │ 4. Recover│ 5. Report   │
│ Disk Image │ Filesystem│    struct    │   Files   │ Generation  │
│ (dd+hash)  │ Structures│  Metadata   │ (extents) │ (txt+csv)   │
└────────────┴───────────┴──────────────┴───────────┴─────────────┘
```

| Module | Binary / Script | Description |
|---|---|---|
| Disk Image Acquisition | `dd` / `sha256sum` | Acquire disk images from devices, verify integrity with hashing |
| XFS Scanner | `build/xfs_scan` | Parse XFS superblock, AGs, enumerate inodes, directory entries & freed extents |
| Btrfs Scanner | `build/btrfs_scan` | Traverse leaf nodes, parse INODE_ITEM/REF, DIR_ITEM, FILE_EXTENT_ITEM |
| Metadata Reconstruction | `build/reconstruct_csv` | Combine scan CSVs, merge timestamps, build unified metadata |
| Recovery Engine | `build/recover_csv` | Reassemble files from extents, preserve filenames, write to `recovered/` |
| Report Generator | `scripts/generate_report.sh` | Produce human-readable forensic recovery report |
| Test Suite | `tests/run_tests.sh` | 29-test automated validation |

---

## Repository Structure

```text
MetaRecoverX/
├── core/                           # C++ source code
│   ├── CMakeLists.txt              # CMake build configuration
│   └── src/
│       ├── xfs_scan.cpp            # XFS filesystem scanner
│       ├── btrfs_scan.cpp          # Btrfs filesystem scanner
│       ├── reconstruct_csv.cpp     # Metadata reconstruction engine
│       └── recover_csv.cpp         # File recovery engine
│
├── scripts/                        # Bash orchestration scripts
│   ├── setup.sh                    # Build script (CMake → g++ fallback)
│   ├── build.sh                    # Direct build wrapper
│   ├── metarecoverx.sh            # Main pipeline script (all commands)
│   ├── detect_usb.sh              # USB device detection
│   ├── run_demo.sh                # Quick demo runner
│   ├── menu.sh                    # Interactive TTY menu (6 options)
│   └── generate_report.sh         # Human-readable report generator
│
├── tests/                          # Test infrastructure
│   └── run_tests.sh               # Automated test suite (29 tests)
│
├── build/                          # Compiled binaries (generated)
│   ├── xfs_scan
│   ├── btrfs_scan
│   ├── reconstruct_csv
│   └── recover_csv
│
├── artifacts/                      # Pipeline outputs (generated)
│   ├── acquire.json               # Acquisition metadata (path, size, hash)
│   ├── hash.txt                   # SHA-256 hash of disk image
│   ├── xfs_scan.csv              # XFS scan results
│   ├── btrfs_scan.csv            # Btrfs scan results
│   ├── metadata.csv              # Unified metadata (with timestamps + recovery_plan)
│   └── metadata.json             # Unified metadata (JSON)
│
├── recovered/                      # Recovered files (generated)
│   ├── file.txt                   # (filename preserved when possible)
│   ├── photo.jpg
│   └── file_0001.bin              # (fallback name if unknown)
│
├── dummy.img                       # 1 MiB test image for demo
├── report.csv                      # Copy of metadata.csv for quick viewing
├── report.txt                      # Human-readable recovery report
└── Makefile                        # Top-level Makefile
```

---

## Getting Started

### Prerequisites (Ubuntu, offline)

- Ubuntu (native or WSL) with a C++17 compiler (`g++`)
- Optional: CMake 3.10+ (for nicer builds; script falls back to `g++` if missing)

### Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/MetaRecoverX.git
cd MetaRecoverX
```

### Build the Project

```bash
chmod +x scripts/*.sh tests/*.sh
./scripts/setup.sh
```

This compiles C++17 binaries into `build/`:

- `build/xfs_scan`
- `build/btrfs_scan`
- `build/reconstruct_csv`
- `build/recover_csv`

You can also build explicitly via:

```bash
./scripts/metarecoverx.sh build
```

---

## How to Run

MetaRecoverX supports multiple modes of operation. All commands are run from the project root.

### 1. Quick Demo (Sanity Check)

Verify that the toolchain and pipeline work end-to-end:

```bash
./scripts/metarecoverx.sh demo
```

What the demo does:

1. Creates a small zero-filled `dummy.img` (1 MiB)
2. Runs acquisition (SHA-256 hash + metadata)
3. Runs XFS and Btrfs scanners
4. Reconstructs unified metadata
5. Recovers file data
6. Generates `report.csv` and `report.txt`

Inspect the result:

```bash
column -t -s, report.csv | head
cat report.txt
```

### 2. Full Pipeline on a Disk Image

If you already have a raw disk image:

```bash
./scripts/metarecoverx.sh pipeline --image /path/to/disk.img
```

Pipeline steps:

```
[1/6] Building binaries...
[2/6] Acquiring image info (SHA-256 hash)...
[3/6] Scanning filesystems (XFS + Btrfs) to CSV...
[4/6] Reconstructing metadata...
[5/6] Recovering file data...
[6/6] Generating report...
```

### 3. USB Forensic Recovery

#### Step 1: Detect USB Devices

```bash
./scripts/detect_usb.sh
# or
./scripts/metarecoverx.sh detect-usb
```

Example output:

```text
NAME  TRAN  SIZE  MOUNTPOINT
----  ----  ----  ----------
sdb   usb   32G
sdc   usb   64G
```

#### Step 2: Run USB Forensic Pipeline

```bash
sudo ./scripts/metarecoverx.sh pipeline-usb --device /dev/sdb
```

⚠️ **WARNING:** Double-check that you have the correct device. Choosing the wrong one may destroy data.

USB recovery steps:

1. Build C++ tools
2. Acquire forensic image via `dd` → `artifacts/usb_sdb_YYYYMMDD_HHMMSS.img`
3. Compute SHA-256 hash → `artifacts/hash.txt`
4. Detect filesystem type via `blkid` (xfs, btrfs, or both)
5. Scan filesystem metadata → CSV files
6. Reconstruct combined metadata → `metadata.csv` + `metadata.json`
7. Recover files → `recovered/`
8. Generate report → `report.txt`

### 4. Interactive TTY Menu

For bare-metal Ubuntu TTYs, use the key-driven menu:

```bash
./scripts/menu.sh
```

Menu options:

| Key | Action |
|---|---|
| `1` | Detect USB storage devices |
| `2` | Run demo pipeline (dummy.img) |
| `3` | Full pipeline on disk image (prompts for path) |
| `4` | USB forensic recovery (prompts for device, confirms before imaging) |
| `5` | View last recovery report |
| `6` | Clean all outputs |
| `q` | Quit |

### 5. Generate a Report

After running any pipeline:

```bash
./scripts/metarecoverx.sh report
```

Produces a human-readable `report.txt` with acquisition info, recovery summary, file listing, and metadata table.

### 6. Run the Test Suite

```bash
./scripts/metarecoverx.sh test
```

Runs 29 automated tests covering build, binaries, pipeline execution, output format validation, bug regression checks, and script syntax.

---

## All Available Commands

```bash
# Build
./scripts/metarecoverx.sh build

# Acquire image metadata
./scripts/metarecoverx.sh acquire --image /path/to/disk.img

# Scan filesystem
./scripts/metarecoverx.sh scan xfs   --image /path/to/disk.img --format csv
./scripts/metarecoverx.sh scan btrfs --image /path/to/disk.img --format csv

# Reconstruct metadata from CSVs
./scripts/metarecoverx.sh reconstruct

# Recover files
./scripts/metarecoverx.sh recover --image /path/to/disk.img

# Full pipeline (build + acquire + scan + reconstruct + recover + report)
./scripts/metarecoverx.sh pipeline --image /path/to/disk.img

# Detect USB devices
./scripts/metarecoverx.sh detect-usb

# USB forensic pipeline
sudo ./scripts/metarecoverx.sh pipeline-usb --device /dev/sdb

# Generate report
./scripts/metarecoverx.sh report

# Run demo
./scripts/metarecoverx.sh demo

# Run tests
./scripts/metarecoverx.sh test

# Interactive menu
./scripts/menu.sh
```

---

## Direct Binary Usage (Advanced)

Advanced users can call the binaries directly:

### XFS Scanner

```bash
build/xfs_scan --image /path/to/disk.img --format csv > artifacts/xfs_scan.csv
```

### Btrfs Scanner

```bash
build/btrfs_scan --image /path/to/disk.img --format csv > artifacts/btrfs_scan.csv
```

### Reconstruct Metadata

```bash
build/reconstruct_csv \
  --out artifacts/metadata.json \
  --csv-out artifacts/metadata.csv \
  artifacts/xfs_scan.csv artifacts/btrfs_scan.csv
```

### Recover Files

```bash
build/recover_csv \
  --image /path/to/disk.img \
  --plan artifacts/metadata.csv \
  --out recovered
```

---

## CSV Metadata Format

### Scanner Output (xfs_scan.csv / btrfs_scan.csv)

```text
fs,inode,path,size,uid,gid,mode,atime,mtime,ctime,extents
```

Example:

```text
xfs,10231,/home/user/docs/file.txt,20480,1000,1000,100644,1698765432,1698765400,1698765432,8192:20480
btrfs,256,/documents/report.pdf,51200,1000,1000,100644,1698700000,1698700000,1698700000,16384:51200
```

### Unified Metadata (metadata.csv — after reconstruct_csv)

```text
global_id,fs,path,size,uid,gid,mode,atime,mtime,ctime,extents,recovery_plan
```

Example:

```text
xfs-10231,xfs,/home/user/docs/file.txt,20480,1000,1000,100644,1698765432,1698765400,1698765432,8192:20480,carve
btrfs-256,btrfs,/documents/report.pdf,51200,1000,1000,100644,1698700000,1698700000,1698700000,16384:51200,carve
```

Fields:

| Column | Description |
|---|---|
| `global_id` | Unique ID: `{fs}-{inode}` (e.g. `xfs-10231`) |
| `fs` | Filesystem type: `xfs` or `btrfs` |
| `path` | Full reconstructed directory path |
| `size` | File size in bytes |
| `uid` | User ID |
| `gid` | Group ID |
| `mode` | POSIX permissions (octal) |
| `atime` | Last access time (Unix epoch seconds) |
| `mtime` | Last modification time (Unix epoch seconds) |
| `ctime` | Last status change time (Unix epoch seconds) |
| `extents` | Physical disk extents: `start:length;start:length` |
| `recovery_plan` | `carve` (recoverable) or `skip` (no extents) |

---

## Output Structure

After a successful pipeline run:

```text
artifacts/
  acquire.json        — acquisition metadata (image path, size, SHA-256 hash)
  hash.txt            — SHA-256 hash file (USB pipeline)
  xfs_scan.csv        — XFS filesystem metadata with timestamps
  btrfs_scan.csv      — Btrfs filesystem metadata with timestamps
  metadata.csv        — unified metadata (all filesystems, with recovery_plan)
  metadata.json       — same metadata in JSON format (with real ISO 8601 timestamps)

recovered/
  file.txt            — recovered file (filename preserved)
  photo.jpg           — recovered file (filename preserved)
  file_0001.bin       — recovered file (filename unknown)

report.csv            — copy of metadata.csv for quick inspection
report.txt            — human-readable forensic recovery report
```

---

## Progress Indicator

For large disk images (>1 GB), both scanners display real-time progress:

```
[xfs_scan] scanning 4294967296 bytes (4096 MiB)...
[xfs_scan] 5% (204 / 4096 MiB)
[xfs_scan] 10% (409 / 4096 MiB)
[xfs_scan] 15% (614 / 4096 MiB)
...
[xfs_scan] 100% scan complete.
[xfs_scan] found 42 inode(s), 35 name(s)
```

Progress updates every 5% on stderr, using carriage return for in-place TTY updates.

---

## Test Suite

Run all 29 automated tests:

```bash
./scripts/metarecoverx.sh test
```

Test categories:

| Category | Tests |
|---|---|
| Build | Compilation succeeds |
| Binaries | All 4 binaries exist and are executable |
| --help flags | Scanners respond to --help |
| Demo pipeline | Full end-to-end pipeline completes |
| Output files | All 6 expected output files created |
| CSV headers | Scanner and metadata CSV headers match specification |
| JSON validity | metadata.json and acquire.json are valid JSON |
| Bug regression | No double-prefix in global_id, real timestamps |
| Feature checks | recovery_plan column present |
| Data content | CSVs have data rows, files recovered |
| Script syntax | All scripts pass `bash -n` validation |

Expected result:

```
══════════════════════════════════════════════
  Results: 29 passed, 0 failed
══════════════════════════════════════════════
  ✓ All tests passed!
```

---

## Windows Users (WSL Recommended)

The scripts are POSIX shell-based. The simplest way to run on Windows is via WSL (Ubuntu):

1. Install WSL with Ubuntu from the Microsoft Store.
2. Open an Ubuntu terminal and navigate to the project:

```bash
cd /mnt/c/Users/YourName/Desktop/MetaRecoverX
```

3. Run the same offline steps:

```bash
chmod +x scripts/*.sh tests/*.sh
./scripts/setup.sh
./scripts/metarecoverx.sh pipeline --image /mnt/c/Path/To/disk.img
```

---

## Safety and Forensic Best Practices

- **Never** run `pipeline-usb` or `dd` against a device unless you are absolutely certain it is the correct source.
- Prefer working on **read-only clones or images** rather than original evidence media.
- Keep MetaRecoverX machines **offline** for forensic workflows.
- **Verify hashes** (`artifacts/hash.txt`) before and after copying images between systems.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `realpath`/`readlink` not found | Script tries both and falls back gracefully |
| CMake not found | Build falls back to direct `g++` compilation automatically |
| Permission denied | Run `chmod +x scripts/*.sh tests/*.sh` |
| Binary not in `build/` | Run `./scripts/build.sh` to recompile |
| USB device not listed | Ensure device is connected; run `lsblk` manually |
| Stale CMake cache | Run `rm -rf build/` then rebuild |
| Tests failing | Run `./scripts/metarecoverx.sh test` to see which test failed |

---

## Project Milestones (2025–2026)

| Date Range | Milestone |
|---|---|
| Sep 10–20, 2025 | Btrfs tree traversal prototype; metadata pipeline outline |
| Sep 20–25, 2025 | Requirements finalized; select sample datasets |
| Sep 25–30, 2025 | Disk image acquisition process finalized |
| Oct 1–2, 2025 | XFS superblock and AG parser drafted and validated |
| Oct 5–10, 2025 | Deleted inode detection and carving (XFS) |
| Oct 10–18, 2025 | Integrate metadata pipeline; timestamps/UID/GID recovery |
| Oct 15–20, 2025 | Btrfs deleted file and snapshot detection |
| Oct 20–25, 2025 | Integrate Btrfs with recovery engine; QA on multiple images |
| Oct 31, 2025 | Integrate XFS modules; optimize carving and scanning performance |
| Nov 7–20, 2025 | CSV/JSON reporting, performance tuning, initial packaging |
| Dec 1–15, 2025 | USB forensic acquisition pipeline (`dd` + `sha256sum` + `blkid`) |
| Dec 15–31, 2025 | Interactive TTY menu; `detect_usb.sh`; pipeline-usb command |
| Jan 5–20, 2026 | Demo pipeline (`dummy.img`); end-to-end pipeline validation |
| Jan 20–31, 2026 | Consolidate all README files into unified `readme5.md` |
| Feb 1–15, 2026 | Code review and audit; identify P0/P1/P2/P3 issues |
| Feb 15–28, 2026 | **P0 fixes:** Fix double-prefix `global_id` bug; extract timestamps (atime/mtime/ctime) from XFS and Btrfs inodes; add `recovery_plan` column to metadata.csv |
| Mar 1–5, 2026 | **P1 fixes:** Real ISO 8601 timestamps in JSON; Btrfs extent type filter (`==108`); CSV comma escaping (RFC 4180); remove duplicate `meta.sh` |
| Mar 5–10, 2026 | **P2 fixes:** Full directory path reconstruction (parent→child chain walking for multi-level paths); expanded TTY menu (6 options with USB confirmation, report viewer, clean) |
| Mar 10–14, 2026 | **P3 fixes:** Progress indicator for large images (5% updates); automated test suite (29 tests); human-readable forensic report generator (`report.txt`); `report` and `test` commands in pipeline; updated `readme5.md` with all commands |

---

## Future Improvements

- Snapshot recovery (Btrfs)
- Advanced carving heuristics
- HTML forensic reports
- Performance optimizations
- Support for additional filesystems (ext4, NTFS)

---

## License

License will be added after academic evaluation. Planned: MIT License or Apache 2.0.

---

## Acknowledgements

Special thanks to the Information Technology Department at SKIT Jaipur for academic guidance, infrastructure support, and project mentorship.
