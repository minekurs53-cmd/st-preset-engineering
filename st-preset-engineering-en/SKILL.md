---
name: st-preset-engineering-en
description: Engineering workflows for SillyTavern (ST) presets: structural autopsy, diagnosis, safe modification, feature add/remove, new-model branch adaptation, full refactoring, and effect verification. Use whenever the user provides a preset .json file, or mentions SillyTavern / ST preset / prompt_order / prompt presets / entries / output template / chain-of-thought / prefill / regex / Tavern Helper scripts and wants to modify, fix, add, remove, troubleshoot, refactor, adapt to a model, or evaluate — including complex presets with script panels, entry management UIs, and lorebook integration. Triggers even for "take a look at this preset" or "just change the length limit".
license: MIT
compatibility: For agent platforms that support the Agent Skills convention (SKILL.md + .agents/skills or equivalent directory). The seven-step skeleton is pure Markdown with no bundled scripts; the companion tool band is Python 3.8+ scripts referenced via preset-profile.json's tool_dir (not distributed with the skill; see repository README).
metadata:
  version: "1.1.0"
---

# ST Preset Engineering

Perform **autopsy → diagnosis → surgery → verification → handover** on SillyTavern preset JSON. Every discipline in this skill is distilled from the hard-won lessons of real reverse-engineering projects (`references/pitfalls.md` — every entry actually happened), not generic best-practice filler.

## Safety Boundary Statement (read before use)

- **File access**: only read/write paths registered in the profile (working copy, out-of-repo output directory); the source preset directory is **read-only**, and `profile.verify_command` runs before and after every operation
- **Network**: no external services are contacted; the mock endpoint listens on 127.0.0.1 only; remote links inside preset scripts are **disabled by default** (`references/red-lines.md` rule 7) — never decide on the user's behalf to execute remote code
- **What it does not do**: the analysis surface stays on a PUBLIC copy with zero lexicon hits (placeholder pipeline, see `references/red-lines.md` rule 2); no hand-editing large JSON (always the generator's three assertions); sensitive artifacts never enter any version control history
- **No secrets**: this skill contains and never requests any API keys or credentials

## Step 0: read the profile (always the first action)

The per-preset instance parameters (paths / red lines / group notation semantics / branch pairing / complexity) live in `preset-profile.json`:

1. Lookup order: current working directory → repository root → `~/.preset-profiles/<preset-name>.json`
2. Not found → first build a minimal profile per `references/profile-spec.md` (S0/S1 fill in most of it), then continue
3. Every path/command in the profile must actually run; **any unreadable reference must be explicitly reported — never silently skipped**

## Step routing (enter as needed; don't run everything)

| Request type | Steps | Notes |
|---|---|---|
| "What's wrong with this preset?" | S0→S1→S3 | The report is the deliverable; hands off |
| Toggle a switch / change route / sampling params | S0→S4(minimal)→S5(L1) | Minute-scale task; don't use a sledgehammer |
| Edit entry body / add or remove entries | S0→S1(only if stale)→S4→S5(L1+L2) | |
| Add a model branch | S0→S1→S3→S4→S5(L1+L2+L3) | Read `references/surgery.md` first |
| Refactor / deep cleanup | Full pipeline; S4→S5 may loop, **re-take ground truth (S2) after each round** | |
| Effect evaluation | S0→S2 | L4 belongs to the user; never self-assess (see Discipline 2) |

## The seven-step skeleton (details in the matching reference)

```
S0 Red lines → S1 Autopsy → S2 Ground truth → S3 Diagnosis → S4 Surgery → S5 Verification → S6 Handover
```

- **S0** (`references/red-lines.md`): source read-only + before/after verification; sensitive-artifact triage (placeholder pipeline / out-of-repo quarantine / no distribution)
- **S1** (`references/autopsy.md`): structure dossier + extension hookpoint inventory + complexity grading
- **S2** (`references/truth-sources.md`): choosing between mock capture / offline assembly / the user's real dump
- **S3** (`references/diagnosis.md`): three defect classes + four-way evidence labels
- **S4** (`references/surgery.md`): generator's three assertions; **hand-editing JSON is forbidden**
- **S5** (`references/verification.md`): three verification gates from cheap to expensive; stop at first failure
- **S6** (`references/handover.md`): change ledger + user manual + state sync

## Three inviolable disciplines

1. **Source is read-only**: every change happens on a working copy; run the read-only verification before and after (command in `profile.verify_command` — if absent, create one first)
2. **Real data > mock > sub-agent roleplay**: effect conclusions may only come from the first two; when unverifiable, write "unverified + what's missing" — **never downgrade the wording to make "not tested" read as "passed"**
3. **Surgery always goes through the generator's three assertions**: uniqueness (`count==1`) / enumeration (diff equals exactly the change list) / equality (byte-identical apart from the enumerated fields). Hand-editing a 1MB JSON is always wrong

## Complexity adaptation (skeleton fixed; only depth changes)

The S1 dossier produces `profile.complexity`:

- **light** (≤50 entries, ≤2 scripts, ≤6 regexes): S2 may use the B2 offline approximation only
- **medium**: standard pipeline
- **heavy** (script panels / multiple extension config blocks / entry-management UI): S3 must add the "script interaction & hookpoint coupling" diagnosis; **before touching any entry, scan the scripts for references to that entry's name/group number** — panel scripts often index entries by name

## Drift self-check (start of every task + every 30 minutes)

1. What **specific** problem will this action save the user in SillyTavern? Can't answer → stop (lesson: self-serving documentation)
2. Am I verifying my own output or writing docs about my own work? (lesson: self-verification)
3. Is there a cheaper real experiment? One hour of mocking beats a doc marathon
4. Is the focus "the user can use this" or "the process is rigorous"? Drift needs the user to pull you back twice — don't wait for a third

## Tool quick reference

Call cards for the eight tools (command / expected output / failure criteria / generality tags): `references/toolmap.md`. All point at the existing tool directory (`profile.tool_dir`) — **never rebuild or copy them**.

## Additional obligations inside a governed repository

If this work happens in a repository with session-close obligations (per that repository's own AGENTS.md / collaboration docs): on close-out, additionally follow its close-out rules — state sync + verification suite rerun + commit — and register newly discovered errors in that repository's errata channel.

**Exemptions and conflicts**: read-only tasks (the "report is the deliverable, hands off" rows in the routing table) trigger no repository write obligations; when a session has an external hard constraint (test rules, an explicit user prohibition on writing to the repo), **the hard constraint wins** — declare every skipped close-out obligation in the handover. Never skip silently, and never break a hard constraint to follow a rule.

## References index (read as needed)

| File | When to read |
|---|---|
| profile-spec.md | Step 0, building/validating the profile |
| red-lines.md | S0; before any operation involving sensitive content or outbound files |
| autopsy.md | S1; on unfamiliar structures/extension blocks |
| truth-sources.md | S2; before any "effect" conclusion |
| diagnosis.md | S3 |
| surgery.md | S4; **mandatory** before touching JSON |
| verification.md | S5; before writing the words "verification passed" |
| handover.md | S6; before session close-out |
| pitfalls.md | When planning large tasks; when confused |
| best-practices.md | Before writing new entry bodies / editing output templates |
| toolmap.md | Before invoking any tool |
