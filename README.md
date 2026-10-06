# RetroBoxDB VirtualBoy

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Nintendo Virtual Boy. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 176 source ZIPs, 76.4 MiB (No-Intro 127, RetroAchievements sets 49); 176 ROM files, 315.1 MiB uncompressed |
| Stored size | populated database 26.5 MiB; public Catalog 2.7 MiB (no ROM data) |
| Ratio | 34.6% of the source ZIPs, 8.4% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 256 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 128 MiB (128 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (78 files, each checked against the DAT hashes): 51.1 MiB/s, 36 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.717 s, TorrentZip 0.99 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.VirtualBoy.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-VirtualBoy/releases/latest/download/RetroBoxDB.VirtualBoy.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-virtualboy-games.csv) / [summary](reports/ra-virtualboy.json), [build report](reports/virtualboy-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

19 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-virtualboy.json`): smallest 512 KiB / 256 MiB at 21.84 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 256 KiB / 128 MiB at 21.87 MiB. ZIPs 76.45 MiB, per-file LZMA 33.69 MiB.

- ROMs are 64 KiB–4 MiB (two files are 16 MiB); the game header sits 0x220 bytes before the end of the image: 20-byte Shift-JIS title, maker code, game code and version, stored in `vb_hardware`.
- The No-Intro folders hold 81 licensed, 45 aftermarket and 1 private ZIP; the RetroAchievements set adds 49 files. RetroAchievements console 28 (whole-file MD5).

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 97 / 53 / 80 |
| DAT coverage per version | 20260428-015207: 78/78; 20260804-192036: 78/80; 20260805-170231: 78/80 |
| Local ROMs in no DAT | 19 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 31, RA only 18, hash not in the latest RA snapshot 0 ([list](reports/ra-virtualboy-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-virtualboy-missing.csv) |
| No-Intro DB Export + Dump Log 20260804-192036 | 81 archives, 81 file identities, 76 documented hardware assertions; Dump Log Verified 21 |
| RetroAchievements (console 28) | 43 games with achievements: 41 with a local ROM (51 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 2 without a No-Intro counterpart |
| Chinese names | 33 of 33 rows translated (23 unique); 32 local ROMs have a Chinese name |
| Populated-database audit | 102 objects, 2 groups, 113 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.VirtualBoy.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.VirtualBoy.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.VirtualBoy.sqlite --discover --ra --catalog RetroBoxDB.VirtualBoy.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
