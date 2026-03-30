# Consultation Prep

Prepare for GP consultations and record encounter outcomes. Generates a 60-second crib sheet from the knowledge store, then captures what happened afterwards.

## Usage

```
/consultation-prep                          # Full flow: prep then post-appointment
/consultation-prep --record-only            # Skip prep, record an encounter directly (REQ-PREP-008)
/consultation-prep --method=econsult        # eConsult: plain text output, no markdown
/consultation-prep --method=phone           # Phone/video: shorter summary
/consultation-prep --method=face-to-face    # Default: full crib sheet
```

## Context Loading

Before any operation, read these files:

1. `data/SCHEMA.md` -- type definitions and field requirements
2. `data/vocabulary-spec.md` -- controlled vocabulary (condition_status, relational_confidence, etc.)
3. `governance.yaml` -- zone rules for data/core/ (strict zone)
4. `ctxt/knowledge-model.md` -- type system, relationship map, common workflows

## Mode Selection

Determine the operating mode:

- If `--record-only` flag is present, skip to **Phase 3: Post-Appointment Recording**
- Otherwise, begin at **Phase 1: Gather Intent**

Determine the encounter method (defaults to `face-to-face`):

| Method | Format | Characteristics |
|--------|--------|----------------|
| `face-to-face` | Full markdown crib sheet | All sections, full detail |
| `econsult` | Plain text, no markdown formatting | Pasteable into eConsult form, no headers/bullets/bold |
| `phone` | Shortened markdown | Conditions + medications + one-liner concerns only |
| `video` | Shortened markdown | Same as phone |

## Phase 1: Gather Intent (REQ-PREP-002)

Ask the user:

> What's the main reason for this appointment?

Wait for the response. This becomes the one-sentence summary at the top of the crib sheet.

Then ask:

> Anything else you want to raise while you're there?

These become the "Patient concerns" section. If the user says nothing, leave the section empty.

## Phase 2: Generate Salience View (REQ-PREP-001, REQ-PREP-003, REQ-PREP-004)

### 2a: Pull from Knowledge Store

Read files from `data/core/` to gather:

1. **Last encounter**: Find the most recent file with `type: encounter`. Note its date -- this is the "since last visit" boundary.
2. **Observations since last encounter**: All files with `type: observation` where `date` is after the last encounter date. Highlight any with `flagged_for_gp: true`.
3. **Active conditions**: All files with `type: condition` where `status` is NOT `resolved`.
4. **Active treatments**: All files with `type: treatment` where `status: active`. Include `dosage`, `response`, and `response_notes`.
5. **Active investigations**: All files with `type: investigation` where `status` is `hypothesis` or `investigating`.
6. **Recent test results**: All files with `type: test-result` from the last 90 days.
7. **Flagged-for-review links**: Any observation or condition with links where `flagged_for_review: true`.

### 2b: Apply Confidence Boundaries (REQ-PREP-004)

When composing the crib sheet, respect the three confidence tiers strictly:

- **`clinician-confirmed`**: State the relationship as established fact. Example: "Vitamin D deficiency (confirmed by Dr Smith, 2025-06-15)"
- **`self-assessed`**: Frame as patient-noticed correlation. Example: "Patient notes: fatigue seems to coincide with B12 dip". These go in the "Patient-noticed patterns" section, not mixed into clinical sections.
- **`unassessed`**: Do NOT include in the crib sheet at all. These have no place in a clinical-facing document.

### 2c: Compose the Crib Sheet

Assemble in this exact section order (GP priority):

```
# GP Consultation -- {YYYY-MM-DD}

## Why I'm here
{One-sentence summary from Phase 1}

## What's changed since last visit ({last encounter date})
{Observations since last encounter, most recent first}
{Highlight flagged_for_gp items}

## Active conditions
{Each condition: name, status, body_system, last_updated}
{Clinician-confirmed details stated as fact}

## Current medications
{Each active treatment: name, dosage, started, response_notes if any}

## What I want to discuss
{Additional concerns from Phase 1, if any}

## Patient-noticed patterns (unverified)
{Self-assessed links only -- framed as "I've noticed X seems to coincide with Y"}
{Explicitly state: these are self-reported correlations, not clinically verified}

## Active investigations
{Each: question, status, key evidence so far}

## Recent test results
{Each: date, test_type, key findings from body text}
```

### 2d: Method Adaptation (REQ-PREP-007)

**eConsult format**: Strip all markdown formatting. No headers, no bold, no bullets. Use line breaks and plain text labels instead. Example:

```
REASON FOR CONTACT
{summary}

WHAT'S CHANGED SINCE LAST VISIT ({date})
- {observation 1}
- {observation 2}

ACTIVE CONDITIONS
{condition}: {status}
...
```

Use dashes for list items (eConsult systems handle these better than bullets). Keep total length under 2000 characters where possible -- eConsult forms have limits.

**Phone/video format**: Include only:
- Why I'm here (one sentence)
- Active conditions (name + status, one line each)
- Current medications (name + dosage, one line each)
- What I want to discuss (brief list)

Omit: What's changed, Patient-noticed patterns, Active investigations, Recent test results. The user can refer to these verbally if needed.

### 2e: Save as Immutable Evidence (REQ-PREP-006)

Write the crib sheet to: `data/core/crib-sheet-{YYYY-MM-DD}.md`

Frontmatter:

```yaml
---
type: evidence
traits: [named, temporal, immutable]
id: DOC-{NNN}
date: {YYYY-MM-DD}
source: lykeward-consultation-prep
category: consultation-crib-sheet
encounter_method: {method}
---
```

Assign the next sequential DOC-NNN ID by checking existing evidence files.

This file is **immutable** once written. It preserves what the patient intended to discuss, distinct from what actually happened. Do not modify it after creation.

Display the crib sheet to the user. Then ask:

> The crib sheet has been saved. Want to proceed to post-appointment recording now, or come back later?

If the user wants to defer, end the skill here. They can return with `/consultation-prep --record-only`.

## Phase 3: Post-Appointment Recording (REQ-PREP-005)

This phase has two sub-phases. Phase 3b can be deferred.

### 3a: Quick Capture

Ask:

> How did it go? What did the GP say?

Let the user talk freely. From their response, extract and confirm:

- **Method**: gp-appointment / econsult / utc / specialist / phone / video
- **Provider**: Clinician name and practice (if mentioned)
- **Key outcomes**: What was discussed, decided, prescribed, referred
- **Next steps**: Follow-ups, tests ordered, referrals made

Create the encounter record in `data/core/encounter-{YYYY-MM-DD}.md`:

```yaml
---
type: encounter
traits: [temporal, immutable, relational]
id: ENC-{YYYYMMDD}-{NNN}
date: {YYYY-MM-DD}
method: {method}
provider: {clinician name and practice, if known}
concerns: [{list of presenting concerns}]
follows: {previous encounter ID, if known}
crib_sheet_ref: {DOC-NNN of the crib sheet, if one was generated}
---
```

Body: narrative summary of what happened, what was decided, what's next.

Assign the next sequential ENC-YYYYMMDD-NNN ID. If a crib sheet was generated in Phase 2, reference it via `crib_sheet_ref`. Link to the previous encounter via `follows` if one exists.

This record is **immutable** once created.

Tell the user:

> Encounter recorded. Want to update conditions, treatments, or investigations now, or come back to that later?

If deferring, end here. The user can return and ask to update specific records manually.

### 3b: Detailed Update (deferrable)

Walk through each outcome from Phase 3a and create or update records as needed.

**New conditions** from the encounter:
- Default to `status: suspected` unless the user explicitly says "the GP diagnosed X" or "Dr Y confirmed X"
- If the user attributes a diagnosis to a clinician: set `status: confirmed`, `confirmed_by: {clinician}`, `diagnosed_date: {date}`
- Create in `data/core/condition-{name}.md`

**Condition status changes** (REQ-PREP-005 clinician attribution rule):
- Any change to `confirmed` or `resolved` MUST include clinician attribution
- Ask: "Who confirmed/resolved this? Was it {provider from encounter}?"
- Set `confirmed_by` field and note the encounter reference in the body
- Other status changes (flaring, stable, managed, etc.) do not require attribution

**New or changed treatments**:
- New prescriptions: create `data/core/treatment-{name}.md` with `status: active`, `prescribed_by`, `targets`
- Dosage changes: update existing treatment file, set new `dosage`, update `last_updated`
- Stopped treatments: set `status: stopped`, `stopped: {date}`

**New investigations**:
- Create `data/core/investigation-{slug}.md` with `status: investigating`, link initial evidence

**Test results** (if results were given at the appointment):
- Create `data/core/test-result-{YYYY-MM-DD}-{qualifier}.md`
- Set `encounter_ref` to the encounter ID

**Tests ordered** (results pending):
- Note in the encounter body text that tests were ordered
- Do NOT create test-result records until results arrive

For every record created or updated, confirm the changes with the user before writing.

## Governance Compliance

All files created by this skill go in `data/core/` (strict zone). Every file must have:

- Full frontmatter per SCHEMA.md type definitions
- Controlled vocabulary values matching `data/vocabulary-spec.md` exactly
- Provenance (source references where applicable)
- Confidence fields where the schema requires them

Before creating any file:
1. Check for duplicates (existing condition/treatment with same name)
2. Assign the next sequential ID for types that use IDs
3. Verify all vocabulary values against `data/vocabulary-spec.md`

## Confidence Boundary Rules

These rules apply throughout the entire skill:

| Scenario | Action | Example |
|----------|--------|---------|
| User says "I think X causes Y" | `relational_confidence: self-assessed` | "I've noticed headaches coincide with screen time" |
| User says "the GP said X causes Y" | `relational_confidence: clinician-confirmed` | "Dr Smith confirmed migraine triggers" |
| Agent notices a correlation | `relational_confidence: unassessed`, flag for review | Pattern detected but not discussed |
| User says "I have X" (no clinician) | `status: suspected` | "I think I have IBS" |
| User says "the GP diagnosed X" | `status: confirmed`, `confirmed_by` required | "Dr Jones diagnosed asthma" |

Never promote confidence without explicit clinician attribution from the user.

## What This Skill Does NOT Do

- Provide medical advice or suggest diagnoses
- Assert causal relationships between health events
- Replace the GP's own clinical notes
- Include the full health history -- only what's relevant NOW
- Modify any immutable record after creation
