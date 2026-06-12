# D2D — ONA Configuration Package
# Discovery to Developer · Hackathon Build Reference
# Scenario: DA Sense Targets & Benchmarks

---

## WHAT IS IN THIS PACKAGE

```
d2d-ona/
  agents/
    agents.md                         ← THE ORCHESTRATOR. Read this first.
  skills/
    skill_01_mode_and_context.md      ← mode_selector + context_loader
    skill_02_idea_exploration.md      ← idea_exploration (most critical skill)
    skill_03_critic_and_assembler.md  ← requirements_critic + brief_assembler
    skill_04_uxr_skills.md            ← research_guide_generator + research_synthesiser + handoff_packager
    skill_05_ba_analysis.md           ← technical_clarifier + change_impact_analyser + edge_case_surfacer
    skill_06_feasibility_and_specs.md ← feasibility_structurer + spec_drafter (3 calls)
    skill_07_consistency_and_delivery.md ← consistency_checker + delivery_agent
  context/
    roche_context_layer.md            ← Context layer structure + what to populate
```

---

## HOW TO USE IN ONA

ONA uses a `skills.md + agents.md` pattern. Here is how these files map:

### agents.md
Upload `agents/agents.md` as the main orchestrator configuration.
This is the state machine. It routes inputs to the right skill at the right time.
It maintains session state across all turns.

### skills.md files
Each file in `skills/` contains one or more skills. In ONA, each skill is a separately
configured callable unit. You can either:
- Upload each skill file individually with its skill ID as the name
- Concatenate all skill files into a single `skills.md` if ONA expects one file

The skill IDs used in `agents.md` must match exactly. They are:
```
mode_selector
context_loader
idea_exploration
requirements_critic
edge_case_surfacer          ← reusable, called with depth param
brief_assembler
research_guide_generator
research_synthesiser
handoff_packager
technical_clarifier
change_impact_analyser
feasibility_structurer
spec_drafter_stories
spec_drafter_logic
spec_drafter_data
consistency_checker
delivery_agent
```

### Context layer
The `context/roche_context_layer.md` defines the structure and content of what must be
populated before the agents run. For the PoC, this content is injected into each skill's
system prompt as context. See population priority order in the file.

---

## TEAM BUILD ASSIGNMENTS

| Team member | Skills to own | Files |
|-------------|--------------|-------|
| Person 3 (PO-side agents) | mode_selector, context_loader, idea_exploration, requirements_critic, brief_assembler | skill_01, skill_02, skill_03 |
| Person 4 (PO–BA bridge) | research_guide_generator, research_synthesiser, handoff_packager, technical_clarifier, change_impact_analyser | skill_04, skill_05 (partial) |
| Person 5 (Spec + Consistency) | edge_case_surfacer, feasibility_structurer, spec_drafter_stories, spec_drafter_logic, spec_drafter_data, consistency_checker | skill_05 (partial), skill_06, skill_07 (partial) |
| Person 6 (Delivery) | delivery_agent | skill_07 (partial) |
| Person 2 (Data Lead) | roche_context_layer population | context/ |
| Person 7 (UI/Demo) | agents.md integration, session state display | agents/ |
| You (Integration Owner) | agents.md orchestration logic, source_chain integrity, cross-skill state | agents/ + review all |

---

## BUILD ORDER (critical path)

1. **Populate context layer first** (Person 2). Without this, all agents are generic.
   Priority: data_architecture.md (write-back constraint) → kpi_catalogue.md → known_constraints.md

2. **Build context_loader** (Person 3). Verify it retrieves the right constraints.
   Test: query write-back capability → must return "exists: false".

3. **Build idea_exploration** (Person 3). This is the most critical skill.
   Test: run 5 turns on the Targets scenario. Verify T0 conflict surfaces at turn 4,
   CES constraint surfaces at turn 5.

4. **Build requirements_critic** (Person 3). Test against the turn-5 brief state.
   Verify write-back is flagged as BREAKING.

5. **Build technical_clarifier + change_impact_analyser** (Person 4).
   Test: verify I1 (write-back) and I2 (user identity) are both BREAKING.

6. **Build edge_case_surfacer deep mode** (Person 5).
   Test: all 10 edge cases from scenario reference document are surfaced.

7. **Build spec_drafter (3 calls)** (Person 5). Gate test: BREAKING items resolved.
   Test: US01–US07 generated. LR-01–LR-05 generated. bp_targets schema complete.

8. **Build consistency_checker** (Person 5).
   Test: the three illustrative conflicts in skill_07 are found.

9. **Build delivery_agent** (Person 6).
   Test: 22 tickets generated. Every ticket has source_chain. Pipeline summary readable.

10. **Wire agents.md** (You + Person 7). Full end-to-end run on demo scenario.

---

## DEMO SCENARIO RUN SCRIPT

The demo follows this exact sequence. Build and test against it.

```
Step 1: PO says "I want to add targets to DA Sense Business Performance"
         → mode_selector activates. PO selects Mode A.
         → context_loader loads DA Sense BP context. Confirms write-back constraint loaded.

Step 2: PO says "Tell me what you're thinking" turn
         → idea_exploration turn 2. Problem anchoring.

Step 3: T0 conflict surface (turn 4)
         → Agent asks about relationship to existing T0 forecast.
         DEMO WOW MOMENT 1: Agent already knows about T0 reference line.

Step 4: CES and NPS constraint surface (turn 5)
         → Agent asks about KPI scope. References the three data buckets by name.

Step 5: Edge cases optional (turns 7–9)
         → PO opts in. edge_case_surfacer light mode. 5 cases presented.

Step 6: Brief assembly (turn 12)
         → brief_assembler. Present for PO approval.

Step 7: UXR decision
         → PO: "Validate with users first."
         → research_guide_generator. Discussion guide shown.
         → [Offline UXR. Return with findings.]
         → research_synthesiser. PO accept/reject flow.
         DEMO WOW MOMENT 2: Team-level sharing emerges from UXR. Data model changes live.

Step 8: BA takes over.
         → technical_clarifier. 4 clarifications presented.
         → change_impact_analyser. 2 BREAKING changes flagged.
         DEMO WOW MOMENT 3: "Write-back not possible today" — BREAKING — surfaced before sprint.

Step 9: Deep edge cases.
         → edge_case_surfacer deep mode. 10 cases. BA reviews.

Step 10: Feasibility agenda.
          → feasibility_structurer. 5 questions for dev team.

Step 11: Spec drafting (3 calls).
          → spec_drafter_stories: US01–US07.
          → spec_drafter_logic: LR-01–LR-05.
          → spec_drafter_data: bp_targets schema.

Step 12: Consistency check.
          DEMO WOW MOMENT 4: Checker finds the team_id conflict and is_active gap.
          BA resolves. Re-run confirms clean.

Step 13: Delivery.
          → 22 tickets generated. Pipeline Summary read aloud.
          DEMO CLOSER: "Started as 'I want to add targets' → 11 turns → UXR → 10 edge cases
          → 2 BREAKING changes caught → 22 tickets. Developer starts with enough context
          to work independently."
```

---

## SOURCE CHAIN FORMAT

Every output item must carry this format:
```
[output_id] → [upstream_output_id] → ... → agent_01_turn_[N]
```

Example for the write-back BREAKING change ticket DE-01:
```
DE-01 → I1 → C1 → req_01 → uxr_finding_none → agent_02_challenge_01 → agent_01_turn_05
```

Reading this backwards: "The write-back ticket came from BREAKING impact I1, which came from
clarification C1, which came from requirement req_01, which was challenged by the Critic in
Challenge 1, which was first surfaced in PO conversation turn 5."

This chain is what the Pipeline Summary narrates. Build it from the start, not at the end.
