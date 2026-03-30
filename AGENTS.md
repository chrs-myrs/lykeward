# Lykeward Agent Guide

Personal health bible -- body-guardian. A temporal health journal with deferred classification, operated by AI agents on behalf of a single patient.

Lykeward captures health observations with minimal friction, lets structure emerge through AI suggestion and clinical confirmation, and generates GP consultation crib sheets automatically. The store tracks symptoms, conditions, treatments, test results, and investigation threads.

## Design Principles

1. **Capture first, classify later** -- observations are free text + date; classification fields are optional and added over time
2. **Co-occurrence is not causation** -- relationships are flagged for clinical review, never asserted as causal
3. **GPs want salience, not completeness** -- consultation views target 60-second scan time
4. **Chronic conditions don't have arcs** -- status snapshots (managed, flaring, stable), not lifecycle stages
5. **Patient observes, clinician diagnoses** -- two independent confidence axes: `observation_confidence` (how sure the patient is about what they noticed) and `relational_confidence` (whether a clinician has confirmed a connection)

## Knowledge Model Quick Reference

### Types

| Type | Purpose | Mutability | Zone |
|------|---------|------------|------|
| `entity` | Named thing with attributes | mutable | core |
| `observation` | Any health data point -- free text + date | mutable | core |
| `assessment` | Evaluative synthesis across records | immutable | research |
| `overview` | Portfolio-level summary | mutable | research |
| `encounter` | GP/specialist interaction -- prep + outcome | immutable | core |
| `condition` | Diagnostic label -- confirmed or suspected | mutable | core |
| `treatment` | Medication, supplement, therapy | mutable | core |
| `test-result` | Lab work, imaging, measurements | immutable | core |
| `investigation` | Diagnostic hypothesis thread | mutable | core |
| `evidence` | External documents (GP letters, referrals) | immutable | core |
| `measurement-series` | Time-series health data | append-only | core |

### Key Traits

| Trait | Contract |
|-------|----------|
| `immutable` | MUST NOT be modified after creation. New information creates new records. |
| `temporal` | Has date information -- `date` or `created`/`last_updated` required |
| `relational` | Connects to other records via entity/follows/references fields |
| `named` | Has unique identity -- `entity` field required |

### Observation Tags

`[symptom]` `[measurement]` `[episode]` `[daily-log]` `[concern]` `[question]` `[pattern]` `[improvement]` `[trigger]` `[medication-change]` `[shift]` `[insight]` `[challenge]` `[goal]` `[observation]` `[action-client]` `[action-owner]` `[decision]`

## Task Routing

| When the user says... | Action | Type | Key fields |
|-----------------------|--------|------|------------|
| "I noticed something" / "I have a headache" / describes a symptom | Create observation | `observation` | date, free text body, optional severity/body_system |
| "I had a GP appointment" / "Saw Dr X today" | Create encounter | `encounter` | date, method, provider; derive crib sheet before, record outcome after |
| "I submitted an eConsult" / "Sent an online consultation" | Create encounter | `encounter` | date, method: econsult, concerns; capture full submission text in body |
| "I'm seeing the GP tomorrow" / "Prepare for appointment" | Generate salience view | (query) | Read active conditions, recent observations, active treatments; produce 60-second summary |
| "Why does X happen?" / "I want to investigate Y" | Create investigation | `investigation` | question, entity, status: hypothesis |
| "The GP said I have Y" / "Diagnosed with Z" | Create or update condition | `condition` | entity, status, confirmed_by, diagnosed_date, body_system |
| "I started taking Z" / "New prescription" | Create treatment | `treatment` | name, type, status: active, started, dosage, targets |
| "Got test results" / "Blood work back" | Create test-result | `test-result` | date, test_type, values in body, encounter_ref if applicable |
| "Run a health check" | Run /knowledge-health | (skill) | Audits frontmatter, vocabulary, governance, referential integrity |
| "What meds am I on?" | Query treatments | (query) | Filter treatments where status: active |
| "What's the story with my headaches?" | Query investigation + linked observations | (query) | Read investigation entity, follow evidence array and linked observations |
| "What should I tell the GP about X?" | Generate concern summary | (query) | Gather observations tagged `[concern]` or `[question]`, plus related conditions |
| "Draft an eConsult" / "Submit online consultation" | Draft eConsult form | `encounter` | Follow `specs/processes/econsult-form.spec.md`. Load context, interview for gaps, draft fields (500 char limit each), save to `var/`. One issue per form. |

### Encounter Workflow (Two-Phase)

**Before the appointment** (user says "I'm seeing the GP tomorrow"):
1. Read active conditions, recent observations (last 30 days), active treatments
2. Read any investigations with status `investigating` or `hypothesis`
3. Read observations tagged `[concern]` or `[question]`
4. Generate a salience view: conditions by status, recent changes, open questions
5. Write to `var/` as a transient crib sheet

**After the appointment** (user says "Saw the GP today"):
1. Create an immutable encounter record with date, method, provider
2. Record outcomes, decisions, new prescriptions in the encounter body
3. Create/update conditions if diagnoses were made or changed
4. Create treatments if new medications were prescribed
5. Link observations that were discussed to the encounter

## Governance Zones

| Zone | Location | Level | Rules |
|------|----------|-------|-------|
| Core | `data/core/` | strict | Full provenance, confidence fields, source references, immutable signals, naming conventions |
| Research | `data/research/` | light | Naming conventions only |
| Internal | `data/internal/` | none | Scratch space, no rules |

**Cross-zone rules:**
- Strict-zone content MUST NOT cite none-zone content as provenance
- Content may be promoted from light to strict by adding full metadata
- Assessments spanning zones should reside in the light zone

## Controlled Vocabulary

All values MUST match `data/vocabulary-spec.md`. Key enumerations:

- **severity**: mild | moderate | severe
- **temporality**: acute | episodic | chronic | recurrent
- **observation_confidence**: certain | likely | uncertain
- **relational_confidence**: clinician-confirmed | self-assessed | unassessed
- **condition_status**: suspected | confirmed | managed | flaring | stable | remission | resolved
- **treatment_response**: helped | partially-helped | no-effect | worsened | side-effect
- **link_type**: co-occurrence | triggers | worsens | alleviates | side-effect | differential

Do not invent new values. Update `data/vocabulary-spec.md` if the domain genuinely needs extension.

## Creating Records

1. Determine the correct **type** from `data/SCHEMA.md`
2. Check the **governance zone** -- which `data/` subdirectory?
3. If strict zone: ensure provenance, confidence, source reference
4. Check for **duplicates** -- read existing files of the same type first
5. Assign sequential **ID** if the type uses IDs (ENC-YYYYMMDD-NNN, TEST-YYYYMMDD-NNN, DOC-NNN)
6. Every file MUST have `type` and `traits` frontmatter
7. Mutable types MUST update `last_updated` when changed
8. Immutable types MUST NOT be modified after creation

**File naming**: `{type}-{qualifier}.md`, kebab-case, all lowercase, hyphens not underscores.

## File Locations

| Path | Contents |
|------|----------|
| `PURPOSE.md` | Project mission and scope |
| `project.yaml` | Project configuration |
| `governance.yaml` | Governance zone definitions |
| `data/SCHEMA.md` | Type system and frontmatter fields |
| `data/vocabulary-spec.md` | Controlled vocabulary values |
| `data/sources.yaml` | External document source mappings (doc:// URI roots) |
| `data/core/` | Strict-zone records (observations, encounters, conditions, treatments, etc.) |
| `data/research/` | Light-zone records (assessments, overviews) |
| `data/internal/` | Scratch space |
| `ctxt/` | Agent context files |
| `ctxt/queries.md` | Query patterns |
| `ctxt/knowledge-model.md` | Detailed knowledge model reference |
| `specs/workspace/` | Workspace specifications |
| `var/` | Transient artifacts (crib sheets, audit reports) |
| `var/audits/` | Health check output |

## External Sources

External document sources are mapped via `data/sources.yaml` using `doc://` URI prefixes. Evidence records reference documents by URI (e.g. `doc://s-docs-current/Medical/filename.pdf`). The NHS App can be accessed via Claude for Chrome to extract GP records, prescriptions, test results, and appointment notes.

## Agent Behaviour

- **Always** check for duplicates before creating records
- **Always** use controlled vocabulary values from `data/vocabulary-spec.md`
- **Never** modify immutable records -- create new records instead
- **Never** assert causation -- flag co-occurrences for clinical review with `relational_confidence: unassessed` or `self-assessed`
- **Never** diagnose -- the patient observes, the clinician diagnoses
- **Ask** before creating a condition with status `confirmed` unless the user explicitly attributes it to a clinician
- **Prefer** creating observations first, then linking to conditions/investigations -- capture before classify

### Knowledge Write-Back

During ANY session — not just health check-ins — write to `data/core/` immediately when:

| Trigger | Action | Example |
|---------|--------|---------|
| **New condition identified** | Create condition record | "I have a bunion" → condition file, status: patient-reported |
| **New investigation question** | Create investigation record | "Why do I keep getting headaches?" → investigation, status: hypothesis |
| **Treatment side effect/response** | Update treatment file | "The new medication is giving me nausea" → update response field |
| **Clinically relevant negative finding** | Create observation with `[observation]` tag | "I've never had asthma" → observation recording the negative |
| **Factual correction** | Update mutable record | "Actually it started in January not February" → update date field |

**Timing**: Write at the point of discovery, not at session end. If a health fact surfaces during a debate, research, or freeform conversation, record it immediately.

**Threshold**: Only patient-specific findings cross into `data/core/`. General medical knowledge, research conclusions, and debate outputs stay in `var/`.

### Provenance Guardrails

1. **Patient-stated only** — only write to `data/core/` what the patient has explicitly stated or confirmed. AI hypotheses, debate persona claims, and general research do NOT become patient records
2. **Source attribution** — every write-back must trace to either (a) something the patient said, or (b) a clinician's documented statement. "The debate concluded X" is not a valid source
3. **No assumed treatments** — never create or update a treatment record based on inference. Only the patient can confirm what they take
4. **Hypotheses stay hypotheses** — investigation records from exploration use "candidate", "possible", "may contribute" — never definitive language
5. **Negative findings need confirmation** — don't record "patient doesn't have X" unless the patient explicitly said so. Absence of mention ≠ negative finding
6. **Gate on uncertainty** — if unsure whether the patient said something or it was inferred during research/debate, ASK before writing to `data/core/`. The cost of asking is low; the cost of enshrining a wrong fact is high
