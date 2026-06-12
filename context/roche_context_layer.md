# Roche Context Layer — Configuration Reference
# D2D · Infrastructure · Standalone service
# All agents query this layer. No agent owns it.
# Version: 1.0 | Scenario: DA Sense Targets & Benchmarks

---

## WHAT THIS IS

The Roche Context Layer is the infrastructure that makes D2D different from generic AI.
It is not an agent. It is a queryable knowledge service that all skills pull from.

Every answer to a PO question about DA Sense, every data constraint surfaced in analysis,
every cross-product suggestion — all come from this layer.

Without this layer populated correctly, all agents produce generic output.
With it populated correctly, all agents produce Roche-specific output.

**Build this first. Everything else depends on it.**

---

## STRUCTURE

```
roche_context_layer/
  products/
    da_sense/
      business_performance/
        kpi_catalogue.md          # All KPI definitions, data sources, calculation rules
        data_architecture.md      # Source systems, schemas, known constraints
        ui_layouts.md             # Cross Portfolio, charts, filter patterns
        existing_features.md      # Features already built (T0 forecast, patient share, etc.)
        existing_user_stories.md  # Current stories (for consistency reference)
        api_contracts.md          # Endpoint definitions, request/response schemas
        known_constraints.md      # CES non-aggregatability, NPS lag, simple average flag
        data_quality_flags.md     # Null rates, collection gaps, methodology variances
    ilab/
      feature_index.md            # What iLab has built (risk scores, threshold alerts)
    ces/
      feature_index.md
    ai_captain/
      feature_index.md
  cross_product/
    feature_index.md              # The differentiator: {feature: [products that have it]}
  industry/
    pharma_targets_benchmarks.md  # How Veeva, IQVIA, Tableau handle goals and RAG
    BI_patterns.md                # Goal lines, reference bands, threshold bands
  zs_standards/
    story_templates.md
    delivery_standards.md
    acceptance_criteria_patterns.md
```

---

## CONTENTS TO POPULATE

### da_sense/business_performance/kpi_catalogue.md

Populate with all 10 KPIs from master_context_document:
- KPI name, definition, data source, endpoint, subheader, footnote
- Backend calculation rules (aggregation logic, top 6 logic, filter logic)
- Frontend display rules (chart types, axis labels, "others" handling)
- Visualisation settings (filters, defaults, options)

Key entries for Targets & Benchmarks scenario:
- Sales Trends: monthly_sales_value_chf, fiscal_ym, top_country/top6product flags
- Sales vs Forecast: T0 forecast reference line ALREADY EXISTS — dual reference risk
- YoY Growth: fiscal quarter completeness rules, 3rd working day rule
- Patient Share: flat file source, non-standard methodology per affiliate
- NPS by Brand: IQVIA CD+, quarterly, simple average aggregation FLAGGED AS INCORRECT
- Share of Voice: same IQVIA constraints as NPS

### da_sense/business_performance/data_architecture.md

Source systems:
- SAP → SDP Snowflake: gtm_glo_int, gtm_glo_src schemas; info_mart for Business Impact KPIs
- IQVIA CD/CD+: fct_channel_performance_metric_latest for Customer Impact KPIs
- CES (Customer Engagement Suite): Coverage, Frequency, Email, Website Traffic

**Critical for Targets scenario — MUST be in this file:**
- No write-back layer exists. All flows are read-only from source systems.
- Any user-stored data (targets, preferences, notes) requires net-new persistence architecture.
- DA Sense has no user identity layer at the backend (verify with engineering).

Known data quality flags:
- NPS/SoV: simple average across countries — flagged as mathematically incorrect
- NPS: 120-day collection lag. Data labelled Q2 may reflect Q4 prior year.
- Patient Share: methodology varies — RxDynamics (Canada), IQVIA MIDAS (Ophtha), Affiliate
  Input Tracker (multiple DAs). Not apples-to-apples at DA level.
- US and CN: not yet automated for Channel Performance KPIs.
- CES KPIs: explicitly non-aggregatable by time, disease, or product.

### da_sense/business_performance/existing_features.md

**Sales vs. Forecast — T0 reference line:**
The Sales vs. Forecast KPI already renders a T0 annual forecast as a horizontal reference line
with percentage attainment displayed. This is a built, shipped feature.
Any new "goal" or "target" feature that also renders a reference line on the Sales chart creates
a dual-reference conflict. Design decision required before implementation.

**Patient Share — competitor data from flat file:**
Competitor patient share data comes from a flat file tracker, not CD+ basket.
This is a known architectural decision and relevant context for any feature touching Patient Share.

**Cross Portfolio table:**
Summary table on the most-visited page. Shows one row per DA with KPI columns.
Currently: no RAG status, no goal lines, no user-defined reference points.
This is the primary target for the Targets & Benchmarks V1 scope.

### cross_product/feature_index.md

```json
{
  "risk_score_display": ["iLab"],
  "threshold_alert": ["iLab"],
  "goal_line_chart": [],
  "rag_status_indicator": [],
  "user_defined_targets": [],
  "write_back_storage": [],
  "chatbot": ["DSNs Ask AI", "AI Captain"],
  "filter_panel_multi_select": ["DA Sense", "CES"],
  "kpi_comparison_table": ["DA Sense", "CES"],
  "competitor_benchmarking": ["DA Sense (NPS, SoV)"]
}
```

Note: goal_line_chart and rag_status_indicator are both empty arrays — this feature would be
a first in Roche internal tools. No reuse opportunity. New pattern to define.
iLab's risk score display pattern is the closest analogue — reference for UI design approach.

### industry/pharma_targets_benchmarks.md

Populate with:
- Veeva Align: inline goal lines in charts, colored status pills in summary tables
- IQVIA analytics platforms: threshold bands in KPI dashboards
- Salesforce Health Cloud: target tracking with RAG in account health views
- Tableau / Power BI / Looker: goal line functionality, reference bands, deviation charts

Best practices to include:
- Targets need a polarity attribute (high = good vs. low = good) to determine RAG direction
- Rank targets require different comparison logic than absolute/percentage targets
- KPI-specific yellow-zone thresholds more meaningful than universal cutoffs
- Goal lines in time-series charts provide trajectory context beyond single RAG status
- Pharma commercial best practice: centrally-aligned targets (from Business Plan) more trusted
  than individually-set ones, but pragmatic tools start with user-defined and migrate to central
- Aggregating percentage-based targets across geographies: weighted average by patient pool
  or revenue is more defensible than simple average

---

## QUERY INTERFACE FOR SKILLS

Skills query the context layer using structured lookups:

```
query_context(
  product: "da_sense_business_performance",
  context_type: "kpi_constraints",
  kpi: "nps_brand",
  fields: ["aggregation_method", "data_lag", "known_flags"]
)
→ returns: { aggregation_method: "simple_average", data_lag_days: 120, flags: ["mathematically_imprecise"] }
```

```
query_context(
  context_type: "cross_product_feature",
  feature: "rag_status_indicator"
)
→ returns: { products_with_feature: [], closest_analogue: "iLab risk score display" }
```

```
query_context(
  context_type: "architecture",
  product: "da_sense",
  query: "write_back_capability"
)
→ returns: { exists: false, notes: "All flows read-only. No persistence layer for user data." }
```

---

## POPULATION PRIORITY ORDER

For the PoC (DA Sense Targets & Benchmarks scenario), populate in this order:

1. **da_sense/business_performance/data_architecture.md** — especially the write-back constraint
2. **da_sense/business_performance/kpi_catalogue.md** — especially NPS, SoV, Sales vs Forecast
3. **da_sense/business_performance/existing_features.md** — T0 forecast reference line
4. **da_sense/business_performance/known_constraints.md** — CES, NPS lag, simple average
5. **cross_product/feature_index.md** — RAG and goal line both empty
6. **industry/pharma_targets_benchmarks.md** — Veeva, Tableau patterns
7. **zs_standards/** — story templates and delivery standards

Items 1–4 are blockers. Items 5–7 enhance quality but do not block the scenario from running.

---

## PoC vs. NORTH STAR

PoC (this hackathon):
- Text-based documents in the above structure
- Populated manually from master_context_document and scenario reference
- Queried via prompt injection into each skill's system prompt
- Figma snapshots described in text (not rendered)

North Star (post-hackathon):
- Live data dictionaries queried via API
- Full codebase + Figma API integration
- Automated null rate and data quality flag extraction from SDP Snowflake
- Cross-product feature index maintained as a live registry
- Real-time API contract retrieval from DA Sense backend
