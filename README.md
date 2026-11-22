# Databricks_Hackathon_2026-UW_MSDS
Which aid projects deserve a second look? This Databricks tool benchmarks every OCHA pooled-fund project against its peers, flags unusual cost-per-beneficiary ratios with confidence and context, and lets reviewers ask questions in plain English through Genie.


# CBPF Beneficiary Targeting Checker

**Databricks Hackathon 2026 · Challenge 1: Smart Beneficiary Targeting Validation**

A decision-support tool for humanitarian pooled-fund managers. It flags CBPF projects whose **cost per beneficiary
per month** is unusually high or low compared with similar projects, explains why, and suggests comparable projects
for benchmarking. A flag means *worth a closer look*, never *wrong*.

## How it works

```
HDX sources (CBPF, CERF, FTS, population, INFORM / HAPI)
        │
        ▼  Lakeflow Job (one notebook per task)
01 ingest_bronze → 02 clean_silver → 03 score_gold ─┬─ 04 flag_and_validate ─┐
                                                    ├─ 05 comparables ───────┼─ 06 explain_llm
                                                    └─ 07 gap_index          │
        ▼                                                                    ▼
Delta gold tables in Unity Catalog (workspace.hackathon_gold)
        │
        ├── AI/BI dashboard (Flags · Checker · Quality, Ask Genie built in)
        ├── Genie Agent (plain-English Q&A, check_proposal() trusted function)
        └── App (optional)
```

1. **Cost metric:** `budget / beneficiaries / months` (USD per person per month).
2. **Peer groups:** sector × country × year, falling back to sector × country, sector × year, then sector,
   when a group has fewer than 8 projects.
3. **Flag:** robust z-score (median + MAD) beyond the threshold → `HIGH` (unusually expensive) or `LOW`
   (unusually cheap, possibly over-counted beneficiaries). Every flag carries a confidence level and reason.
4. **Excluded:** logistics, coordination and telecom projects (their "beneficiaries" are agencies), and projects
   without budget, beneficiaries, duration or country.
5. **Comparables:** 5 projects matched on sector, country, year, budget size and org type, never on cost.

## Repository layout

| Folder | Contents |
| --- | --- |
| `notebooks/` | Databricks notebooks, one per pipeline step (see `notebooks/README.md`) |
| `dashboards/` | AI/BI dashboard export (`.lvdash.json`) |
| `genie/` | Genie Agent configuration: setup SQL, instructions, example queries, benchmarks |
| `jobs/` | Lakeflow Job definition |
| `app/` | Optional Databricks App |
| `docs/` | Data sources, table schema, validation results, pitch |

## How to run

1. Import `notebooks/` into a Databricks workspace (Free Edition works), keeping all notebooks in one folder so
   `%run ./NB0-Scoring_Library` resolves.
2. Run in order: NB1 → NB2 → NB3 (validate, writes `gold_model_config`) → NB4 (score) → teammate tasks
   (planned vs actual, LLM explanations, gap index) → NB9 (Genie setup). Or run the Lakeflow Job in `jobs/`.
3. Import `dashboards/*.lvdash.json` (Dashboards → ▾ → Import dashboard from file).
4. Create the Genie Agent with the files in `genie/`.

## Results

See [`docs/validation_results.md`](docs/validation_results.md).

## Limits (read before using the flags)

* Beneficiary numbers are self-reported and planned, not verified.
* INFORM severity is a single April 2024 snapshot, used as context only.
* Planted-error tests measure whether a 5× error is detectable, not how much real misreporting exists.
* High cost can be legitimate (remote access, insecurity, surgery, construction). The tool supports reviewers and
  does not decide.

## Team

| Member | Area |
| --- | --- |
| @Nirubama12 | Bronze, silver, scoring, validation, Genie Agent |
| utg1209 | _add your area_ |
| paartk | LLM explanations, gap index |

## Data

All data comes from the Humanitarian Data Exchange (HDX). Raw files are not committed; see
[`docs/data_sources.md`](docs/data_sources.md) for links and licences.
