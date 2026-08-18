# Matillion Analytics — Snowflake Pipeline Workspace

![Matillion](https://img.shields.io/badge/Matillion-ETL%20%2F%20DPC-19A0D2)
![Snowflake](https://img.shields.io/badge/Snowflake-Cloud%20DW-29B5E8)
![Amazon S3](https://img.shields.io/badge/Amazon%20S3-Staging-569A31)
![Pipelines](https://img.shields.io/badge/pipelines-13-blue)

A working set of **Matillion pipelines built against Snowflake** — semi-structured JSON handling, Excel-driven table iteration, and file-loop patterns, plus the sales datasets they run on.

Everything here is a real, importable pipeline definition (`.orch.yaml` for orchestration, `.tran.yaml` for transformation) — not screenshots or notes.

---

## What is inside

| Folder | What it holds |
|---|---|
| `Practice/` | File-iterator and fixed-iterator loading patterns, plus a `filter_high_amounts` transformation |
| `Project/Project1/` | Six months of raw sales extracts (`sales_mar` → `sales_aug`, CSV) |
| `Project/Project_Json_Excl_TableItr/` | The **GreenWave Technologies** build — DDL, orchestration and transformation pipelines |
| `.../PracticeMat/` | `flatten-variant` demos for semi-structured data |
| `.../PracticeMat/Flow_Components/` | Flow control demos — unconditional, `AND`, and `OR` connectors |

---

## The GreenWave Technologies build

The flagship pipeline in this repo. `GreenWave Technologies Demo.orch.yaml` chains:

```
Start → S3 Load → Create Table → Excel Query → Table Iterator → Run Transformation → End Success
```

Supporting pieces:

| File | Role |
|---|---|
| `DDL.sql` | Creates `DEV_DB` / `TEST_DB` / `PROD_DB`, matching schemas, and `VARIANT` landing tables — `GW_CUSTOMER_ACCOUNTS`, `GW_ORDERS`, `GW_ORDER_ITEMS`, `GW_ITEMS`, `CUST_JSON`, `IP_RANGE` |
| `Create SHEET_NAMES.tran.yaml` | Builds the sheet-name driver table the iterator loops over |
| `Calculate Profit and Revenue.tran.yaml` | The business logic — joins, calculator, aggregate, rename |
| `Newmini.orch.yaml` | Trimmed variant of the same flow |

**VARIANT** (Snowflake's schemaless column type) means raw JSON lands untouched and is unpacked later with `extract-nested-data-sf` / `flatten-variant` — no loss on ingest, no schema guesswork at load time.

---

## Component coverage

Across the 13 pipelines, these Matillion components are exercised:

| Area | Components used |
|---|---|
| Load | `s3-load`, `excel-query`, `database-query`, `fixed-flow` |
| Iteration | `table-iterator`, `file-iterator`, `fixed-iterator` |
| Flow control | `if`, `and`, `or`, `end-success`, `end-failure` |
| Semi-structured | `extract-nested-data-sf`, `flatten-variant` |
| Transform | `join`, `filter`, `calculator`, `aggregate`, `rename`, `convert-type` |
| Write | `create-table`, `create-table-v2`, `rewrite-table`, `sql-executor` |

---

## Running these pipelines

1. In Matillion, open your project → **Import** → select the `.orch.yaml` / `.tran.yaml` files.
2. Point the Snowflake environment at your own account, warehouse and database. The DDL assumes `COMPUTE_WH` and a `DEMODATABASE` / `PUBLIC` starting point.
3. Upload the CSVs in `Project/Project1/` (or your own) to an S3 bucket and update the S3 Load components to that bucket.
4. Run `DDL.sql` in a Snowflake worksheet first — the pipelines expect those tables to exist.

**Nothing runs unchanged.** Bucket names, warehouse names and credentials in these files are from the original environment; they are placeholders for yours.

---

## Notes worth keeping

- **`.orch` vs `.tran`** — orchestration pipelines move data and control flow; transformation pipelines only reshape data already inside Snowflake. Mixing the two up is the most common early mistake.
- **Iterate on a driver table, not a hardcoded list.** `Create SHEET_NAMES` exists so the loop reads its inputs from data — add a sheet, and the pipeline picks it up with no edit.
- **Land raw, transform later.** VARIANT columns first, structure second, keeps ingestion from breaking every time an upstream field changes.
