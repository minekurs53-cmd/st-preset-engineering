# autopsy.md — S1: Autopsy (structure dossier)

> **When to read**: S1; when encountering unfamiliar structures/extension blocks.
> **Deliverable**: structure dossier (p1 report) + extension hookpoint inventory + complexity grading → backfill profile.structure / complexity.
> **Core principle**: every autopsy conclusion must be reproducible (a command or a probe script) — **never eyeball-read a 1MB JSON**.
> **Without a tool**: if contract Card 5 cannot run, build the dossier tool yourself per toolmap's method (the six-section report IS the output contract), pass its self-test, then run — **never skip the dossier and eyeball instead**.

## 1. Run the structure dossier

```bash
python <tool_dir>/p1_evidence.py report --preset <preset.json>   # any preset (contract Card 5)
python <tool_dir>/p1_evidence.py report --target <registered>    # a registered source version ⛔ (the version list is instance data)
python <tool_dir>/p1_evidence.py drift                           # cross-version drift (when multiple versions exist)
```

How to read the report (each section = one risk class):

| Section | Content | What to look at |
|---|---|---|
| §1 Pipeline table | Per entry: switch state / phase / group / role / length / variable ops | **The enabled column is the effective state** (`prompts[].enabled` is a stale artifact — never trust it); whether the role distribution matches family semantics (assistant = pseudo-prefill carrier); **absolute-injection entries** (`injection_position=1`, not in the inline order, injected into chat history by depth) and **custom `sp=true` entries** listed separately — reordering them does not move the injection point (measured) |
| §2 Slot table | setvar/addvar writers vs getvar readers | **Write-only slots = dead slots** (the original had 7, all historical corpses); multiple setvar writers on one slot = a timing landmine (enabling a disabled entry can wipe the main slot — zombie-entry check); **"read" is decided across three places: entry bodies + scripts + regexes** (lorebooks/character cards out of scope — declare); NO_STATIC_WRITER (zero writers in all three) = presumed runtime write (manual /setvar or external injection), pending user confirmation |
| §3 Group table | Notation × phase → members and ON counts | **Mutually exclusive group with ON≠1 = configuration violation**; but first confirm notation semantics (see §4 known limits); **single-member groups carry no group semantics** (usually decorative prefixes — annotate as "naming convention" to reduce noise) |
| §4 Orphan table | In prompts[] but not in order | Orphan = the feature doesn't exist (invisible/untoggleable); list functional orphans separately from junk orphans; **data-container entries** (unusually large char count, body is serialized JSON, e.g. `SPreset配置` = an extensions copy) listed separately and marked "do not wire back" |
| §5 Regex table | Flags of all 14+ regexes | **Double-False and not disabled = error** (permanently rewrites chat history); record pattern literals into the contract list |
| §6 Script table | Scripts / remote URLs | External scripts = risk; record script/button names into the hookpoint inventory |
| Top-level params | temperature / max_tokens / media_inlining etc. | **Connection-bound fields don't take effect from presets** (the cpp lesson) — these belong in handover notes, not JSON |

## 2. Extension hookpoint inventory (the core move for complex presets)

For each sub-block under `extensions`, register a hookpoint and answer three questions: **who reads it? who writes it? what does changing it affect?**

```jsonc
// Hookpoint registration format (goes into profile.structure.extension_hookpoints)
[
  {"block": "regex_scripts", "kind": "ST native regex", "status": "effective",   // effective|dormant|unknown
   "note": "The regex-activation layer; the only place that must be changed"},
  {"block": "tavern_helper.scripts", "kind": "Tavern Helper scripts", "status": "effective",
   "note": "2 scripts with floating-window UI; index entries by name/group number → scan references before touching entries"},
  {"block": "SPreset", "kind": "extension binding layer", "status": "dormant",
   "note": "A second copy of the same-name regexes; fully inert without SPreset installed — when changing regexes, sync both or drop the binding layer"}
]
```

**Law of three copies**: when the same data (e.g., the same-named regexes) appears in multiple places, first determine at runtime which copy is effective (the binding layer is inert without the extension), **change only the effective layer, sync or delete the rest — never maintain both in parallel**.

**UI script coupling**: panel/button scripts often index entries by **entry-name or group-number strings** — before S4 renames/deletes an entry, run a repo-wide reference scan over script bodies (search the entry name as a literal).

## 3. Complexity grading (drives pipeline depth)

| Level | Criteria (any one triggers) | Pipeline impact |
|---|---|---|
| light | ≤50 entries, ≤2 scripts, ≤6 regexes | S2 may use the B2 offline approximation only |
| medium | default | standard pipeline |
| heavy | script panels/UI, ≥2 extension config blocks, >200 entries, >20 regexes | S3 adds the "script interaction & hookpoint coupling" diagnosis; S4 requires a UI reference scan first; S5 adds script-layer regression |

When the criteria and substance disagree (e.g. "≥2 extension blocks but no scripts no UI" grading heavy), grade by the letter and annotate the substantive level in `complexity.note` — a distorted criterion beats grading down at will.

## 4. Known limits (declare honestly; never pretend generality)

- **Zone/group recognition must be style-agnostic**: at least five real styles exist — 〈…〉 heading entries, paired boundary entries (——X begin——/——X end——), emoji+category-name prefixes, HTML-tag paired entries (`<background>…</background>`), "----X" hyphen headings. **The same skill will meet mutually incompatible zone families**: built-in style recognition is untrustworthy; the profile's `zone_title_re`/`zone_pair_start_re`/`zone_pair_end_re`/`zone_category_re` data-driven rules **take precedence over built-ins**; if the profile has none and recognition fails → switch to a targeted-extraction probe; never build on a wrong recognition. (All five forms appeared in one test run: 〈〉/——pairs/HTML pairs/emoji prefixes/━━ separator headings; the emoji-prefix and ━━ separator classes needed `zone_category_re` and the "capture group 1 = zone name" semantics — see profile-spec.)
- **Multi-copy "effective-layer determination + sync verification" must be tooled, not hand-diffed**: the same data (e.g. a regex set) may live in several layers (native layer + SPreset.RegexBinding + a container-entry JSON); with multiple layers, first determine which is effective (the binding layer is inert without the extension), **change only the effective layer, sync or delete the rest**; per-layer consistency and the effective-layer verdict land on a reproducible command or diff, never "eyeball byte-by-byte" (measured: three byte-identical copies / two identical layers / empty binding — all three shapes must be self-provable).
- **Three evidence forms for rules (mutex / pick-one)** — all legitimate, all must be checked:
  1. Heading-entry clustering (legend/description entries spell out mark semantics)
  2. In-name annotations (entry names contain "(pick one)"/"(pick1)"/"mutex" etc. — measured: 47 hits in one preset; missing this = missing whole clusters)
  3. Slot-table cross-check (same-slot multi-writers × the two forms above → rule candidate set)
  Register `block_rules` from the intersection of the evidence lines; a single evidence line grades as "presumed", flagged for user confirmation.
- **Legend retrieval, three forms**: the legend may live in a must-read/description entry body (sensitive entries can't be read whole — extract by line via a probe), may be embedded in entry names (emoji+category), or may not exist at all (rely on in-name annotations + slot-table inference). No description entries ≠ no rules.
- **Emoji anchors are unstable**: entry names in community presets carry emoji with variation selectors; cross-script matching often fails. **All programmatic locating uses numeric/ASCII substrings** (e.g., "1600-2500"), never full emoji names.
- **Multiple order blocks**: prompt_order may contain blocks for several character_ids (legacy/grouping); register each block; the effective block is usually the largest / the one with dummyId 100001 (inferred — label provenance); only touch the target block.
- **Three identifier forms** (classify before auditing; the root cause of reference-scan false positives):
  1. **UUID** — custom entries; stable, usable as a reference anchor
  2. **ST built-in slot names** ('main'/'nsfw'/'jailbreak'/'scenario'/'charPersonality'/'worldInfoBefore'…; full list in the local ST source's built-in identifier table in PromptManager) — carry runtime fill semantics; even renamed/repurposed they are still ST-filled (presumed); never treat as ordinary short words
  3. **Custom short words** ('nsfw'/'SPresetSettings' etc.) — default high false positives in reference scans (see toolmap Card 9)
- **Probe output discipline**: self-written probes print only numbers/structure/aliases, **never sensitive body text** — the probe script itself may end up in git.

## 5. Reference scan (S3 prerequisite / mandatory before S4 on heavy presets)

UI scripts index entries by name/group number (the §2 coupling problem); the reverse check is the reference scan — tool contract in toolmap Card 9. Key points:

- Three consumer classes: script bodies, regex pattern/replaceString, entry bodies.
- **False-positive criterion**: a hit counts only if it can be explained as **consumption semantics for that entry**; exclude language keywords / CSS numbers / regex backreferences (`$N`) / standard slot words / the script's own identifiers. Hits on short names (pure digits / ≤2 chars) and built-in slot names are untrustworthy by default — verify item by item.
- Measured lesson: skipping this step, a rename/deletion silently breaks panel features; on script-less presets the scan output is empty (the empty case still validates the chain).

## 6. S1 close-out check

- [ ] p1 report written to disk (outside the repo if it contains sensitive names)
- [ ] The five structural risks are quantified: orphans / dead slots / double-False / mutex violations / remote dependencies
- [ ] Absolute-injection / custom sp=true / data-container entries annotated separately
- [ ] Every hookpoint has answers to the three questions (mark unknowns)
- [ ] complexity graded and backfilled into the profile
- [ ] group_notation / zone_marker_style confirmed or listed as "ask the user"
- [ ] Reference-scan baseline captured (S3/S4 both reconcile against it)
