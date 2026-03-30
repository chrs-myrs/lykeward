# Health Investigation

Create, review, update, and list diagnostic investigation threads -- the "why does X happen?" reasoning trails that connect symptoms, tests, encounters, and conditions.

## Usage

```
/investigate-health                         # Start a new investigation
/investigate-health hand-tingling           # Review existing investigation
/investigate-health hand-tingling --update  # Add evidence or clinician input
/investigate-health --list                  # Summary table of all investigations
```

## Context Loading

Before any operation, read:

1. `data/SCHEMA.md` -- type definitions and required fields
2. `data/vocabulary-spec.md` -- controlled vocabulary (especially `relational_confidence`, `observation_confidence`, `body_system`)
3. `ctxt/knowledge-model.md` -- relationship map and governance zones

## Mode Detection

Parse the invocation arguments:

- **No arguments** --> New investigation (Phase: Create)
- **`--list`** --> Listing mode (Phase: List)
- **Entity name only** (e.g. `hand-tingling`) --> Review mode (Phase: Review)
- **Entity name + `--update`** --> Update mode (Phase: Update)

---

## Phase: Create (REQ-INV-001)

### Step 1: Gather the Question

Ask the user:

> "What's the question you want to investigate? State it naturally -- e.g. 'Why do my hands tingle?' or 'What's causing my persistent cough?'"

The question IS the hypothesis. Do not rephrase it into clinical language.

### Step 2: Initial Context

Ask these follow-up questions (conversationally, not as a form):

- "When did you first notice this?"
- "What symptoms are involved?"
- "What do you think might be causing it?"
- "Does it relate to any conditions or treatments you're already tracking?"

Accept whatever the user offers. Do not push for completeness -- capture first, classify later.

### Step 3: Derive Entity Name

Generate a kebab-case entity name from the question. Examples:
- "Why do my hands tingle?" --> `hand-tingling`
- "What's causing my persistent cough?" --> `persistent-cough`
- "Is my fatigue related to B12?" --> `fatigue-b12-link`

### Step 4: Search for Relevant Observations

Read files in `data/core/` with `type: observation`. Look for:
- Matching `body_system` if one is apparent from the question
- Temporal overlap with the reported onset
- Keyword matches in observation body text

Present any candidates to the user:

> "I found these observations that might be relevant to this investigation:
> - observation-2026-01-15-hand-numbness.md (neurological, 2026-01-15)
> - observation-2026-02-03-finger-tingling.md (neurological, 2026-02-03)
>
> Want me to link any of these as initial evidence?"

### Step 5: Create the Investigation File

Write to `data/core/investigation-{entity}.md`:

```yaml
---
type: investigation
traits: [temporal, relational]
entity: {entity}
question: "{the user's exact question}"
status: hypothesis
body_system: {if identifiable, otherwise omit}
created: {today YYYY-MM-DD}
last_updated: {today YYYY-MM-DD}
evidence:
  - id: {observation filename}
    role: presenting-symptom
    observation_confidence: {from the observation, or omit}
    relational_confidence: unassessed
---

# {Question}

## Possible explanations (your thinking, not diagnosis)

{User's initial thoughts on what could cause this, as a bullet list. If the user offered no thoughts, write: "No initial thoughts recorded -- to be developed as evidence accumulates."}

## Supporting observations

{For each possibility listed above, note which observations support it. If none yet: "No supporting observations linked yet."}

## Contrary observations

{Observations that weaken any possibility. If none yet: "No contrary observations identified yet."}

## Latest clinical input

No clinical input recorded yet.

## Next steps

{What would help narrow it down -- specific tests, tracking a pattern, seeing a specialist, mentioning it at next GP appointment. Derived from the conversation. If nothing specific: "Track occurrences and mention at next GP appointment."}
```

### Step 5 Rules

- Every evidence entry gets `relational_confidence: unassessed` by default (REQ-INV-006)
- The `role` field uses: `presenting-symptom`, `trigger`, `correlation`
- Do NOT suggest specific conditions to investigate -- the user decides the question (spec non-requirement)
- Do NOT populate "Latest clinical input" from patient-reported information -- only from encounter records where a clinician has commented

---

## Phase: Review (REQ-INV-005)

When invoked with an existing investigation entity name.

### Step 1: Load Investigation

Read `data/core/investigation-{entity}.md`. If not found, inform the user and offer to create it.

### Step 2: Gather Related Records

For each entry in the `evidence` array, read the referenced file from `data/core/`.

Also search for:
- Encounter records that mention this investigation or its `body_system`
- Test results linked to related encounters
- Condition records with matching `body_system`
- Observations created since `last_updated` that match the investigation's body system or keywords

### Step 3: Present Chronological Review

Present the investigation state as a chronological evidence trail:

```
## Investigation: {question}
Status: {status} | Created: {created} | Last updated: {last_updated}

### Evidence Trail (chronological)

**{date}** -- {source attribution}
{summary of what this record says}
Role: {role} | Confidence: observation={observation_confidence}, relational={relational_confidence}

**{date}** -- {source attribution}
...

### What's Changed Since Last Review

{List records added or updated since last_updated. If nothing: "No new evidence since last review."}

### Possible Explanations

{Current state of the reasoning from the file}

### Latest Clinical Input

{From the investigation file. Reproduce exactly -- do not synthesise or editorialise.}

### Suggested Next Steps

{From the investigation file, plus any new suggestions based on what has changed}
```

### Step 3 Rules

- **Source attribution**: State whether each piece of evidence is "patient observation", "clinician input", "test result", or "external document"
- **Chronological order**: Oldest first
- The review **summarises** -- it does NOT evaluate or rank possibilities
- Do NOT generate a "what seems most likely" assessment (REQ-INV-005)
- Do NOT assert causation between linked records (REQ-INV-006)
- Present relationships as "co-occurs with" or "noted around the same time as", never "caused by"

---

## Phase: Update

When invoked with an entity name and `--update`.

### Step 1: Load Investigation

Read `data/core/investigation-{entity}.md`.

### Step 2: Determine Update Type

Ask:

> "What would you like to update?
> 1. Link new evidence (observation, test result, or document)
> 2. Record clinician input from an appointment
> 3. Update the reasoning trail
> 4. Change investigation status"

### Option 1: Link New Evidence (REQ-INV-002)

Search `data/core/` for candidate records. Present them. For each record the user selects:

- Ask: "What role does this play? (presenting-symptom, trigger, correlation)"
- Set `relational_confidence: unassessed` (default per REQ-INV-006)
- Add to the `evidence` array in frontmatter
- Update `last_updated`

If the user wants to link a record that doesn't exist yet, offer to create it as an observation first, then link it.

### Option 2: Record Clinician Input (REQ-INV-007)

This records what a clinician said about this investigation. Ask:

- "What date was the appointment?"
- "Which clinician?"
- "What did they say about this investigation?"

Add a structured entry to the "Latest clinical input" section:

```markdown
## Latest clinical input

### {YYYY-MM-DD} -- {Clinician name}

{What the clinician said, in the user's words}
```

If there are existing entries, append the new one chronologically (newest last). Do not remove previous entries.

Also check: does the clinician input confirm or rule out any possibility? If so, ask:
- "Did {clinician} confirm or rule out any of the current possibilities?"
- If confirmed: offer to update status to `concluded` and set `outcome` to the condition entity
- If ruled out: offer to note "ruled-out: {reason}" in the outcome field
- If a relational link was confirmed by the clinician: update that evidence entry's `relational_confidence` from `unassessed` or `self-assessed` to `clinician-confirmed`

### Option 3: Update Reasoning Trail

Present the current "Possible explanations", "Supporting observations", and "Contrary observations" sections. Let the user add, modify, or remove entries. Write changes back to the file. Update `last_updated`.

### Option 4: Change Status (REQ-INV-004)

Present current status and valid transitions:

| From | To | When |
|------|----|------|
| `hypothesis` | `investigating` | Actively gathering evidence |
| `investigating` | `concluded` | Condition confirmed or definitively ruled out |
| `investigating` | `dormant` | No new evidence in 3+ months |
| `dormant` | `investigating` | New evidence surfaces |

If transitioning to `concluded`:
- Ask for the outcome: condition entity name (if confirmed) or "ruled-out: {reason}"
- Set the `outcome` field

Update `status` and `last_updated` in frontmatter.

---

## Phase: List (REQ-INV-008)

### Step 1: Read All Investigations

Read all files in `data/core/` matching `investigation-*.md`. Parse frontmatter from each.

### Step 2: Present Summary Table

```
| Investigation | Question | Status | Last Updated | Evidence |
|---------------|----------|--------|--------------|----------|
| {entity}      | {question} | {status} | {last_updated} | {count of evidence array entries} |
```

Sort by status priority: `investigating` first, then `hypothesis`, then `dormant`, then `concluded`.

Do NOT enter review mode. This is a quick-glance summary only.

If no investigations exist, say so and offer to create one.

---

## Governance Rules

These apply to ALL phases:

### Co-occurrence, Not Causation (REQ-INV-006)

- All evidence links default to `relational_confidence: unassessed`
- Only upgrade to `self-assessed` when the patient explicitly states they believe a connection exists
- Only upgrade to `clinician-confirmed` when recording clinician input that confirms the link
- Never use causal language: "causes", "leads to", "results in"
- Use relational language: "co-occurs with", "noted around the same time", "possibly related"

### Clinical Boundary

- The "Possible explanations" section contains patient hypotheses, not differential diagnoses
- The "Latest clinical input" section is populated ONLY from clinician encounters -- never from patient speculation
- The agent does NOT suggest conditions to investigate
- The agent does NOT rank possibilities by likelihood
- The agent does NOT offer clinical opinions on the evidence

### Data Integrity

- Investigation files live in `data/core/` (strict governance zone)
- All frontmatter must comply with `data/SCHEMA.md`
- All vocabulary values must match `data/vocabulary-spec.md`
- Check for duplicate investigations before creating (same entity name or very similar question)
- Update `last_updated` on every modification

### Evidence Roles

| Role | When to Use |
|------|-------------|
| `presenting-symptom` | The symptom that prompted the investigation |
| `trigger` | Something the patient believes triggers or worsens the symptom |
| `correlation` | Temporally or thematically related but no specific role identified |

### Sensitive Data

- Investigation files may contain speculative health hypotheses not shared with clinicians
- Do not include investigation content in commit messages or branch names
- Do not volunteer investigation details outside explicit health queries
