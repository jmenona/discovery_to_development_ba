# skill: research_guide_generator
# D2D · Act 1 · UXR Branch (conditional)
# Generates a UXR discussion guide from the PO's accumulated conversation context.
# The guide is grounded in specific product context, not generic.
# Stimulus materials include wireframe from idea_exploration.

---

## TRIGGER
Invoked when PO chooses to validate with users after brief approval.
Params: { approved_brief, critique_output, wireframe_state }

---

## SYSTEM PROMPT

You are generating a UXR discussion guide for user research on a Roche internal tool feature.
The guide must be grounded in DA Sense Business Performance context — not generic research questions.
Every question must reference the actual product, actual KPIs, or actual UI patterns.

Include stimulus materials referencing the wireframe from idea_exploration.
Target participants are Roche internal users who use DA Sense today.

---

## RESEARCH OBJECTIVES (Targets & Benchmarks scenario)

1. Validate the interpretation gap — do users track goals outside DA Sense today, and how?
2. Preference: user-defined vs. team-aligned vs. centrally-defined targets?
3. RAG comprehension: does color status make sense for each KPI's natural variability?
4. Dual reference test: Sales vs. Forecast already has a T0 line — does adding a second
   target line help or confuse?
5. Time dimension: how do users naturally think about goals? Annual / quarterly / campaign-based?
6. Access: should targets be personal or visible to the whole team viewing the same DA?

---

## USER SEGMENTS TO RECRUIT

- 3–4 CLT Leads (global, cross-DA perspective)
- 3 Country I&A Leads (affiliate-level, metric-heavy users)
- 2 Country Commercial Leads (action-oriented, less analytical — test RAG intuitiveness)

---

## DISCUSSION GUIDE STRUCTURE

Section 1 — Warm-up (10 min)
  Walk me through how you typically review Business Performance for your DA in DA Sense.
  What do you do when you see a Sales or NPS number — what's your first question?
  Do you currently track goals or targets outside DA Sense? How?

Section 2 — Interpretation gap (15 min)
  [Show current Cross Portfolio screenshot]
  Looking at this table — if you see NPS at 32%, how do you know if that's good or bad?
  What information would need to be on this screen for you to feel confident it's on track?

Section 3 — Target preference (15 min)
  [Show wireframe with Set Target panel]
  If you could define a target for NPS here — who should set it? Just you, your team, or
  should it come centrally from the Business Plan?
  If your colleague sets a different NPS target for the same brand — how does that affect
  how you use this feature?

Section 4 — Dual reference test (10 min)
  [Show Sales vs. Forecast chart with BOTH T0 forecast line AND user goal line]
  You can see two reference lines here. Which one do you look at first?
  If they differ significantly — say T0 says you're at 59% but your goal says 72% — which
  number do you report in your CLT deck?

Section 5 — RAG comprehension (10 min)
  [Show Cross Portfolio table with RAG indicators]
  What does yellow mean to you for NPS? For Sales? Are those the same?
  How close to the target should something be before it turns green?

Section 6 — Close (5 min)
  If this feature was live tomorrow, what's the first target you'd set?
  What would make you not use this feature?

---

## STIMULUS MATERIALS

1. Current Cross Portfolio screenshot (from loaded context)
2. Wireframe: Cross Portfolio + RAG indicators (from idea_exploration final state)
3. Wireframe: Set Target modal
4. Side-by-side: current Sales vs. Forecast view vs. proposed version with both reference lines

---

## OUTPUT FORMAT

```
{
  "guide_id": "03_{feature}_{product_short}",
  "research_objectives": [string],
  "participant_segments": [
    { "segment": string, "count": integer, "rationale": string }
  ],
  "discussion_guide": {
    "total_duration_minutes": integer,
    "sections": [
      {
        "section": string,
        "duration_minutes": integer,
        "questions": [string],
        "stimulus": string | null
      }
    ]
  },
  "stimulus_materials": [string],
  "key_probes_from_critic": [string],
  "screener_criteria": string
}
```

---
---

# skill: research_synthesiser
# D2D · Act 1 · UXR Branch (conditional)
# Maps UXR findings against the approved brief.
# PO reviews each finding and makes accept/reject/modify decisions.
# Output is the validated requirements — the formal handoff artifact to the BA.

---

## TRIGGER
Invoked when PO returns with UXR findings and uploads them to the session.
Params: { approved_brief, uxr_findings_raw, critique_output }

---

## SYSTEM PROMPT

You are mapping UXR research findings against a validated product brief.
Your job: for each finding, determine whether it confirms, modifies, or contradicts a
brief element, or introduces something new.
Present each finding to the PO with a clear recommendation. Wait for PO accept/reject/modify.
Build the final validated requirements from PO decisions.

Every requirement in the output must carry a source_chain linking it to:
original brief element → UXR finding → PO decision.

---

## FINDING CLASSIFICATION

For each UXR finding, classify as:
- CONFIRMS: finding validates an existing brief element. Action: strengthen the requirement.
- MODIFIES: finding suggests a change to an existing element. Action: present PO with proposed change.
- CONTRADICTS: finding challenges a core brief assumption. Action: present PO with decision.
- NEW: finding introduces a requirement not in the original brief. Action: present as scope addition.

---

## EXPECTED FINDINGS (Targets & Benchmarks scenario)

Finding 1 — CONFIRMS
  "Users maintain shadow Excel tracking of targets today — the interpretation gap is real."
  PO action: confirm. Strengthens problem_statement.

Finding 2 — MODIFIES
  "RAG indicators are immediately intuitive to analytical users; less so to commercial leads
  who need plain-language 'on track / at risk' label alongside the color."
  PO action: accept → add plain-language label as acceptance criterion for RAG component.

Finding 3 — MODIFIES (high impact on data model)
  "Pure user-defined targets create inconsistency anxiety. Users want team-level targets
  or a recommended default with optional override."
  PO action: accept → changes target_authority from purely personal to team-level shared
  with personal override. This adds is_shared flag and owner attribution to data model.

Finding 4 — MODIFIES (chart design decision)
  "Dual reference lines on Sales chart are confusing when they differ significantly.
  Users want to set their own goal AS the reference, replacing T0 as primary display."
  PO action: accept → user goal line = primary dashed; T0 forecast = secondary lighter line.
  This resolves the dual-reference conflict surfaced by requirements_critic.

Finding 5 — NEW (potential scope addition, flag for BA)
  "Users want to see goal trajectory over time — 'am I likely to hit my target by year-end?'"
  PO action: note as future release item, out of V1 scope. Add to backlog.

---

## OUTPUT FORMAT — Validated Requirements

```
{
  "validated_req_id": "04_{feature}_{product_short}",
  "source_brief_id": "01_{feature}_{product_short}",
  "uxr_session_summary": {
    "participants": integer,
    "segments": [string],
    "date": string
  },
  "validated_requirements": [
    {
      "req_id": "req_01",
      "requirement": string,
      "source_chain": "brief_cap_N → uxr_finding_N → po_decision_accept",
      "uxr_status": "confirmed" | "modified" | "new",
      "po_decision": "accept" | "modify" | "reject",
      "po_decision_notes": string | null,
      "priority": "must" | "should" | "could"
    }
  ],
  "out_of_scope_items": [
    {
      "item": string,
      "reason": string,
      "backlog_candidate": boolean
    }
  ],
  "open_decisions_for_ba": [
    {
      "decision": string,
      "context": string,
      "recommended_default": string | null
    }
  ],
  "handoff_notes": string
}
```

---
---

# skill: handoff_packager
# D2D · Act 1 · Handoff step
# Packages the full PO journey into a clean handoff artifact for the BA.
# This is what the BA reads before beginning Act 2.

---

## TRIGGER
Invoked after research_synthesiser completes (or after brief_assembler if no UXR).
Params: { approved_brief, critique_output, validated_requirements | null, uxr_complete }

---

## OUTPUT

The handoff package contains:
1. Validated requirements with source_chains
2. Key decisions log (what the PO decided and why)
3. Open decisions for BA (things that need resolution or assumption in Act 2)
4. Critique summary (what was challenged, what was resolved, what remains)
5. Wireframe reference (which wireframe state to reference in Figma)

This document is presented to the BA as their starting point.
The BA does not need to re-read the full PO conversation — everything is captured here.
