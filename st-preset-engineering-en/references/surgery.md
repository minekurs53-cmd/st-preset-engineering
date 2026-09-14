# surgery.md — S4: Surgery (generator mode)

> **When to read**: S4; **mandatory before touching JSON**.
> **Why**: hand-editing a 1MB JSON is always wrong and unauditable. The generator + three assertions make "what changed and what didn't" the machine's call — the same discipline covered an 186-entry mega-surgery and a one-entry tweak in real practice.

## The generator's three assertions (skeleton template)

```python
import json, io, copy, hashlib
doc = copy.deepcopy(json.load(io.open(SRC, encoding="utf-8")))
changes = []  # expected changes, enumerated

def edit_once(idx_or_name, old, new, label):
    """Assertion 1 uniqueness: the target string occurs exactly once in that entry, else abort"""
    c = doc["prompts"][idx]["content"]
    assert c.count(old) == 1, f"{label} matched {c.count(old)} times (expected 1), aborting"
    doc["prompts"][idx]["content"] = c.replace(old, new)
    changes.append(label)

# …… all edits ……

# Assertion 2 enumeration: the changed-entry set == the expected set
diff = [i for i,(a,b) in enumerate(zip(src["prompts"], doc["prompts"]))
        if json.dumps(a,sort_keys=True)!=json.dumps(b,sort_keys=True)]
assert diff == sorted(expected_idx), f"changed entries mismatch: {diff}"

# Assertion 3 equality: byte-identical except the enumerated fields (top level / extension layers likewise)
for k in doc:
    if k not in ALLOWED_CHANGED_KEYS:
        assert json.dumps(src[k],sort_keys=True)==json.dumps(doc[k],sort_keys=True), f"{k} unexpectedly changed"

json.dump(doc, open(OUT,"w",encoding="utf-8",newline="\n"), ensure_ascii=False, indent=4)
```

## Calibrate assertions first (lesson)

- "Byte-identical" style assertions **must be calibrated on real differences first**: noise from derived artifacts (placeholder renumbering, reordering) must be normalized away, or acceptance fails forever or passes vacuously.
- Noise handling for assembly reconciliation: normalize `«BLOCK:NNN»`/`«NAME:…»` to `«BLOCK:X»` before doing per-message diffs.
- Emoji anchors always fail: **locate entries with numeric/ASCII substrings** ("1600-2500"), never full emoji names.

## Operation notes for the five change classes

| Class | Notes |
|---|---|
| Toggle switches | **Paired operation** (❗1+❗2 together); enumerate **all** members that flip (including flipping a default-ON to OFF — a missed enumeration = double-template accident); after flipping, exactly one ON per group. **For gear-style switches, add one step**: scan the repo for other ON entries hard-coding the old gear value (including Chinese numerals, e.g. hard minimums like "body must reach 1200 characters") — such semantic conflicts are invisible to L1 (all group rules green); only a body scan or L3 catches them |
| Edit entry body | Run the **contract-counterpart scan** first (consumers of tag/format literals in regexes and scripts); one edit = one `edit_once`; changing a tag = sync every consumer |
| Add entry | **Copy key set and role from the nearest same-family entry** (a real same-family entry is the key-set template; don't copy a static template — it may lack attach_*/injection_order keys); the new entry **must go into order** (`{"identifier":…, "enabled":false}`), inserted at the end of its family zone; new entries default OFF |
| Delete entry | Delete from prompts[] **and** order together (otherwise ghost references); scan scripts/regexes for references first; deleting a disabled entry carrying setvar is actually mine-clearing (zombie landmine) |
| Edit regex | Double-False ban (split display-side/prompt-side); **multi-layer copy sync** (native layer + extension binding layer); change pattern literals and entry bodies together |

## Before and after surgery

- **Before**: S0 read-only verification (before); confirm the working copy is not the source; sensitive artifacts go outside the repo.
- **After**: S0 verification (after, proving the source untouched) → S5 three-gate verification → sha256 into the handover notes.
- **Multi-round surgery**: after each S4→S5 round, **return to S2 and re-take ground truth** — the previous round changed assembly behavior; the old truth is void.

## High-risk zone checklist (check before operating)

- [ ] `{{setvar::main-slot::}}` inside disabled entries — enabling wipes the accumulated slot (zombie landmine); delete the entry or extract the setvar
- [ ] The tail of assistant-role entries = the pseudo-prefill continuation point — **never append instructions after the continuation point** (they get treated as the model's own words)
- [ ] setvar in Main-Prompt-class initialization entries — deleting a dead-slot initialization is safe (renders empty forever), but first prove "empty forever" (initial value is the empty string and nobody writes it)
- [ ] Always-on anchors (🔒) — touching one affects every route; changes must pass L3
- [ ] Entry names/group numbers referenced by UI scripts — repo-wide reference scan before renaming/deleting
