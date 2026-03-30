# Knowledge Store Health Check

Run a quick audit of this project's knowledge store. Produces a scorecard with fix-this-first recommendations.

## Usage

```
/knowledge-health          # Quick audit
/knowledge-health --deep   # Include cross-reference and content checks
```

## Workflow

### Phase 1: Read Configuration

1. Read `data/SCHEMA.md` for type definitions and required fields
2. Read `data/vocabulary-spec.md` for controlled vocabulary values (if present)
3. Read `governance.yaml` for zone rules (if present)

### Phase 2: Static Audit

Check every file in `data/` (recursively):

**Frontmatter completeness** (per type from SCHEMA.md):
- [ ] Every file has `type` and `traits` fields
- [ ] Required fields for each type are present
- [ ] Dates are valid YYYY-MM-DD format
- [ ] Mutable files have `last_updated` that reflects recent activity

**Vocabulary compliance** (if vocabulary-spec.md exists):
- [ ] All controlled field values match the vocabulary spec
- [ ] No invented values outside the spec

**Naming compliance**:
- [ ] All filenames are kebab-case
- [ ] Session/communication files include dates

**Governance zone compliance** (if governance.yaml exists):
- [ ] Strict-zone files have provenance, confidence, source references
- [ ] No provenance laundering (strict files citing none-zone sources)
- [ ] Light-zone files follow naming conventions

**Referential integrity**:
- [ ] All `entity` references resolve to an entity file
- [ ] All `follows` chains are unbroken
- [ ] All `references` arrays point to existing files
- [ ] No orphaned files (files not referenced by anything)

**Immutability contract**:
- [ ] Files with `immutable` trait have not been modified (check git history if available)

### Phase 3: Scorecard

```markdown
# Knowledge Store Health — {date}

## Summary
- Files checked: {N}
- Pass: {N} | Warn: {N} | Fail: {N}
- Overall: {GREEN|YELLOW|RED}

## Findings
{ordered by severity}

## Fix This First
1. {highest priority fix}
2. {second priority}
3. {third priority}
```

Write scorecard to `var/audits/{date}-knowledge-health.md`.

### --deep Flag

When `--deep` is specified, also check:
- Content quality: do observations have substantive bodies (not just frontmatter)?
- Tag usage: are observation tags being used consistently?
- Staleness: which entities haven't been updated in >30 days?
- Coverage: are there entity types with no observations?
