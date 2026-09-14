# handover.md — S6: Handover

> **When to read**: S6; before session close-out.
> **Why it cannot be skipped**: a fix without handover is an unfixed fix — the next maintainer (or the next session) needs coordinates. One real project started work on wrong premises three times, all traceable to missing handovers.

## Deliverables checklist (scale to surgery size)

| Surgery scale | Must deliver |
|---|---|
| Minimal (toggle/params) | In-conversation note of the changed points + L1 result |
| Medium (body edits / add-remove entries) | Change notes (per change: what changed + evidence + verification result) |
| Large (refactor / multi-round) | **Change ledger** (each entry with evidence) + **user manual** (task cards / health-check commands / troubleshooting table) + unverified list |

## Change-ledger format (one row per change)

```markdown
| # | Entry | Change | Evidence (measured / structural fact / presumed) | Verification status |
```
- Unverified changes go in a **separate section** stating exactly what's missing (API / ST / N turns / user data) — never folded into "done".

## Minimal structure of the user manual

1. Three iron laws (where the effective layer lives / orphan = nonexistent / always run the health check after changes)
2. Switch map (notation semantics + model pairing table + paired operations)
3. Task cards (phrased the way users ask: "want a different model" "want a different length" "want a new style" …)
4. Health-check commands + expected-value table
5. Symptom → troubleshooting table (garbled output → check double templates; history loss → check regexes; …)
6. Unverified list and red lines

## State sync (inside a governed repository, per that repository's own collaboration docs)

1. **Entry state file**: add this session's row (outputs + established facts); overwrite stale state overturned by this session
2. **User-direction file**: did what the user should know / decide change?
3. **Errata channel**: every error discovered this session, numbered sequentially (evidence + disposition columns)

**30-second close-out self-check**:
- Any stale markers in the state file that conflict with the latest facts?
- Decision items in the user-direction file refreshed?
- Does the errata channel's highest number include every new finding of this session?

## User test protocol template (L4 delivery)

```markdown
Test: <output filename> (first 16 of sha: …, switch state: <statement>)
Play: N turns of comparable scenarios
Watch three numbers:
1. <metric1> (comparison baseline <value>)
2. <metric2> (comparison baseline <value>)
3. <metric3> (comparison baseline <value>)
Send me the results → I backfill the dossier/matrix
```

## Before committing

- [ ] Verification suite all green (verification.md §suite)
- [ ] Files pending commit have zero lexicon hits; sensitive artifacts confirmed outside the repo
- [ ] Commit message states "what changed + evidence + unverified items" (let git log itself be the ledger)
