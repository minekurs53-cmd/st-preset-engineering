# st-preset-engineering

English | [简体中文](README.md)

**SillyTavern Preset Engineering Skill** — an Agent Skill that performs **autopsy → diagnosis → surgery → verification → handover** on any SillyTavern (ST) preset JSON.

> **Design goal in one sentence**: let an AI agent that carries content-safety review safely reverse-engineer, diagnose, maintain, and refactor community presets containing sensitive content — keeping sensitive body text outside the agent's "context and artifacts", minimizing platform blocking and output truncation.

This is not yet another preset-writing tutorial. Every discipline is distilled from real reverse-engineering projects (full-cycle practice on a heavy preset: hundreds of entries / 1.5MB / multiple scripts / multiple regexes), including every pitfall along the way — `st-preset-engineering-en/references/pitfalls.md` attaches a real case and a structural defense to every lesson.

## Features

### The seven-step skeleton (enter as needed; not everything runs every time)

```
S0 Red lines → S1 Autopsy → S2 Ground truth → S3 Diagnosis → S4 Surgery → S5 Verification → S6 Handover
```

| Step | What it does | Key discipline |
|---|---|---|
| S0 | Red lines | Source read-only + before/after hash verification; sensitive-artifact triage; remote dependencies disabled by default |
| S1 | Autopsy | Structure dossier (orphans/dead slots/groups/regexes/scripts) + extension hookpoint inventory + complexity grading |
| S2 | Ground truth | Choosing among three truth levels: mock capture / offline assembly / the user's real dump |
| S3 | Diagnosis | Structural/behavioral/effect defect classes; every conclusion carries an evidence label (measured / structural fact / presumed / unverified) |
| S4 | Surgery | Generator's three assertions (uniqueness / enumeration / equality) — **hand-editing large JSON is forbidden** |
| S5 | Verification | Three verification gates from cheap to expensive; stop at first failure |
| S6 | Handover | Change ledger + user manual + honestly labeled unverified list |

### Request-type routing + complexity adaptation

"What's wrong with this preset?" runs only S0→S1→S3 (the report is the deliverable); "toggle a switch" runs only S0→S4(minimal)→S5(L1) — no sledgehammers for minute-scale tasks; refactoring runs the full pipeline. Entry / script / regex counts decide the light / medium / heavy tier; heavy presets force an extra "script-UI reference scan on entry names".

### Instance parameters separated from the skeleton: changing preset family = changing the profile

Every family-specific fact (paths / red-line commands / group-notation semantics / model branch pairing / slot contracts / complexity) lives in `preset-profile.json`; the skeleton files stay unchanged across presets. `examples/preset-profile.sample.json` is a ready-to-adapt sanitized template.

### File list

```
st-preset-engineering-en/
├── SKILL.md              # Skeleton + routing + disciplines + safety boundary statement
└── references/           # Loaded on demand; the details live here
    ├── red-lines.md      # S0: source read-only / sensitive triage / double-False ban / connection fields
    ├── autopsy.md        # S1: dossier reading / hookpoint inventory / complexity grading
    ├── truth-sources.md  # S2: three-level truth decision table / chain-of-thought channel warning
    ├── diagnosis.md      # S3: three defect classes / four evidence labels / report template
    ├── surgery.md        # S4: generator's three assertions / five change classes / high-risk zones
    ├── verification.md   # S5: three gates / the bar for "verification passed"
    ├── handover.md       # S6: deliverables / state sync / test protocol template
    ├── pitfalls.md       # Lesson library: every entry actually happened (case + defense)
    ├── best-practices.md # 20 field-tested practices before writing bodies / editing templates
    ├── profile-spec.md   # Full preset-profile.json schema + minimal template
    └── toolmap.md        # Tool call cards: command / expected output / failure criteria / generality tags
```

## Safety highlights: letting a "reviewed" agent handle sensitive presets safely

### The scenario

SillyTavern community presets commonly carry adult-content guidance and adversarial instruction engineering. When an agent with content-safety review **directly reads** such JSON: sensitive text enters its context → hits the platform's safety policy → **refusals, truncated outputs, interrupted tasks**; and that text keeps spreading through diagnosis reports and git commits. The common workaround is "don't let agents touch it", leaving maintainers hand-editing 1.5MB JSON — exactly the path this skill's generator-with-three-assertions proved always wrong.

### The idea: keep the "reviewable content surface" clean

Reviewers can only match text that is **present**. This skill keeps sensitive body text off the stage: the analysis surface works on a sanitized PUBLIC copy (zero lexicon hits); originals exist only inside isolated directories and tool pipelines; actual byte-level modification is done by generator scripts at the **file layer** — originals never pass through the model's context.

### Five layers of measures

1. **Placeholder pipeline (core)**. A lexicon-driven sanitizer replaces sensitive text with `«BLOCK:NNN»` / `«NAME:NNN|category»` placeholders while **fully preserving the machine-readable skeleton** (identifier / prompt_order / macros & variable names / XML tags / regex flags), and can be **restored byte-identically** (`verify-structure` proves "changes confined exactly to mapped slots"; `restore` reverses). The agent works end-to-end on a PUBLIC copy with zero lexicon hits.

2. **Lexicon criteria, not eyeballing**. What counts as "sensitive" is decided by the lexicon's hit count, not by "I think it's fine". A companion `explain` subcommand guards against over-sanitization — one generic word measurably misclassified 32% of content; an over-strict boundary forces workarounds (pitfalls.md lesson L7).

3. **Sensitive-artifact triage**. Isolation strength scales with analysis depth: `placeholder-pipeline` (long-term analysis: PUBLIC copy may leave the machine + mapping table locked in a gitignored directory) / `quarantine` (copy in a gitignored directory; artifacts never leave the repo) / `none` (no sensitive content). **Artifacts containing sensitive text never enter git history**; the tooling locks the mapping table's directory by force — being refused means it's in the wrong place; fix the path, don't bypass the tool.

4. **Red lines first + permission boundary statement**. S0 is step one — rules before any file operation: source preset read-only, hash verification before and after changes, outbound files must have zero lexicon hits, remote links inside preset scripts **disabled by default** (never decide on the user's behalf to run remote code), mock endpoint listening on 127.0.0.1 only. SKILL.md opens with the Safety Boundary Statement so the behavior envelope is visible before use.

5. **Lean context**. SKILL.md is ~100 lines (cap 500); the details live in 11 references loaded on demand. No redundant text in context = smaller review exposure and less truncation pressure.

### Why this reduces truncation (mechanism, not promise)

- **Nothing to match**: the main analysis surface (the PUBLIC copy) has zero lexicon hits — the reviewer has no sensitive text to match → no reason to trigger blocking.
- **Minimized exposure**: sensitive text exists only in isolated directories / tool pipelines / outside the repo — never in context summaries, never in reports, never in git history. Every layer has a programmatic "zero hits" criterion that can be self-verified before a task.
- **Self-verifying repository**: all text files in this repository pass lexicon audit with zero artifact-surface hits; no keys, no credentials, no real paths.

### Honest statement (boundaries)

This skill **reduces** the exposure of sensitive content in context and artifacts and provides re-runnable self-verification commands, but **does not promise "never truncated"** — platform review policy is outside this skill's control; in `quarantine` mode the agent still reads a local quarantined copy (artifacts never leave the machine). What can be verified: the release is clean, the working surface is clean, the artifact surface is clean — each checkable by command.

## Installation

```bash
git clone https://github.com/minekurs53-cmd/st-preset-engineering.git
```

- **Directory-based** (Claude Code and other platforms honoring the skills directory convention): put the whole `st-preset-engineering-en/` (English) or `st-preset-engineering/` (Chinese) directory into `~/.agents/skills/` (user-level) or `<project>/.agents/skills/` (project-level).
- **Bundle-based** (claude.ai → Settings → Capabilities → Skills → upload): upload the zip attached to the Release, or package the skill directory yourself.
- **Verify the install**: the agent should read the profile first (or build a minimal one), then enter S0 — if it starts editing JSON without red lines, it isn't running this skill.

## Quick start

1. **Prepare instance parameters**: create `preset-profile.json` in the working directory or repo root. For a cold start, copy the "minimal profile" template from `references/profile-spec.md`; S0/S1 auto-fill most of it. `examples/preset-profile.sample.json` is a filled-in sanitized sample.
2. **Trigger with a sentence**, e.g.:
   - "Take a look at this preset — what's wrong with it?"
   - "Change the length gear of this preset"
   - "Add a new model branch to this preset"
   - "Refactor this preset and clear out the zombie entries"

## Requirements & dependencies

| Item | Requirement |
|---|---|
| Agent platform | Any platform honoring the Agent Skills convention (SKILL.md frontmatter + directory loading) |
| Skill body | Pure Markdown; no bundled scripts; no external network access |
| Companion tool band | Structure dossier / sanitizer pipeline / mock endpoint / offline assembly / audit — Python 3.8+ scripts, **not distributed with this repository**; paths registered via `profile.tool_dir` |
| Without the tool band | S0 hash verification works with any sha256 tool; the dossier and assembly reconciliation need equivalent tooling of your own. Step 0 of the skill mandates: unreadable references must be explicitly reported — **never silently skipped** |

> **Why the tool band isn't bundled**: the sanitizer depends on a sensitive-term lexicon; distribution is undecided (see FAQ). Decoupling the skill layer from the tool layer is deliberate — changing preset family changes the profile, not the skeleton.

## Compatibility

- For agent platforms supporting the Agent Skills convention (SKILL.md frontmatter + `.agents/skills` directory or claude.ai upload).
- Works on any SillyTavern preset JSON; dedicated disciplines for complex presets with script panels, entry-management UIs, and lorebook integration (heavy tier).
- The English and Chinese skills are structurally identical; install either one — no need for both.

## FAQ

**Q: Where do I get the tool band?**
A: Not distributed with the repository for now (the sanitizer depends on a sensitive-term lexicon; distribution undecided). Without it you can still run S0/S3/S6 (diagnosis and handover need no tools); S1/S2/S4/S5's dossier and assembly reconciliation need equivalent tooling of your own — the skill explicitly reports any failed reference instead of skipping it silently.

**Q: My preset has no sensitive content?**
A: Set `sensitive_strategy: "none"` in the profile; the pipeline runs unchanged and skips sanitization. All other disciplines (source read-only / generator's three assertions / three-gate verification) stay exactly the same.

**Q: Does this skill teach agents to write adversarial content?**
A: No. It handles the **engineering structure and maintenance discipline** of presets; adversarial gear semantics are not registered into profiles by default (register less, expose less), the sanitizer keeps sensitive text off the analysis surface, and the repository itself passes lexicon audit.

**Q: Differences between the English and Chinese versions?**
A: Content is one-to-one identical; only the language differs. The Chinese version's trigger words cover Chinese colloquial requests ("酒馆预设", "翻个开关") more naturally; the English version fits English workflows. Pick whichever matches your usage.

**Q: Can it work with different preset families?**
A: That is exactly the purpose of `preset-profile.json` — changing family = changing profile, skeleton untouched. Under unfamiliar group-notation systems the skill downgrades group interpretation to advisory and asks the user face to face — **never guessing** (measured: foreign notation systems produce group false positives; recorded in pitfalls).

## Contributing

Issues and PRs welcome. Before submitting: no real paths / credentials / preset body text; new disciplines need a real case and a structural defense (see the registration format in pitfalls.md).

## License

[MIT](LICENSE)

## Acknowledgments

- **The "Kedai" (可待) preset and its original author**: every discipline in this skill is distilled from reverse-engineering and maintenance practice on this Chinese SillyTavern community preset — its prompt engineering (group notation system, chain-of-thought channels, model branch pairing, regex post-processing) was both the research object and the source of all the experience.
- **The Kedai preset engineering project**: multi-session, multi-version structure dossiers and a full-coverage engineering practice. The seven-step skeleton, the lesson library, and the best practices all precipitated from it.
- [Anthropic Agent Skills specification](https://github.com/anthropics/skills) and the runoob Skills tutorial: references for SKILL.md structure, progressive disclosure, and release security practices.
