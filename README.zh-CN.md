# RetroBoxDB VirtualBoy

[English](README.md) | 中文

任天堂 Virtual Boy的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 176 个，76.4 MiB（No-Intro 127 个，RetroAchievements 集合 49 个）；解压后 ROM 176 个，315.1 MiB |
| 入库后大小 | 完整库 26.5 MiB；公开 Catalog 2.7 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 34.6%，为解压后 ROM 总量的 8.4% |
| 使用的技术 | 存储 v4：256 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 128 MiB 的 LZMA2 实体组（字典 128 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，78 个文件，逐个按 DAT 哈希校验）：51.1 MiB/s，平均 36 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 0.717 秒，TorrentZip 平均 0.99 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.VirtualBoy.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-VirtualBoy/releases/latest/download/RetroBoxDB.VirtualBoy.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-virtualboy-games.csv)／[汇总](reports/ra-virtualboy.json)、[构建报告](reports/virtualboy-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 19 种块／组组合（`assessment/data/storage-experiment-virtualboy.json`）：最小为 512 KiB / 256 MiB 21.84 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 256 KiB / 128 MiB 21.87 MiB。ZIP 76.45 MiB，逐文件 LZMA 33.69 MiB。

- ROM 为 64 KiB–4 MiB（另有两个 16 MiB 的文件）；游戏头位于镜像末尾前 0x220 字节：20 字节 Shift-JIS 标题、厂商代码、游戏代码和版本，存入 `vb_hardware`。
- No-Intro 目录有 81 个授权、45 个 Aftermarket 和 1 个 Private ZIP，RA 目录另有 49 个文件。RetroAchievements 主机 28（整文件 MD5）。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 97／53／80 |
| 各版 DAT 覆盖 | 20260428-015207：78/78；20260804-192036：78/80；20260805-170231：78/80 |
| 不在任何 DAT 的本地 ROM | 19 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 31，仅 RA 收录 18，哈希不在最新 RA 快照 0（[清单](reports/ra-virtualboy-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-virtualboy-missing.csv) |
| No-Intro DB Export＋Dump Log 20260804-192036 | 81 个档案、81 个文件身份、76 条有文档的硬件声明；Dump Log Verified 21 |
| RetroAchievements（console 28） | 有成就的游戏 43 个：本地有 ROM 41（51 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 2 |
| 中文名 | 33 条记录中 33 条有中文（23 个唯一名）；本地 ROM 32 个有中文名 |
| 完整库审计 | 102 个对象、2 个组、113 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.VirtualBoy.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.VirtualBoy.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.VirtualBoy.sqlite --discover --ra --catalog RetroBoxDB.VirtualBoy.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
