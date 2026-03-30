---
id: dormancy-check
type: feature-spec
purpose: Surface forgotten investigation threads and stale health concerns
criticality: IMPORTANT
failure_mode: Open health investigations quietly forgotten; conditions go unmonitored
governed-by:
  - ../workspace/knowledge-operations.spec.md
  - ../workspace/constitution.spec.md
specifies:
  - ../../.claude/skills/dormancy-check/skill.md
---

# Dormancy Check Skill

## Purpose

Scans the knowledge store for investigation threads, active conditions, and treatments that haven't been reviewed recently. Surfaces them for the user to decide: still investigating, move to dormant, or actively resume.

## Requirements

### REQ-DORM-001: Investigation Staleness
- Scan all investigations with status: investigating or hypothesis
- Default threshold: flag any with no new linked observations in 90+ days
- Threshold is configurable per investigation via a `review_interval` field (e.g., investigations awaiting specialist referral may legitimately be quiet for 6+ months)
- Present: "Your [investigation name] hasn't had new evidence since [date]. Still investigating?"

### REQ-DORM-002: Condition Review
- Scan conditions with status: managed or flaring
- Flag any not referenced by a recent encounter (180+ days)
- Present: "Your [condition] was last discussed with a GP on [date]. Worth a review?"

### REQ-DORM-003: Treatment Audit
- Scan treatments with status: active
- Flag any with no treatment_response recorded
- Flag any active for 180+ days without a recent encounter reference
- Flag treatments active for 365+ days without a recent encounter reference: "Your annual medication review for [treatment] may be due."
- Present: "You've been on [treatment] since [date]. How's it going?"

### REQ-DORM-004: User Decision for Each Flag
For each flagged item, offer:
- **Still active** — keep current status, update last_reviewed
- **Move to dormant** — change investigation status or note condition as stable
- **Resume** — the flag reminds me to act; schedule a check-in or GP visit
- **Resolved** — close the investigation or mark condition resolved. Requires attribution: "Was this resolved by a clinician, or did symptoms resolve on their own?" Set investigation outcome accordingly to maintain the observation/diagnosis boundary

### REQ-DORM-005: Lightweight Output
- Run on demand only: `/dormancy-check` — do NOT run automatically on session start
- Optionally, surface a one-line count on session start ("3 items need review — run /dormancy-check to see them") but do not present the full list unprompted
- If nothing is stale: "All clear — no dormant threads."
- If items flagged: present as a simple numbered list, handle one at a time
- Total interaction under 2 minutes for typical store

### REQ-DORM-006: Bridge to Action
- After processing flags, offer actionable next steps: "Would you like to add any of these to your next GP visit concerns?"
- If yes, set `flagged_for_gp: true` on relevant items
- This bridges dormancy-check to consultation-prep

## Non-Requirements
- Does NOT create new observations or investigations
- Does NOT audit data quality (that's /knowledge-health)
- Does NOT suggest clinical actions — only surfaces what needs attention
