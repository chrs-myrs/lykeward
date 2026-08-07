# Export Allowlist — lykeward public template

This file is the outward-sync guard for the `chrs-myrs/lykeward` public template repo. Only the files listed here may be synced outward from the private instance (`~/projects/personal/lykeward`) on future export runs. Everything else in the instance — all of `data/` (core, research, internal, sources.yaml), the condition-named research subtree, `var/`, `.sessions/`, `registries/`, and any gitignored personal config — stays instance-only, permanently.

## Allowed outward-sync paths

- `README.md`
- `AGENTS.md` — generalise: strip any condition-named research-store section, strip any skill row not present in this template
- `PURPOSE.md`
- `governance.yaml`
- `project.yaml`
- `.gitignore` — sync only entries relevant to this template's own directory structure; drop instance-only entries (e.g. condition-named store ignores, skills not included in the template)
- `data/SCHEMA.md`
- `data/vocabulary-spec.md`
- `specs/` (all files) — generalise: strip any condition-named or profile-specific sections
- `.claude/skills/` — per-skill inclusion decision; a skill is included only if its content is generic health-tracking methodology with no condition names, personal names, or personal research-project references. See the out-of-tree EXPORT-REVIEW artifact (delivered alongside this staging tree, never committed — it discusses excluded content) for the current per-skill decision (`fettle` is currently excluded).
- `ctxt/`

## Explicitly never read or synced (instance hard-exclusions)

- `data/core/`, `data/research/`, `data/internal/`, `data/sources.yaml`
- The condition-named research subtree, and any mention of it or the condition elsewhere
- `var/`, `.sessions/`, `.tmp/`, `.scratch/`
- `.claude/settings.local.json`, `.claude/projects/`
- Any personal name, clinician identity, clinical value, condition name, or NHS identifier appearing inside an otherwise-allowed file

## Process

1. Fresh clone of this public repo.
2. Copy-with-appraisal each allowlisted file from the instance; scan before staging.
3. Generalise where a clean strip is possible (delete the personal/condition-specific section, keep the rest); exclude the whole file if it can't be cleanly generalised.
4. Write the EXPORT-REVIEW artifact OUT OF TREE (sibling of the staging clone, never committed — it necessarily discusses excluded sensitive content) with a diff summary and scan attestation.
5. Commit locally. A human (Chris) reviews and pushes — this repo never pushes itself.
