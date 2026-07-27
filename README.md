# GitHub ETL Pipeline

A scheduled ETL pipeline that pulls trending Python repositories from the GitHub Search API daily, transforms the data, and loads it into a local SQLite database. Built as part of a public data engineering learning series on [Towards Data Science](https://towardsdatascience.com/).

---

## What it does

Every day at 9am UTC, a GitHub Actions workflow spins up a fresh environment, runs the pipeline, and shuts down. No server required.

The pipeline:
- **Extracts** the top 30 trending Python repositories created in the last 24 hours from the GitHub Search API
- **Transforms** the data using pandas: drops rows with missing descriptions, adds a `viral` flag for repos with over 50,000 stars, and sorts by star count
- **Loads** the cleaned data into a SQLite database with idempotency handling to prevent duplicates

---

## Architecture

<img width="2720" height="2080" alt="github_etl_pipeline_architecture" src="https://github.com/user-attachments/assets/00c63479-8b6c-48af-80e7-082a40fce543" />

---

## Project structure

<img width="397" height="181" alt="Screenshot 2026-07-27 110117" src="https://github.com/user-attachments/assets/abcf0b99-275e-4c28-a19f-8b9a495efc1a" />

---

## How to run locally

**1. Clone the repo**

```bash
git clone https://github.com/ibbysalam/github-etl.git
cd github-etl
```

**2. Install dependencies**

```bash
pip install -r requirements.txt
```

**3. Run the pipeline**

```bash
python pipeline.py
```

A `github_repos.db` file will appear in the project folder after the first run.

---

## How to inspect the data

**Option 1: Python**

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect('github_repos.db')
df = pd.read_sql('SELECT * FROM repos ORDER BY stars DESC', conn)
conn.close()

print(df.head(10))
```

**Option 2: DB Browser for SQLite**

Download [DB Browser for SQLite](https://sqlitebrowser.org/), open `github_repos.db`, and browse the `repos` table visually. No code needed.

**Option 3: SQLite CLI**

```bash
sqlite3 github_repos.db
```

Then run SQL directly:

```sql
SELECT name, owner, stars, viral FROM repos ORDER BY stars DESC LIMIT 10;
```

To check how many rows are loaded per day:

```sql
SELECT loaded_at, COUNT(*) as count FROM repos GROUP BY loaded_at ORDER BY loaded_at DESC;
```

---

## Schema

| Column | Type | Description |
|---|---|---|
| name | TEXT | Repository name |
| owner | TEXT | GitHub username of the owner |
| stars | INTEGER | Star count at time of extraction |
| forks | INTEGER | Fork count at time of extraction |
| language | TEXT | Primary language (Python for all rows) |
| description | TEXT | Repository description |
| url | TEXT | GitHub URL (used as unique key for dedup) |
| created_at | TEXT | Timestamp the repo was created on GitHub |
| viral | TEXT | "Yes" if stars > 50,000, otherwise "No" |
| loaded_at | TEXT | Date the row was loaded by the pipeline |

---

## Scheduling

The pipeline runs on a cron schedule defined in `.github/workflows/schedule.yml`:

```yaml
on:
  schedule:
    - cron: '0 9 * * *'
  workflow_dispatch:
```

`workflow_dispatch` allows manual triggering from the GitHub Actions tab without waiting for the scheduled time.

---

## Part of a learning series

This pipeline is built and documented across a series of articles on Towards Data Science:

1. [My 12-month self-study roadmap](https://towardsdatascience.com/from-data-analyst-to-data-engineer-my-12-month-self-study-roadmap/)
2. [Building my first ETL pipeline as a complete beginner](https://towardsdatascience.com/i-built-my-first-etl-pipeline-as-a-complete-beginner-heres-exactly-how/)
3. [Making the pipeline production-ready](https://towardsdatascience.com/i-thought-data-engineering-was-just-writing-scripts-i-was-wrong/)
4. [Scheduling the pipeline with GitHub Actions]([https://towardsdatascience.com/](https://towardsdatascience.com/i-tried-to-schedule-my-etl-pipeline-heres-what-i-didnt-expect/))
