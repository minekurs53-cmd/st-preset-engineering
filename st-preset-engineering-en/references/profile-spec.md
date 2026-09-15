# profile-spec.md — preset-profile.json field specification

> **When to read**: Step 0, building/validating the profile.
> **Principle**: the profile is the skill's instance-parameter injection point. The skeleton (SKILL.md) is invariant across presets; **changing preset family = changing the profile**. The profile holds only "facts verifiable on this machine"; every path/command must actually run.

## Lookup order and failure handling

1. Current working directory → 2. repository root → 3. `~/.preset-profiles/<preset-name>.json`
- None found: build the skeleton fields from the "minimal profile" below; S0/S1 fill in most of the structure section. **Timing allowance**: during the skeleton phase `verify_command`/`tool_dir` may hold placeholders (planned paths) — tools don't exist yet at first build, which is normal; backfill immediately after each tool is built and passes its self-test, from which point "every command runs" takes effect
- A profile-referenced file/command fails to run: **report explicitly which reference failed** — never silently skip. The way out is not a substitute with different behavior: implement the equivalent tool locally per `toolmap.md`'s tool-building method and the matching contract card, pass its self-test, register it in the profile, then continue
- `tool_dir` being null is not a failure: proceed with the minimal flow first, and build a card's tool only when you reach a step that needs it (method discipline 5: pass the self-test before touching real data)

**Where to put a new profile** (choose by session nature; declare in the handover):
- Long-term maintenance of this preset within a project → repository root `preset-profile.json` (committed, versioned with the repo)
- Cross-project / one-off analysis → `~/.preset-profiles/<preset-name>.json` (never in any repo)
- Session forbidden from writing to the repo → output directory, noting in the handover "the formal location should be X; per constraints this went to a temp directory"
- **Multi-preset independent sandbox** (testing several presets in one session without polluting each other's cold start) → one working folder per preset, profile inside each folder — read the first three options as "one profile follows one preset"; a sandbox just makes the folder the isolation unit

## Full schema (annotated)

```jsonc
{
  "preset": {
    "name": "<preset-name>-rebuilt",                 // display name
    "source_dir_readonly": "<source-preset-dir>/",   // read-only source (may be null: no-source scenarios)
    "working_copy": "<working-copy-path>.json",      // current working copy (sensitive body → outside the repo)
    "verify_command": "python <tool_dir>/01_source_manifest.py",  // read-only verification; expect [verify] OK
    "tool_dir": "<tool_dir>",                        // tool directory (relative to repo root or absolute); null = no local tools yet — build per toolmap contracts, then register
    "sensitive_strategy": "placeholder-pipeline",    // placeholder-pipeline | quarantine | none
    "outbound_files_pattern": "*.public.*",          // outbound-safe pattern (zero lexicon hits)
    "hash_baseline": {"file": "source-filename", "sha256_16": "e91b1ef8ac231f30"} // read-only baseline for smoke/foreign files
  },
  "structure": {
    "switch_layer": "prompt_order",                  // universal fact; don't change
    "order_blocks": [100001],                        // character_id list (some presets have multiple blocks)
    "group_notation": {"❗": "exactly one per group", "❔": "zero or one", "❕": "any number", "🆎": "all on / all off", "🔒": "always-on anchor"},
    // ↑ must be refilled for every new preset! Unfamiliar notation → stop and ask the user; never guess (the sample notation is one author's private system)
    "zone_marker_style": "〈…〉 heading entries",     // how phase boundaries are recognized; presets without markers write "none"
    // ↑ zone_marker_style is display-only; **what actually drives the tools are the optional regex fields below (data-driven, style-agnostic)**.
    //   Six real-world styles measured: 〈…〉 heading entries / paired boundary entries (——X begin—— … ——X end——) / same-name boundary pairs (toggle type) / emoji+category-name prefixes / HTML-tag paired entries (`<background>…</background>`) / "----X" hyphen headings.
    //   Built-in style recognition is untrustworthy (measured: the first version recognized one style and missed all others) — profile regexes take precedence over tool built-ins.
    "zone_title_re": "^----[^\\s-]+",                // optional: heading-entry recognition regex (style-agnostic)
    "zone_pair_start_re": null,                      // optional: paired boundary — begin-entry regex (null if none)
    "zone_pair_end_re": null,                        // optional: paired boundary — end-entry regex
    //   ↑ Same-name boundary pairs (toggle type): when start/end are filled with the **same** regex, the tool treats it as "second occurrence of the same name = close" (measured: boundary entries share the name at open and close — ——X—— appearing twice; first opens, second closes; differently-named boundaries still use sequential open semantics)
    "zone_category_re": null,                        // optional: emoji+category-name-prefix style — per-entry category-name extraction regex (style-agnostic)
    // ↑ the 3rd style (emoji+category-name prefixes) has no title/pair field to use; measured as needing this extra field (`^[^︱丨]{1,6}[︱丨]\s*([^-\s丨︱]{1,10})`, capture group 1 = category name).
    //   ⚠️ When a regex has capture groups, **capture group 1 = the zone name** (e.g. `━━━━ X ━━━━` separator entries only want a pure category-name label, taken via the capture group) — same for title_re.
    "block_rules": [                                 // optional: data-driven group rules (machine-checked by the tool; no semantic guessing)
      // {"kind": "count_in", "zone": "<phase>", "field": "style", "max": 2},
      // {"kind": "mutex", "members": ["«NAME:1|system»", "«NAME:2|system»", "«NAME:3|system»"]}
      //   mutex members = the intersection of three evidence lines: same-slot multi-writers × in-name annotations ("(pick one)" etc.) × mark clustering
    ],
    "branch_zone": "〈filler features〉",             // phase name containing model branches (null for branchless presets)
    "model_branches": [ /* see below */ ],
    "live_slots": ["<family-prefix>_preprocessing", "…"],  // live slot list (from the S1 dossier slot table)
    "extension_hookpoints": [                        // optional: extension hookpoint inventory (autopsy §2's three-question conclusions land here)
      // [{"block": "regex_scripts", "kind": "ST native regex", "status": "effective",
      //   "note": "who reads / who writes / what changing it affects"}]
    ],
    "tag_contracts": [ /* see below */ ],
    "default_switch_state": "GLM route B: ❗1 GLM tail + ❗2📅 non-prefill output template ON; anti-defect ON / anti-flat OFF"
  },
  "adaptation": {
    "user_models": ["glm-5.3", "deepseek"],          // models the user actually uses (real usage beats factory defaults)
    "matrix": "<methodology-docs>/model-capability-matrix.json",
    "sop_adapt": "<methodology-docs>/new-model-adaptation-SOP.md",
    "sop_feature": "<methodology-docs>/new-feature-SOP.md"
  },
  "complexity": {
    "entries": 186, "scripts": 4, "regexes": 15,
    "extension_blocks": ["regex_scripts", "tavern_helper", "SPreset"],
    "level": "heavy",                                // light | medium | heavy; criteria in autopsy.md
    "note": null                                     // optional: annotate when criteria and substance disagree (e.g. "heavy by the letter; no scripts no UI, substance ≈ medium")
  }
}
```

## model_branches entry format

```jsonc
{
  "model": "GLM",
  "tail": "❗1 GLM tail",                    // ❗1 tail-input entry name
  "template": "❗2📅 non-prefill output template",   // ❗2 output-template entry name (may be shared)
  "slot_written": "<family-prefix>_cot_nonprefill_locator",  // the locator slot this route's ❗1 setvar writes
  "slot_read_by_template": null,             // the locator slot the template getvar reads; **must equal slot_written or be null** —
                                             //  different slots = broken pairing (measured: breaks silently, only symptom is degraded output); mandatory dossier check
  "extra_params": "chat completion source must be the matching provider"  // model-side precondition (omit if none)
}
```

## tag_contracts entry format (mandatory check before S4 body edits)

```jsonc
[
  {"literal": "<thinking>", "consumers": ["strip-redundant-cot prompt (regex, thinking only)"]},
  {"literal": "</(?:think|thinking)>", "consumers": ["cot-beautify (regex, both tags)"]},
  {"literal": "<details><summary>(live|mini)summary", "consumers": ["floors 10+ keep-mini-summary (regex; S9: change one side = wipe the whole floor)"]}
]
```
How to generate: scan the preset for "literals inside regex find/replaceString" ∩ "format markers inside entry bodies"; the intersection is the contract. UI scripts count as consumers too (references by entry name / group number).

## Minimal profile (cold-start template)

```jsonc
{
  "preset": {"name": "<filename>", "source_dir_readonly": null, "working_copy": "<user-given path>",
             "verify_command": null, "tool_dir": null, "sensitive_strategy": "none",
             "outbound_files_pattern": null, "hash_baseline": {"file": "<path>", "sha256_16": "<compute in S0>"}},
  "structure": {"switch_layer": "prompt_order", "order_blocks": [], "group_notation": {},
                "zone_marker_style": "unknown", "zone_title_re": null, "zone_pair_start_re": null,
                "zone_pair_end_re": null, "zone_category_re": null, "block_rules": [],
                "extension_hookpoints": [], "branch_zone": null, "model_branches": [],
                "live_slots": [], "tag_contracts": [], "default_switch_state": "unknown"},
  "adaptation": {"user_models": [], "matrix": null, "sop_adapt": null, "sop_feature": null},
  "complexity": {"entries": 0, "scripts": 0, "regexes": 0, "extension_blocks": [], "level": "unknown"}
}
```
S0 fills hash_baseline; after S1's `p1 --preset`, most of structure/complexity is filled; **when group_notation or zone_marker_style can't be determined, ask the user face to face**; with the user absent (async task) → register "presumed + pending confirmation" and continue the rest — **never guess semantics and press on**.

## Acceptance rules

- An `unknown` field in the profile ≠ failure, but **any step depending on it must first resolve it** (run a tool or ask the user)
- Sensitive strategy `quarantine`: working copy in a gitignored directory; `none`: no sensitive content, may be committed directly
- Adversarial/jailbreak-style gear semantics are **not registered by default** (entry names are self-describing; register less, expose less) — unless the user explicitly asks
