# skill: feasibility_structurer
# D2D · Act 2 · Stage 4
# Takes unresolved technical questions from clarifications + impact analysis.
# Formats them as a structured discussion agenda for the dev team.
# Not a list of worries — a facilitation tool the BA can run in a meeting.

---

## TRIGGER
Invoked after edge_case_surfacer (deep mode) completes.
Params: { clarification_output, impact_output, edge_case_output }

---

## SYSTEM PROMPT

You are preparing a technical feasibility discussion agenda for the development team.
Your job: take every unresolved question from the BA analysis phase and structure it
as a discussion item with context, options, and a named decision owner.

The output should be usable as a meeting agenda without further editing.
Each item should be self-contained — the dev team should not need to read the full
requirements to understand what is being asked.

---

## AGENDA ITEMS (Targets & Benchmarks scenario)

FQ1 — Write-back architecture pattern
Context: DA Sense has no write-back layer today. We need to store user-defined targets.
Question: What is the preferred write-back pattern?
Options:
  (a) Dedicated microservice — most flexible, highest maintenance cost
  (b) Snowflake write-back table — stays within existing data infrastructure,
      may have latency implications
  (c) External lightweight datastore (Postgres/DynamoDB) — fast, separate from analytics layer
Decision owner: Platform Engineering
Blocking: All data engineering and backend tickets

FQ2 — User identity availability
Context: Storing user-specific targets requires a user_id in every target-setting API call.
Question: Does the current DA Sense session token include a user_id the backend receives?
If not, what is the path to adding it within the current auth framework?
Decision owner: Backend / Auth team
Blocking: BE-01, BE-05 (user identity layer, team sharing logic)

FQ3 — Cross Portfolio query performance
Context: Joining a target table to the Cross Portfolio KPI query adds a lookup per KPI per row.
Question: What is the acceptable latency increase on Cross Portfolio page load?
Is caching a target-lookup layer feasible?
Decision owner: Backend
Blocking: BE-04 (Cross Portfolio query update)

FQ4 — FX conversion for absolute Sales targets
Context: User sets a Sales target in CHF. Underlying SAP data for Germany comes in EUR,
converted at CES A25 rates.
Question: Where does the conversion happen for target storage and comparison?
Options:
  (a) Store target in CHF — conversion at write time
  (b) Store in local currency — conversion at read time against same CES A25 rate
Decision owner: Data Engineering + Backend
Blocking: DE-03 (FX handling ticket)

FQ5 — Global target proration to country level
Context: If a global target is set for a brand, should country-level views automatically
inherit a prorated version?
Question: What is the proration logic? Historical revenue share? Equal split? User-defined?
Decision owner: PO (business logic decision) + Backend (implementation)
Blocking: BE-06 (default + inheritance logic)

---

## OUTPUT FORMAT

```
{
  "feasibility_id": "05d_{feature}_{product_short}",
  "agenda_items": [
    {
      "id": "FQ1",
      "title": string,
      "context": string,
      "question": string,
      "options": [string] | null,
      "decision_owner": string,
      "blocking_tickets": [string],
      "estimated_discussion_time_minutes": integer
    }
  ],
  "total_items": integer,
  "estimated_total_time_minutes": integer,
  "suggested_attendees": [string]
}
```

---
---

# skill: spec_drafter_stories
# D2D · Act 2 · Stage 5 · Call 1 of 3
# Generates user stories with acceptance criteria.
# GENERATIVE ONLY — no analysis. Analysis is already complete.
# Stories follow ZS story template. Every story links to a validated requirement and source_chain.
# Gate: all BREAKING changes must have confirmed resolution strategies before this runs.

---

## TRIGGER
First of three sequential spec_drafter calls.
Gate: breaking_changes_flagged = true AND all BREAKING items have resolution strategies confirmed.
Params: { validated_requirements, clarification_output, impact_output, edge_case_output }

---

## SYSTEM PROMPT

You are a senior BA at Roche writing user stories for DA Sense Business Performance.
Follow the ZS story template exactly: "As a [persona], I can [action] so that [outcome]."
Every acceptance criterion must be testable. Every edge case must reference the data source.

Use the personas from the validated requirements: CLT Lead, Country I&A Lead, Global Brand Lead.
Stories must be consistent with the validated requirements — do not introduce new scope.
Do not include analysis, commentary, or alternatives. Just the stories.

---

## STORY TEMPLATE

```
Story ID: US_N
Title: [Short descriptive title]
Story: As a [persona], I can [action] so that [outcome].
Acceptance criteria:
  - AC1: [testable criterion]
  - AC2: [testable criterion]
  - AC3: [edge case handling criterion]
Linked logic rule: LR-N
Linked data: [table or column reference]
Source chain: req_N → agent_01_turn_N
Priority: must | should | could
```

---

## STORIES TO GENERATE (Targets & Benchmarks scenario)

US01 — Set Target panel
As a CLT Lead, I can open the Set Target panel from any KPI cell in the Cross Portfolio table
so that I can define a target without leaving my current view.
AC: Panel opens inline; pre-populates KPI, DA, and time period from current filter context;
can be saved or cancelled without navigating away.
Linked logic: LR-01 | Linked data: bp_targets table | Priority: must

US02 — RAG status indicator in table
As any user, I see a RAG status indicator next to each targeted KPI in the Cross Portfolio table
so that I immediately understand if performance is on or off track.
AC: Indicator is green/yellow/red per threshold rules; tooltip on hover shows target value,
current value, deviation, and threshold definition; indicator is grey when no target is set.
Linked logic: LR-02 | Linked data: bp_targets, KPI query result | Priority: must

US03 — Goal line in trend charts
As a user viewing a Sales or Growth trend chart, I see a dashed goal line at my target value
so that I can assess trajectory toward the target over time.
AC: Goal line renders at correct Y-axis position; labelled "Target"; appears only when a target
exists; user goal is primary dashed line; T0 forecast rendered as secondary lighter reference;
legend clearly distinguishes the two.
Linked logic: LR-03 | Linked data: bp_targets | Priority: must

US04 — Smart defaults when no target is set
As a user, when no target is set for a KPI, the system shows a smart default so that I never
see an empty or broken state on first use.
AC: Sales vs Forecast default = T0 annual forecast × (days elapsed / 365); YoY Growth default
= annual forecast-based; NPS default = above competitor average where data available;
if fewer than 2 competitors: show "Insufficient competitor data" — no default applied.
Linked logic: LR-04 | Linked data: computed at read time, not stored | Priority: must

US05 — Yellow-zone threshold configuration
As a user, I can configure the yellow-zone threshold when setting a target so that I can
calibrate sensitivity to my KPI's natural variance.
AC: Yellow threshold field in Set Target panel; defaults to 90% of target for positive-polarity
KPIs; stored per target record in bp_targets; visible in hover tooltip.
Linked logic: LR-02 | Linked data: bp_targets.yellow_threshold | Priority: should

US06 — Data staleness warning for IQVIA KPIs
As a user, when NPS or SoV data is more than one full reporting cycle old, I see a data
staleness warning alongside the RAG indicator so that I do not make decisions on outdated comparisons.
AC: Warning icon with tooltip "Data from [date] — IQVIA updates quarterly"; no warning
for SAP data (monthly); staleness threshold = 90 days for IQVIA, 35 days for CES, 5 business
days for SAP.
Linked logic: LR-05 | Linked data: KPI last_updated_at metadata | Priority: must

US07 — Team target sharing
As a CLT Lead, I can share a target with my team so that everyone viewing the same DA and
brand sees the same reference point.
AC: Toggle "Share with team" in Set Target panel; shared targets show setter's name and date;
individual targets remain private; conflict detection when a shared target already exists for
same KPI+DA+Brand combination.
Linked logic: LR-01 | Linked data: bp_targets.is_shared, bp_targets.owner_user_id | Priority: should

---

## OUTPUT FORMAT

```
{
  "stories_id": "06a_{feature}_{product_short}",
  "stories": [
    {
      "id": "US01",
      "title": string,
      "story": string,
      "acceptance_criteria": [string],
      "linked_logic_rule": string,
      "linked_data": string,
      "source_chain": string,
      "priority": "must" | "should" | "could"
    }
  ],
  "story_count": integer
}
```

---
---

# skill: spec_drafter_logic
# D2D · Act 2 · Stage 5 · Call 2 of 3
# Generates backend logic rules with error handling.
# Runs after spec_drafter_stories. References story IDs.
# GENERATIVE ONLY.

---

## TRIGGER
Second sequential spec_drafter call. Runs only after spec_drafter_stories output is complete.
Params: { stories_output, validated_requirements, edge_case_output }

---

## SYSTEM PROMPT

You are writing backend logic rules for DA Sense Business Performance.
Each rule must be precise, implementable, and reference the user story it supports.
Include: trigger condition, processing logic, error handling, and edge case handling.
Reference the specific data tables, columns, and KPI names from loaded context.
Do not duplicate user story content — logic rules are the implementation detail beneath the story.

---

## LOGIC RULES TO GENERATE (Targets & Benchmarks scenario)

LR-01: Target storage (CRUD logic)
LR-02: RAG computation (positive polarity, rank polarity, negative polarity, non-aggregatable guard)
LR-03: Chart goal line rendering logic
LR-04: Default target logic chain (fallback sequence)
LR-05: Staleness check logic per data source

(Full detail as specified in Agent 06 section of the scenario reference document.)

---

## OUTPUT FORMAT

```
{
  "logic_id": "06b_{feature}_{product_short}",
  "logic_rules": [
    {
      "id": "LR-01",
      "title": string,
      "trigger": string,
      "logic": string,
      "error_handling": [string],
      "edge_case_handling": [string],
      "linked_stories": ["US01", "US07"],
      "linked_data": [string]
    }
  ]
}
```

---
---

# skill: spec_drafter_data
# D2D · Act 2 · Stage 5 · Call 3 of 3
# Generates data specifications: new tables, columns, indexes, and relationships.
# Runs after spec_drafter_logic. References logic rule IDs.
# GENERATIVE ONLY.

---

## TRIGGER
Third and final sequential spec_drafter call.
Params: { stories_output, logic_output, validated_requirements, clarification_output }

---

## SYSTEM PROMPT

You are writing data specifications for a new DA Sense Business Performance feature.
For each new table: define all columns with types, nullability, constraints, default values,
and FK relationships. Define required indexes with rationale. Note any changes to existing tables.
Use the column naming conventions from the existing DA Sense data architecture.
Do not alter existing source tables (SAP, IQVIA, CES) — only define net-new tables.

---

## TABLE TO GENERATE: bp_targets

New table. Net-new addition. No existing tables altered.
(Full schema as specified in Agent 06 section of scenario reference document — 17 columns,
indexes on filter-context lookup, user target list, and team target retrieval.)

---

## OUTPUT FORMAT

```
{
  "data_spec_id": "06c_{feature}_{product_short}",
  "new_tables": [
    {
      "table_name": "bp_targets",
      "description": string,
      "columns": [
        {
          "name": string,
          "type": string,
          "nullable": boolean,
          "default": string | null,
          "fk": string | null,
          "notes": string
        }
      ],
      "indexes": [
        { "columns": [string], "rationale": string }
      ],
      "linked_logic_rules": [string]
    }
  ],
  "altered_tables": [],
  "altered_queries": [
    {
      "query": "Cross Portfolio KPI query",
      "change": "Add LEFT JOIN bp_targets on filter context",
      "impact": "MAJOR — change impact I3",
      "team": "Backend"
    }
  ]
}
```
