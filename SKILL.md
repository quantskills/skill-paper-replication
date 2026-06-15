---
name: paper-replication
description: Systematically reproduce quantitative finance papers with framework-neutral scripts, third-party market data, standalone research backtests, and packaged outputs.
version: 2.0.2
author: PandaAI
license: GPL-3.0-only
metadata:
  organization: QuantSkills
  organization_url: https://github.com/quantskills
  repository: skill-paper-replication
  repository_url: https://github.com/quantskills/skill-paper-replication
  project_type: skill
  collection: paper-replication
  creator: abgyjaguo
  maintainer: abgyjaguo
  tags: [quant, research, paper, replication, backtest]
  compatible_agents: [codex, hermes, claude-code, aider, cursor, generic-cli-agent]
---

# Paper Replication

Use this skill to reproduce quantitative finance papers in any agent framework that can read
this directory and run local Python scripts.

The primary pipeline is standalone: paper search/download, PDF extraction, formula reconstruction,
Pandas-based research backtest, charts, metrics, and deliverable packaging.

## Framework-Neutral Usage

Agents should resolve paths relative to this `SKILL.md` file:

```text
paper-replication/
  SKILL.md
  README.md
  requirements.txt
  scripts/
  templates/
  references/
```

Do not assume any agent-specific install folder. When invoking scripts, use the actual
local path to this skill directory.

## Dependencies

Install dependencies in the active Python environment:

```bash
python -m pip install -r /path/to/paper-replication/requirements.txt
```

## Output Location

Generated paper artifacts must go under:

```text
/home/coder/project/replication/paper-replication/{paper_id}/
```

Never write generated paper artifacts back into the skill directory.

## Quick Start

```bash
cd /home/coder/project/replication/paper-replication

python /path/to/paper-replication/scripts/run_research.py \
  --pipeline \
  --paper-id 2201.06635 \
  --symbols rb,if,au \
  --strategy tsmom
```

## Workflow

1. Search or select a paper.
   Use `scripts/search_arxiv.py`, or accept a user-provided arXiv ID/PDF.

2. Extract the paper.
   Use `scripts/extract_paper.py` with PyMuPDF. Focus on formulas, asset universe,
   data period, benchmark metrics, transaction cost assumptions, and signal timing.

3. Define the implementation plan.
   State the data source, signal logic, portfolio construction, normalization,
   costs, and validation metrics before coding.

4. Run standalone reproduction.
   Use `scripts/reproduce_paper.py`. The default Chinese futures data path uses akshare;
   CSV data is acceptable when the paper requires a custom dataset.

5. Validate results.
   Compare the reproduction metrics against the paper and explain gaps using data
   coverage, signal delay, universe differences, costs, or normalization choices.

6. Package deliverables.
   Save extracted notes, metrics JSON, equity/weights CSVs, and charts in the paper
   output directory.

## Data Source Rules

- Keep third-party data sources for market data access.
- Default Chinese futures data source: akshare `futures_zh_daily_sina`.
- International data may use yfinance when appropriate.
- User-provided CSV is allowed when the paper or user supplies a dataset.
- Always record the data source, date range, and latest available date.
- Always guard against look-ahead bias with shifted weights/signals.

## Standalone Strategies

Supported by `scripts/reproduce_paper.py`:

| Strategy | Signal | Portfolio |
| --- | --- | --- |
| `tsmom` | sign of lookback return | inverse volatility |
| `csmom` | cross-sectional return rank | signal weighted |
| `risk_parity` | long-only risk allocation | covariance based |
| `trend_vol` | short lookback trend | inverse volatility |

## Quality Checks

- Verify `df["date"].max()` or equivalent latest timestamp is plausible.
- Confirm no future bars enter signal calculation.
- Check output files exist and are non-empty.
- Report replication gaps honestly instead of forcing a match.
