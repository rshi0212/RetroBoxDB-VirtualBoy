VirtualBoy Catalog, storage v4 (256 KiB blocks, 2 solid LZMA2 groups of up to 128 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release of this platform.
- One schema for all twenty-three platforms: the header tables of the eight new platforms (Game Gear, PC Engine, SuperGrafx, MSX, MSX2, Virtual Boy, Game & Watch, Super A'Can: `pce_hardware`, `msx_hardware`, `vb_hardware`) and `rom_annotations` exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color, PC Engine<->SuperGrafx, MSX<->MSX2): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 176 ZIPs (nointro 127, retroachievements 49), 76.4 MiB (176 ROM files, 315.1 MiB uncompressed). Populated database: 26.5 MiB (34.6% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 97 ROM records, 53 games, 80 releases; DAT versions: 20260428-015207, 20260804-192036, 20260805-170231.
- RetroAchievements: 41 of 43 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 51.1 MiB/s (78 files); single file with a cold cache 0.717 s (ROM) / 0.99 s (TorrentZip) on average.
- Full audit of the populated database: 102 objects, 2 groups, 113 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-VirtualBoy/blob/main/README.zh-CN.md)
