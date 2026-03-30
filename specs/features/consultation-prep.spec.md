---
id: consultation-prep
type: feature-spec
purpose: Generate a 60-second salience view for GP consultations
criticality: IMPORTANT
failure_mode: Patient arrives at GP unprepared; appointment time wasted on history reconstruction
governed-by:
  - ../workspace/knowledge-operations.spec.md
  - ../workspace/constitution.spec.md
specifies:
  - ../../.claude/skills/consultation-prep/skill.md
---

# Consultation Prep Skill

## Purpose

Generates a single-page GP consultation crib sheet from the knowledge store. Designed for the 60-second absorption target — the GP should be able to scan it and understand: what's changed, what's active, what the patient is concerned about.

## Requirements

### REQ-PREP-001: Salience View Generation
- Produce a single-page markdown document: `data/core/crib-sheet-YYYY-MM-DD.md`
- Sections in order of GP priority:
  1. **One-sentence summary** — what brings you in today
  2. **What's changed** since last encounter
  3. **Active conditions** with current status
  4. **Current medications** with response notes
  5. **Patient concerns** — what you want to discuss
  6. **Patient-noticed patterns (unverified)** — co-occurrences the patient has noticed. These are self-reported correlations. They have not been clinically verified.
  7. **Active investigations** — what's being looked into
  8. **Recent test results** if any

### REQ-PREP-002: Ask the User What's Driving This Visit
- Before generating, ask: "What's the main reason for this appointment?"
- This becomes the one-sentence summary
- Follow up: "Anything else you want to raise?"

### REQ-PREP-003: Pull from Knowledge Store
- Read all observations since last encounter
- Read observations with `flagged_for_gp: true` since last encounter
- Read all active conditions and treatments
- Read all active investigations
- Read any recent test results
- Read flagged-for-review links

### REQ-PREP-004: Respect Confidence Boundaries
- Self-assessed links marked explicitly: "Patient notes: X seems to coincide with Y"
- Clinician-confirmed links stated as fact
- Unassessed links not included in the crib sheet

### REQ-PREP-005: Post-Appointment Recording
- After the appointment, prompt: "How did it go? What did the GP say?"
- Create encounter record in `data/core/`
- Two-phase recording:
  - Phase (a) Quick capture: what happened, what was decided
  - Phase (b) Detailed update: condition/treatment/investigation changes. Phase (b) can be deferred
- Any condition status change to `confirmed` or `resolved` must include explicit clinician attribution (who said it, when)
- Any new condition created from encounter outcome defaults to `status: suspected` unless the user attributes a diagnosis to the clinician
- This is the bidirectional flow: prep derives → outcome records back

### REQ-PREP-006: Immutable Crib Sheet
- The generated crib sheet is immutable once created (type: evidence)
- It becomes part of the encounter record as a reference
- This preserves what the patient intended to discuss vs what actually happened

### REQ-PREP-007: Encounter Method Adaptation
- If the encounter method is eConsult, generate a text-pasteable format (no markdown formatting, plain text)
- If phone/video, generate a shorter summary
- Default (face-to-face) uses full crib sheet format

### REQ-PREP-008: Standalone Encounter Recording
- The post-appointment recording capability (REQ-PREP-005) must be invocable independently of the preparation phase
- If the user runs `/consultation-prep` after an unplanned appointment, skip REQ-PREP-001 and REQ-PREP-002 and proceed directly to REQ-PREP-005

## Non-Requirements
- Does NOT replace the GP's own clinical notes
- Does NOT provide medical advice or suggest diagnoses
- Does NOT include the full health history — only what's relevant NOW
