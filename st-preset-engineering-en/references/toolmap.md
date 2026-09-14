# toolmap.md — Tool call cards

> **When to read**: before invoking any tool.
> **Generality tags**: ✅ = works for any preset; 🔶 = works but output contains instance-specific interpretation; ⛔ = depends on instance data; replace per family.
> All paths relative to `profile.tool_dir`. **When a tool refuses you, first check whether you put something in the wrong place — never bypass the tool.**

## 01_source_manifest.py — read-only source verification ✅

- Command: `python 01_source_manifest.py`
- Expected: `[verify] OK — all source file hashes match the manifest`
- Failure criterion: any `[FAIL]` / hash mismatch → **stop all surgery**; find out who touched the source first
- Timing: S0 (once before, once after); for other presets use your own hash_baseline (first 16 hex of sha256)

## 02_sanitize_nsfw.py — sanitizer ✅ (lexicon swappable)

| Subcommand | Purpose | Notes |
|---|---|---|
| `classify --in <preset> --out <CSV>` | Lexicon classification (SAFE/MIXED/OVERRIDE/EXPLICIT) | The CSV contains original text → the tool forces writes into the sensitive directory only |
| `apply --in --out --map --public-json` | Produces the quarantined copy + PUBLIC copy | PUBLIC copy = zero lexicon hits = safe to export |
| `verify-structure --src --san --map` | Verifies "changes confined exactly to mapped slots" | Run after surgery; guards the skeleton |
| `explain` | Diagnoses which lexicon entry triggered sanitization | **Always run before tuning the lexicon** — one generic word measurably misclassified 32% of content |
| `restore` | Byte-identical restore from the sanitized copy | Verifies pipeline integrity |

## 03_p0_pipeline.py — one-click sanitize + full acceptance ✅

- Command: `python 03_p0_pipeline.py [--all]`
- Expected: `verdict: all passed ✅`
- Mandatory whenever the sanitizer pipeline / lexicon was touched

## p1_evidence.py — structure dossier (S1 core) ✅

```bash
python p1_evidence.py report --preset <any preset.json>   # single-version dossier (orphans/dead slots/groups/regexes/scripts)
python p1_evidence.py report --target <registered>        # registered source version ⛔ (the target list is instance data)
python p1_evidence.py drift                               # cross-version drift (when multiple versions exist)
```
- Reading guide: autopsy.md §1; the five risk criteria: verification.md L1
- Limit: group interpretation depends on notation + phase markers — unfamiliar notation systems downgrade to advisory; ask the user

## p5_groups_build.py — group registry self-check ⛔ (registry is instance data)

- Command: `python p5_groups_build.py --check` (verify only; without --check it rewrites the file)
- Expected: `self-check PASS: N groups…`; the registry reconciles group-by-group with p1 report §3
- General case: never reuse this registry across preset families; use profile.model_branches + group-mutex semantics instead

## p5_agent_eval.py — offline assembly + assertions 🔶

```bash
python p5_agent_eval.py assemble --config default --pub <PUBLIC copy> --tag <name> --out-dir <out-of-repo dir>
python p5_agent_eval.py check <reply.txt> --template <registered template> --turn N
```
- `assemble`: ✅ any PUBLIC copy; produces the message sequence for before/after per-message diff
- `check`: ⛔ **the template is instance-specific** (length ranges / tag literals / option enums), and applies only to the **tag-channel** chain-of-thought — never score reasoning-channel models (responses carry reasoning_content) with it

## p2_mock_llm.py — mock endpoint (S2 request-side ground truth) ✅

- Command: `python p2_mock_llm.py --port 9999`
- Setup: ST → Custom (OpenAI-compatible) → `http://127.0.0.1:9999/v1` + any key + any model name → Connect → send a message → read the dump
- Dumps go to out-of-repo temp directories; temporary chats, zero residue afterwards
- Proves: the assembly layer; **not what the model sees in real user runs** (the script layer rewrites request history)

## 04_audit_for_git.py — pre-commit audit ✅

- Command: `python 04_audit_for_git.py`
- Reading: the **hit column** (first column) — files pending commit must be 0; the heuristic column (third column) is a coarse filter for descriptive wording (combinations like "coverage/rules/compliance" trigger it); committed docs carry 1 too — **the hit column is authoritative**
- Mandatory before every commit; new files with hits >0 → alias/sanitize; never force them in
