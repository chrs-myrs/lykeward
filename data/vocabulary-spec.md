---
type: vocabulary
scope: personal-health
version: 1.0.0
---

# Vocabulary Specification

## severity

| Value | Definition |
|-------|-----------|
| `mild` | Noticeable but not limiting daily activities |
| `moderate` | Affects daily activities or causes concern |
| `severe` | Significantly limits function or causes distress |

## temporality

| Value | Definition |
|-------|-----------|
| `acute` | Sudden onset, expected to resolve |
| `episodic` | Comes and goes, distinct episodes |
| `chronic` | Persistent, ongoing for months or years |
| `recurrent` | Resolved previously but has returned |

## observation_confidence

| Value | Definition |
|-------|-----------|
| `certain` | Clearly observed or measured — no doubt |
| `likely` | Fairly confident but some uncertainty |
| `uncertain` | Noticed something but unsure of the nature |

## relational_confidence

| Value | Definition |
|-------|-----------|
| `clinician-confirmed` | A healthcare professional has confirmed this connection |
| `self-assessed` | Patient believes these are connected (flag for clinical review) |
| `unassessed` | No assessment of whether these are related (default) |

## treatment_response

| Value | Definition |
|-------|-----------|
| `helped` | Symptoms improved with treatment |
| `partially-helped` | Some improvement but not fully resolved |
| `no-effect` | No noticeable change |
| `worsened` | Symptoms got worse during treatment |
| `side-effect` | Treatment caused new unwanted effects |

## condition_status

| Value | Definition |
|-------|-----------|
| `suspected` | Under investigation, not yet confirmed |
| `confirmed` | Clinician-diagnosed |
| `managed` | Chronic condition under active management |
| `flaring` | Managed condition currently worsening |
| `stable` | Condition present but not causing problems |
| `remission` | Previously active, currently quiescent |
| `resolved` | No longer present |

## link_type

| Value | Definition |
|-------|-----------|
| `co-occurrence` | These happen at the same time (temporal correlation only) |
| `triggers` | One appears to trigger the other (patient observation) |
| `worsens` | One makes the other worse |
| `alleviates` | One helps the other |
| `side-effect` | Treatment causing unwanted effect |
| `differential` | Alternative explanation for the same symptoms |

## Edge-Case Guidance

## Governance
- These values are the ONLY valid options
- Do not invent new values without updating this spec
