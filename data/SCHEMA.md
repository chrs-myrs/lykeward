# Knowledge Store Schema

Domain: personal-health
Extends: core

## Types

| Type | Purpose | Traits | Mutability |
|------|---------|--------|------------|
| `entity` | Named thing with attributes — the anchor | named, temporal, relational | mutable |
| `observation` | Any health data point — free text + date. No required classification at capture time. | temporal, relational | mutable |
| `assessment` | Evaluative synthesis across multiple records | temporal, immutable, relational | immutable |
| `overview` | Portfolio or practice-level summary | temporal, relational | mutable |
| `encounter` | GP/specialist interaction — prep derived from store, outcome recorded back | temporal, immutable, relational | immutable |
| `condition` | Diagnostic label — confirmed or suspected by clinician. Status snapshots, no lifecycle arcs. | named, temporal, relational | mutable |
| `treatment` | Medication, supplement, therapy — tracks response over time | temporal, relational | mutable |
| `test-result` | Lab work, imaging, measurements — structured values with clinical context | temporal, immutable, relational | immutable |
| `investigation` | Diagnostic hypothesis — "why does X happen?" Patient initiates, clinician informs. | temporal, relational | mutable |
| `evidence` | External documents — GP letters, referral forms, discharge letters, NHS records | named, temporal, immutable | immutable |
| `measurement-series` | Time-series health data — BP, weight, B12 levels, eye prescriptions | temporal, relational | append-only |
| `genetic-profile` | Genomic test anchor — WGS, genotyping, microbiome sequencing | named, temporal, relational | mutable |

## Frontmatter Fields

### entity

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `entity` |
| `traits` | Yes | `[named, temporal, relational]` |
| `entity` | Yes | Kebab-case identifier |
| `created` | Yes | YYYY-MM-DD |
| `last_updated` | Yes | YYYY-MM-DD |

### observation

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `observation` |
| `traits` | Yes | `[temporal, relational]` |
| `date` | Yes | YYYY-MM-DD |
| `severity` | No | mild | moderate | severe (patient-assessed) |
| `temporality` | No | acute | episodic | chronic | recurrent (AI suggests) |
| `body_system` | No | neurological | respiratory | musculoskeletal | dermatological | mental-health | gastrointestinal | cardiovascular | endocrine | immunological | sensory | urological | oral | hepatic | genetic |
| `observation_confidence` | No | certain | likely | uncertain |
| `flagged_for_gp` | No | Boolean — true if this observation should be raised at next GP consultation. Set by /health-check-in clinical flags. |
| `links` | No | Array of {to, type, relational_confidence, flagged_for_review} |

### assessment

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `assessment` |
| `traits` | Yes | `[temporal, immutable, relational]` |
| `entity` | Yes | Subject being assessed |
| `date` | Yes | YYYY-MM-DD |
| `sessions_covered` | No | List of session numbers |

### overview

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `overview` |
| `traits` | Yes | `[temporal, relational]` |
| `created` | Yes | YYYY-MM-DD |
| `last_updated` | Yes | YYYY-MM-DD |

### encounter

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `encounter` |
| `traits` | Yes | `[temporal, immutable, relational]` |
| `id` | Yes | ENC-YYYYMMDD-NNN |
| `date` | Yes | YYYY-MM-DD |
| `method` | Yes | gp-appointment | econsult | utc | specialist | phone | video |
| `provider` | No | Clinician name and practice |
| `concerns` | No | List of presenting concerns |
| `follows` | No | Previous encounter reference |

### condition

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `condition` |
| `traits` | Yes | `[named, temporal, relational]` |
| `entity` | Yes | Kebab-case condition name |
| `status` | Yes | suspected | confirmed | managed | flaring | stable | remission | resolved |
| `diagnosed_date` | No | YYYY-MM-DD when confirmed |
| `body_system` | Yes | Primary body system |
| `confirmed_by` | No | Clinician who confirmed |
| `created` | Yes | YYYY-MM-DD |
| `last_updated` | Yes | YYYY-MM-DD |

### treatment

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `treatment` |
| `traits` | Yes | `[temporal, relational]` |
| `name` | Yes | Treatment name |
| `treatment_type` | Yes | medication | supplement | therapy | lifestyle | procedure |
| `status` | Yes | active | paused | stopped | completed |
| `started` | Yes | YYYY-MM-DD |
| `stopped` | No | YYYY-MM-DD |
| `dosage` | No | Current dosage |
| `prescribed_by` | No | Clinician |
| `targets` | No | Condition or symptom this treats |
| `response` | No | helped | partially-helped | no-effect | worsened | side-effect |
| `response_notes` | No | Freeform response description |

### test-result

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `test-result` |
| `traits` | Yes | `[temporal, immutable, relational]` |
| `id` | Yes | TEST-YYYYMMDD-NNN |
| `date` | Yes | YYYY-MM-DD |
| `test_type` | Yes | blood | imaging | measurement | assessment | screening | genetic |
| `ordered_by` | No | Clinician who ordered |
| `encounter_ref` | No | Encounter that triggered this |

### investigation

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `investigation` |
| `traits` | Yes | `[temporal, relational]` |
| `entity` | Yes | Kebab-case investigation name |
| `question` | Yes | The driving question — e.g. 'why do my hands tingle?' |
| `status` | Yes | hypothesis | investigating | concluded | dormant |
| `body_system` | No | Primary body system |
| `outcome` | No | Condition entity if confirmed, or 'ruled-out: reason' |
| `created` | Yes | YYYY-MM-DD |
| `last_updated` | Yes | YYYY-MM-DD |
| `evidence` | No | Array of {id, role, observation_confidence, relational_confidence} |

### evidence

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `evidence` |
| `traits` | Yes | `[named, temporal, immutable]` |
| `id` | Yes | DOC-NNN |
| `date` | Yes | Document date |
| `source` | Yes | Who produced this |
| `category` | Yes | gp-letter | referral | discharge | test-report | prescription | insurance |

### measurement-series

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `measurement-series` |
| `traits` | Yes | `[temporal, relational]` |
| `metric` | Yes | What is being measured |
| `unit` | No | mmHg, kg, pmol/L, etc. |
| `body_system` | No | Related body system |
| `created` | Yes | YYYY-MM-DD |
| `last_updated` | Yes | YYYY-MM-DD |

### genetic-profile

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always `genetic-profile` |
| `traits` | Yes | `[named, temporal, relational]` |
| `entity` | Yes | Kebab-case identifier (e.g. `nebula-wgs-2021`) |
| `source` | Yes | Provider (e.g. `nebula-genomics`) |
| `test_date` | Yes | YYYY-MM-DD (or YYYY-MM if day unknown) |
| `test_method` | Yes | wgs | wes | genotyping-array | microbiome |
| `coverage` | No | Sequencing depth (e.g. `0.4x`) |
| `reference_genome` | No | Reference build (e.g. `GRCh37`) |
| `sample_id` | No | Lab sample identifier |
| `data_location` | No | URI or path to raw data files |
| `created` | Yes | YYYY-MM-DD |
| `last_updated` | Yes | YYYY-MM-DD |

## Traits

| Trait | Meaning |
|-------|---------|
| `named` | Has a unique identity — entity field required |
| `temporal` | Has date information — date or created/last_updated required |
| `relational` | Connects to other records via entity/follows/related references |
| `immutable` | Cannot be modified after creation — agents MUST refuse modification |

## Observation Tags

- `[shift]` — Moment of change or breakthrough
- `[insight]` — Realisation or new understanding
- `[challenge]` — Difficulty or blocker
- `[goal]` — Aspiration or target
- `[observation]` — Neutral factual note
- `[action-client]` — Action for the subject
- `[action-owner]` — Action for the store maintainer
- `[decision]` — Choice made
- `[symptom]` — Physical or mental health symptom
- `[measurement]` — Numeric health measurement
- `[episode]` — Specific occurrence of a recurring issue
- `[daily-log]` — Routine daily health check-in
- `[concern]` — Something the patient is worried about
- `[question]` — Something to ask a clinician
- `[pattern]` — A correlation or pattern the patient has noticed
- `[improvement]` — Positive change noticed
- `[trigger]` — Something that seems to cause or worsen symptoms
- `[medication-change]` — Dose change, new medication, or stopped medication
- `[genetic-risk]` — Polygenic risk score finding from genomic data
- `[pharmacogenomics]` — Drug metabolism variant
- `[carrier-status]` — Recessive condition carrier finding
- `[microbiome]` — Microbiome composition finding

## Naming Conventions

- Files: `{type}-{qualifier}.md` or `session-{n}.md`
- Entity IDs: kebab-case
- All lowercase, hyphens not underscores

## Immutability Contract

Records with the `immutable` trait MUST NOT be modified after creation.
New information goes in new records.
