# Knowledge Model Context

Detailed reference for agents operating the Lykeward health knowledge store. Load this file when performing knowledge operations -- creating records, querying patterns, generating consultation views, or auditing store health.

## Type System

### entity

Anchor record for a named thing (person, body part, system).

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `entity` |
| `traits` | yes | `[named, temporal, relational]` |
| `entity` | yes | Kebab-case identifier |
| `created` | yes | YYYY-MM-DD |
| `last_updated` | yes | YYYY-MM-DD |

Mutable. Update `last_updated` on change.

### observation

Any health data point. Free text body + date. Classification fields are optional and deferred -- capture first, classify later.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `observation` |
| `traits` | yes | `[temporal, relational]` |
| `date` | yes | YYYY-MM-DD |
| `severity` | no | mild / moderate / severe |
| `temporality` | no | acute / episodic / chronic / recurrent |
| `body_system` | no | neurological / respiratory / musculoskeletal / dermatological / mental-health / gastrointestinal / cardiovascular / endocrine / immunological / sensory |
| `observation_confidence` | no | certain / likely / uncertain |
| `links` | no | Array of {to, type, relational_confidence, flagged_for_review} |

Mutable. Tags in body text: `[symptom]` `[measurement]` `[episode]` `[daily-log]` `[concern]` `[question]` `[pattern]` `[improvement]` `[trigger]` `[medication-change]`.

### encounter

GP or specialist interaction. Two-phase: derive crib sheet before, record outcome after. Immutable once created.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `encounter` |
| `traits` | yes | `[temporal, immutable, relational]` |
| `id` | yes | ENC-YYYYMMDD-NNN |
| `date` | yes | YYYY-MM-DD |
| `method` | yes | gp-appointment / econsult / utc / specialist / phone / video |
| `provider` | no | Clinician name and practice |
| `concerns` | no | List of presenting concerns |
| `follows` | no | Previous encounter reference |

Immutable. Do not modify after creation.

### condition

Diagnostic label. Status snapshots, not lifecycle arcs. Clinician confirms; patient suspects.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `condition` |
| `traits` | yes | `[named, temporal, relational]` |
| `entity` | yes | Kebab-case condition name |
| `status` | yes | suspected / confirmed / managed / flaring / stable / remission / resolved |
| `diagnosed_date` | no | YYYY-MM-DD when confirmed |
| `body_system` | yes | Primary body system |
| `confirmed_by` | no | Clinician who confirmed |
| `created` | yes | YYYY-MM-DD |
| `last_updated` | yes | YYYY-MM-DD |

Mutable. Update `last_updated` and `status` when condition changes.

### treatment

Medication, supplement, therapy, lifestyle change, or procedure.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `treatment` |
| `traits` | yes | `[temporal, relational]` |
| `name` | yes | Treatment name |
| `treatment_type` | yes | medication / supplement / therapy / lifestyle / procedure |
| `status` | yes | active / paused / stopped / completed |
| `started` | yes | YYYY-MM-DD |
| `stopped` | no | YYYY-MM-DD |
| `dosage` | no | Current dosage |
| `prescribed_by` | no | Clinician |
| `targets` | no | Condition or symptom this treats |
| `response` | no | helped / partially-helped / no-effect / worsened / side-effect |
| `response_notes` | no | Freeform response description |

Mutable. Update status and response over time.

### test-result

Lab work, imaging, measurements, assessments, screenings.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `test-result` |
| `traits` | yes | `[temporal, immutable, relational]` |
| `id` | yes | TEST-YYYYMMDD-NNN |
| `date` | yes | YYYY-MM-DD |
| `test_type` | yes | blood / imaging / measurement / assessment / screening |
| `ordered_by` | no | Clinician who ordered |
| `encounter_ref` | no | Encounter that triggered this |

Immutable. Values and interpretation in body text.

### investigation

Diagnostic hypothesis thread. Patient initiates with a question; evidence accumulates; clinician informs conclusion.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `investigation` |
| `traits` | yes | `[temporal, relational]` |
| `entity` | yes | Kebab-case investigation name |
| `question` | yes | The driving question (e.g. "why do my hands tingle?") |
| `status` | yes | hypothesis / investigating / concluded / dormant |
| `body_system` | no | Primary body system |
| `outcome` | no | Condition entity if confirmed, or "ruled-out: reason" |
| `created` | yes | YYYY-MM-DD |
| `last_updated` | yes | YYYY-MM-DD |
| `evidence` | no | Array of {id, role, observation_confidence, relational_confidence} |

Mutable. Accumulates evidence over time. Transitions through statuses as clinical information arrives.

### evidence

External documents. GP letters, referral forms, discharge letters, NHS records.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `evidence` |
| `traits` | yes | `[named, temporal, immutable]` |
| `id` | yes | DOC-NNN |
| `date` | yes | Document date |
| `source` | yes | Who produced this |
| `category` | yes | gp-letter / referral / discharge / test-report / prescription / insurance |

Immutable. Transcribed or summarised content in body.

### assessment

Evaluative synthesis across multiple records.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `assessment` |
| `traits` | yes | `[temporal, immutable, relational]` |
| `entity` | yes | Subject being assessed |
| `date` | yes | YYYY-MM-DD |
| `sessions_covered` | no | List of session numbers |

Immutable. Lives in `data/research/` (light zone).

### overview

Portfolio or practice-level summary.

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `overview` |
| `traits` | yes | `[temporal, relational]` |
| `created` | yes | YYYY-MM-DD |
| `last_updated` | yes | YYYY-MM-DD |

Mutable. Lives in `data/research/` (light zone).

### measurement-series

Time-series health data (blood pressure readings, weight, B12 levels, eye prescriptions).

| Field | Required | Description |
|-------|----------|-------------|
| `type` | yes | `measurement-series` |
| `traits` | yes | `[temporal, relational]` |
| `metric` | yes | What is being measured |
| `unit` | no | mmHg, kg, pmol/L, etc. |
| `body_system` | no | Related body system |
| `created` | yes | YYYY-MM-DD |
| `last_updated` | yes | YYYY-MM-DD |

Append-only. New readings added to body, never modify existing entries.

## Relationship Map

```
observation ──links──> condition, treatment, investigation
encounter ──concerns──> condition, observation
encounter ──follows──> encounter (chain)
condition ──related──> treatment, investigation, observation
treatment ──targets──> condition
test-result ──encounter_ref──> encounter
investigation ──evidence──> observation, test-result, evidence
assessment ──entity──> any named record
measurement-series ──related──> condition, treatment
evidence ──referenced by──> investigation, encounter
```

## Governance Zone Map

| Zone | Directory | Level | Requirements |
|------|-----------|-------|--------------|
| Core | `data/core/` | strict | provenance, confidence fields, source references, immutable signals, naming conventions |
| Research | `data/research/` | light | naming conventions |
| Internal | `data/internal/` | none | scratch space |

**Placement guide:**
- Observations, encounters, conditions, treatments, test-results, investigations, evidence, measurement-series --> `data/core/`
- Assessments, overviews --> `data/research/`
- Drafts, scratch notes, working summaries --> `data/internal/`

## Common Workflows

### Recording an observation

1. User describes a symptom, measurement, or health event
2. Create file in `data/core/` as `observation-YYYY-MM-DD-{qualifier}.md`
3. Set required frontmatter: type, traits, date
4. Body: free text description, relevant tags
5. Optional: suggest severity, body_system, temporality -- but do not require them
6. If related to known condition/investigation, add links with `relational_confidence: self-assessed`

### Creating an encounter (after appointment)

1. Assign next ID: ENC-YYYYMMDD-NNN
2. Create in `data/core/` as `encounter-YYYY-MM-DD.md`
3. Record: method, provider, concerns discussed, outcomes, decisions
4. Create follow-on records: new conditions, treatments, test-results as needed
5. Link related observations that were discussed

### Starting an investigation

1. User asks "why does X happen?" or "I want to investigate Y"
2. Create in `data/core/` as `investigation-{question-slug}.md`
3. Set question, status: hypothesis, body_system if known
4. Link existing observations as initial evidence
5. As evidence accumulates, update evidence array and status

### Generating a salience view (pre-appointment)

1. Read all conditions where status is not `resolved`
2. Read observations from last 30 days
3. Read active treatments (status: active)
4. Read investigations with status `investigating` or `hypothesis`
5. Read observations tagged `[concern]` or `[question]`
6. Compose a 60-second scan document grouped by urgency:
   - Active concerns and open questions
   - Recent changes (new symptoms, medication changes, status changes)
   - Current medication list
   - Open investigations
7. Write to `var/salience-view-YYYY-MM-DD.md`

### Updating a condition status

1. Read existing condition file
2. Update `status` field to new value from vocabulary
3. Update `last_updated`
4. If confirming: set `confirmed_by`, `diagnosed_date`
5. If the status change came from an encounter, note the encounter reference in body

### Querying active medications

1. Read all files in `data/core/` with `type: treatment`
2. Filter where `status: active`
3. Present: name, dosage, started date, targets, prescribed_by
