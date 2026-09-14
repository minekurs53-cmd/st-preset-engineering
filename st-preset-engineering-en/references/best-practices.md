# best-practices.md — Practices checklist before writing bodies / editing templates

> **When to read**: before writing new entry bodies, editing output templates, or designing switch structures.
> **Sources**: mature-preset reverse engineering + community tutorials (well-known preset wikis, official docs, negative-instruction research) + measured calibration. Every item has survived real runs.

## Structure and ordering

1. **U-shaped attention**: identity/frame at the head; format requirements / output templates / tail input pressed to the end; reference material centered; chat history near the bottom — head and tail carry the most weight (positional-bias research consensus + community measurement). **Confirm every new constraint lands in the tail zone through its emission point**.
2. **One effective layer**: switch state is read only from `prompt_order[].order[].enabled`; any assumption based on the entry's own `enabled` field will be wrong (stale snapshot).
3. **Three iron laws of group numbers**: never renumber old groups; new groups take "global max + 1"; never reuse a number across phases — so long-time users' "keep my current checks" keep pointing at the same features. **Supplement (easy to miss)**: when adding a new member to an existing mutex family (e.g. a branch for a new model), **join the existing group**; a separate new group degrades "double-ON misconfiguration" from L1-detectable to silent.
4. **Pairing via shared slots**: the branch↔template binding lives in variable-slot read/write relations registered in the profile, **never name-based pairing**; at dossier time verify "the slot the template reads == the slot this route writes".
5. **Zero orphans**: every added entry goes into order; every removed entry takes its order row with it; never "keep it in prompts[] as a spare".

## Instruction writing

6. **Positive-first, bans as backstop**: showing the pattern to follow beats piling up "don't do Y" (pink-elephant effect); every ban comes **paired with a positive replacement** ("ban X → rewrite as Y").
7. **Make bans self-checkable**: write "X is forbidden" as a check the model can execute ("after each paragraph, verify this paragraph contains zero X") plus a lexicon an external program can assert against.
8. **Anchor length by module**: pure numeric ranges drift ~12% in the same direction in measurement; "≈N characters per module × M modules + pre-writing allocation + post-writing self-check" measured effective. **Write the per-gear allocation numbers directly inside the gear entry** — don't make the model do division.
9. **Closed enumerations**: have the model "choose from these categories" rather than "vary the kinds" — open-ended wording induces invented categories (measured).
10. **Spend emphasis sparingly**: bold / ‼️ / threat-style wording only on the highest-value constraints; emphasis everywhere = emphasis nowhere.
11. **Restraint with examples**: inline examples must be few and diverse + an explicit "never copy this example's phrasing" — otherwise style degenerates into a fixed cadence.

## Output templates / chain-of-thought

12. **One tag, one literal**: the whole preset uses a single chain-of-thought tag name (variants and hedged spellings count as contradictions); two tag names in one request = chain-of-thought output in the wrong place.
13. **Tags are contracts**: template tags ↔ beautify/removal regexes ↔ compression regexes are literally coupled — scan consumers repo-wide before changing any side (measured: changing the summary tag wiped whole floors of messages).
14. **Choose the chain-of-thought carrier per model**: reasoning-channel models (API returns reasoning_content) must not be asked for body tags; only tag channels get body templates + literal enforcement. Mixing = double chain-of-thought.
15. **Pseudo-prefill needs continuation rules**: when a request ends with an assistant closing tag, the template must explicitly say "continue the body directly from the closing tag; do not reopen chain-of-thought" — otherwise measured behavior reopens it and unbalances tags.
16. **Role is behavior**: same-family entries share one role (user×N / system×1 / assistant×1 is a real distribution); new entries copy the family role, never go by feel.

## Parameters and metadata

17. **Connection fields stay out of presets**: `custom_prompt_post_processing` and other isConnection fields have no effect when written into JSON — put them in handover notes for UI verification. **If a deep SOP conflicts with this list, this list + source forensics win**, and the conflict gets registered in the errata channel.
18. **Machine-readable metadata**: add top-level name/version/changelog — a version number written only inside a "must-read" body vanishes when the user toggles it off.
19. **Resistance packs per model**: register the author-verified resistance combinations per model (resistance_pack); the default state converges to "a legal configuration for at least one mainline model", never a jack-of-all-trades default.
20. **Model self-reported counters are unreliable**: counters maintained by the model (refresh every N turns / trigger every M times) drift — bind triggers to a signal the model already maintains stably (e.g. a counter marker it writes every turn), or count floors objectively in a script.
