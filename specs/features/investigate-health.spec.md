---
id: investigate-health
type: feature-spec
purpose: Create and manage diagnostic investigation threads
criticality: IMPORTANT
failure_mode: Health concerns investigated informally without tracking progress or evidence
governed-by:
  - ../workspace/knowledge-operations.spec.md
  - ../workspace/constitution.spec.md
specifies:
  - ../../.claude/skills/investigate-health/SKILL.md
---

# Health Investigation Skill

## Purpose

Creates and manages diagnostic investigation threads — the "why does X happen?" streams that connect symptoms, tests, encounters, and conditions into a reasoning trail. Investigations are the core structural innovation of this knowledge store.

## Requirements

### REQ-INV-001: Question-Driven Creation
- Start every investigation with a question: "Why do my hands tingle?"
- The question IS the hypothesis
- Ask: "When did you first notice? What symptoms are involved? What do you think might be causing it?"
- Create investigation file in `data/core/` with status: hypothesis

### REQ-INV-002: Evidence Linking
- Link existing observations to the investigation via the evidence array
- For each linked observation, assign a role (presenting-symptom, trigger, correlation)
- Observation confidence set by patient; relational confidence defaults to self-assessed
- AI suggests relevant observations from the store based on body_system and temporal overlap

### REQ-INV-003: Reasoning Trail Structure
Investigation files contain these sections:
- **Question**: The driving hypothesis
- **Possible explanations (your thinking, not diagnosis)**: What could this be? (list of possible conditions)
- **Supporting observations**: Observations supporting each possibility
- **Contrary observations**: Observations that weaken each possibility
- **Latest clinical input**: Populated ONLY from encounter records where a clinician has commented on this investigation. If no clinician input exists, this section states "No clinical input recorded yet."
- **Next Steps**: What would help narrow it down (tests, specialist, tracking)

### REQ-INV-004: Status Management
- hypothesis → investigating (when actively gathering evidence)
- investigating → concluded (when a condition is confirmed or ruled out)
- investigating → dormant (when no new evidence in 3+ months)
- dormant → investigating (when new evidence surfaces)
- Update status based on encounter outcomes (GP confirms/rules out)

### REQ-INV-005: Investigation Review
- When invoked with an existing investigation name, review its current state:
  - Present evidence in date order with source attribution (patient observation vs clinician input)
  - Highlight what's changed since last review
  - Suggest next steps
- The review summarises chronologically — it does NOT evaluate or generate a "what seems most likely" assessment
- This is the "what's the story with my tingling?" query

### REQ-INV-006: Co-occurrence, Not Causation
- Investigation reasoning trails are patient-sourced hypotheses
- All connections marked relational_confidence: self-assessed unless a clinician has confirmed
- The investigation presents evidence — it does NOT diagnose

### REQ-INV-007: Clinician Input Recording
- When post-appointment recording (from consultation-prep or standalone) updates an investigation, the clinician's input is recorded in a structured `clinician_input` section
- Each entry includes: date, clinician, and what they said
- This is the only section that carries clinical weight

### REQ-INV-008: Lightweight Listing
- Support a listing mode: "What are my active investigations?"
- Returns a summary table (name, question, status, last updated, evidence count)
- Does not enter full review mode

## Non-Requirements
- Does NOT replace clinical differential diagnosis
- Does NOT suggest specific conditions to investigate (patient decides the question)
- Does NOT auto-close investigations — patient or clinician decides when concluded
