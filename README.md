# CVE Patch Collection Dashboard

Live dashboard for daily automated CVE patch crawling from four vulnerability databases.

**Dashboard**: [https://llf3239cc-coder.github.io/incomplete-patch-detection-platform/](https://llf3239cc-coder.github.io/incomplete-patch-detection-platform/)

## Data Sources

| Source | Description | Method |
|--------|-------------|--------|
| **GitHub Advisory (GHSA)** | GitHub Security Advisory Database | Git repo parsing |
| **NVD (CVEListV5)** | NIST National Vulnerability Database | Git repo parsing |
| **OSV** | Open Source Vulnerabilities (Google) | ZIP download from GCS |
| **Snyk** | Snyk Vulnerability Database | Web scraping |

## Pipeline

A daily cron job runs the following steps:

1. **Sync repos** - Clone/pull advisory-database and cvelistV5
2. **GHSA Collector** - Parse GitHub Advisory JSON files
3. **NVD Collector** - Parse CVEListV5 JSON files
4. **OSV Collector** - Download and parse OSV all.zip
5. **Snyk Collector** - Scrape security.snyk.io (optional, slow)
6. **Merge Patches** - Deduplicate and merge all sources into `merged_cves`
7. **Patch Downloader** - Download `.patch` diffs from GitHub commit URLs
8. **Export Stats** - Write daily snapshot to `data/history.json`, commit and push

## Changing the Daily Run Time

The pipeline runs via cron. To change the schedule:

```bash
crontab -e
```

Edit the schedule (first two fields are minute and hour in UTC):

```
# Current: 2:00 AM UTC
0 2 * * * PYTHONPATH=/home/lyuye_ubuntu/data /home/lyuye_ubuntu/data/.venv/bin/python3 pipeline/daily_pipeline.py >> /home/lyuye_ubuntu/data/log/cron_pipeline.log 2>&1

# Example: 8:00 AM UTC
0 8 * * * PYTHONPATH=...

# Example: every 12 hours
0 */12 * * * PYTHONPATH=...
```

## Manual Run

```bash
cd /home/lyuye_ubuntu/data
PYTHONPATH=/home/lyuye_ubuntu/data .venv/bin/python3 pipeline/daily_pipeline.py
```

Options:
- `--skip-snyk` - Skip Snyk collector (saves hours)
- `--skip-download` - Skip patch file download
- `--year-filter 2025,2026` - Filter by CVE year

## Dashboard Features

- Total CVE counts per source with daily deltas
- CVE growth over time (stacked area chart)
- Daily new CVEs (stacked bar chart)
- Patch download progress vs merged CVEs
- Source breakdown (doughnut chart)
- Pipeline run history with per-step status
- Dark/light theme toggle (persisted in browser)

## Repository Structure

```
index.html          # Dashboard (static HTML + Chart.js)
data/
  history.json      # Daily stats snapshots (auto-updated by pipeline)
README.md
```

## GitHub Pages Setup

In the repo Settings > Pages:
- Source: **Deploy from a branch**
- Branch: **main**, folder: **/ (root)**
