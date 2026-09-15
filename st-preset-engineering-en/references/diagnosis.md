# diagnosis.md — S3: Diagnosis

> **When to read**: S3.
> **Deliverable**: a risk list / diagnosis report — every defect = description + evidence (file:line, command output, or dump name) + evidence label + suggested fix path. **Diagnosis never modifies anything.**

## Three defect classes (different verification paths; never mix)

| Class | Examples | Who can confirm |
|---|---|---|
| **Structural** | Orphan entries, order stale refs (the classic hot-upgrade residue), dead slots, broken pairing (template reads a slot ≠ this route writes it), tag contradictions (two tag names in one request), double-False regexes, mutex groups with ON≠1, remote dependencies, zombie setvar landmines | The S1 dossier alone (structural fact) |
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

1. Walk the S1 dossier through the six structural reconciliations (orphans / order stale refs / dead slots / double-False / mutex / remote dependencies) — all **mandatory checks**, never samples. Grade remote dependencies per red-lines rule 8 (I/W/E). **Confirm the input type first**: no `prompts` key at top level / `entries` is a dict (lorebook signature) → this route does not apply; output an "input recognition + downgrade note" report, **never diagnose it as an empty preset** (measured: a lorebook input produced an empty report with exit 0, nearly misread).
2. Pairing integrity: for every model branch, the template's getvar slot == the route's ❗1 setvar slot? (Measured lesson: three routes self-consistent, one broken, zero errors — the only symptom was degraded output.)
3. Contract scan (three-way consistency; tool contracts = Card 5 §5 + Card 9): tag/format literals in entry bodies ↔ regex patterns ↔ script references — all three agree? **Regex-side caveat**: literal extraction only recognizes `</?tag>` shapes; consumers hidden inside regex syntax patterns (a bare `think` in find rather than `<think>`) are false negatives — every tag the matrix marks "one-sided" gets one targeted substring recheck. **Alternation-form tags** (`<(a|b|c)>`) are missed by `</?tag>`-shape extraction entirely — the same "one-sided but actually two-sided" trap, caught by the same targeted substring recheck (measured: `<(thinking|suggestions|disclaimer)>` reported one-sided, actually two-sided).
4. Behavioral defects: compare S2 ground truth against template requirements item by item (length/format/triggers/counters), **give a number for every item** (hit rate, occurrence counts, drift events). **Default path with no ground truth** (both B1/B2 unreachable: no local ST, quarantine strategy produces no PUBLIC copy): everything behavioral goes to the "unverified list" with **what is missing written item by item** (missing dump / missing N turns / which path) — never write "passed", never omit silently. A diagnosis report is complete only when the six-mandatory-check reconciliation AND the unverified list are both present.
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
### Extension hookpoint inventory
- <the S1 hookpoint three-question conclusions land here (block / kind / status / note table)>
### Unverified list
- <item> — missing <what>
```

## Cross-preset shared infrastructure (recognizing same-author / sibling-preset copies)

Same-family or sibling presets copy infrastructure between each other (same-name entries + same inj_order, regex families copied over, same-family floating-window scripts) — if this cross-preset relationship is not registered, a single report misses context like "this is part of a sibling-preset series" (measured: two presets shared same-name entries with identical inj_order and params, regex families copied between them, same-family floating windows; cross-referencing the two reports locates it). When several presets are diagnosed in one session:

- Same-name entries (same inj_order/same state) appearing across presets → register "cross-preset shared infrastructure" in each report's Info and cross-reference;
- Same-family regexes/scripts copied between presets → note the source preset and the sync surface;
- Cross-preset verdicts still list structural facts only; never guess author intent.

## Discipline

- All diagnosis conclusions go into the **single errata channel** (if the repository has one); never let multiple documents diverge.
- Intent questions ("why did the author write it this way") are honestly labeled "undecidable" — only structure and behavior can be judged, never minds.
- Found a wrong document/prior-session conclusion → register an erratum (numbered sequentially), **never quietly rewrite history**.
