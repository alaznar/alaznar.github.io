---
title: "FAERS Adverse Events · SQL Project"
excerpt: "SQLite-powered exploration of FDA adverse event reports"
teaser: /assets/img/FAERS.png
---

***Challenge***  
Build a beginner‑friendly yet solid SQL pipeline (with `sqlite3` in Python) to ingest, model, and query FDA Adverse Event Reporting System (FAERS) quarterly extracts. Answer practical safety questions such as top reactions, suspect drugs, and outcomes over time.

***Focus***  
Designing a simple relational schema from the FAERS text files; loading data into SQLite; crafting readable SQL for joins across `DEMO`, `DRUG`, `REAC`, `OUTC`, etc.; validating results with sanity checks and small visualizations.

***Results***  
- Lightweight SQLite database created from raw `$`‑delimited FAERS files  
- Reusable queries for: most frequent reactions (PT), primary‑suspect drugs, outcomes (death, hospitalization, etc.), and country breakdowns  
- A clean Jupyter notebook showing step‑by‑step loading, schema, and analysis

***Stack***  
Python · sqlite3 · SQL · pandas · matplotlib · Jupyter Notebook

- [Repo](https://github.com/alaznar/FAERS-Ozempic)
- [Click here to see the Project](https://github.com/alaznar/FAERS-Ozempic/blob/main/FAERS.ipynb)
