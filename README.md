# Deneb Vega Templates — PBIP Extension

This repository extends the excellent open-source work of **Andrzej Leszkiewicz** ([@avatorl](https://github.com/avatorl)), whose [Deneb-Vega-Templates](https://github.com/avatorl/Deneb-Vega-Templates) collection provides ready-to-use Vega chart templates for [Deneb](https://deneb-viz.github.io/), the custom visual for Power BI.

All original template files and folder structure are preserved. This fork layers in two things:

1. **A universal sample dataset** — a single, clean CSV that maps to the field requirements of every non-spatial template in the library
2. **Power BI Project (PBIP) files** — one per individual chart template, so each visualisation can be opened directly in Power BI Desktop without any manual wiring

---

## The Original Templates

Andrzej's templates implement the chart taxonomy of the [Financial Times Visual Vocabulary](https://ft.com/vocabulary) — a practitioner's guide to choosing the right chart type for the story you want to tell. Each folder in this repo corresponds to one FT Visual Vocabulary category.

62 templates across 9 categories + 1 advanced template:

| Category | Templates | Count |
|---|---|---|
| [Change over Time](#change-over-time) | Line, Column, Column & Line, Area, Slope, Candlestick | 6 |
| [Correlation](#correlation) | Scatterplot, Bubble Plot, Heatmap, Connected Scatterplot, Column & Line Timeline | 5 |
| [Deviation](#deviation) | Diverging Bar, Back-to-Back Bar, Diverging Stacked Bar, Surplus-Deficit Line | 4 |
| [Distribution](#distribution) | Histogram, Boxplot, Violin, Dot Plot, Dot Strip, Barcode, Population Pyramid | 7 |
| [Flow](#flow) | Waterfall, Sankey, Network | 3 |
| [Magnitude](#magnitude) | Bar, Bar (2 series), Column, Column (2 series), Lollipop, Radar, Marimekko, Bullet, Proportional Symbol, Grouped Symbol, Isotype, Parallel Coordinates | 12 |
| [Part-to-Whole](#part-to-whole) | Pie, Donut, Treemap, Stacked Column, Gridplot, Waterfall, Marimekko, Parliament, Venn, Voronoi | 10 |
| [Ranking](#ranking) | Bar (Ordered), Column (Ordered), Lollipop, Slope, Bump, Dot Strip, Ordered Proportional Symbol | 7 |
| [Spatial](#spatial) | Choropleth, Proportional Symbol, Contour Map, Heat Map, Flow Map, Scaled Cartogram, Equalised Cartogram, Dot Density | 8 |
| [Advanced](#advanced-templates) | HR Sankey | 1 |

---

## The Universal Dataset

### Origin

The sample data is derived from the **EU Superstore** dataset — a fictional retail orders dataset covering 15 European countries across 2015–2018. It is a well-understood, realistic dataset with natural hierarchies that make charts visually interesting.

### What we did

The raw orders data (10,000 line items, 20 columns) was transformed into a compact, generic dataset optimised for Deneb template showcasing:

- **Aggregated** to month × all categorical dimensions, reducing to ~6,100 rows
- **Columns renamed** to generic names that align directly with Deneb template field conventions
- **Discount rate replaced** by Profit Margin as the `Rate` field — discount rate is a row-level attribute that cannot be meaningfully aggregated; margin is meaningful at any grain
- **Margin clamped** to [−1, 1] — a handful of small-revenue, high-loss rows produced extreme outliers

### Schema

| Column | Type | Values / Range | Source |
|---|---|---|---|
| `Date` | date | `2015-01-01` → `2018-12-01` (48 months) | Order Date, truncated to month |
| `L1-Category` | text | Furniture · Office Supplies · Technology | Category |
| `L2-Category` | text | 17 sub-categories (Bookcases, Chairs, Phones…) | Sub-Category |
| `Segment` | text | Consumer · Corporate · Home Office | Segment |
| `L1-Region` | text | Central · North · South | Region (sales territories) |
| `L2-Region` | text | 15 countries (Austria, France, Germany, UK…) | Country |
| `Value` | decimal | 2.96 – 7,958 | SUM(Sales) — always positive |
| `Value 2` | decimal | −3,059 – 3,979 | SUM(Profit) — can go negative |
| `Count` | integer | 1 – 73 | SUM(Quantity) |
| `Rate` | decimal | −1 – 0.5 | Profit Margin = Value 2 / Value |

**Two natural hierarchies** emerge from this schema:

- **Product**: `L1-Category` → `L2-Category`
- **Geographic**: `L1-Region` → `L2-Region`

Any template that requires two categorical dimensions can use either hierarchy, or cross-cut between them (e.g. Category × Region as a heatmap).

**File:** `_data/universal.csv`

### Coverage

The universal dataset covers the field requirements of all non-spatial templates. A small number of charts need a specialist dataset due to their unique data shape:

| Template | Reason | Status |
|---|---|---|
| Candlestick | Requires OHLC columns (Open, High, Low, Close) | Planned |
| Waterfall | Requires Period, Total flag, and Sort order | Planned |
| Population Pyramid | Requires two symmetric value columns (Left, Right) | Planned |
| Parliament Diagram | Requires Index, Party, Seats | Planned |
| Diverging Stacked Bar | Requires Likert-scale Characteristic + Score | Planned |
| Spatial (all) | Requires Lat/Long or Area Codes | Deferred |

---

## Power BI Project Files (PBIP)

### What a PBIP is

A PBIP is a human-readable, source-control-friendly alternative to the binary `.pbix` format. It is a folder of JSON and TMDL files that Power BI Desktop reads directly. Each chart in this library gets its own PBIP, generated automatically by a script from the template file and the universal dataset.

### Actual file structure (derived from manual example)

```
change-over-time/
├── line.deneb-template.json              ← source Vega template (input to generator)
├── line.pbip                             ← entry point; references Report folder
├── line.Report/
│   ├── .platform                         ← Fabric metadata; unique logicalId per PBIP
│   ├── definition.pbir                   ← links Report → SemanticModel by relative path
│   ├── StaticResources/SharedResources/
│   │   └── BaseThemes/CY26SU04.json      ← base PBI theme (static)
│   └── definition/
│       ├── report.json                   ← Deneb visual GUID, theme ref, settings (static)
│       ├── version.json                  ← static
│       └── pages/
│           ├── pages.json                ← page order (static; same page ID reused)
│           └── {pageId}/
│               ├── page.json             ← 1280×720 canvas (static)
│               └── visuals/
│                   └── {visualId}/
│                       └── visual.json   ← GENERATED: field bindings + Vega spec
└── line.SemanticModel/
    ├── .platform                         ← Fabric metadata; unique logicalId per PBIP
    ├── definition.pbism
    ├── diagramLayout.json
    └── definition/
        ├── model.tmdl                    ← static
        ├── database.tmdl                 ← static
        ├── relationships.tmdl            ← static (date table relationship)
        ├── cultures/en-US.tmdl           ← static
        └── tables/
            ├── universal.tmdl            ← CSV path injected at generation time
            ├── DateTableTemplate_*.tmdl  ← static (auto date table)
            └── LocalDateTable_*.tmdl     ← static (auto date table)
```

### Generator architecture

PBIPs are produced by a PowerShell generator script (`_shared/generate-pbips.ps1`) driven by a manifest file (`_shared/pbip-manifest.json`). Static boilerplate lives in `_shared/pbip-scaffold/` and is copied verbatim; only `visual.json` and name tokens are generated dynamically.

#### What changes per template

| File | What changes | How |
|---|---|---|
| `.pbip` | Report folder path string | Name substitution |
| `definition.pbir` | SemanticModel folder path string | Name substitution |
| `.platform` (×2) | `displayName` + `logicalId` | Name substitution + fresh UUID |
| `universal.tmdl` | Absolute path to `_data/universal.csv` | Injected at generation time (see below) |
| `visual.json` | Field projections + Vega spec | Generated from template + manifest |

Everything else is copied unchanged from the scaffold.

#### The manifest (`_shared/pbip-manifest.json`)

Each entry declares which universal columns bind to which template dataset fields. For fields where the template name exactly matches a universal column (`Date`, `L1-Category`, etc.) the generator infers the binding automatically. For aliases (`Category` → `L2-Category`, `Value X` → `Value`) the manifest provides an explicit override.

```json
{
  "change-over-time/line": {
    "fields": {
      "Date":         { "kind": "column",  "source": "Date" },
      "Sum of Value": { "kind": "measure", "source": "Value", "fn": "Sum" }
    },
    "sort": { "field": "Date", "direction": "Ascending" }
  },
  "correlation/scatterplot": {
    "fields": {
      "Category": { "kind": "column",  "source": "L2-Category" },
      "Value X":  { "kind": "measure", "source": "Value",   "fn": "Sum" },
      "Value Y":  { "kind": "measure", "source": "Value 2", "fn": "Sum" }
    },
    "sort": { "field": "Category", "direction": "Ascending" }
  }
}
```

Templates not in the manifest are skipped by the generator with a warning.

#### How `visual.json` is built

1. The Vega spec is read from the template file. Placeholder field names (`__0__`, `__1__`, …) are replaced with the real field names from `usermeta.dataset` (e.g. `__0__` → `"Date"`, `__1__` → `"Sum of Value"`).
2. The result is JSON-encoded as a single-quoted string literal (PBI's format for `jsonSpec`): internal single quotes become `'`, line endings become `\r\n`.
3. The `projections` array is built from the manifest: `kind: "column"` entries become `Column` projections; `kind: "measure"` entries become `Aggregation` projections (Function 0 = Sum).
4. Interactivity flags and `jsonConfig` are taken directly from `usermeta.interactivity` and `usermeta.config`.

#### ID consistency

| ID | Scope | Strategy |
|---|---|---|
| Page name / folder (`{pageId}`) | Internal to one report | Pinned constant — reused across all PBIPs |
| Visual name / folder (`{visualId}`) | Internal to one page | Pinned constant — reused across all PBIPs |
| `logicalId` in `.platform` | Fabric Git integration | Fresh UUID generated per PBIP |
| Column `lineageTag` UUIDs in `universal.tmdl` | Fabric column tracking | Pinned — SemanticModel is identical in every PBIP |
| Relationship UUID in `relationships.tmdl` | Internal | Pinned |

#### Dynamic CSV path

The absolute path to `_data/universal.csv` is injected into `universal.tmdl` at generation time. The generator detects the repo root from its own location (`$PSScriptRoot`) so it works correctly regardless of where the repo is checked out — no manual path editing required. If you open a generated PBIP on a different machine, run `generate-pbips.ps1` once to re-stamp the correct path for that machine.

#### Known fragile points

1. **Spec string encoding** — the Vega spec must be serialised as a single-quoted PBI string literal with exact escaping. A single wrong character produces a silently empty visual. Validate by diffing the generated `visual.json` against the manual reference file before bulk generation.
2. **PBI schema drift** — `report.json`, `page.json`, and `visual.json` reference versioned Microsoft JSON schemas. If Power BI Desktop updates these schemas, the scaffold may need updating. Check after major PBI Desktop releases.

#### Testing sequence

1. Generate PBIP for **line chart** — diff `visual.json` byte-for-byte against `manual-example-line-chart/` reference
2. Open in Power BI Desktop — visual renders, data loads
3. Generate PBIP for **bar chart** — different field types, no sort on date
4. Generate PBIP for **scatterplot** — three fields, manifest alias override
5. Open both in Power BI Desktop
6. If all three pass, generate the remaining templates

Test output goes to `_test-output/` (gitignored). Do not commit generated PBIPs until the test sequence passes.

### Status

| Category | Templates | PBIP Status |
|---|---|---|
| Change over Time | Line, Column, Area, Column & Line, Slope | 📋 Planned |
| Change over Time | Candlestick | ⏸ Awaiting specialist data |
| Correlation | Scatterplot, Bubble, Heatmap, Connected Scatter, Col & Line Timeline | 📋 Planned |
| Deviation | Diverging Bar, Back-to-Back Bar, Surplus-Deficit Line | 📋 Planned |
| Deviation | Diverging Stacked Bar | ⏸ Awaiting specialist data |
| Distribution | Boxplot, Violin, Dot Plot, Dot Strip, Barcode, Histogram | 📋 Planned |
| Distribution | Population Pyramid | ⏸ Awaiting specialist data |
| Flow | Waterfall, Sankey, Network | ⏸ Awaiting specialist data |
| Magnitude | Bar, Bar (2 series), Column, Column (2 series), Lollipop, Radar, Marimekko, Proportional Symbol, Parallel Coordinates | 📋 Planned |
| Magnitude | Bullet, Grouped Symbol, Isotype | ⏸ Awaiting specialist data |
| Part-to-Whole | Pie, Donut, Treemap, Stacked Column, Gridplot, Marimekko, Venn, Voronoi | 📋 Planned |
| Part-to-Whole | Waterfall, Parliament | ⏸ Awaiting specialist data |
| Ranking | All 7 | 📋 Planned |
| Spatial | All 8 | ⏸ Deferred |

---

## Automation Skills (Planned)

The longer-term vision is a set of Claude Code slash-command skills that turn this template library into a front-end toolkit for autonomously generating Power BI reports with Deneb visuals.

### Tier 1 — Theme integration

| Skill | What it does |
|---|---|
| `/apply-pbi-theme` | Maps a PBI theme JSON onto a Deneb template: `dataColors` → Vega `range.category`, `textClasses` → axis/legend/title fonts and sizes, `background` → Vega background. Writes the result into `usermeta.config`. |
| `/extract-theme-tokens` | Parses a PBI theme and prints a clean summary of its design tokens (palette, fonts, sizes) for inspection. |
| `/create-vega-config` | Builds a Vega config block from a palette array, font name, and base size — for when you don't have a full PBI theme. |

### Tier 2 — Template manipulation

| Skill | What it does |
|---|---|
| `/remap-fields` | Replaces `__0__`, `__1__` etc. with real column names throughout a template spec and updates the `dataset` metadata block. |
| `/validate-template` | Checks a template against Deneb 1.9.0 structural requirements before import. |
| `/clone-template` | Copies a template to a new file and optionally remaps fields in one step. |

### Tier 3 — Autonomous generation

| Skill | What it does |
|---|---|
| `/suggest-visual` | Given a dataset schema, recommends chart templates from this library that fit, with rationale. |
| `/generate-pbip` | Scaffolds a full PBIP project for a given template, wired to the universal CSV via the shared semantic model. |
| `/auto-configure` | Given a dataset schema and a target template: remaps fields, applies a theme, and produces a ready-to-open PBIP. Composes the skills above. |

---

## Working with Deneb Templates

Read the official Deneb documentation: https://deneb-viz.github.io/templates

---

## Original Author

All Vega template code is the work of **Andrzej Leszkiewicz**.

- Website: [Power of Business Intelligence](https://powerofbi.org/)
- YouTube: [@power-of-bi](https://www.youtube.com/@power-of-bi)
- Twitter: [@avatorl](https://twitter.com/avatorl)
- LinkedIn: [@avatorl](https://www.linkedin.com/in/avatorl/)

If a template has saved you time on a commercial project, consider sponsoring Andrzej directly via the Sponsor button on his original repository.

---

## Resources

- [Financial Times Visual Vocabulary](https://ft.com/vocabulary)
- [Deneb for Power BI](https://deneb-viz.github.io/)
- [Vega visualization grammar](https://vega.github.io/vega/)
- [Original template gallery](https://powerofbi.org/deneb-vega-templates/)

---

## Template Gallery

### Change over Time

> Show how values evolve across a continuous time axis.

| Template | Description |
|---|---|
| Line | Single or multi-series trend over time |
| Column | Discrete time periods as columns |
| Column & Line | Dual-axis combining column and line series |
| Area | Trend with emphasis on magnitude below the line |
| Slope | Change between exactly two time points |
| Candlestick | OHLC price movement (financial) |

---

### Correlation

> Show the relationship between two or more variables.

| Template | Description |
|---|---|
| Scatterplot | X vs Y relationship across categories |
| Bubble Plot | Scatterplot with a third variable encoded as size |
| Heatmap | Matrix of two categoricals coloured by value |
| Connected Scatterplot | Scatterplot with time-ordered path |
| Column & Line Timeline | Two metrics on a shared time axis |

---

### Deviation

> Show how values differ from a reference point.

| Template | Description |
|---|---|
| Diverging Bar | Bars extending left or right from a zero baseline |
| Back-to-Back Bar | Two groups mirrored on a shared axis |
| Diverging Stacked Bar | Likert-scale style stacked deviation |
| Surplus-Deficit Line | Filled area above/below a reference line |

---

### Distribution

> Show the spread and shape of a dataset.

| Template | Description |
|---|---|
| Histogram | Frequency distribution of a continuous variable |
| Boxplot | Five-number summary across categories |
| Violin | Distribution shape with density estimation |
| Dot Plot | Individual values across two points per category |
| Dot Strip | Individual values scattered along an axis |
| Barcode | Dense strip of marks for large distributions |
| Population Pyramid | Back-to-back distribution for two groups |

---

### Flow

> Show movement, connections, or cumulative change.

| Template | Description |
|---|---|
| Waterfall | Running total with incremental up/down steps |
| Sankey | Weighted flows between nodes |
| Network | Node-link diagram of relationships |

---

### Magnitude

> Compare the size of things.

| Template | Description |
|---|---|
| Bar Chart | Horizontal comparison across categories |
| Bar Chart (2 series) | Side-by-side horizontal bars |
| Column Chart | Vertical comparison across categories |
| Column Chart (2 series) | Side-by-side vertical bars |
| Lollipop | Bar alternative with reduced ink |
| Radar | Multi-axis radial comparison |
| Marimekko | Two-dimensional proportional mosaic |
| Bullet Graph | Performance vs target with qualitative ranges |
| Proportional Symbol | Size-encoded circles per category |
| Grouped Symbol | Icon-based unit chart |
| Isotype | Pictogram / unit chart |
| Parallel Coordinates | Multi-variable profile comparison |

---

### Part-to-Whole

> Show how parts make up a total.

| Template | Description |
|---|---|
| Pie Chart | Classic proportional circle |
| Donut Chart | Pie with centre space for a label |
| Treemap | Hierarchical rectangles sized by value |
| Stacked Column | Segments stacked to show composition |
| Gridplot | Equal-area grid coloured by proportion |
| Waterfall | Cumulative decomposition |
| Marimekko | Two-dimensional proportional mosaic |
| Parliament Diagram | Seat allocation in a hemicycle |
| Venn Diagram | Overlapping set membership |
| Voronoi | Area-divided plane by category |

---

### Ranking

> Show items in order.

| Template | Description |
|---|---|
| Bar Chart (Ordered) | Sorted horizontal bars |
| Column Chart (Ordered) | Sorted vertical bars |
| Lollipop | Sorted lollipop |
| Slope | Rank change between two periods |
| Bump Chart | Rank over time across multiple periods |
| Dot Strip (Ordered) | Ranked individual values |
| Ordered Proportional Symbol | Ranked size-encoded circles |

---

### Spatial

> Show data with a geographic dimension. *(PBIP support deferred.)*

| Template | Description |
|---|---|
| Choropleth | Areas coloured by value |
| Proportional Symbol | Sized circles at geographic locations |
| Contour Map | Density contours over a map |
| Heat Map | Colour intensity at lat/long points |
| Flow Map | Directional flows between areas |
| Scaled Cartogram | Areas scaled by value |
| Equalised Cartogram | Equal-area grid map |
| Dot Density | One dot per unit at geographic location |

---

### Advanced Templates

| Template | Description | Data |
|---|---|---|
| HR Sankey | Employee flow between roles/departments | Specialist CSV included |

---

*© Original Vega template code: Andrzej Leszkiewicz. PBIP extension and dataset: this repository.*
