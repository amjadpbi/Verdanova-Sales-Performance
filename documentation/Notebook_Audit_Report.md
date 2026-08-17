# Verdanova — Fabric Notebook Audit Report

**Scope:** The five notebooks in `./Notebooks/` only.
**Type:** Audit only. No project files were modified.
**Notebooks reviewed:**

| # | File | Cells | Role |
|---|------|-------|------|
| 01 | `Notebook 01 — Generate Operational Source Data.ipynb` | 25 | Generate synthetic source data → Files → Bronze |
| 02 | `Notebook 02 – Profile Operational Data.ipynb` | 17 | Profile Bronze data quality |
| 03 | `Notebook 03 – Transform Operational Data to Silver.ipynb` | 14 | Bronze → Silver (cleanse/standardize) |
| 04 | `Notebook 04 – Build Gold Analytical Model.ipynb` | 19 | Silver → Gold (dimensions + facts) |
| 05 | `Notebook_5_Gold_Validation.ipynb` | 23 | Validate the Gold layer |

---

## 1. Overall Assessment

The notebook set is **solid, coherent, and already better documented than most learning projects**. The medallion flow (Bronze → Silver → Gold → Validation) is logical, the table lineage is clean, and every notebook opens with a structured H1 header stating its objective. The intentional data-quality injections in Notebook 01, profiled in 02 and resolved in 03, form a genuinely good teaching arc.

The weaknesses are almost entirely about **consistency and explanation**, not correctness of design:

- **Documentation depth swings wildly** — from a bare bold label (`**Customers**`) to an 86-line header (NB05) or a 7-subsection cell (NB04 cell 13). There is no single standard.
- **Heading style is mixed** — some steps use real Markdown headings (`##`), most use bold text as fake headings (`**Label**`). This is the single most visible inconsistency.
- **The "why" is usually missing.** Steps are labeled by *what* they operate on (a table name) but rarely explain *what layer the data is in, what transformation is happening, or why*. For a DP-600 learning project this is the biggest lost opportunity.
- **A few documentation statements no longer match the code** (NB04 column lists; NB04/NB05 header table names), which actively misleads a learner.
- **Some cross-notebook wiring is undocumented** — most notably NB01 depends on a table (`sales`) that only exists after NB04 runs, and NB05 assumes a `Sales.Category` column that NB04's committed code does not create.

None of these require rewriting logic. The pipeline works. This is a **documentation and consistency pass**, not a refactor.

**Overall grade:** structurally sound; documentation is good in places, absent in others, and inconsistent throughout.

---

## 2. Recommended Documentation Standard

**Recommendation: a single "structured-concise" standard — short explanatory prose, consistently applied, with the depth concentrated in the notebook header and closing summary rather than spread across every cell.**

Concretely:

- **Notebook header (once per notebook):** the current H1 + `## Objective / Inputs / Outputs` pattern is right. Detail here is justified — keep it.
- **Each processing step:** a real Markdown heading (`###`) **plus one or two sentences** answering: *what layer is this, what is happening, and why.* Not a bare label. Not a paragraph.
- **Closing summary (once per notebook):** a short "what this notebook produced / proved" cell, like NB03's `## Silver Layer Completed`.

**Why this level (and not shorter or longer):**

- **Shorter (bare bold labels, as in NB02/NB03) is not enough for a learning project.** `**Customers**` tells you nothing you can't already see in the code. When you revisit this in six months, the value is in the one sentence explaining *"Silver cleanse: drop duplicate customers, fill missing email, standardize Region casing — the issues found in profiling."*
- **Longer (NB04 cell 13's 7 subsections, NB05's 86-line header) tips toward a textbook.** That depth is fine **once**, in a header, but is too heavy to repeat on every cell and creates the inconsistency seen now.

The good news: **the target style already exists in the project** — NB04's dimension cells (`## Dimension - Products` + two sentences of purpose) are almost exactly the standard to adopt everywhere.

---

## 3. Notebook-by-Notebook Audit

### Notebook 01 — Generate Operational Source Data

**Current structure:** Rich H1 header (cell 0) → CRM → ERP (Products, Orders, Order Lines) → Marketing → Excel Sales Targets. Each domain generates data, writes to `Files/`, and loads a `bronze_*` Delta table.

**Strengths**
- Excellent opening header (cell 0): Objective, Business Context, Process, Expected Output with a file tree and table list.
- Intentional data-quality issues are deliberately injected and commented (cells 2, 10, 22) — the foundation of the whole learning arc.
- Later sections (cells 22, 24) are genuinely well-organized internally with numbered comment blocks.

**Inconsistencies**
- **Two documentation styles inside one notebook.** CRM and early ERP use terse bold labels (`**Save to Lakehouse Files**`, `**Generate Products**` — cells 3, 9, 11, 13, 15, 17). Marketing/Excel/Order-Lines use structured `##` headings with an **Output** list (cells 19, 21, 23).
- **Sectioning is uneven.** ERP gets `# Section 2 - Enterprise Resource Planning` (cell 8) but there is **no "Section 1 - CRM"** heading — CRM just starts. Marketing (cell 21) and Excel (cell 23) use `##` with no "Section N" prefix. Three different sectioning conventions.
- The "Expected Output" file tree is a fenced code block in cell 0 but an unfenced indented block in cell 8.

**Markdown issues**
- **Cell 1** is a note-to-self, not documentation: `<mark>**Note: I will write code for Microsoft Fabric Notebook, not generic Jupyter Notebook.**</mark>`. It uses raw HTML and doesn't belong in the reader-facing flow.
- **Cell 5** wraps a whole sentence in bold **and nests more bold inside it** (`**... the Lakehouse **Files** area into the **Bronze layer**...**`). Nested `**` breaks Markdown rendering. It is also being used as a paragraph-length "heading."

**Code-cell organization**
- Cells are already grouped sensibly (generate → write Files → load Bronze). **No cell needs splitting.**
- Cells 2, 10, 20, 22, 24 each re-`import` and re-create the Spark session. Harmless in Fabric (cells run independently), so this is a *consistency* note only, not a defect.
- **Cross-notebook dependency (important):** cell 22 reads `spark.table("sales")` to size the marketing revenue envelope. `sales` is a **Gold** table built in **Notebook 04**. So Notebook 01 cannot be run top-to-bottom on a clean lakehouse — cell 22 fails until NB04 has run once. This ordering is real and **undocumented**. (Flag only — do not "fix" by changing logic.)

**Recommended changes**
- Standardize step labels to `###` headings + one sentence (mandatory consistency item — see §6).
- Add a "Section 1 - CRM" heading to match the ERP/Marketing/Excel sections.
- Move the cell-1 note out of the reader flow (or convert to a normal italic aside).
- Add one sentence in/above cell 22 documenting that it depends on the Gold `sales` table and therefore on a prior NB04 run.

---

### Notebook 02 — Profile Operational Data

**Current structure:** H1 header (cell 0) → record counts → one profile block per Bronze table (customers, products, orders, order lines, campaigns, sales targets).

**Strengths**
- **Internally the most consistent notebook.** Every section uses the same terse bold label + a focused profiling cell.
- Clean, readable profiling pattern (count → nulls → duplicates → distinct values).
- Header explicitly states "No data is modified" — good framing for a profiling step.

**Inconsistencies**
- Uses **only** bold labels — no `##` step headings. Internally consistent, but the opposite extreme from NB04/NB05.
- Import style differs from NB03/04: explicit `from pyspark.sql.functions import col, count` rather than the wildcard used later.

**Markdown issues**
- **The "Data quality observations" promised in the header (cell 0 Scope) is never written.** Every check is present in code, but there is no Markdown that states *what was actually found* or *what Silver will do about it*.
- **Cells 7 and 8 have no label.** They sit under the "Product Profile" heading and add a pricing-relationship analysis (UnitPrice vs StandardCost), but nothing tells the reader what that extra analysis is for.

**Code-cell organization**
- Well-scoped. No splitting needed.
- `df` and `products` are reused across cells — fine in a notebook.

**Recommended changes**
- Add a short one-sentence explanation to each profile ("what issue am I looking for here"), and a label for cells 7–8.
- **Highest-value item:** add a closing "Profiling Findings" Markdown cell listing the concrete issues found (missing email, duplicate customers, `NORTH` casing, null category on ProductID 9, duplicate product, null platform, `drinkware` casing, duplicate campaign) and noting they are resolved in NB03. This closes the learning loop between injected issues (NB01) and fixes (NB03).

---

### Notebook 03 — Transform Operational Data to Silver

**Current structure:** H1 header (cell 0) → one cleanse-and-write cell per table → `## Silver Layer Completed` summary (cell 13).

**Strengths**
- **Cleanest, most uniform code pattern in the set:** read `bronze_*` → dedupe → handle nulls → standardize → cast types → write `silver_*`. Same shape every time.
- **Has a proper closing summary (cell 13)** — the model other notebooks should follow.
- Consistent use of `.option("overwriteSchema", "true")`.

**Inconsistencies**
- Step labels are again bare bold table names (`**Customers**`, `**Products**`, …) — no `##`, and no explanation.
- Uses wildcard `from pyspark.sql.functions import *` in every cell, unlike NB02's explicit imports.

**Markdown issues**
- **No step explains which profiled issue it resolves.** This is the cleansing notebook — the natural place to say "Region had a `NORTH` value; `initcap` standardizes it" — yet each cell is unlabeled beyond the table name. Biggest learning gap here.
- **Cell 4** hard-codes `when(col("ProductID") == 9, lit("Cleaning"))` under the comment "Fix intentional data-quality issue." The comment is good, but a reader won't know *why ProductID 9* (its Category was nulled in NB01) or why the replacement is `Cleaning`. One sentence would make it self-explanatory.
- Fill-value choices are undocumented business decisions: Email → `unknown@verdanova.com`, **Platform → `Instagram`**, SalesTarget → `0`. Imputing a *specific* platform is non-obvious and deserves a one-line rationale. (Documentation only — do **not** change the value.)

**Code-cell organization**
- Excellent. No splitting needed.

**Recommended changes**
- Convert labels to `###` + one sentence tying the transformation to the profiled issue.
- Add a one-line rationale next to the ProductID-9 fix and the imputation fill values.

---

### Notebook 04 — Build Gold Analytical Model

**Current structure:** H1 header (cell 0) → Calendar → Customers → Products → Regions → Sales fact → Marketing Campaign → Sales Target → "Date Format Refine" → "Category Gold table."

**Strengths**
- Contains the **best step-level documentation in the project**: `## Dimension - Products` + a two-sentence purpose (cells 1, 3, 5, 7, 9). **This is the recommended standard.**
- The Sales fact build (cell 10) is clean and readable.
- **Cell 16 has good defensive validation** — it counts invalid date conversions and only overwrites `Sales` if the count is zero. Keep this.

**Inconsistencies (this notebook has the widest spread of any)**
- **Four different Markdown styles in one notebook:**
  1. `## Dimension - Name` + 2 sentences (cells 1–9) — concise, good.
  2. `# Marketing Campaign` + `## Source / Grain / Output Columns` (cell 11) — medium, and jumps to H1.
  3. `# Sales Target` + **seven** subsections incl. "Architectural Note" (cell 13) — very detailed.
  4. Bare bold labels `**Date Format Refine**`, `**Category Gold table**` (cells 15, 17) — back to the terse style.
- Heading level jumps between `##` (dimensions) and `#` (Marketing, Sales Target).
- Import style is mixed *within the notebook*: wildcard `import *` (cells 2, 8, 10) vs explicit `import col, to_date` (cells 12, 14, 16, 18).

**Markdown issues (documentation no longer matches code — needs correction)**
- **Cell 11 "Output Columns" omits `Region`**, but cell 12 selects `Region`. The list is wrong.
- **Cell 13 "Output Columns" and "Transformations" omit `YearMonthKey`**, but cell 14 creates and selects it. The list and the transformation description are incomplete.
- **Cell 0's Business Model names don't match the tables produced:** header says facts `CampaignPerformance` and `SalesTargets`, but the code writes `marketing_campaign` and `SalesTarget`; header lists no `Category` dimension though cell 18 tries to build one.

**Code-cell organization**
- **Cell 15/16 "Date Format Refine" is a late patch** to the `Sales.OrderDate` type, appended after Sales is already built in cell 10. It works, but it reads as a fix-up with no explanation of why it's needed after the fact.
- **Cell 18 "Category Gold table" was never executed** (`execution_count = None`, no outputs) **and references `Sales.Category`, a column cell 10 does not create.** As written it would fail. The Gold-build notebook currently *ends on an unexecuted, likely-broken cell.* (Flag for a decision — do not fix logic in this audit.)

**Recommended changes**
- Adopt the cell 1–9 style everywhere in this notebook; demote the cell 11/13 H1s to `##` and trim to the standard depth.
- **Correct the three documentation-accuracy defects** (Region, YearMonthKey, header table names) — see §6, mandatory.
- Add one sentence to cell 15 explaining why the date refine is needed.
- Decide the fate of cell 18 (complete it, or mark it clearly as incomplete/TODO). Add a closing summary cell to match NB03/NB05.

---

### Notebook 05 — Gold Validation

**Current structure:** Very detailed H1 header (cell 0, 86 lines) → existence check → key integrity → referential integrity → SalesTarget grain → business validation (Sales, Campaign) → Sales-vs-Target → target coverage → Campaign-vs-Sales → hardcoded final summary.

**Strengths**
- **Conceptually the strongest notebook.** The header lays out 7 validation areas, explains *why* (find orphans before building the semantic model), and documents intentional synthetic-data limitations (campaign revenue not reconciling to Sales). Excellent learning value.
- Clean referential-integrity technique (`left_anti` joins, cell 6).
- Good business-rule checks with an explicit tolerance on `LineAmount` (cell 12).

**Inconsistencies**
- **Depth whiplash:** the 86-line header sits above steps labeled with bare bold text (`**Key Integrity**`, `**Inspect Duplicate**`). Same short-vs-long tension as the rest of the set, at its most extreme.
- **Table-name casing is inconsistent in code:** reads `products`, `customers`, `regions`, `salestarget` (lowercase) while NB04 wrote `Products`, `Customers`, `Regions`, `SalesTarget`. Works only because Spark resolves names case-insensitively; still confusing to read.
- Header uses idealized names (`Product`, `Customer`, `Region`, `MarketingCampaign`) that don't match the physical tables (`products`, `customers`, `regions`, `marketing_campaign`).

**Markdown issues**
- **Cells 15 and 16 have no label** (the Sales-vs-Target monthly comparison and the achievement-% summary). They also rely on variables defined in earlier cells (`sales`, `calendar`, `targets`, `spark_sum`), so they only work if the notebook is run in order — worth a note.
- **Cell 22 "Gold Validation Summary" is hardcoded.** It prints `0` for every violation and "All 192 target combinations…" as literal strings, not values computed from the checks above. If the data changed, the summary would silently misreport. For a learning notebook, at minimum it should say these are the observed results of a specific run.

**Code-cell organization**
- Logical and well-scoped. No splitting needed.
- **Reproducibility flag:** cells 15 and 20 group `Sales` by `Category` and ran successfully (`exec = 3` and `8`), which means the executed lakehouse had a `Sales.Category` column — but NB04's committed cell 10 does not produce one. The committed code and the executed state have drifted. (Flag only.)

**Recommended changes**
- Keep the header depth (it's justified once), but standardize the per-step labels to `###` + one sentence.
- Align the header's table names and in-code casing with the actual physical table names.
- Label cells 15–16.
- Make the final summary reflect computed values, or explicitly frame it as "results observed on the latest run."

---

## 4. Cross-Notebook Consistency Issues

These should be standardized across the whole set:

1. **Heading style (highest impact).** Bold-text pseudo-headings (`**Label**`) vs real Markdown headings (`##`/`###`) are mixed both within and across notebooks. Pick one (real headings) everywhere.
2. **Documentation depth.** Ranges from bare labels (NB02/03) to multi-subsection cells (NB04 c13, NB05 header). Converge on the §2 standard.
3. **The "why" / learning narrative.** The Bronze-inject → profile → Silver-fix arc is fully implemented in code but never stated in Markdown. Document the traceability in NB02 (findings) and NB03 (fixes).
4. **Closing summaries.** NB03 and NB05 have them; NB01, NB02, NB04 do not. Add to all.
5. **Import style.** Explicit imports (NB02) vs wildcard `import *` (NB03, parts of NB04) vs mixed (NB04). Pick one convention.
6. **Table naming.** Gold tables mix PascalCase (`Customers`, `Sales`, `SalesTarget`, `Calendar`, `Regions`, `Products`, `Category`) with a lone snake_case `marketing_campaign`, and NB05 refers to several in all-lowercase. *Physically renaming tables is out of scope for this audit* (it touches the semantic model), but the inconsistency should be recorded and at least documented consistently.
7. **File-name convention.** Three styles: `Notebook 01 —` (em dash, U+2014), `Notebook 0N –` (en dash, U+2013), and `Notebook_5_Gold_Validation` (underscores, "5" not "05"). Purely cosmetic to the reader; note that renaming files can affect Fabric item references, so treat as deliberate, low-priority.
8. **Terminology.** "CampaignPerformance" / "Marketing Campaign" / "marketing_campaign" and "SalesTargets" / "Sales Target" / "SalesTarget" are used interchangeably. Settle on one display term per entity.

---

## 5. Recommended Unified Structure

A single template, adapted (not forced) to each notebook's role:

```text
1. Header (H1 + Objective / Inputs / Outputs)   ← detail is welcome here, once
2. Configuration / Imports                       ← where applicable
3. Processing steps                              ← each: ### heading + 1–2 sentence "what layer / what / why"
4. Closing summary                               ← "what this notebook produced or proved"
```

Applied per notebook:

- **NB01 (generate):** Header → per source system: `### <System> — Generate / Land to Files / Load Bronze` → summary of Bronze tables created.
- **NB02 (profile):** Header → `### Profile: <table>` (one line on what's being checked) → **Findings** summary.
- **NB03 (Silver):** Header → `### Silver: <table>` (one line naming the issue fixed) → keep existing summary.
- **NB04 (Gold):** Header → `### Dimension - <name>` / `### Fact - <name>` (already the good style) → add summary.
- **NB05 (validate):** Keep the detailed header → `### <validation area>` per check → keep/repair the summary.

Do **not** force this on cells where it doesn't fit (e.g., the short refine/patch cells) — a single sentence is enough there.

---

## 6. Priority

Deliberately kept tight; only genuine clarity/consistency problems are Mandatory.

### Mandatory
- **M1 — Adopt one documentation standard and one heading style** (§2). Replace bold-text pseudo-headings with real `###` headings and add the one-sentence "what/why" per step, consistently across all five notebooks. This resolves the largest and most pervasive inconsistency.
- **M2 — Make documentation match the code.** Correct the inaccurate statements that will mislead a learner:
  - NB04 cell 11: add `Region` to "Output Columns."
  - NB04 cell 13: add `YearMonthKey` to "Output Columns" and "Transformations."
  - NB04 cell 0 and NB05 cell 0: use the actual table names (`marketing_campaign`, `SalesTarget`, `products`/`customers`/`regions`) or clearly note the physical names.

### Recommended
- **R1 — Document the learning arc.** Add NB02 "Findings" and one-line issue→fix notes in NB03. (Highest learning value.)
- **R2 — Add closing summary cells** to NB01, NB02, NB04 (match NB03/NB05).
- **R3 — Label the unlabeled code cells:** NB02 cells 7–8; NB05 cells 15–16.
- **R4 — Document cross-notebook wiring:** the NB01 → `sales` (Gold) dependency and required run order; note the NB04/NB05 `Sales.Category` drift for awareness.
- **R5 — Resolve NB04 cell 18 (`Category`):** either complete it or mark it clearly as incomplete/TODO so the notebook doesn't end on an unexecuted, broken cell. (Decision only — logic changes are a later phase.)
- **R6 — Pick one import convention** (explicit vs wildcard) across the set.
- **R7 — Make NB05 cell 22 summary** reflect computed values, or explicitly label it as "results from the latest run."

### Optional
- **O1 — Remove/relocate** the NB01 cell-1 note-to-self and fix the nested-bold Markdown in NB01 cell 5.
- **O2 — Standardize file names** (dash/number convention). Cosmetic; note the Fabric-reference caveat.
- **O3 — Align in-code table-name casing** in NB05 with the physical PascalCase names (readability only; behavior is unchanged).
- **O4 — Add a "Section 1 - CRM"** heading in NB01 for symmetry with the other sections.

---

## 7. What Should NOT Be Changed

These are already fine and should remain untouched:

- **The medallion architecture and table lineage** (`bronze_*` → `silver_*` → Gold). The dependency graph is clean and correct.
- **The intentional data-quality injections** in NB01 (null email, `NORTH` casing, duplicates, null category, `drinkware`, null platform). They are the pedagogical core — keep them exactly as they are.
- **All data-generation logic and business rules:** the cost/markup model, seasonality weights, regional/category distributions, the marketing-revenue envelope (60% of actual sales), and `random.seed(42)`. Do not alter values or logic.
- **NB03's per-table cleanse pattern** and its `## Silver Layer Completed` summary — this is the model for the others.
- **NB04's dimension header style** (`## Dimension - <name>` + purpose) — standardize *toward* this, don't remove it.
- **NB04 cell 16's defensive validation** (check invalid dates before overwrite) — good practice, keep.
- **NB05's 7-area validation design** and referential-integrity approach (`left_anti` joins, tolerance on `LineAmount`).
- **The consistent `.option("overwriteSchema", "true")`** on writes.
- **Per-cell re-imports / Spark session re-creation and variable reuse** — harmless in Fabric; not worth "cleaning up."
- **Cell counts / cell boundaries.** No cell needs to be split; the current grouping (config / generate / write / load / validate) is appropriate.

---

**AUDIT COMPLETE — NO PROJECT FILES MODIFIED**
