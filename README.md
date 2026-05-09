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

### Plan

Each chart template will be accompanied by a Power BI Project (`.pbip`) file in its folder. A PBIP is a human-readable, source-control-friendly alternative to the binary `.pbix` format, introduced in Power BI Desktop as a developer-first authoring experience.

**Structure per chart:**

```
change-over-time/
├── line.deneb-template.json        ← original Vega template
├── line.pbip                       ← Power BI project entry point
├── line.Report/
│   ├── definition.pbir
│   ├── report.json
│   └── pages/
│       └── ReportPage/
│           ├── page.json
│           └── visuals/
│               └── DenebVisual/
│                   └── visual.json ← Deneb configured with this template
└── line.SemanticModel/             ← references _data/universal.csv
    ├── definition.pbism
    └── model.bim
```

A **shared semantic model** at `_shared/UniversalData.SemanticModel/` avoids duplicating the model definition across every chart. Each PBIP report references it via relative path.

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
