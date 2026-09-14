# autopsy.md — S1: Autopsy (structure dossier)

> **When to read**: S1; when encountering unfamiliar structures/extension blocks.
> **Deliverable**: structure dossier (p1 report) + extension hookpoint inventory + complexity grading → backfill profile.structure / complexity.
> **Core principle**: every autopsy conclusion must be reproducible (a command or a probe script) — **never eyeball-read a 1MB JSON**.

## 1. Run the structure dossier

```bash
python <tool_dir>/p1_evidence.py report --preset <preset.json>   # any preset
python <tool_dir>/p1_evidence.py report --target <registered>    # a registered source version
python <tool_dir>/p1_evidence.py drift                           # cross-version drift (when multiple versions exist)
```

How to read the report (each section = one risk class):

| Section | Content | What to look at |
|---|---|---|
| §1 Pipeline table | Per entry: switch state / phase / group / role / length / variable ops | **The enabled column is the effective state** (`prompts[].enabled` is a stale artifact — never trust it); whether the role distribution matches family semantics (assistant = pseudo-prefill carrier) |
| §2 Slot table | setvar/addvar writers vs getvar readers | **Write-only slots = dead slots** (the original had 7, all historical corpses); multiple setvar writers on one slot = a timing landmine (enabling a disabled entry can wipe the main slot — zombie-entry check) |
| §3 Group table | Notation × phase → members and ON counts | **Mutually exclusive group with ON≠1 = configuration violation**; but first confirm notation semantics (see §4 known limits) |
| §4 Orphan table | In prompts[] but not in order | Orphan = the feature doesn't exist (invisible/untoggleable); list functional orphans separately from junk orphans |
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

## 4. Known limits (declare honestly; never pretend generality)

- **Group interpretation depends on notation + phase markers**: p1 clusters by "❗N-style marks + 〈phase〉 boundaries". For unfamiliar presets using a different notation system or without phase markers, the group table **downgrades to advisory** — notation semantics must be asked of the user or found in a legend inside a must-read entry body. **Never guess.**
- **Emoji anchors are unstable**: entry names in community presets carry emoji with variation selectors; cross-script matching often fails. **All programmatic locating uses numeric/ASCII substrings** (e.g., "1600-2500"), never full emoji names.
- **Multiple order blocks**: prompt_order may contain blocks for several character_ids (legacy/grouping); register each block; only touch the target block.
- **Probe output discipline**: self-written probes print only numbers/structure/aliases, **never sensitive body text** — the probe script itself may end up in git.

## 5. S1 close-out check

- [ ] p1 report written to disk (outside the repo if it contains sensitive names)
- [ ] The five structural risks are quantified: orphans / dead slots / double-False / mutex violations / remote dependencies
- [ ] Every hookpoint has answers to the three questions (mark unknowns)
- [ ] complexity graded and backfilled into the profile
- [ ] group_notation / zone_marker_style confirmed or listed as "ask the user"
