---
id: knowledge-health
type: feature-spec
purpose: Read-only audit of the knowledge store for frontmatter, vocabulary, governance, referential-integrity, and immutability compliance
criticality: MEDIUM
failure_mode: Store drifts into inconsistency (invalid vocabulary, broken references, governance violations, modified immutable records) undetected
governed-by:
  - ../workspace/knowledge-operations.spec.md
  - ../workspace/constitution.spec.md
specifies:
  - ../../.claude/skills/knowledge-health/SKILL.md
---

# Knowledge Store Health Check

## Purpose

Read-only audit of the personal health knowledge store (`data/`). Produces a scorecard with severity-ranked findings and a "fix this first" list, written to `var/audits/`. Detects drift that the capture skills do not prevent: missing frontmatter, invalid vocabulary, broken cross-references, governance-zone violations, and modified immutable records. The audit reports — it never repairs.

## Requirements

### REQ-KHEALTH-001: Configuration-Driven Audit
- Read `data/SCHEMA.md` (types + required fields), `data/vocabulary-spec.md` (controlled values), and `governance.yaml` (zone rules) before auditing
- Audit rules derive from these sources — the skill MUST NOT hardcode the schema, so the audit stays correct as the schema evolves

### REQ-KHEALTH-002: Frontmatter Completeness
- Every record has `type` and `traits`
- Required fields for each type (per SCHEMA.md) are present
- Dates are valid YYYY-MM-DD; mutable records carry `last_updated`

### REQ-KHEALTH-003: Vocabulary Compliance
- All controlled field values match `data/vocabulary-spec.md` exactly
- No invented values outside the spec

### REQ-KHEALTH-004: Referential Integrity
- `entity` references resolve to an existing entity file
- `follows` chains are unbroken; `links`/`references` point to existing files
- Flag orphaned records and **inconsistent link shapes** (the canonical observation link is `{to, type, relational_confidence}`; flag divergent forms such as `{id, link_type}`)

### REQ-KHEALTH-005: Governance Compliance
- Strict-zone (`data/core/`) records have provenance, confidence, and source references
- No provenance laundering (strict-zone records citing none-zone sources)
- Light/none zones follow their (looser) rules

### REQ-KHEALTH-006: Immutability Contract
- Records with the `immutable` trait are unchanged since creation (check git history where available)

### REQ-KHEALTH-007: Naming Compliance
- Filenames are kebab-case, all lowercase; date-primary records include the date

### REQ-KHEALTH-008: Scorecard Output
- Write a scorecard to `var/audits/{date}-knowledge-health.md`
- Include: files checked, pass/warn/fail counts, overall GREEN/YELLOW/RED, findings ranked by severity, and a "fix this first" list
- The audit is read-only — it MUST NOT modify any record

### REQ-KHEALTH-009: Deep Mode (`--deep`)
- Adds content-quality checks (observations have substantive bodies, not just frontmatter), tag-usage consistency, staleness (entities not updated in >30 days), and coverage gaps (types with no records)

## Non-Requirements
- Does NOT fix issues — remediation is the user's or another skill's responsibility
- Does NOT capture new health data (that's `/health-check-in`)
- Does NOT audit any external research store outside `data/` — this audit is scoped to the personal knowledge store only
