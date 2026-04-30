---
name: semantic-link-labs-notebook
description: >
  Create, structure, and populate Microsoft Fabric notebooks that use the semantic-link-labs Python library
  to programmatically interact with Fabric items (Semantic Models, Reports, Lakehouses, Warehouses, Workspaces).
  Trigger this skill whenever the user mentions Semantic Link Labs, sempy_labs, Fabric notebooks for Power BI automation,
  model BPA, Vertipaq Analyzer, Direct Lake migration, report analysis in Fabric, TOM (Tabular Object Model) in Fabric,
  or any task involving programmatic management of Power BI semantic models or reports from a notebook.
  Also trigger when the user wants to automate Fabric workspace operations, refresh semantic models via code,
  deploy or backup semantic models, analyze report metadata, or migrate import models to Direct Lake.
---

# Semantic Link Labs in Microsoft Fabric Notebooks

This skill teaches you how to create Microsoft Fabric notebooks that use the `semantic-link-labs` library to
programmatically manage Power BI semantic models, reports, lakehouses, and other Fabric items.

Semantic Link Labs is an open-source Python library (by Michael Kovalsky, hosted on Microsoft's GitHub) that extends
the core `semantic-link` (`sempy`) library with 300+ additional functions. It is designed for use exclusively inside
Microsoft Fabric notebooks and uses the identity of the notebook runner for authentication — no explicit credential
handling is required.

## Table of Contents

1. [Environment & Installation](#1-environment--installation)
2. [Notebook Structure Conventions](#2-notebook-structure-conventions)
3. [Core Imports](#3-core-imports)
4. [Key Subpackages at a Glance](#4-key-subpackages-at-a-glance)
5. [Workspace Context & Parameter Defaults](#5-workspace-context--parameter-defaults)
6. [Semantic Model Analysis](#6-semantic-model-analysis)
7. [Tabular Object Model (TOM) Wrapper](#7-tabular-object-model-tom-wrapper)
8. [Report Inspection & Validation](#8-report-inspection--validation)
9. [Direct Lake Operations](#9-direct-lake-operations)
10. [Semantic Model Lifecycle](#10-semantic-model-lifecycle)
11. [Lakehouse & Workspace Operations](#11-lakehouse--workspace-operations)
12. [Admin Functions](#12-admin-functions)
13. [Common Pitfalls & Error Handling](#13-common-pitfalls--error-handling)
14. [Reference Links](#14-reference-links)

---

## 1. Environment & Installation

### One-time install per session

The first cell of every notebook should install the library. Use the `%pip` magic (not `!pip`) so the package is
available to the running Spark session immediately.

```python
%pip install semantic-link-labs
```

### Persistent install via a Fabric Environment (recommended for teams)

Instead of running `%pip install` in every notebook, add `semantic-link-labs` as a library in a custom Fabric
Environment. Steps:

1. In your Fabric workspace, create a new **Environment** item.
2. Under **Public Libraries**, search for `semantic-link-labs` and add it.
3. Publish the environment.
4. In your notebook's top navigation bar, select this environment from the **Environment** dropdown.

This way every notebook that uses the environment gets the library without a per-session install.

### Language setting

Set the notebook's primary language to **PySpark (Python)**. All `sempy_labs` functions are Python-only.

---

## 2. Notebook Structure Conventions

Organize notebook cells in this order:

1. **Install cell** — `%pip install semantic-link-labs` (or skip if using a Fabric Environment).
2. **Import cell** — all `import` statements.
3. **Parameters cell** — define `dataset`, `workspace`, `lakehouse`, and other variables. Keeping parameters in a
   single cell makes the notebook reusable across workspaces.
4. **Task cells** — one logical task per cell (e.g., run BPA, list pages, export report).
5. **Markdown cells** — use markdown headings above each task cell to describe what it does.

This mirrors the pattern used in the official sample notebooks on the semantic-link-labs GitHub repository.

---

## 3. Core Imports

Import only the subpackages you need. Here is the full roster for reference:

```python
import sempy.fabric as fabric          # Core Semantic Link (pre-installed in Fabric runtime)
import sempy_labs as labs               # Main labs package
import sempy_labs.report as rep         # Report-level functions
import sempy_labs.lakehouse as lake     # Lakehouse-level functions
from sempy_labs.tom import connect_semantic_model   # TOM wrapper (context manager)
from sempy_labs.report import ReportWrapper         # Report inspection object
from sempy_labs import directlake       # Direct Lake utilities
from sempy_labs import admin            # Tenant / workspace admin functions
```

For most tasks you only need:

```python
import sempy_labs as labs
import sempy.fabric as fabric
```

---

## 4. Key Subpackages at a Glance

| Subpackage | Purpose |
|---|---|
| `sempy_labs` | Top-level functions: BPA, Vertipaq Analyzer, refresh, backup/restore, deploy, translate, model size |
| `sempy_labs.tom` | `connect_semantic_model` context manager, TOM wrapper with 60+ helper methods |
| `sempy_labs.report` | `ReportWrapper`, report BPA, rebind, clone, save as PBIP |
| `sempy_labs.directlake` | Generate Direct Lake models, check guardrails, show unsupported objects, update connections |
| `sempy_labs.lakehouse` | Lakehouse table management and shortcuts |
| `sempy_labs.admin` | Tenant-level scanning, workspace access, sensitivity labels, sharing links |
| `sempy_labs.spark` | Custom Spark pool management and workspace Spark settings |
| `sempy_labs.migration` | Import/DirectQuery to Direct Lake migration automation |

---

## 5. Workspace Context & Parameter Defaults

Almost every function accepts a `workspace` parameter. The default behavior (`workspace=None`) resolves like this:

1. If the notebook has an attached lakehouse → uses that lakehouse's workspace.
2. If no lakehouse is attached → uses the workspace the notebook lives in.

Best practice: always define your parameters explicitly in a dedicated cell so the notebook is portable.

```python
dataset   = "Sales_Model"          # Name or UUID of the semantic model
workspace = "Production Workspace"  # Name or UUID; None = current workspace
lakehouse = "Sales_Lakehouse"       # Name or UUID (for functions that save to a lakehouse)
```

You can use `fabric.resolve_workspace_id(workspace)` to convert a name to an ID if needed.

---

## 6. Semantic Model Analysis

### Best Practice Analyzer (BPA)

Scans a semantic model against a set of rules and surfaces modeling issues (unused columns, missing descriptions,
floating-point types, etc.).

```python
import sempy_labs as labs

dataset   = "Sales_Model"
workspace = None

# Basic run — displays an interactive HTML table in the notebook output
labs.run_model_bpa(dataset=dataset, workspace=workspace)

# Extended run — includes Vertipaq Analyzer stats for advanced rules
labs.run_model_bpa(dataset=dataset, workspace=workspace, extended=True)

# Export results to a delta table in the attached lakehouse
labs.run_model_bpa(dataset=dataset, workspace=workspace, export=True)

# Translate rules to another language
labs.run_model_bpa(dataset=dataset, workspace=workspace, language="French")
```

### Vertipaq Analyzer

Shows detailed memory statistics (table sizes, column cardinality, encoding, dictionary sizes).

```python
import sempy_labs as labs

dataset   = "Sales_Model"
workspace = None

# Returns a dict of DataFrames: tables, columns, partitions, relationships
stats = labs.vertipaq_analyzer(dataset=dataset, workspace=workspace)
for name, df in stats.items():
    print(name)
    display(df)

# Export to delta tables in the attached lakehouse
stats = labs.vertipaq_analyzer(dataset=dataset, workspace=workspace, export="table")

# Export to a .zip file in the lakehouse Files section
stats = labs.vertipaq_analyzer(dataset=dataset, workspace=workspace, export="zip")
```

### Semantic model size

```python
import sempy_labs as labs
import sempy.fabric as fabric

dataset   = "Sales_Model"
workspace = None

# Single model
size = labs.get_semantic_model_size(dataset=dataset, workspace=workspace)
print(f"Model size: {size}")

# All non-default models in the workspace
sizes = {}
for _, r in fabric.list_datasets(workspace=workspace, mode="rest").iterrows():
    d_id = r["Dataset Id"]
    if not labs.is_default_semantic_model(dataset=d_id, workspace=workspace):
        sizes[r["Dataset Name"]] = labs.get_semantic_model_size(dataset=d_id, workspace=workspace)
print(sizes)
```

### Semantic model object usage across reports

```python
import sempy_labs as labs

dataset   = "Sales_Model"
workspace = None

usage_df = labs.list_semantic_model_object_report_usage(
    dataset=dataset,
    workspace=workspace,
    include_dependencies=True,
)
display(usage_df)
```

---

## 7. Tabular Object Model (TOM) Wrapper

The `connect_semantic_model` function returns a context manager that wraps the .NET Tabular Object Model.
Use it to read or modify model objects (tables, columns, measures, relationships, RLS, etc.) in-process.

**Important:** XMLA read/write endpoints must be enabled in the workspace settings if you set `readonly=False`.
Changes made with `readonly=False` are saved automatically when the context manager exits.

### Basic read-only connection

```python
from sempy_labs.tom import connect_semantic_model

dataset   = "Sales_Model"
workspace = None

with connect_semantic_model(dataset=dataset, workspace=workspace, readonly=True) as tom:
    # List every table and column
    for t in tom.model.Tables:
        for c in t.Columns:
            print(f"'{t.Name}'[{c.Name}]")
```

### Add a measure (read/write)

```python
from sempy_labs.tom import connect_semantic_model

dataset   = "Sales_Model"
workspace = None

with connect_semantic_model(dataset=dataset, workspace=workspace, readonly=False) as tom:
    tom.add_measure(
        table_name="Sales",
        measure_name="Total Revenue",
        expression="SUM(Sales[Revenue])",
        description="Sum of revenue across all transactions",
    )
```

### Format all DAX expressions

```python
from sempy_labs.tom import connect_semantic_model

dataset   = "Sales_Model"
workspace = None

with connect_semantic_model(dataset=dataset, workspace=workspace, readonly=False) as tom:
    tom.format_dax()  # formats measures, calc columns, calc items, RLS, calc tables
```

### Auto-generate measure descriptions with an LLM

Requires an F64 or higher capacity.

```python
from sempy_labs.tom import connect_semantic_model

dataset   = "Sales_Model"
workspace = None

with connect_semantic_model(dataset=dataset, workspace=workspace, readonly=True) as tom:
    descriptions_df = tom.generate_measure_descriptions()
display(descriptions_df)
```

### Read Vertipaq stats via TOM annotations

```python
from sempy_labs.tom import connect_semantic_model

dataset   = "Sales_Model"
workspace = None

with connect_semantic_model(dataset=dataset, workspace=workspace, readonly=True) as tom:
    tom.set_vertipaq_annotations()  # reads stats into memory without writing if readonly=True
    for t in tom.model.Tables:
        print(f"{t.Name} : {tom.total_size(object=t)} bytes")
```

### Retrieve the .bim file

```python
from sempy_labs.tom import connect_semantic_model

dataset   = "Sales_Model"
workspace = None

with connect_semantic_model(dataset=dataset, workspace=workspace, readonly=True) as tom:
    bim = tom.get_bim()
    print(bim)
```

---

## 8. Report Inspection & Validation

### ReportWrapper (requires PBIR format)

`ReportWrapper` gives programmatic access to a Power BI report's internal structure. It only works with reports
saved in the modern PBIR format (not legacy .pbix). If the report is in legacy format, `ReportWrapper` will
raise an error — wrap initialization in `try/except`.

```python
from sempy_labs.report import ReportWrapper

report_name = "Sales Dashboard"
workspace   = None

try:
    rpt = ReportWrapper(report=report_name, workspace=workspace)
except Exception as e:
    print(f"Report may not be in PBIR format: {e}")
```

### List pages, visuals, filters, bookmarks

```python
from sempy_labs.report import ReportWrapper

report_name = "Sales Dashboard"
workspace   = None

rpt = ReportWrapper(report=report_name, workspace=workspace)

# Pages
display(rpt.list_pages())

# Visuals across all pages
display(rpt.list_visuals())

# Filters
display(rpt.list_filters())

# Bookmarks
display(rpt.list_bookmarks())
```

### Find broken visuals

Broken visuals reference semantic model objects that no longer exist.

```python
from sempy_labs.report import ReportWrapper

report_name = "Sales Dashboard"
workspace   = None

rpt = ReportWrapper(report=report_name, workspace=workspace)
objects_df = rpt.list_semantic_model_objects(extended=True)

# Rows where 'Valid Semantic Model Object' is False are broken references
broken = objects_df[objects_df["Valid Semantic Model Object"] == False]
display(broken)
```

### Report Best Practice Analyzer

```python
import sempy_labs.report as rep

report_name = "Sales Dashboard"
workspace   = None

rep.run_report_bpa(report=report_name, workspace=workspace)

# Export results to delta table
rep.run_report_bpa(report=report_name, workspace=workspace, export=True)
```

### Rebind a report to a different semantic model

```python
import sempy_labs.report as rep

rep.rebind_report(
    report="Sales Dashboard",
    dataset="New_Sales_Model",
    report_workspace="Reports Workspace",
    dataset_workspace="Models Workspace",
)
```

### Save a report as .pbip (for version control or CI/CD)

```python
import sempy_labs.report as rep

rep.save_report_as_pbip(
    report="Sales Dashboard",
    workspace=None,
    thick_report=True,    # include the semantic model definition
    live_connect=True,    # maintain live connection to the service
    lakehouse="My_Lakehouse",
)
# The .pbip files appear in the lakehouse's Files section.
```

---

## 9. Direct Lake Operations

### Generate a Direct Lake semantic model from lakehouse tables

```python
from sempy_labs import directlake

directlake.generate_direct_lake_semantic_model(
    dataset="New_DL_Model",
    lakehouse_tables=["Sales", "Products", "Calendar"],
    workspace=None,
    lakehouse="Sales_Lakehouse",
    overwrite=False,
    refresh=True,
)
```

### Check Direct Lake guardrails for your SKU

```python
from sempy_labs import directlake

sku = labs.get_sku_size(workspace=None)
guardrails = directlake.get_direct_lake_guardrails(sku_size=sku)
display(guardrails)
```

### Show objects unsupported by Direct Lake

```python
from sempy_labs import directlake

tables_df, columns_df, measures_df = directlake.show_unsupported_direct_lake_objects(
    dataset="Sales_Model",
    workspace=None,
)
display(tables_df)
display(columns_df)
```

### Update a Direct Lake model's source connection

```python
from sempy_labs import directlake

directlake.update_direct_lake_model_connection(
    dataset="Sales_DL_Model",
    workspace=None,
    source="New_Lakehouse",
    source_type="Lakehouse",
)
```

---

## 10. Semantic Model Lifecycle

### Refresh

```python
import sempy_labs as labs

dataset   = "Sales_Model"
workspace = None

# Full refresh
labs.refresh_semantic_model(dataset=dataset, workspace=workspace)

# Refresh specific tables only
labs.refresh_semantic_model(dataset=dataset, workspace=workspace, tables=["Sales", "Products"])

# Visual real-time progress
labs.refresh_semantic_model(dataset=dataset, workspace=workspace, visualize=True)
```

### Backup and Restore

```python
import sempy_labs as labs

# Backup
labs.backup_semantic_model(
    dataset="Sales_Model",
    file_path="Sales_Model_backup.abf",
    allow_overwrite=True,
    apply_compression=True,
    workspace=None,
)

# Restore
labs.restore_semantic_model(
    dataset="Sales_Model_Restored",
    file_path="Sales_Model_backup.abf",
    allow_overwrite=True,
    ignore_incompatibilities=True,
    workspace=None,
    force_restore=True,
)
```

### Deploy to another workspace

```python
import sempy_labs as labs

labs.deploy_semantic_model(
    source_dataset="Sales_Model",
    source_workspace="Dev Workspace",
    target_dataset="Sales_Model",
    target_workspace="Prod Workspace",
    refresh_target_dataset=False,
    overwrite=True,
)
```

### Translate model metadata

```python
import sempy_labs as labs

labs.translate_semantic_model(
    dataset="Sales_Model",
    workspace=None,
    languages=["French", "German", "Japanese"],
    exclude_characters="_",
)
```

### Create a semantic model from a .bim file

```python
import sempy_labs as labs

bim_dict = { ... }  # a parsed Model.bim dictionary

labs.create_semantic_model_from_bim(
    dataset="My_New_Model",
    bim_file=bim_dict,
    workspace=None,
)
```

---

## 11. Lakehouse & Workspace Operations

### List lakehouses and their tables

```python
import sempy_labs.lakehouse as lake

lakehouses = lake.list_lakehouses(workspace=None)
display(lakehouses)

tables = lake.get_lakehouse_tables(lakehouse="Sales_Lakehouse", workspace=None)
display(tables)
```

### List all items in a workspace

```python
import sempy.fabric as fabric

items = fabric.list_items(workspace=None)
display(items)
```

### Workspace role assignments

```python
from sempy_labs import admin

access_df = admin.list_workspace_access_details(workspace=None)
display(access_df)
```

---

## 12. Admin Functions

Admin functions provide tenant-level visibility. The running user must have appropriate admin permissions.

```python
from sempy_labs import admin

# List all workspaces the calling user can see
workspaces = admin.list_workspaces()
display(workspaces)
```

---

## 13. Common Pitfalls & Error Handling

### ReportWrapper fails on legacy reports

`ReportWrapper` only works with reports saved in PBIR format. If the report is a legacy `.pbix`-sourced report,
initialization will throw an error. Always wrap it:

```python
try:
    rpt = ReportWrapper(report=report_name, workspace=workspace)
except Exception as e:
    print(f"Cannot inspect report — it may be in legacy format: {e}")
```

### XMLA read/write not enabled

If you use `connect_semantic_model(..., readonly=False)` and XMLA read/write is not enabled on the workspace,
you will get an authentication or connection error. To fix:

- Go to **Workspace Settings → General → Data model settings**.
- Enable **Users can edit data models in the Power BI service**.

### Spark session must be active

Functions that access the data plane (e.g., reading lakehouse tables, exporting to delta) require an active
Spark session. Simply running any cell in a PySpark notebook starts the session. If the session has timed out,
run a trivial cell like `print("ok")` to restart it.

### Lakehouse must be attached for default paths

When exporting to delta tables (`export=True` or `export="table"`), the function writes to the lakehouse
attached to the notebook. If no lakehouse is attached, the function will fail. Attach a lakehouse via the
notebook's left-side explorer panel before running export functions.

### connect_semantic_model is a context manager

Always use `with connect_semantic_model(...) as tom:` — never instantiate `TOMWrapper` directly. The context
manager handles session setup, .NET interop initialization, and saving changes on exit (for read/write mode).

### Rate limits on large workspaces

When iterating over many semantic models (e.g., checking sizes for every model in a workspace), add brief
pauses between calls to avoid throttling:

```python
import time
for _, r in dfD.iterrows():
    size = labs.get_semantic_model_size(dataset=r["Dataset Id"], workspace=workspace)
    time.sleep(0.5)
```

---

## 14. Reference Links

| Resource | URL |
|---|---|
| Semantic Link Labs GitHub (source + README) | https://github.com/microsoft/semantic-link-labs |
| API Reference Docs (ReadTheDocs) | https://semantic-link-labs.readthedocs.io/en/stable/ |
| PyPI Package | https://pypi.org/project/semantic-link-labs/ |
| Official Sample Notebooks | https://github.com/microsoft/semantic-link-labs/tree/main/notebooks |
| Wiki Code Examples | https://github.com/microsoft/semantic-link-labs/wiki/Code-Examples |
| Fabric Notebook Documentation | https://learn.microsoft.com/en-us/fabric/data-engineering/author-execute-notebook |
| Community Walkthrough (Musili Adebayo) | https://community.fabric.microsoft.com/t5/Notebook-Gallery/How-to-Use-Semantic-Link-Labs-in-Microsoft-Fabric-Notebook/td-p/4803810 |

### Official sample notebooks on GitHub

These Jupyter notebooks can be imported directly into a Fabric workspace:

- **Model Optimization.ipynb** — BPA, Vertipaq Analyzer, translations, DAX formatting
- **Report Analysis.ipynb** — ReportWrapper, broken visuals, report BPA
- **Migration to Direct Lake.ipynb** — full import-to-Direct-Lake migration workflow
- **Semantic Model Management.ipynb** — backup, restore, deploy, refresh
- **Semantic Model Refresh.ipynb** — advanced refresh with partitions and visualization
- **Best Practice Analyzer Report.ipynb** — BPA at scale across workspaces
- **Capacity Migration.ipynb** — P-SKU to F-SKU capacity migration
- **Delta Analyzer.ipynb** — analyze delta table health
- **Service Principal.ipynb** — service principal authentication patterns
- **SQL.ipynb** — SQL endpoint and warehouse operations
- **Query Scale Out.ipynb** — QPU read replica management
