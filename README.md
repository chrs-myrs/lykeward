# Lykeward

*From Old Norse "lik" (body) + Old English "weard" (guardian). The body-guardian.*

A personal health knowledge store operated by AI agents. Lykeward captures health observations with minimal friction, lets structure emerge through AI suggestion and clinical confirmation, and generates GP consultation crib sheets automatically.

## How It Works

Lykeward is a structured markdown knowledge base designed to be operated through [Claude Code](https://docs.anthropic.com/en/docs/claude-code). There's no application to install -- just clone the repo, open it in Claude Code, and start talking about your health.

```
git clone https://github.com/chrs-myrs/lykeward.git
cd lykeward
claude
```

Your health data lives in `data/core/` (gitignored by default). The repo contains only the schema, governance rules, vocabulary, specifications, and agent instructions.

## Usage Examples

**Recording a symptom:**
> "I've had a persistent headache behind my left eye for three days"

Creates an observation record with date, severity, body system, and optional links to conditions.

**After a GP appointment:**
> "Saw Dr Smith today about the headaches"

Creates an immutable encounter record with outcomes, decisions, and links to observations discussed.

**Preparing for an appointment:**
> "I'm seeing the GP tomorrow about my breathing"

Generates a 60-second salience view: active conditions, recent observations, current treatments, and open questions.

**Tracking a medication:**
> "Started amoxicillin 500mg three times daily"

Creates a treatment record with dosage, prescriber, targets, and response tracking.

**Investigating a pattern:**
> "Why do I always get worse in winter?"

Creates an investigation thread linking observations, test results, and clinical findings.

**Drafting an eConsult:**
> "I need to submit an eConsult about this rash"

Runs a structured interview, pulls context from the store, and drafts optimised form fields within character limits.

**Running a health check:**
> "/knowledge-health"

Audits frontmatter compliance, vocabulary usage, governance zone rules, and referential integrity across all records.

## Design Principles

1. **Capture first, classify later** -- everything enters as free text + date; classification is optional and added over time
2. **Co-occurrence is not causation** -- relationships are flagged for clinical review, never asserted as causal
3. **GPs want salience, not completeness** -- consultation views target 60-second scan time
4. **Chronic conditions don't have arcs** -- status snapshots (managed, flaring, stable), not lifecycle stages
5. **Patient observes, clinician diagnoses** -- two independent confidence axes

## Knowledge Model

| Type | Purpose | Mutability |
|------|---------|------------|
| `observation` | Any health data point -- free text + date | mutable |
| `condition` | Diagnostic label -- confirmed or suspected | mutable |
| `treatment` | Medication, supplement, therapy | mutable |
| `encounter` | GP/specialist interaction | immutable |
| `test-result` | Lab work, imaging, measurements | immutable |
| `investigation` | Diagnostic hypothesis thread | mutable |
| `evidence` | External documents (GP letters, referrals) | immutable |
| `entity` | Named thing (provider, practice) | mutable |
| `measurement-series` | Time-series data (BP, weight) | append-only |
| `assessment` | Evaluative synthesis across records | immutable |
| `overview` | Portfolio-level summary | mutable |

## Governance Zones

| Zone | Location | Level |
|------|----------|-------|
| Core | `data/core/` | Strict -- full provenance, confidence fields, source references |
| Research | `data/research/` | Light -- naming conventions only |
| Internal | `data/internal/` | None -- scratch space |

## Project Structure

```
lykeward/
  CLAUDE.md            # Agent instructions (loaded by Claude Code)
  AGENTS.md            # Agent guide (knowledge model, task routing)
  PURPOSE.md           # Project mission and scope
  governance.yaml      # Zone definitions and rules
  data/
    SCHEMA.md          # Type system and frontmatter fields
    vocabulary-spec.md # Controlled vocabulary values
    core/              # Health records (gitignored)
    research/          # Synthesis records (gitignored)
    internal/          # Scratch space (gitignored)
  specs/               # Specifications (workspace, features, processes)
  ctxt/                # Agent context files
  var/                 # Transient artifacts (gitignored)
  .claude/
    skills/            # Custom skills (health-check-in, consultation-prep, etc.)
```

## Getting Started

1. Clone the repo
2. Open in Claude Code
3. Create `data/sources.yaml` if you have external document sources (GP letters, etc.)
4. Start recording: *"I noticed a pain in my lower back yesterday"*

The agent will create properly typed records, check for duplicates, use controlled vocabulary, and maintain governance rules automatically.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI or IDE extension)
- Optional: [Claude for Chrome](https://chromewebstore.google.com/detail/claude-for-chrome) for NHS App extraction

## Licence

MIT
