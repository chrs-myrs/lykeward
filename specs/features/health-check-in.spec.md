---
id: health-check-in
type: feature-spec
purpose: Regular health interview capturing current symptoms, treatment responses, and updates
criticality: CRITICAL
failure_mode: Health observations not captured between appointments; knowledge store stagnates
governed-by:
  - ../workspace/knowledge-operations.spec.md
  - ../workspace/constitution.spec.md
specifies:
  - ../../.claude/skills/health-check-in/skill.md
---

# Health Check-In Skill

## Purpose

Structured interview that captures the user's current health state — what they're experiencing, how treatments are working, what's changed. Produces observation records, treatment updates, and flags for clinical attention. Designed for regular use (weekly or as-needed).

## Requirements

### REQ-CHECKIN-001: Adaptive Interview
- Start with an open question: "How are you feeling? Anything new or changed?"
- Adapt follow-up questions based on the response
- If active investigations exist, ask about relevant symptoms specifically
- If active treatments exist, ask about response/side effects
- Do NOT run through a fixed checklist — adapt to what the user says

### REQ-CHECKIN-002: Capture Observations
- Create observation records in `data/core/` for each health data point
- Observations are free text with optional severity and body_system tags
- AI appends suggested tags to the observation record silently
- Only prompt the user for classification if severity is ambiguous or a clinical flag is warranted
- Follow the "capture first, classify later" principle — never force classification

### REQ-CHECKIN-003: Update Active Treatments
- Only ask about treatments started or changed in the last 30 days, treatments with no recorded response, or treatments previously flagged
- Do NOT ask about every active treatment every session
- Update treatment response field if user reports change
- If treatment stopped: update status, record reason
- If new side effect: create observation linked to treatment

### REQ-CHECKIN-004: Flag for Clinical Attention
- Flag *changes* in severity (e.g., mild last week, severe now) — not just absolute severity
- If a symptom is new and unexplained, suggest "you might want to mention this to your GP"
- If a treatment response is "worsened" or "side-effect", flag for GP discussion
- Present flags at the end, not during the interview (don't alarm)
- Safety netting: if user describes severity as "severe" or uses emergency language, the session summary includes a predetermined safety message: "You described [X] as severe. If you need urgent help, contact NHS 111 or call 999." This is informational, not diagnostic

### REQ-CHECKIN-005: Link to Existing Knowledge
- New observations automatically linked to relevant active investigations
- AI suggests links based on body_system and symptom similarity
- Links default to relational_confidence: unassessed (never assert causation)
- Observations flagged by REQ-CHECKIN-004 receive `flagged_for_gp: true` — this creates the explicit data bridge to consultation-prep

### REQ-CHECKIN-006: Session Summary
- End with a summary of what was captured
- List new observations, treatment updates, and flags
- Suggest next actions if any (book GP, start investigation, update condition status)

### REQ-CHECKIN-007: Session Duration Target
- Typical check-in should complete in under 5 minutes
- If user reports nothing new, acknowledge and complete in under 2 minutes
- A "nothing new" check-in still updates `last_reviewed` on active items to maintain dormancy accuracy

## Non-Requirements
- Does NOT produce a GP crib sheet (that's /consultation-prep)
- Does NOT manage investigations (that's /investigate-health)
- Does NOT audit the store (that's /knowledge-health)
