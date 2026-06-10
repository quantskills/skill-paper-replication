# Paper Replication Agent v2.0.2

Framework-neutral skill for reproducing quantitative finance papers.

## 中文说明

`paper-replication` 是一个面向量化金融论文复现的 Codex/Agent 技能包。它不绑定特定智能体框架，只要目标环境可以读取本目录并运行本地 Python 脚本，就可以完成论文检索、PDF 提取、公式与策略逻辑整理、研究型回测、图表生成和结果打包。

适合用于：

- 复现 arXiv 或本地 PDF 中的量化金融论文。
- 将论文中的信号、组合构建、成本假设和评价指标转成可运行的 Pandas 回测。
- 使用 akshare、yfinance 或用户提供的 CSV 数据进行实验。
- 输出复现笔记、指标 JSON、净值/权重 CSV 和图表文件，便于后续审阅或报告撰写。

基本流程：

1. 用 `scripts/search_arxiv.py` 搜索或下载论文，也可以直接提供本地 PDF。
2. 用 `scripts/extract_paper.py` 提取论文文本、公式、数据区间、资产范围和评价指标。
3. 明确数据源、信号计算、组合构建、成本、换仓和指标口径。
4. 用 `scripts/reproduce_paper.py` 或 `scripts/run_research.py --pipeline` 运行复现实验。
5. 检查数据是否为空、最新日期是否合理、信号是否做了滞后处理，并解释与论文结果的差异。

所有生成结果应写入：

```text
/home/coder/project/replication/paper-replication/{paper_id}/
```

不要把复现实验输出写回 skill 目录本身。

The skill can live in any project or agent workspace. A third-party agent only needs to:

1. Read this directory.
2. Install `requirements.txt`.
3. Invoke scripts by their actual local path.
4. Write outputs under `/home/coder/project/replication/paper-replication/`, not into the skill folder.

Data sources remain third-party sources such as akshare, yfinance, or user-provided CSV files.

## Install Dependencies

From any agent framework:

```bash
python -m pip install -r /path/to/paper-replication/requirements.txt
```

No framework-specific install location is required. Use the actual local path to this
skill directory.

## Project Root

Generated artifacts must go under:

```text
/home/coder/project/replication/paper-replication/{paper_id}/
```

Example:

```bash
python /path/to/paper-replication/scripts/run_research.py \
  --pipeline \
  --paper-id 2201.06635 \
  --symbols rb,if,au \
  --strategy tsmom
```

## Output Layout

```text
/home/coder/project/replication/paper-replication/{paper_id}/
  reports/{paper_id}.pdf
  reports/extracted_{paper_id}.md
  reports/metrics_{strategy}.json
  data/equity_{strategy}.csv
  data/weights_{strategy}.csv
  charts/chart_{strategy}.png
```

## Single Steps

Search arXiv:

```bash
python /path/to/paper-replication/scripts/search_arxiv.py \
  "momentum futures" \
  --max 5 \
  --download
```

Extract a PDF:

```bash
python /path/to/paper-replication/scripts/extract_paper.py \
  --pdf /home/coder/project/replication/paper-replication/2201.06635/reports/2201.06635.pdf \
  --markdown \
  --output /home/coder/project/replication/paper-replication/2201.06635/reports/extracted_2201.06635.md
```

Run standalone reproduction:

```bash
python /path/to/paper-replication/scripts/reproduce_paper.py \
  --symbols rb,if,au \
  --strategy tsmom \
  --start 2020-01-01 \
  --end 2024-12-31 \
  --output-dir /home/coder/project/replication/paper-replication/2201.06635
```

## Data Sources

- Chinese futures: `akshare.futures_zh_daily_sina`.
- International instruments: `yfinance` when needed.
- Custom paper datasets: CSV with required `date,open,high,low,close,volume` columns.

Always record the data source, requested date range, loaded date range, latest available date,
and any stale-data caveat.

## Arguments

### `run_research.py`

| Argument | Default | Notes |
| --- | --- | --- |
| `--paper-id` | - | arXiv paper ID, e.g. `2201.06635` |
| `--paper` | - | arXiv search query |
| `--pdf` | - | Local PDF path |
| `--pipeline` | `false` | Run the full workflow |
| `--symbols` | `rb,if,au` | Comma-separated instruments |
| `--strategy` | `tsmom` | `tsmom`, `csmom`, `risk_parity`, `trend_vol` |
| `--start` | `2020-01-01` | Backtest start date |
| `--end` | `2024-12-31` | Backtest end date |
| `--target-vol` | `0.10` | Annual target volatility |
| `--skip-reproduce` | `false` | Skip standalone backtest |

### `reproduce_paper.py`

| Argument | Default | Notes |
| --- | --- | --- |
| `--symbols` | `rb,if,au` | Comma-separated instruments |
| `--strategy` | `tsmom` | `tsmom`, `csmom`, `risk_parity`, `trend_vol` |
| `--start` | `2020-01-01` | Start date |
| `--end` | `2024-12-31` | End date |
| `--target-vol` | `0.10` | Annual target volatility |
| `--cost` | `0.0001` | Transaction cost rate |
| `--capital` | `1000000` | Initial capital |
| `--output-dir` | `/home/coder/project/replication/paper-replication` | Output directory |

## Validation

- Confirm the loaded data is non-empty.
- Confirm latest data date is not unexpectedly stale.
- Use shifted weights/signals to avoid look-ahead bias.
- Check generated JSON/CSV/PNG files are non-empty.
- Explain metric gaps against the paper instead of hiding them.
