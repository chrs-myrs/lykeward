# Health Check-In

Regular health interview capturing current symptoms, treatment responses, and updates. Designed for weekly or as-needed use.

## Usage

```
/health-check-in              # Full adaptive interview
/health-check-in --quick      # Fast "anything new?" check (under 2 min)
```

## Workflow

### Phase 1: Load Context

Before asking anything, silently read the store to understand current state:

1. Read all files in `data/core/` with `type: investigation` and `status: investigating` or `hypothesis`
2. Read all files with `type: treatment` and `status: active`
3. Read all files with `type: condition` and `status: active` or `managed` or `flaring`
4. Read recent observations (last 30 days) to understand current baseline
5. Build a selective treatment probe list:
   - Treatments started or changed in last 30 days
   - Treatments with no `response` field recorded
   - Treatments previously flagged for review

### Phase 2: Adaptive Interview

**Open**: "How are you feeling? Anything new or changed since last time?"

**If user says "nothing new" or similar** (--quick path):
- Acknowledge: "All clear. I've updated the review timestamps on your active items."
- Update `last_reviewed` on all active investigations, conditions, and treatments
- End session. Target: under 2 minutes.

**If user reports something**, adapt follow-ups based on response:
- If they mention a body area → ask about severity, duration, pattern
- If active investigations exist for that area → ask specifically: "How's the [investigation question] going? Any change?"
- If treatments on the probe list relate → ask: "Still taking [treatment]? How's it going?"
- Do NOT run through a fixed checklist — follow the conversation

**Treatment probing** (selective, not exhaustive):
- Only ask about treatments on the probe list from Phase 1
- If user mentions a treatment naturally, update that one too
- Never ask about every active treatment in one session

### Phase 3: Capture Records

For each health data point mentioned:

**Create observation** in `data/core/`:
```yaml
---
type: observation
traits: [temporal, relational]
date: YYYY-MM-DD
body_system: <AI suggests silently>
severity: <if mentioned>
temporality: <if inferable>
observation_confidence: <certain|likely|uncertain>
links: []
flagged_for_gp: false
---

<Free text description of what the user said>
```

- AI appends suggested `body_system` and `temporality` tags silently — do NOT ask the user to confirm tags
- Only prompt the user if severity is ambiguous and matters for clinical flagging
- Link to relevant active investigations via the `links` array (default `relational_confidence: unassessed`)

**Update treatments** if response reported:
- Edit the treatment file's `response` field
- If stopped: change `status: stopped`, add `stopped: YYYY-MM-DD` and reason
- If new side effect: create a new observation linked to the treatment

### Phase 4: Clinical Flags

After the interview (not during — don't alarm), assess:

1. **Severity changes**: Compare new observation severity against recent observations for the same body_system. Flag if escalated (e.g., mild → severe)
2. **New unexplained symptoms**: If a symptom doesn't relate to any active investigation, note: "This is new — you might want to mention it to your GP"
3. **Treatment concerns**: If response is `worsened` or `side-effect`, flag for GP discussion
4. **Safety netting**: If user described severity as "severe" or used emergency language (can't breathe, chest pain, worst headache ever), include in summary:

> **You described [X] as severe. If you need urgent help, contact NHS 111 or call 999.**

This is informational, not diagnostic.

Set `flagged_for_gp: true` on all flagged observations — this creates the data bridge to `/consultation-prep`.

### Phase 5: Session Summary

```markdown
## Check-In Summary — YYYY-MM-DD

### New Observations
- [list of observations created with severity if recorded]

### Treatment Updates
- [list of treatment changes]

### Flags for GP
- [list of flagged items, if any]

### Suggested Actions
- [book GP if flags exist]
- [start investigation if new unexplained symptom]
- [nothing needed if all clear]
```

## Anti-Patterns

**NEVER**:
- Run through a fixed symptom checklist
- Ask about every active treatment
- Diagnose or suggest conditions
- Assert causal relationships between symptoms
- Skip safety netting for severe symptoms
- Force classification on observations
- Take longer than 5 minutes for a typical session
