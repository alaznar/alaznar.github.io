---
title: "FAERS Adverse Events · SQL Project"
excerpt: "SQLite-powered exploration of FDA adverse event reports"
teaser: /assets/img/FAERS.png
---

***Challenge***  
Build a beginner‑friendly yet solid SQL pipeline (with `sqlite3` in Python) to ingest, model, and query FDA Adverse Event Reporting System (FAERS) quarterly extracts. Answer practical safety questions such as top reactions, suspect drugs, and outcomes over time.

***Focus***  
Designing a simple relational schema from the FAERS text files; loading data into SQLite; crafting readable SQL for joins across `DEMO`, `DRUG`, `REAC`, `OUTC`, etc.; handling partial dates; creating indexes; and validating results with sanity checks and small visualizations.

***Results***  
- Lightweight SQLite database created from raw `$`‑delimited FAERS files  
- Reusable queries for: most frequent reactions (PT), primary‑suspect drugs, outcomes (death, hospitalization, etc.), and country breakdowns  
- Examples of dechallenge/rechallenge filters and therapy windows  
- A clean Jupyter notebook showing step‑by‑step loading, schema, and analysis (beginner style, well commented)

***Stack***  
Python · sqlite3 · SQL · pandas · matplotlib · Jupyter Notebook

- [Repo](https://github.com/alaznar/FAERS-Ozempic)
- [Click here to see the Project](https://github.com/alaznar/FAERS-Ozempic/blob/main/FAERS.ipynb)

---

### What’s inside

- **Schema (beginner‑friendly):**  
  `DEMO` (one row per case/version) · `DRUG` (one+ per case) · `REAC` (one+ MedDRA PT per case) · `OUTC` · `RPSR` · `THER` · `INDI`.  
  Includes simple integer/text types and a few practical indexes (e.g., `PRIMARYID`, `CASEID`, `DRUG_SEQ`).

- **ETL steps (notebook):**  
  1) Read raw `.TXT` files with `'$'` separator  
  2) Minimal cleaning (trim, uppercase codes, parse partial dates as text `YYYY`, `YYYYMM`, `YYYYMMDD`)  
  3) Create tables and insert in chunks  
  4) Add indexes and quick row counts to verify loads

- **Sample queries:**  
  - Top 20 reaction PTs and their counts  
  - Primary‑suspect (`ROLE_COD='PS'`) drugs linked to those reactions  
  - Outcomes distribution from `OUTC` (DE, LT, HO, …)  
  - Country split (`OCCR_COUNTRY`) and reporter type (`OCCP_COD`)  
  - Therapy windows join (`THER` + `DRUG`) for simple time filters

- **Plots (quick checks):**  
  Bar charts for top PTs/drugs and outcome proportions to validate SQL outputs.

---

### Why SQLite?
Small, fast, zero‑setup, and perfect for learning SQL with a real‑world healthcare dataset. The notebook shows all steps with clear comments so you can reuse or adapt it to a larger RDBMS later.

---

### Limitations & notes
Spontaneous reports can’t be used to estimate incidence or infer causality; counts are descriptive only. Some dates are partial (year/month only), and the latest case version is used in quarterly extracts.

