# Lykeward — Personal Health Bible

*Lykeward: from Old Norse "lík" (body) + Old English "weard" (guardian). The body-guardian.*

## Problem

Health information is scattered across NHS App, GP letters, notes, e-consult drafts, symptom logs, blood test results, medication history, and memory. When seeing the GP, assembling a coherent picture requires manual reconstruction. Connections between symptoms and conditions are noticed informally but never tracked. Investigation threads exist implicitly but have no structural home.

## Goal

A single, AI-agent-operated knowledge store that:
- Captures health observations with minimal friction (free text + date)
- Lets structure emerge over time through AI suggestion and clinical confirmation
- Surfaces patterns and connections for clinical review
- Generates GP consultation crib sheets automatically
- Tracks investigation hypotheses from question through evidence to conclusion
- Maintains a complete medication and treatment history with response tracking
- Produces a 60-second salience view for GP consultations

## Scope

### In Scope
- Personal health observations (symptoms, measurements, episodes)
- GP and specialist encounters (bidirectional: prep + outcome)
- Conditions (diagnosed, suspected, managed)
- Treatments and medications (active, stopped, response)
- Test results (blood work, imaging, assessments)
- Investigation threads (diagnostic hypotheses)
- External evidence documents (GP letters, referrals)
- Health measurement series (blood pressure, weight, B12)

### Out of Scope
- Family member health (single person — extensible later)
- Clinical decision-making (the store surfaces patterns; clinicians diagnose)
- NHS system integration (markdown-based, may export to FHIR later)

## Design Principles

1. **Capture first, classify later** — everything enters as an observation
2. **Co-occurrence is not causation** — relationships flagged for review, never asserted
3. **GPs want salience, not completeness** — 60-second design target
4. **Chronic conditions don't have arcs** — status snapshots, not lifecycles
5. **Patient observes, clinician diagnoses** — two independent confidence axes
