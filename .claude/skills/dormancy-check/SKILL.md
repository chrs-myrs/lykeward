# Dormancy Check

Surface forgotten investigation threads, stale conditions, and unreviewed treatments. On-demand only.

## Usage

```
/dormancy-check              # Full scan with interactive triage
/dormancy-check --count      # One-line summary count only (suitable for session start)
```

## Context Loading

Before any scan, read these files:

1. `data/SCHEMA.md` -- type definitions and required fields
2. `data/vocabulary-spec.md` -- controlled vocabulary (condition_status, treatment_response)
3. `ctxt/knowledge-model.md` -- relationship map and field reference

## Phase 1: Scan

Read all files in `data/core/` recursively. Parse YAML frontmatter from each. Build three collections:

### 1A: Investigation Staleness (REQ-DORM-001)

Select files where `type: investigation` AND `status` is `investigating` or `hypothesis`.

For each investigation:
- Find the most recent linked observation by reading the `evidence` array and resolving each referenced observation's `date` field. If no evidence array exists, fall back to `last_updated`.
- Calculate days since the most recent evidence date.
- Read `review_interval` from frontmatter if present (integer, days). Default threshold: **90 days**.
- Flag if days since last evidence exceeds the threshold (per-investigation `review_interval` overrides the 90-day default).

### 1B: Condition Review (REQ-DORM-002)

Select files where `type: condition` AND `status` is `managed` or `flaring`.

For each condition:
- Find the most recent encounter that references this condition. Search all `type: encounter` files -- check `concerns` arrays and body text for the condition's `entity` value.
- Calculate days since that encounter's `date`. If no encounter references this condition at all, use `last_updated` as the reference date.
- Flag if **180+ days** since last encounter reference.

### 1C: Treatment Audit (REQ-DORM-003)

Select files where `type: treatment` AND `status` is `active`.

For each treatment, check three independent flags:

1. **No response recorded**: Flag if `response` field is absent or empty.
2. **Stale (180+ days)**: Find the most recent encounter referencing this treatment (search encounter `concerns` and body text for the treatment `name`). Flag if 180+ days since that encounter, or if no encounter references it at all and `started` is 180+ days ago.
3. **Annual medication review (365+ days)**: Flag if `started` is 365+ days ago AND no encounter has referenced this treatment in the last 365 days. This is a separate, more urgent flag.

A single treatment may carry multiple flags. Present the most urgent one.

### Count Mode

If `--count` was specified, skip to the output:

- If zero flags: output nothing (silent).
- If flags exist: output a single line: `"{N} items need review -- run /dormancy-check to see them"` and stop. Do not present the full list.

## Phase 2: Present (REQ-DORM-005)

If nothing is flagged:

> All clear -- no dormant threads.

Stop here.

If items are flagged, present a numbered summary list. Group by category but keep it flat:

```
Dormancy check found {N} items:

Investigations
  1. {investigation entity} -- no new evidence since {date} ({N} days)
  2. ...

Conditions
  3. {condition entity} -- last discussed with GP on {date} ({N} days)
  4. ...

Treatments
  5. {treatment name} -- no treatment response recorded
  6. {treatment name} -- active since {date}, not discussed with GP in {N} days
  7. {treatment name} -- annual medication review may be due (active {N} days)
  ...
```

Then say: `"Let's go through these one at a time. Starting with #{first}."` and proceed to Phase 3.

## Phase 3: Triage (REQ-DORM-004)

Process each flagged item sequentially. For each item, present the contextual prompt and offer four options:

### Contextual prompts by type

**Investigation**: "Your **{entity}** investigation ("{question}") hasn't had new evidence since {date}. Still investigating?"

**Condition**: "Your **{entity}** was last discussed with a GP on {date}. Worth a review?"

**Treatment (no response)**: "You've been on **{name}** since {started}. How's it going?"

**Treatment (stale)**: "You've been on **{name}** since {started}, not discussed with a GP in {N} days. Still needed?"

**Treatment (annual review)**: "Your annual medication review for **{name}** may be due. Started {started}, last GP discussion {date}."

### Decision options

Present these options for each item. Use AskUserQuestion with single-select:

| Option | Label | Action |
|--------|-------|--------|
| A | Still active | Keep current status. Set `last_updated` to today on the record. Add `last_reviewed: {today}` to frontmatter if not present. |
| B | Move to dormant | **Investigation**: set `status: dormant`. **Condition**: set `status: stable` and add a body note: "Moved to stable via dormancy review {date}." **Treatment**: no dormant status exists -- ask whether to set `status: paused` instead. Update `last_updated`. |
| C | Resume / act on this | Flag for follow-up. Ask: "Want to add this to your next GP visit concerns?" If yes, set `flagged_for_gp: true` on the record. This is the bridge to consultation-prep (REQ-DORM-006). |
| D | Resolved | **Investigation**: set `status: concluded`. Then ask for attribution (AskUserQuestion, single-select): "Was this resolved by a clinician, or did symptoms resolve on their own?" Options: (1) Clinician confirmed -- set `outcome` to the condition entity if known, ask for `confirmed_by`. (2) Self-resolved -- set `outcome: self-resolved`. (3) Ruled out -- set `outcome: ruled-out: {reason}`, ask for reason. **Condition**: set `status: resolved`, update `last_updated`. Ask for attribution: same pattern. **Treatment**: set `status: completed`, set `stopped: {today}`, update `last_updated`. If no `response` recorded, ask for it now using vocabulary values from `treatment_response`. |

### Mutation rules

- Only modify mutable records (`immutable` trait must NOT be present)
- Always update `last_updated` to today on any modified record
- Use only vocabulary-spec.md values for controlled fields
- Do not create new records. This skill only updates existing ones.
- Do not modify encounter files (they are immutable)

After processing each item, move to the next. Keep responses tight -- one item, one question, one update.

## Phase 4: Bridge to Action (REQ-DORM-006)

After all items are triaged, check if any items were marked with option C (Resume) or had `flagged_for_gp: true` set during this session.

If yes:

> {N} items flagged for GP attention. Run /consultation-prep when you're ready to prepare for your next appointment.

If no items were flagged for GP:

> All items triaged. Nothing flagged for GP.

## Constraints

- **On-demand only**: Never run automatically. Never present the full list without `/dormancy-check` being invoked.
- **Under 2 minutes**: Keep the interaction lightweight. One question per item, no deep-dives. If the user wants to explore an item in depth, suggest they do so after the dormancy check completes.
- **No clinical assertions**: Surface what needs attention. Do not suggest diagnoses, treatment changes, or clinical actions. The agent observes; the clinician advises.
- **No new records**: This skill modifies existing records only. It does not create observations, encounters, or any other record type.
- **Vocabulary compliance**: All status transitions and field values must use controlled vocabulary from `data/vocabulary-spec.md`. Do not invent values.
- **Immutability contract**: Never modify records with the `immutable` trait (encounters, assessments, test-results, evidence).

## Valid Status Transitions

These are the only status changes this skill may make:

| Type | From | To | Via option |
|------|------|----|-----------|
| investigation | investigating | dormant | B |
| investigation | hypothesis | dormant | B |
| investigation | investigating | concluded | D |
| investigation | hypothesis | concluded | D |
| condition | managed | stable | B |
| condition | flaring | stable | B |
| condition | managed | resolved | D |
| condition | flaring | resolved | D |
| treatment | active | paused | B |
| treatment | active | completed | D |
