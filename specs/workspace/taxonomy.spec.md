---
id: taxonomy
type: workspace-spec
purpose: Define directory structure and content placement rules
criticality: HIGH
failure_mode: Records placed in wrong governance zone or files created in wrong location
references:
  - ../../governance.yaml
  - ../../data/SCHEMA.md
---

# Taxonomy

## Directory Structure

```
lykeward/
├── PURPOSE.md                    # Project mission and scope
├── AGENTS.md                     # Agent bootstrap and task routing
├── project.yaml                  # Project configuration
├── governance.yaml               # Governance zone definitions
├── data/                         # All knowledge store content
│   ├── SCHEMA.md                 # Type system and frontmatter fields
│   ├── vocabulary-spec.md        # Controlled vocabulary values
│   ├── core/                     # Strict zone -- full provenance required
│   ├── research/                 # Light zone -- naming conventions only
│   └── internal/                 # None zone -- scratch space
├── ctxt/                         # Agent context files
│   ├── queries.md                # Query patterns
│   └── knowledge-model.md        # Detailed model reference
├── specs/                        # Specifications
│   └── workspace/                # Workspace-level specs
│       ├── knowledge-operations.spec.md
│       ├── constitution.spec.md
│       └── taxonomy.spec.md
├── var/                          # Transient artifacts (not committed)
│   ├── audits/                   # Health check output
│   └── salience-view-*.md        # Pre-appointment crib sheets
└── .sessions/                    # Session tracking
```

## Content Placement

### data/core/ (strict zone)

Primary knowledge records requiring full provenance and confidence metadata.

**Types placed here:** observation, encounter, condition, treatment, test-result, investigation, evidence, measurement-series, entity

**Rules:** provenance required, confidence fields mandatory, source references mandatory, immutable signals enforced, kebab-case naming

### data/research/ (light zone)

Analytical and summary records. Naming conventions apply but provenance is optional.

**Types placed here:** assessment, overview

**Rules:** kebab-case naming only

### data/internal/ (none zone)

Scratch space for drafts, working notes, and temporary content. No governance rules.

**Appropriate for:** draft observations before promotion, working notes, temporary analysis

### ctxt/

Agent context files loaded on demand. Not knowledge records.

### specs/workspace/

Workspace specifications that govern how the project operates. Not knowledge records.

### var/

Transient artifacts generated during sessions. Not committed to git.

- `var/audits/` -- knowledge-health output
- `var/salience-view-*.md` -- pre-appointment crib sheets

## Naming Conventions

| Pattern | Used for | Example |
|---------|----------|---------|
| `{type}-{qualifier}.md` | Most records | `observation-2026-03-28-headache.md` |
| `{type}-YYYY-MM-DD.md` | Date-primary records | `encounter-2026-03-28.md` |
| `{type}-{entity-slug}.md` | Entity-primary records | `condition-vitamin-b12-deficiency.md` |
| `investigation-{question-slug}.md` | Investigations | `investigation-hand-tingling.md` |
| `evidence-DOC-NNN.md` | Evidence documents | `evidence-DOC-001.md` |
| `measurement-series-{metric}.md` | Measurement series | `measurement-series-blood-pressure.md` |

All filenames: kebab-case, all lowercase, hyphens not underscores.

## ID Formats

| Type | Format | Example |
|------|--------|---------|
| encounter | ENC-YYYYMMDD-NNN | ENC-20260328-001 |
| test-result | TEST-YYYYMMDD-NNN | TEST-20260328-001 |
| evidence | DOC-NNN | DOC-001 |
