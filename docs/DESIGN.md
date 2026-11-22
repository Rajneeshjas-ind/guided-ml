# guided-ml design notes

Each stage is designed as a **step card** before any code is written. A card is agreed, cross-checked from six expert angles (ML engineer, statistician, data-quality engineer, UX designer, software engineer, hiring manager), challenged once by a devil's advocate, and only then implemented.

## How every stage works

- **One shared session** carries the data, every decision and the log through all stages.
- **Same three steps in every stage:** `analyze()` returns suggestions with reasons; `apply(decisions)` makes the changes; `check()` raises an error or warning if something broke.
- **The original file is never modified.** Every change is logged so it can be replayed or undone.
- **Destructive actions need confirmation:** dropping rows or columns, changing the target.
- **Before the train/test split, only structural checks are allowed.** Distributions, outlier limits, category frequencies, target relationships and "how predictive is this column" wait until after the split (Stage 3).

---

## Stage 0: Load + data quality

**Goal:** read the file correctly and fix structural problems before anything else, so every later stage works on trustworthy data.

**Input:** a file path (CSV, Excel or Parquet; sheet name for Excel).

| Step | What the tool does | What the user decides |
|---|---|---|
| 0.1 Missing-value strings | Reads everything as text first, then counts tokens that often mean "missing" (`""`, `NA`, `N/A`, `null`, `None`, `NaN`, `?`, `-`) per column | Which tokens mean missing in which column. Example: in Ames Housing, `NA` in `PoolQC` means "no pool", not missing. |
| 0.2 Data types | Suggests a type per column: number, number stored as text (`1,200`, `$5`), date, yes/no, category, free text, ID-like | Accept or change each conversion |
| 0.3 Basic info (structural) | Rows, columns, missing % per column, unique values per column, constant columns, ID-like columns, high-cardinality text | Nothing to decide; information only |
| 0.4 Exact duplicates | Finds rows that are identical on every column except ID-like columns; shows the count and examples | Remove or keep. Identical rows can be genuine, e.g. the same customer buying twice. |

**Outputs**
- A cleaned working copy of the data in the session (original untouched)
- `schema.json`: for each column, the original type, the chosen type and its missing-value tokens
- Decision-log entries for every choice

**Automatic checks**
- File exists, is readable and is not empty
- Duplicate or blank column names are reported
- A type conversion that creates new blanks is reported with the count (e.g. "12 values could not be converted")
- Row counts before and after duplicate removal are logged

**Expert notes**
- *Data-quality engineer:* step 0.1 must come before type detection, or `NA` strings silently turn a real category into missing data.
- *ML engineer:* removing exact duplicates before the split prevents the same row landing in both train and test.
- *UX designer:* show a five-row preview beside every decision, so the user sees what the change does.

**Devil's advocate:** "Why not let pandas guess everything?" Because pandas' defaults turn `NA` into missing and can misread codes like ZIP codes as numbers. Those mistakes are invisible until a model behaves strangely.

**Done when:** a file loads, every column has a confirmed type, duplicates are resolved, and `schema.json` plus the log are written.

---

## Stage 1: Frame the problem

**Goal:** the user states what the model is for before any analysis. The answers become machine-readable settings that later stages use automatically.

**All six questions are required.**

| # | Question | Example | Used later for |
|---|---|---|---|
| 1 | What decision will this model support? | "Decide which customers get a retention offer" | Report wording |
| 2 | Which column is the target? (picked from the columns) | `SalePrice`, `Churn` | Every stage |
| 3 | Regression or classification? (suggested from the target) | `Churn` (yes/no) → classification | Models and metrics |
| 4 | One row = one what? | one house, one customer, one transaction | Split: if one customer has many rows, the tool asks for the customer ID column and uses a group split |
| 5 | When is the prediction made? | "At signup", "the month before renewal" | Leakage screen: columns not known at that moment are flagged |
| 6 | Which error costs more? | Classification: a missed churner or a wasted offer? Regression: over- or under-predicting? | Choice of metric |

**"I'm not sure" on Q5** counts as an answer. The tool explains why the question matters, and every column is flagged for manual review in Stage 2.

**How Q6 becomes a metric**

| Situation | Suggested metric |
|---|---|
| Classification, balanced classes, equal costs | Accuracy and ROC-AUC |
| Classification, rare positive class or missed positives costly | PR-AUC, recall, F1 |
| Classification, false alarms costly | Precision |
| Regression, errors equally bad | RMSE (big misses weigh more) or MAE (every unit of error weighs the same) |
| Regression, one direction worse | MAE plus a note on asymmetric (quantile) loss |

**Output:** `frame.json` with `decision`, `target`, `task_type`, `positive_class` (classification), `row_unit`, `group_column` (if needed), `prediction_time`, `error_cost`, `metric`.

**Automatic checks**
- The target exists and is not ID-like
- Rows with a missing target are counted and, with confirmation, removed
- Classification targets have at least two classes; class counts are shown so the split can be stratified
- If the user's task type conflicts with the suggestion (e.g. a numeric target with only two values), the tool asks again
- If a date column exists, a warning explains why a random split may leak the future into training

**Expert notes**
- *ML engineer:* Q4 and Q5 are the most skipped questions in beginner projects and the most important; they drive the split and the leakage screen.
- *Statistician:* the metric is chosen here, before any model is trained, so results cannot be cherry-picked later.
- *Hiring manager:* no ML library asks these questions. This stage is guided-ml's clearest differentiator and should be the README screenshot.

**Devil's advocate:** six required questions may feel like a form. Mitigation: every question shows an example from a demo dataset, and answers can be edited later.

**Done when:** `frame.json` is written with all six answers and the chosen metric.

---

## Decision log

| Date | Decision | Why |
|---|---|---|
| 2026-11-22 | Load and check data quality before framing (Stage 0 before Stage 1) | Users need to see the columns and fix structural problems before choosing a target |
| 2026-11-22 | Only structural checks before the split | Statistics computed on the full data leak test information into decisions |
| 2026-11-22 | Exact duplicates are shown and confirmed, never removed silently | Identical rows can be genuine in transaction data |
| 2026-11-22 | All six framing questions required; "not sure" allowed for prediction time | Framing drives split, leakage screen and metric; uncertainty is flagged rather than blocked |
