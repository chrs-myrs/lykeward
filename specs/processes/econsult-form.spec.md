---
id: econsult-form-process
type: behavior-spec
purpose: Define requirements for expert completion of NHS eConsult forms
tags: [process, medical, communication, healthcare]
criticality: IMPORTANT
failure_mode: Incomplete medical context leads to inadequate GP triage, missed diagnoses, or unnecessary appointments
pattern: meta-process
form_hours: 8am to 6:30pm
---

# eConsult Form Completion Process

## Summary

Defines requirements for expertly completing NHS eConsult forms. Process uses structured interview to gather complete medical context, integrates existing Lykeward health store data, and optimises form fields for clear medical communication within character limits.

## Requirements

### Pre-Interview Profile Check

- [!] Must check existing health store data before asking questions
  - Check `data/core/condition-*.md` for current diagnoses and ongoing treatments
  - Check `data/core/treatment-*.md` for current prescriptions (status: active)
  - Check `data/core/encounter-*.md` for previous similar consultations
  - Check `data/core/entity-*.md` for relevant care team members
  - Purpose: Avoid redundant questions, use existing context efficiently

- [!] Must identify information gaps that require questions
  - Compare what's needed for form against what's known from store
  - Ask only what's unknown or changed since last update
  - Principle: **Minimal sufficient questioning**

### Field Completion Requirements

#### Field: "Describe the problem"
- [!] Must state specific problem clearly in first sentence
- [!] Must optimize 500-character limit: lead with critical info, include symptom/location/severity/duration

#### Field: "How long has it been going on for? Is it getting better or worse?"
- [!] Must provide clear timeline with progression
- [!] Must signal urgency appropriately

#### Field: "Have you tried anything to help?"
- [!] Must list interventions with outcomes
- [!] Must reference relevant active treatments

#### Field: "Is there anything you're particularly worried about?" (Optional)
- [!] Should include when genuine medical concern exists

#### Field: "How would you like us to help?"
- [!] Must state clear, actionable request

#### Field: "Would you like to speak with a particular person?" (Optional)
- [!] Should specify if the patient has seen someone about this problem before
  - Continuity of care improves outcomes — seeing the same clinician avoids re-explaining
  - Check `data/core/encounter-*.md` for previous consultations on the same issue
  - Practice cannot guarantee the named person; may mean a longer wait
  - If no prior clinician for this issue, leave blank

#### Field: "When are the best times to contact you" (Optional)
- [!] Should specify if genuine constraints exist

### Photo Attachment Guidelines

- [!] Must attach photos when they aid assessment (up to 5)
- [!] Should not attach when no visual component

### Medical Communication Standards

- [!] Must use medically clear language
- [!] Must include complete relevant context (conditions, medications, previous episodes)
- [!] Must signal severity appropriately — neither dramatise nor minimise
- [!] Must optimise character limits intelligently — critical info first, concise phrasing

### Post-Submission

- [!] Must create encounter record in `data/core/` with type: encounter, method: econsult
- [!] Must link to relevant observations and conditions

## Notes

- eConsult forms are triage tools, not emergency services
- Forms designed for single health concerns (one issue per submission)
- GP has access to full medical records — form provides acute context
- Responses typically within 2 working days
- Practice hours: 8am-6:30pm for contact and form availability
- Maximum 5 photos per submission
