# skill: consistency_checker
# D2D · Reusable skill · Invoked after spec_drafter_data
# Cross-references stories ↔ logic rules ↔ data spec to find contradictions.
# SEPARATED from spec drafting deliberately — re-run on conflict resolution without re-running
# the full drafting sequence.
# This is the second biggest demo wow moment — the pause between drafts appearing and
# the checker revealing a conflict builds tension and demonstrates rigor.

---

## TRIGGER
Invoked by orchestrator after all three spec_drafter calls complete.
Params: { stories_output, logic_output, data_spec_output }

---

## SYSTEM PROMPT

You are running a consistency check across three specification artefacts:
user stories, backend logic rules, and data specifications.

Your job: find every contradiction, missing link, and inconsistency across the three documents.
This is an analytical pass — do not suggest fixes unless the fix is unambiguous.
For each issue found, present it clearly to the BA with the conflicting items identified.

Do not rewrite any artefact. Just surface what is inconsistent. The BA resolves.

---

## CHECKS TO RUN

### Check 1 — Field name consistency
For every field referenced in a user story or acceptance criterion:
  Does the field exist in the data spec with the exact same name?
  If a story says "owner_user_id" but data spec uses "created_by_user_id" → CONFLICT.

### Check 2 — Story coverage of logic rules
For every logic rule: is there at least one story that tests it?
  If LR-02 (RAG computation) has no acceptance criterion that tests yellow-zone threshold → GAP.

### Check 3 — Logic rule coverage of data columns
For every data column in bp_targets: is it referenced by at least one logic rule?
  If bp_targets.is_active exists but no logic rule addresses archiving or deactivation → GAP.

### Check 4 — Enum consistency
If a story uses an enum value (e.g. "positive" polarity) and the data spec defines the
column as ENUM('positive', 'negative') — do the values match exactly?

### Check 5 — Edge case coverage
For each edge case from edge_case_surfacer: is there an acceptance criterion in some story
that addresses it? If not: which story should own it?
  If EC9 (CES non-aggregatable RAG block) has no acceptance criterion → assign to US02.

### Check 6 — Impact item traceability
For each BREAKING and MAJOR change item from change_impact_analyser: is there at least one
story or logic rule that addresses it?
  If I2 (user identity integration) is not addressed in any story → GAP in US07 or a new story needed.

---

## CONFLICTS FOUND (Targets & Benchmarks scenario — illustrative)

Conflict 1 (field name mismatch):
  US07 acceptance criterion references "team_id" but bp_targets schema has no team_id column.
  The schema uses is_shared (boolean) + owner_user_id to represent team sharing.
  Resolution needed: either add team_id to data spec or update US07 AC to reference is_shared.

Conflict 2 (missing edge case coverage):
  EC10 (concurrent target edits — last write wins) has no acceptance criterion in any story.
  Recommended: add to US07 or create US08.

Conflict 3 (logic rule gap):
  bp_targets.is_active column exists in data spec. No logic rule defines when is_active is
  set to false. Archiving/expiry logic is undocumented.
  Recommended: add LR-06 for target lifecycle management or document as out of scope.

---

## OUTPUT FORMAT

```
{
  "consistency_id": "07_{feature}_{product_short}",
  "conflicts": [
    {
      "id": "CC1",
      "type": "field_name_mismatch" | "missing_story" | "missing_logic" |
              "enum_mismatch" | "edge_case_uncovered" | "impact_unaddressed",
      "description": string,
      "item_a": { "artefact": "story" | "logic" | "data", "id": string },
      "item_b": { "artefact": "story" | "logic" | "data", "id": string } | null,
      "resolution_options": [string],
      "ba_resolution": null,
      "resolved": false
    }
  ],
  "gaps": [
    {
      "id": "CG1",
      "description": string,
      "affected_artefact": string,
      "recommendation": string
    }
  ],
  "clean_count": integer,
  "conflict_count": integer,
  "gap_count": integer
}
```

---

## RESOLUTION FLOW

After presenting findings:
1. BA reviews each conflict and gap.
2. For each: BA chooses a resolution option or provides a custom resolution.
3. Orchestrator applies the resolution to the affected artefact (stories, logic, or data spec).
4. Consistency check runs again on the changed artefact only (targeted re-run, not full re-run).
5. When all conflicts resolved: set consistency_check_complete = true.

---
---

# skill: delivery_agent
# D2D · Act 2 · Stage 7 · Final skill
# Generates Jira-ready tickets grouped by engineering team.
# Generates the Pipeline Summary — the narrative of the full discovery-to-developer journey.
# Every ticket carries a complete source_chain traceable to the original PO idea.

---

## TRIGGER
Invoked after consistency_check_complete = true.
Params: { stories_output, logic_output, data_spec_output, consistency_output,
          all_analysis_outputs, handoff_package, approved_brief }

---

## SYSTEM PROMPT

You are generating developer-ready Jira tickets for DA Sense Business Performance.
Every ticket must carry: title, description, acceptance criteria, technical notes,
dependencies on other tickets, impacted team, and a source_chain.

The source_chain format is: ticket → user story → requirement → PO decision or critique
resolution → conversation turn.

After generating all tickets, generate the Pipeline Summary — a narrative that reads
the source chains backwards: from 22 tickets down to the single PO idea they came from.

A developer reading any ticket should have enough context to make decisions independently
without needing to loop back to the PO or BA.

---

## TICKET GROUPS

### Data Engineering (4 tickets)
DE-01: Design and implement bp_targets write-back layer
  [BREAKING dependency — must be first. Blocks all other tickets.]
DE-02: Implement default target computation logic
DE-03: Add FX handling for absolute Sales targets in CHF
DE-04: Implement data staleness check service

### Backend (6 tickets)
BE-01: Implement user identity layer integration
BE-02: Build Target CRUD API (GET, POST, PUT, DELETE /targets)
BE-03: Implement RAG computation service
BE-04: Update Cross Portfolio query with LEFT JOIN on bp_targets
BE-05: Implement team target sharing logic
BE-06: Implement fallback default logic chain

### Frontend (7 tickets)
FE-01: Build Set Target panel component
FE-02: Build RAG status indicator component
FE-03: Integrate RAG indicators into Cross Portfolio table
FE-04: Add goal line rendering to KPI trend charts
FE-05: Redesign Sales vs. Forecast chart (user goal primary, T0 secondary)
FE-06: Build hover tooltip component
FE-07: Design and implement empty states

### QA (5 tickets)
QA-01: Test RAG computation edge cases
QA-02: Test target persistence across filter changes
QA-03: Test rank target with dynamic competitor set
QA-04: Test concurrent target edit scenario
QA-05: Test Sales vs. Forecast chart with both reference lines

Total: 22 tickets

---

## TICKET TEMPLATE

```
Ticket ID: [TEAM]-[N]
Title: [Action verb] [component/feature] [context]
Team: Data Engineering | Backend | Frontend | QA
Depends on: [ticket IDs] | none
Priority: P0 (BREAKING) | P1 (MAJOR) | P2 (MINOR)

Description:
[2–4 sentences explaining what needs to be built and why, with enough context for the
developer to make implementation decisions independently.]

Acceptance criteria:
  - [testable criterion]
  - [testable criterion]
  - [edge case handling]

Technical notes:
  - [specific implementation detail, schema reference, or constraint to be aware of]

Source chain: [TEAM-N] → [US_N] → [req_N] → [agent_decision] → [agent_01_turn_N]
```

---

## PIPELINE SUMMARY TEMPLATE

```
PIPELINE SUMMARY — [Feature Name] · [Product]

JOURNEY NARRATIVE:
The PO entered with a single observation: "[original PO statement]."
Through [N] conversation turns, this crystallised into [brief description of what was built].

The Requirement Critic identified [N] challenged assumptions. The most consequential:
[description of the most important BREAKING change and what would have happened without D2D.]

UXR [validated | was skipped]. [If validated: key finding and its impact on data model.]

The BA identified [N] BREAKING changes, [N] MAJOR changes, [N] deep edge cases, and
[N] feasibility questions requiring dev team discussion before sprint planning.

[Any specific resolution that was reached during BA phase.]

STATS:
- PO conversation turns: N
- Capabilities in brief: N
- Assumptions challenged by Critic: N
- Breaking changes identified: N
- Deep edge cases: N
- User stories: N
- Backend logic rules: N
- New tables: N (table_name, N columns)
- Jira tickets generated: N
- Teams impacted: N (list)
- Requirements with full source_chain: N/N

TRADITIONAL TIMELINE COMPARISON:
- Typical: 3–4 months from idea to developer-ready spec
- D2D workflow: 2–3 weeks (with UXR); 1 week (without UXR)
- Critical finding discovered at: [stage] (vs. Sprint [N] in traditional process)
```

---

## OUTPUT FORMAT

```
{
  "delivery_id": "08_{feature}_{product_short}",
  "tickets": [
    {
      "id": "DE-01",
      "title": string,
      "team": string,
      "depends_on": [string],
      "priority": "P0" | "P1" | "P2",
      "description": string,
      "acceptance_criteria": [string],
      "technical_notes": [string],
      "source_chain": string
    }
  ],
  "ticket_count": integer,
  "by_team": {
    "data_engineering": integer,
    "backend": integer,
    "frontend": integer,
    "qa": integer
  },
  "pipeline_summary": string,
  "dependency_graph": [
    { "ticket": string, "blocks": [string] }
  ]
}
```
