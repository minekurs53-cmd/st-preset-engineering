# toolmap.md — Tool contract cards & the tool-building method

> **When to read**: before invoking any tool; whenever `profile.tool_dir` is missing, not runnable, or empty and a tool must be built (**mandatory**).
> **Generality tags**: ✅ = works for any preset; 🔶 = works but output contains instance-specific interpretation; ⛔ = depends on instance data; replace that part per family.
> **When a tool refuses you**: first check whether you put something in the wrong place — never bypass the tool.

## The tool-layer model of this skill

This skill only prescribes the **capability contract** each tool must satisfy (input → decision logic → output format → expected output); it binds to no concrete implementation. What lives in the directory `profile.tool_dir` points at is decided locally: either a set of ready-made reference scripts, or equivalent tools newly built in this session against the contract cards. **Tool names appear only in command examples; any tool implementing the same contract may be registered in `profile.tool_dir` as a replacement.**

## The tool-building method (follow when tool_dir is not runnable)

**Trigger**: at Step 0 you find `tool_dir` null, the directory absent, or a card's tool failing to run → implement the equivalent tool locally (Python 3.8+ stdlib is enough) per the matching contract card, register it in the profile, then continue. Build only the card the current step needs — **never the whole set up front**; before a step that depends on a card, its tool must have passed the card's self-test.

Five disciplines (violating any one = the tool is unfit; rewrite):

1. **Instance data lives outside the code**: no preset-family names, no absolute source paths, no lexicon content in code — everything is injected via CLI arguments, the profile, or standalone data files (e.g. a lexicon JSON). A tool with hard-coded instance data dies with the preset family, which is as good as not written.
2. **Read-only & write discipline**: the source preset directory is read-only; file writes use explicit `newline="\n"` (Windows text mode once made a "byte-identical restore" differ by 4896 bytes); generated JSON uses `ensure_ascii=False`. **On a Windows console with GBK encoding, tool output containing emoji/CJK throws `UnicodeEncodeError` or turns into mojibake** — reconfigure **both stdout and stderr** at the tool entry (`sys.stdout.reconfigure(encoding="utf-8")` **and** `sys.stderr.reconfigure(encoding="utf-8")`), or set `PYTHONIOENCODING=utf-8` before running. **Easy to miss: reconfiguring stdout only** — `sys.exit("CJK error message")` goes to stderr; unreconfigured, a calling script captures mojibake and every assertion/string match silently fails (measured: the same "refused to write" error looked fine by hand but mojibaked when captured by a script).
3. **Minimal output surface**: tool output contains only structure, counts, and aliases (`«NAME:NNN|category»`) — **never entry body text**; tool output may end up in reports or git.
4. **Conclusions must be diff-able**: output format is stable and deterministic (same input, byte-identical across runs), and verdicts land on a one-line greppable marker (e.g. `[verify] OK`) — the expected output doubles as the acceptance assertion.
5. **Self-test first**: build a minimal fixture first (hand-write a 3-entry toy preset JSON) and pass the card's "self-test" item before touching a real preset. A real preset is not a test environment. The fixture must cover every code branch — measured: a backslash-escaping error in an on-disk regex didn't trigger in the "no script" branch and was only caught by the "with script" branch's self-test; hand-inlined runs being fine ≠ the on-disk version being fine.

Declare every newly built tool in the handover (tool name / which card it implements / whether self-test passed).

---

## Card 1 — read-only source verification (S0) ✅

- **Contract**: scan all `.json` files in the source directory (**recursive, including subdirectories**; pass the source directory explicitly via `--dir`) → record bytes + sha256 per file → compare against the baseline manifest. Report every addition/removal/hash change; with no differences, emit one line containing `[verify] OK`; exit code 0 = match, 1 = mismatch. **Also supports `--files <f1> [f2…]` single-file/subset mode** — when source files are scattered across a shared directory (multi-user preset folders) or only a few files are in scope, use a file list so `--dir` recursion doesn't sweep in other people's presets (measured: 9 scattered source files in a shared folder); each mode gets its own manifest.
- **Reference command**: `python 01_source_manifest.py` (first run uses `--write` to create the baseline; `--no-names` makes the manifest committable — a nameless manifest pairs with files by sorted index)
- **Write guard**: the manifest file must **never be written into the scanned directory** — the manifest itself would count as a "new file"; the first self-test run steps on exactly this (it actually happened)
- **Expected**: `[verify] OK — all source file hashes match the manifest`
- **Failure criterion**: any mismatch → **stop all surgery**; find out who touched the source first
- **Timing**: S0 (once before, once after); fallback without a tool: compute sha256 (first 16 hex) of source files into `profile.hash_baseline`, compare before and after
- **Self-test**: flip one byte of any source file → must report a mismatch; restore it → must report OK

## Card 2 — sensitive-term scan (S0 criterion) ✅ (lexicon swappable)

- **Contract**: read a standalone lexicon file (JSON, schema: `[{"text": "...", "category": "..."}]`, category enum `override / explicit / illegal`) → compile match patterns (case-insensitive, non-overlapping) → count hits on the target file. **Output only file names, hit counts, and hit term names**, never context; run per file over a repo to get the "zero hits on files pending commit" verdict. There is no canonical starter lexicon — build one of 30–60 terms across the three categories, register it in the profile, and declare it as self-built in the handover. **Any text file can be scanned**: non-JSON (.md/.txt etc.) is scanned as whole text with region `text` — "zero hits on files pending commit" often covers half-markdown doc sets; accepting only JSON errors out the whole batch (measured: 10 of 19 delivery docs were .md and failed on first run).
- **Reference command**: `python 04_audit_for_git.py` (reading: the **hit column** is authoritative; the heuristic column is a coarse filter for descriptive wording — committed docs carry 1 too)
- **Hit triage**: hits split into two kinds — **structural hits** (identifier / field-name hits, e.g. the ST built-in slot names `nsfw`/`jailbreak` — not a content-sensitivity signal) and **content hits** (body-text hits). Classify by hit term name first; only content hits drive the triage decision. **Separate structural/content hits by region** (content/identifier/extension/key/other) via a `--regions`-style flag so conclusions like "all 85 hits land in `other` = purely structural" are reproducible (measured: a lorebook input's 85 hits all landed in other; one preset's `nsfw` term split across content/identifier/extension) — replaces "judge structural vs content by eyeballing".
- **Expected**: zero (content) hits on files pending commit; new files with hits >0 → alias/sanitize; never force them in
- **Discipline**: what counts as sensitive is decided by lexicon hit counts, **never eyeballing** — one generic word measurably misclassified 32% of content as sensitive (lesson L7 in pitfalls)
- **Self-test**: add a test term to the lexicon → the hit count of a fixture containing it must increase by exactly 1

## Card 3 — sanitizer pipeline (executor of S0 red-line 2) ✅ (lexicon swappable)

- **Contract** (five subcommands):
  - `classify --in <preset> --out <CSV>`: lexicon classification per entry (SAFE / MIXED / OVERRIDE / EXPLICIT); the CSV contains original text → the tool must refuse to write the CSV outside the sensitive directory
  - `apply --in --out --map --public-json`: produces the quarantined copy + mapping table + PUBLIC copy. Sensitive body text is replaced with `«BLOCK:NNN»` / `«NAME:NNN|category»` placeholders while the **machine-readable skeleton is fully preserved** (identifier / prompt_order / macros & variable names / XML tags / regex flags); the PUBLIC copy = zero lexicon hits = safe to export
  - `verify-structure --src --san --map`: proves "changes confined exactly to mapped slots" — mandatory after surgery; guards the skeleton
  - `restore --in --map --out`: byte-identical restore from the sanitized copy (diff location reports byte offset only, never context)
  - `explain --in`: diagnoses which lexicon entry triggered sanitization; **always run before tuning the lexicon** (guards over-strict sanitization; see the 32% lesson in Card 2)
- **Write discipline**: the mapping table is forced into a gitignored directory; the tool refusing = you put it in the wrong place — change the path, don't bypass
- **Self-test**: `restore` output must byte-diff empty against the source; the PUBLIC copy scanned by Card 2 must have zero hits
- **Applies**: under `sensitive_strategy: "quarantine"` the apply/verify-structure/restore steps can be skipped; but **`classify`/`explain` still run** (they only read the preset + lexicon and produce no sensitive artifacts) — the over-strict-lexicon recheck under quarantine goes through these two steps, there is no "whole card unavailable" dead end. Under `none`, skip the whole card.

## Card 4 — pipeline acceptance driver ✅ (optional)

- **Contract**: runs the full Card-3 chain (classify → apply → scan → verify-structure → restore) plus the Card-2 taint recheck over the registered file list, then summarizes one line verdict.
- **Reference command**: `python 03_p0_pipeline.py [--all]`; expected: `verdict: all passed ✅`
- **Timing**: mandatory whenever the sanitizer pipeline / lexicon was touched
- **Note**: it is only a driver chaining Cards 1–3; a fresh environment may skip it and run the cards by hand

## Card 5 — structure dossier (S1 core) ✅

- **Contract**: takes **any preset JSON** (read-only), prints a six-section report to stdout —
  - **Input-type gate**: at the entry, probe the top-level structure — no `prompts` key / `entries` is a dict (lorebook v2 signature) → print an input-recognition notice (top-level keys + what non-preset format it is) and **refuse to emit an empty report** (measured: a lorebook input produced a "well-formed but empty" report with exit 0, nearly misread as an empty preset). Put the preset gate at the entry; missing `prompt_order`/`extensions` is reported as a field-level notice.
  1. **Pipeline table**: per-entry switch state (per `prompt_order`'s `enabled`; `prompts[].enabled` is a distorted leftover) / phase / group mark / role / length / macro ops. **Emit every order block separately**; "which block is effective" is an inference (usually the largest block / the one with dummyId 100001) — label its provenance in the report. **Stale-ref reconciliation**: order referencing identifiers that do not exist in prompts[] (the classic residue of a hot-upgrade rename/version swap; measured) must be listed separately; the on= column distinguishes orphan (in prompts, not in order) from stale (in order, not in prompts) — together with the orphan table this forms a two-way reconciliation. **Also annotate two assembly forms** (not part of the inline-order flow; reordering does not move their injection point):
     - **Absolute-injection entries** (`injection_position=1`): injected into chat history by `injection_depth` (source `openai.js:1219-1224/1305`); mark enabled@effective-block + depth + inj_order;
     - **Custom `sp=true` entries** (`system_prompt=true`, identifier not a built-in slot): relative assembly has a filter path (`openai.js:1214`), needs manual review.
     (Measured: several presets had absolute-injection entries (some ON, depth=2), and some carried `agent*` sp=true entries — the first dossier version missed both dimensions; patched in later.)
  2. **Slot table**: setvar/addvar writers vs getvar readers (write-only = dead slot; multiple writers on one slot = timing landmine). **"Read" is decided across three places: entry bodies + scripts + regexes**; lorebooks/character cards are out of scope — declare the limit. New common flag reading: **NO_STATIC_WRITER** (zero writers across all three places) = the slot is probably written at runtime (manual `/setvar` or external injection) — reading template: "zero static writers (sibling slots all have writers) → presumed runtime write, pending user confirmation". **Suspicious slot-name marking**: slot names containing regex metacharacters (`[` `]` `*` `(` `\` etc.) get a `#suspicious (regex-syntax residue)` tag and a separate count — text templates inside huge scripts get mis-extracted as slot names (measured). **External-source notice**: when extension blocks contain external-source/host declaration keys (`*source_worldbook`, `*_entries`, `*_in_host`, `database_companion` patterns), add a fixed notice line — "this preset declares external sources; dead-slot conclusions are limited to entries/scripts/regexes" (measured: 35 dead slots coexisting with external-worldbook declarations; without the notice, external consumption reads as dead)
  3. **Group table**: clusters by the marks in `profile.group_notation` × phase, with each group's ON count. **Single-member groups carry no group semantics** (usually decorative prefixes) — annotate as "naming convention (structural fact)". **Notation-system agnostic**: any concrete mark spelling (❗❔🔒 etc.) is an example only; real presets may use entirely different systems (paired boundary entries, emoji+category names, in-name annotations like "(pick one)") — the tool must not hard-code any mark→semantic mapping; everything comes from the profile; unregistered → downgrade to "facts only", ask the user
  4. **Orphan table**: entries in prompts[] but not in order. When the "functional/junk" split is undecidable at scale (>10 orphans), cluster by three clues: name semantics / slot references / character count. **Distinguish "data-container entries" from functional orphans**: orphans with unusually large character counts whose body is serialized JSON (e.g. `SPreset配置`-class entries = a byte copy of extensions.SPreset) → annotate "container candidate (data copy, not a functional entry)" with a **do-not-wire-back** warning (wiring back injects the whole config body into the request) — annotate, don't guess; semantics still go to the user (measured: one orphan was an 18460-char JSON container, identified by hand).
  5. **Regex table**: each regex's flags; `markdownOnly=false` and `promptOnly=false` and not disabled → flag as error (double-False ban, see red-lines rule 5). **Copy-layer reconciliation**: with multiple layers (native + binding/container layers), besides the per-layer listing you must emit reconciliation lines — baseline the native layer: `only-native list / only-copy list / same-name field diffs (find/repl lengths, flags)`. "Listing layers separately" ≠ "reconciling" (measured: two layers out of sync — 8 only-native, 2 only-copy, same-name find length 30 vs 54 — only obtainable by an ad-hoc manual comparison)
  6. **Script table**: scripts and remote URLs (external links = risk, graded per red-lines rule 8); top-level params (connection-bound fields annotated per the accurate wording in red-lines rule 6). **injectScript extraction contract**: extract the argument region by **balanced parentheses** (a single-arg regex gets truncated at the first nested right paren, losing the URL); **list every call in full** (showing only the first = silent loss); any display truncation must be explicitly annotated
- **Reference command**: `python p1_evidence.py report --preset <any preset.json>`; multi-version drift: run the dossier per adjacent version and diff (the reference implementation also has `--target` / `drift` subcommands, whose version lists are instance data ⛔)
- **Reading guide**: autopsy.md §1; the six reconciliation criteria: verification.md L1
- **Known limit**: group interpretation depends on notation + phase markers — under an unfamiliar notation system, **list facts without verdicts**, downgrade to advisory, ask the user, never guess
- **Discipline**: macro extraction outputs "op + variable name" only, never body text (same as method discipline 3)
- **Self-test**: on a hand-made fixture, assert all six sections' counts one by one — at least 5 entries: 1 always-on + a mutex pair (2 members) + 1 orphan + 1 double-False regex, plus one nested-args and one multi-call injectScript sample. **Add three new samples**: 1 absolute-injection entry (`injection_position=1`) asserted to be flagged, 1 custom `sp=true` entry asserted to be listed, and 1 lorebook-shaped input without a `prompts` key asserted to trip the input-type gate instead of emitting an empty report. **Four more samples**: 1 stale reference (order referencing a non-existent identifier) asserted to land in the stale-ref list, 1 same-name boundary pair asserted to toggle closed (later entries no longer inherit the zone), 1 copy-layer sample (one only-native / one only-copy / one same-name diff) asserted to produce reconciliation lines, and 1 regex-metacharacter slot name asserted to be marked suspicious.

## Card 6 — group registry self-check ⛔ (registry is instance data)

- **Contract**: reconciles a group registry (JSON) against Card-5 report §3 group by group, printing `self-check PASS: N groups…` or per-group differences.
- **Reference command**: `python p5_groups_build.py --check` (verify only; without `--check` it rewrites the file)
- **General case**: never reuse an old registry across preset families — replace it with profile.model_branches + group-mutex semantics, or build a fresh registry for the new family and re-run the self-check
- **Self-test**: corrupt one group's ON count in the fixture registry → must report FAIL

## Card 7 — offline assembly (S2-B2 / S5-L2) 🔶

- **Contract**:
  - `assemble`: reads a PUBLIC copy → replays ST assembly semantics per prompt_order order, enabled state, phase marks, and role semantics → outputs a per-message sequence (JSON) for before/after per-message diff. Does not expand macros (placeholders stay verbatim; normalize `«BLOCK:X»` per surgery.md before diffing)
  - `check`: runs template assertions on a reply text (length ranges / tag literals / option enums)
- **Reference command**: `python p5_agent_eval.py assemble --config default --pub <PUBLIC copy> --tag <name> --out-dir <out-of-repo dir>`; `check <reply.txt> --template <registered template> --turn N`
- **Generality**: `assemble` ✅ any PUBLIC copy; `check` ⛔ **the template is instance-specific**, and applies only to the **tag-channel** chain-of-thought — never score reasoning-channel models (responses carry reasoning_content) with it (truth-sources.md channel warning)
- **Honest labeling**: label every output "**offline approximation**" — it re-implements assembly semantics in a tool, and its divergence from ST ground truth (message merging / runtime additions / connection-field handling) must be kept in mind
- **Self-test**: two runs on the same input are byte-identical (determinism); after toggling one switch, the assemble output differs by exactly the expected message

## Card 8 — mock endpoint (S2-B1 request-side ground truth) ✅

- **Contract**: a minimal OpenAI-compatible mock listening on `127.0.0.1` only — `GET /v1/models` returns a single model; `POST /v1/chat/completions` dumps the **entire request body** to disk (including SSE when stream=true) plus a "safe summary" (message count / role sequence / prefill presence & length / model / stream, **no body text**); returns a fixed reply to every request
- **Reference command**: `python p2_mock_llm.py --port 9999`
- **Setup**: ST → Custom (OpenAI-compatible) → `http://127.0.0.1:9999/v1` + any key + any model name → Connect → send a message → read the dump
- **Discipline**: dumps go to out-of-repo temp directories; temporary chats, zero residue afterwards
- **Proves**: the assembly layer; **not what the model sees in real user runs** (the script layer rewrites request history — see truth-sources.md)
- **Self-test**: after one request, the dump file exists with complete messages; grep finds no entry body text in the meta summary
- **Fallback**: any OpenAI-compatible mock that echoes request bodies to local files works — full-body dumping plus a body-free safe summary are the only hard requirements

## Card 9 — reference scan (S3 three-way consistency / pre-S4 UI-reference check for heavy presets) ✅

- **Contract**: takes a preset JSON (read-only) + a target-literal list (entry names / identifiers / group numbers) → searches the **three consumer classes — script bodies, regex pattern/replaceString, entry bodies** — for each literal → outputs a "literal → consumer location" list (aliased pairs). Verdicts carry two grades:
  - `(id)` hit = exact identifier match (ASCII word boundary) — **verify item by item before treating it as a real reference**; built-in slot names ('main'/'nsfw'/'scenario'…, list in autopsy §6) and short names (pure digits / ≤2 chars) are high-false-positive by default; spot-check and you'll see (measured: 6 short-name spot checks, all false — CSS numbers / `$1` backreferences / JS properties / module ids)
  - `(name)` hit = entry-name literal — same rule; short generic words always misfire
- **Large target sets**: support `--hits-only` (hit lines + a summary line only) — measured: 431 targets produced 429 "zero hit" lines of pure noise; curbs read cost
- **Counting caveat**: the `contracts` tag matrix counts literals as substrings — prefix containment double-counts (`think`'s count includes `thinking`); annotate that caveat in output (or split by longest match)
- **False-positive criterion**: a hit counts as a real reference only if it can be explained as **consumption semantics targeting that entry**. Exclude: language keywords / CSS numbers / regex backreferences (`$N`) / the script's own identifiers and module ids / standard slot words.
- **False-negative blind spot (ties into the `contracts` matrix)**: a tag literal written as an alternation (e.g. `<(thinking|suggestions|disclaimer)>`) is not recognized by `</?tag>`-shape extraction → the matrix wrongly reports "one-sided". Every tag the matrix marks "one-sided" gets one **targeted substring recheck** (search the bare word in the other two sides) before a verdict (measured: `<(thinking|suggestions|disclaimer)>` reported "one-sided" but is truly two-sided; fixed by the substring recheck).
- **Reference command**: `python p1_evidence.py refscan --preset <preset.json>` (the reference implementation also has a `contracts` subcommand emitting the tag-contract matrix)
- **Expected**: every entry name / group number referenced by a UI script has a consumer location in this list; a missing one = orphaned reference (a risk item before renaming/deletion)
- **Timing**: S3 contract scan (three-way consistency); before S4 rename/delete (mandatory for heavy presets)
- **Self-test**: a script body in the fixture contains some entry name → must hit that entry; an entry with no consumers → zero hits
