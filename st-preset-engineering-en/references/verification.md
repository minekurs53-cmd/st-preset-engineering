# verification.md — S5: Verification (three gates, cheap to expensive)

> **When to read**: S5; before writing the words "verification passed".
> **Discipline**: the gates are ordered cheap to expensive; **stop at the first failure** — L1 red means never entering L2. Each gate answers a different question; none substitutes for another.
> **Without a tool**: when a gate's tool cannot run, build it per the matching toolmap contract card (L1 = Card 5, L2 = Card 7, L3 = Card 8), pass its self-test, then verify — **never skip a gate, never substitute eyeballing**.
> **No environment ≠ no tool**: tools can be built; environments cannot — L3/B1 needs a local ST install, L2/B2 needs a PUBLIC copy (quarantine strategy produces none). When the environment lacks conditions: mark that gate's conclusions "unverified + what's missing" honestly — **never bridge with softened wording**, and never let a lower gate's all-green imply a higher gate passed.

## L1 Structural layer (minutes; always required)

```bash
python <tool_dir>/p1_evidence.py report --preset <output.json>
```

| Criterion | Expected | What a violation means |
|---|---|---|
| Orphan entries | 0 (or identical to before, each with a stated reason) | An entry not wired into order = dead work / ghost reference |
| order stale refs | 0 (or identical to before, each with a stated reason) | order referencing a non-existent identifier = old-version residue / broken naming; the common accident of hot-upgrade renames |
| Dead slots (write-only) | 0 (a new dead slot = debt introduced this round) | Someone wrote a variable nobody will ever read |
| Mutex-group ON count | Exactly 1 per group (by-design exceptions registered in the dossier) | Configuration violation / double template |
| Branch pairing | Template's getvar slot == this route's written slot | Broken pairing = silent degradation |
| Double-False regexes | 0 | Permanently rewrites chat history (error) |
| Absolute-injection / sp=true annotation | Same as before, each with a stated reason | Reordering does not affect them — missing the annotation makes changes to injection shape falsely green |

⚠️ **L1 blind spot (measured)**: L1 all green ≠ semantic consistency. Gear values / length minimums hard-coded in the body of other ON entries (e.g. after a length-gear change, a thinking-gear entry still forbidding "body under 1200 characters") **pass every group-rule check** — gear-change/linked changes must additionally run: a repo-wide body scan of ON entries (numeric anchors) or an L3 assembly-truth manual review.

## L2 Assembly layer (before/after reconciliation)

```bash
python <tool_dir>/p5_agent_eval.py assemble --config default --pub <before.PUBLIC> --tag before --out-dir <dir>
python <tool_dir>/p5_agent_eval.py assemble --config default --pub <after.PUBLIC>  --tag after  --out-dir <dir>
```

- **Criterion: after placeholder normalization, the per-message diff == exactly the expected change set**. One extra = unintended change; one missing = an omission.
- Calibrate noise handling first (lesson: placeholder numbers shift when entries are added/removed — normalize `«BLOCK:NNN»`→`«BLOCK:X»` before comparing).
- Label conclusions "**offline approximation**". The `p5 check` asserter applies only to **tag-channel** chain-of-thought; outputs of reasoning-channel models never enter body tags — never score them with it.

## L3 Mock ground truth (mandatory when switches/entries/param structure changed)

- B1-capture the post-change preset's assembled request (see truth-sources.md).
- Criterion: assembly shape matches design (template unique, locator placement correct, macro expansion clean, no leakage).
- State the limit as you state the conclusion: this proves the assembly layer only; the user's script layer may rewrite further — that sentence must accompany the conclusion delivered to the user.

## L4 Effect layer (always belongs to the user)

- Deliver a **test protocol**: baseline statement (switch state + file + sha16) → N turns of comparable scenarios → 3–5 countable metrics (each with a comparison baseline) → "send me the results; I'll backfill the dossier".
- **Forbidden**: sub-agents role-playing the model, imagined models, self-written asserters scoring your own output.
- A deliverable that never ran L4 writes, in the effect column: "**unverified — missing N turns of real testing**". That is not a defect statement; it is honesty.

## Verification suite (rerun at every close-out inside a governed repository)

```bash
python <tool_dir>/01_source_manifest.py    # [verify] OK
python <tool_dir>/03_p0_pipeline.py        # verdict: all passed (whenever the sanitizer pipeline was touched)
python <tool_dir>/p5_groups_build.py --check  # PASS (whenever the group registry was touched)
python <tool_dir>/04_audit_for_git.py      # zero hits on files pending commit (before every commit)
```

## The bar for the words "verification passed"

Only when all hold: L1 all green + L2 diff == expected + (for structural changes) L3 shape correct + the effect column honestly says "unverified" or cites the user's real test data. **Mixing the three layers once = the report is void and must be rewritten.**
