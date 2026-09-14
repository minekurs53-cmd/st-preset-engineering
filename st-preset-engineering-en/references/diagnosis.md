# diagnosis.md — S3: Diagnosis

> **When to read**: S3.
> **Deliverable**: a risk list / diagnosis report — every defect = description + evidence (file:line, command output, or dump name) + evidence label + suggested fix path. **Diagnosis never modifies anything.**

## Three defect classes (different verification paths; never mix)

| Class | Examples | Who can confirm |
|---|---|---|
| **Structural** | Orphan entries, dead slots, broken pairing (template reads a slot ≠ this route writes it), tag contradictions (two tag names in one request), double-False regexes, mutex groups with ON≠1, remote dependencies, zombie setvar landmines | The S1 dossier alone (structural fact) |
| **Behavioral** | Length out of range, format drift, bans not holding, triggers not triggering, counter drift | Output data needed (real model > mock) |
| **Effect** | Is the writing good? Does it feel right? Is the user satisfied? | **Only the user** (L4) |

## Four-way evidence labels (every conclusion carries exactly one)

| Label | Meaning | Evidence required |
|---|---|---|
| **Measured** | Obtained from real runs | dump filename / command output snippet / per-turn data |
| **Structural fact** | Provable from static structure | `file:line` or a reproducible command |
| **Presumed** | Grounded but unverified | Explicitly write "presumed = …, to be replaced by measurement" |
| **Unverified** | Missing conditions | **Must state what is missing** (API / ST / N turns / needs user data) — never bridge the gap with softened wording |

## Diagnosis procedure

1. Walk the S1 dossier through the five structural risks (orphans / dead slots / double-False / mutex / remote dependencies) — all **mandatory checks**, never samples.
2. Pairing integrity: for every model branch, the template's getvar slot == the route's ❗1 setvar slot? (Measured lesson: three routes self-consistent, one broken, zero errors — the only symptom was degraded output.)
3. Contract scan: tag/format literals in entry bodies ↔ regex patterns ↔ script references — all three agree?
4. Behavioral defects: compare S2 ground truth against template requirements item by item (length/format/triggers/counters), **give a number for every item** (hit rate, occurrence counts, drift events).
5. Summarize ordered by error (irreversible damage / silent failure) → warn (systematic deviation) → info (improvement opportunities); each item gets a "fix direction" (not the fix itself — fixes happen in S4).

## Report format (minimal template)

```markdown
## Diagnosis: <preset name> (baseline: <switch-state statement>)
### Error
- [E1] <defect> — structural fact (<command / file:line>) → fix direction: <…>
### Warning
- [W1] <defect> — measured (<dump/data>, <numbers>) → fix direction: <…>
### Info
- [I1] <improvement opportunity> — presumed (<basis>)
### Unverified list
- <item> — missing <what>
```

## Discipline

- All diagnosis conclusions go into the **single errata channel** (if the repository has one); never let multiple documents diverge.
- Intent questions ("why did the author write it this way") are honestly labeled "undecidable" — only structure and behavior can be judged, never minds.
- Found a wrong document/prior-session conclusion → register an erratum (numbered sequentially), **never quietly rewrite history**.
