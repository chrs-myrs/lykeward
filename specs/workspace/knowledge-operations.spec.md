---
id: knowledge-operations
type: workspace-spec
purpose: Define how to work with this project's knowledge store
criticality: CRITICAL
failure_mode: AI agents create inconsistent records or violate governance contracts
governed-by:
  - ../../data/SCHEMA.md
references:
  - ../../data/vocabulary-spec.md
  - ../../governance.yaml
---

# Knowledge Store Operations

## Overview

This project uses a convention-based knowledge store. All knowledge lives in `data/` as markdown files with YAML frontmatter. The frontmatter conventions make files discoverable by AI agents without schema documentation — but this spec defines the operating rules.

Read `data/SCHEMA.md` for the type system and frontmatter fields.
Read `data/vocabulary-spec.md` for controlled vocabulary values (if present).
Read `governance.yaml` for zone-specific quality rules (if present).
Read `ctxt/queries.md` for query patterns.

## Creating Records

### Before creating any file:
1. Determine the correct **type** from SCHEMA.md
2. Check the **governance zone** — which directory does this belong in?
3. If in a strict zone: ensure you have provenance, confidence, and source reference
4. Check for **duplicates** — read existing files of the same type first
5. Assign the next sequential **ID** if the type uses IDs

### Frontmatter contract:
- Every file MUST have `type` and `traits` fields
- Types with `immutable` trait MUST NOT be modified after creation
- Mutable types MUST update `last_updated` when changed
- All controlled vocabulary values MUST match `data/vocabulary-spec.md`

### File naming:
- `{type}-{qualifier}.md` (e.g., `entity-tfl.md`, `communication-20260328-001.md`)
- Kebab-case, all lowercase
- Sessions/communications: include date in filename

## Governance Zones

If `governance.yaml` exists, respect zone rules:

| Zone | Location | Rules |
|------|----------|-------|
| **Strict** | `data/core/` | Full provenance, confidence, source references required. Immutable signals. |
| **Light** | `data/research/` | Naming conventions only. Provenance optional. |
| **None** | `data/internal/` | No rules. Scratch space. |

### Cross-zone rules:
- Strict-zone content MUST NOT cite none-zone content as provenance
- Content may be promoted from light to strict (requires adding full metadata)
- Assessments spanning zones should live in the light zone

## Querying the Store

See `ctxt/queries.md` for standard query patterns. Key operations:

- **Find by type**: Read all files, filter by `type` frontmatter
- **Find by entity**: Filter by `entity` frontmatter field
- **Trace timeline**: Read timeline files, follow chronological entries
- **Cross-reference**: Follow `entity`, `follows`, `references` fields between files

## Self-Audit

Periodically check store health:
1. **Frontmatter completeness**: Every file has required fields per SCHEMA.md
2. **Vocabulary compliance**: All controlled values match vocabulary-spec.md
3. **Referential integrity**: All cross-references resolve to actual files
4. **Governance compliance**: Files in strict zones have required provenance
5. **Immutability**: No immutable records have been modified
6. **Naming**: All files follow kebab-case conventions

## Write-Back from Unstructured Sessions

Structured skills (health-check-in, consultation-prep) have explicit record creation steps. But health facts also surface during unstructured work — debates, investigations, research, freeform conversation. These must be captured immediately, not deferred.

### What crosses into `data/core/`

| Finding | Record Type | Default Status | Notes |
|---------|------------|---------------|-------|
| New anatomical finding or diagnosis | `condition` | `patient-reported` or `suspected` | Only `confirmed` if clinician-attributed |
| New health question worth tracking | `investigation` | `hypothesis` | Follow investigate-health structure |
| Treatment response discovered | update to existing `treatment` | — | Include context for how the response was discovered |
| Clinically relevant negative finding | `observation` with `[observation]` tag | — | Clear negative statement: "Patient confirms no history of X" |
| Correction to existing record | edit to mutable record | — | Only mutable types; immutable records stay unchanged |

### What stays in `var/`

- General medical research (not patient-specific)
- Debate transcripts and persona arguments
- AI-generated hypotheses not confirmed by the patient
- Draft documents and brainstorm output
- Session summaries and investigation artifacts

### Provenance rules for unstructured write-back

Every record written to `data/core/` from an unstructured session must satisfy:

1. **The patient explicitly stated or confirmed the fact** — not inferred, not assumed, not concluded by an AI persona
2. **The source is traceable** — "patient stated during [date] session" or "clinician documented in [encounter]"
3. **Treatments are never assumed** — only the patient can confirm what they take. If a supplement or medication is mentioned speculatively, do NOT create a treatment record without explicit patient confirmation
4. **Hypotheses use hedged language** — "candidate mechanism", "possible", "may contribute". Never "causes", "is due to", "confirms"
5. **Negatives need explicit confirmation** — "I've never had cold sores" is a valid negative finding. Not mentioning cold sores is NOT a negative finding
6. **When in doubt, ask** — if the fact came from debate, research, or AI reasoning rather than the patient's own words, confirm before writing to `data/core/`
