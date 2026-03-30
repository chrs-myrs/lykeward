# Knowledge Store Query Patterns

## How to navigate this store

### Find information about a subject
1. Read the entity file (`type: entity`) — profile, attributes, current state
2. Read their timeline (`type: timeline`) — chronological overview
3. Read sessions (`type: session`) in date order — detailed records

### Trace a theme across sessions
1. Read all session files for the entity in chronological order
2. Look for observations tagged with the relevant category (e.g., `[shift]`, `[challenge]`)
3. Extract tagged observations and arrange by date to see evolution

### Produce a cross-cutting analysis
1. Read ALL session files for the entity
2. Extract observations by tag type and group by theme, not by session
3. Write the analysis as `type: assessment` or `type: overview`

### Check data quality
1. List all files and verify frontmatter completeness (type, traits, entity, date)
2. For sessions: verify `follows` chain is unbroken
3. For entities: verify `last_updated` reflects most recent activity
4. Check referential integrity: every `entity` reference should have a matching entity file

### Query across multiple subjects
1. Read all entity files to identify active subjects
2. For each: count sessions, note latest date, identify key themes
3. Produce a portfolio-level summary aggregating across all subjects

### Create new records
- **Session**: `type: session`, include `follows` reference, mark `immutable`
- **Entity**: `type: entity`, include `traits: [named, temporal, relational]`
- **Observation**: `type: observation`, link to entity and optionally related_session
- **Assessment**: `type: assessment`, mark `immutable`, reference sessions_covered
