# truth-sources.md — S2: Ground Truth

> **When to read**: S2; before drawing any "effect" conclusion.
> **Why it cannot be skipped**: this skill's first discipline is "real data > mock > sub-agent roleplay". "Verification" without ground truth is self-verification — one real project burned three sessions learning this.

## Ground-truth path decision table

| Question I need answered | Which path | Why |
|---|---|---|
| Did the assembled request change after this edit? Which messages changed? | **B2 offline assembly** (run before and after, per-message diff) | Cheap and repeatable; conclusions must be labeled "offline approximation" |
| What request does SillyTavern actually assemble? | **B1 mock capture** | Request-side ground truth; needs a local ST |
| What does the model actually see and output? | **User's real request dump / real chats** | The only complete ground truth; ask the user |
| Is the effect good? Did the fix work? | **L4: the user's real model testing** | Never self-assess |

## B1 mock capture (request-side ground truth)

```bash
python <tool_dir>/p2_mock_llm.py --port 9999
```
ST side: API connections → Chat Completion source → Custom (OpenAI-compatible) → `http://127.0.0.1:9999/v1` + any key + any model name → Connect → select the preset → send one message → read the dump (**into an out-of-repo temp directory**).

- Can prove: entry assembly, switch effect, locator placement, template uniqueness, macro expansion — the **assembly layer**.
- Cannot prove: model behavior, effects, **rewrites by the user's script layer** (see below).
- Discipline: temporary chat, answer "no" to regex/script hook prompts, restore settings afterwards and delete the copy — zero residue.

## B2 offline assembly (approximation)

```bash
python <tool_dir>/p5_agent_eval.py assemble --config default --pub <PUBLIC copy> --tag <name> --out-dir <out-of-repo dir>
```
- Input must be a PUBLIC copy (zero lexicon hits) — red line.
- **Label every output "offline approximation"**; it re-implements assembly semantics in a tool, and its divergence from ST ground truth (message merging, runtime additions, connection-field handling) must be kept in mind.

## User's real data (highest value)

- **Ten turns of real chat > ten thousand words of analysis**. Forms to request from the user: chat export jsonl / request-response dumps / three concrete metrics.
- ⚠️ **Script-layer shadowing**: for presets with tool scripts, the request history the user actually sends may already have been rewritten (real case: the first turns compressed into a summary, the opening message removed, pseudo-user messages injected). Therefore:
  - B1 only proves the **assembly layer**, not "what the model sees";
  - For reconciling "what the model actually saw", the user's **actual request dump** is authoritative;
  - In S3, treat "script-layer rewriting" as an independent variable — never mis-attribute it to preset behavior.

## Chain-of-thought channel warning (assertion applicability)

- Some models emit chain-of-thought via body tags (`<thinking>…</thinking>`); others via the API-level `reasoning_content` (zero body tags).
- **Literal tag assertions apply only to "tag-channel" models**; for reasoning-channel models, searching the body for tags finds nothing forever (measured: 0 hits in ten turns) — it does not mean the model isn't thinking.
- To determine the channel: check the response for `reasoning_content` / ST's `reasoning_type`.

## The standard way to hand L4 to the user

The workflow ends by delivering a **test protocol**, never a self-assessment:
1. Baseline statement (which switch state, which file, first 16 hex of the sha)
2. Play pattern (N turns of comparable scenarios)
3. **3–5 countable metrics** (e.g., body-length hit rate, module trigger rate, forbidden-mark count per turn — each with a comparison baseline)
4. Say explicitly: "Send me the results; I'll backfill the dossier/matrix."
