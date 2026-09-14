# pitfalls.md — Lesson Library (every entry actually happened)

> **When to read**: when planning large tasks; when confused.
> **Format**: lesson → how it happened (real case) → the structural defense in this skill. **These are not theoretical risks; they are tuition already paid.**

| # | Lesson | Real case | Structural defense |
|---|---|---|---|
| L1 | **Self-serving documentation**: writing docs about the work replaces the work; needing "docs about managing docs" = documentation surplus | In one real project, nearly a third of all sessions went to maintaining consistency of a documentation corpus running to hundreds of thousands of characters | SKILL.md hard cap ≤300 lines; routing table controls pipeline weight; drift self-check question 1 |
| L2 | **Self-verification**: imagined models + sub-agent roleplay + self-written asserters, conclusions passed on as "verified" | Three consecutive "verification" sessions were all invalid, overturned by a later session | Discipline 2; L4 externalized to the user; assertion applicability boundaries mandatory |
| L3 | **State loss across sessions**: correction counters stuck at old values, SOPs treated as verified, "blocked" while the source code sat on the local disk — the next session starts on false premises | Happened 3 times, always discovered only by the next session | S6's close-out trio + 30-second self-check |
| L4 | **Focus drift**: sliding from "usable by the user" toward "process rigor / tooling completeness", pulled back by the user twice | The user said it outright, twice: "right direction, wrong focus"; "feels like a loop" | Drift self-check question 4; the routing table hard-codes the minimal correct path |
| L4b | **Control-experiment inertia**: "get baseline data first" becomes a procrastination excuse; a fix backed by measured direction should run in parallel with effect comparison | After shipping v2, defaulted to waiting for the user's 10 control turns — the user pointed out "the original preset fluctuates too" | After S3, branch immediately: fixes supported by data → S4; genuine open questions → unverified list |
| L5 | **Missing control group**: with only one data set, effects cannot be attributed (preset problem or model problem?) | Ten output-side turns yielded only the user's current configuration | The test protocol includes comparison baselines; grab a cheap control group whenever possible |
| L5b | **Acceptance assertions not calibrated first**: idealized "byte-identical" is unreachable on derived artifacts (placeholder shifts); the first run always fails and reworks | Assembly reconciliation took two iterations to pass | verification.md hard-codes the normalization step |
| L6 | **Distorting-field warning too late**: a known trap (the `enabled` artifact) not surfaced early in tool output keeps biting others | A community tool's fallback rules produced wrong results for that family | The S1 dossier lists "distorting fields" as a fixed check item |
| L6b | **Emoji anchor drift**: emoji in entry names carry variation selectors; cross-script matching fails | The same entry name written differently in two scripts; first-run assertion failure | surgery.md: all programmatic locating via numeric/ASCII substrings |
| L7 | **Over-strict safety has costs too**: a compliance tool refusing to write out mapping tables containing original text is correct, but blocking even temporary analysis channels pushes people to work around it | The classify→apply flow had to be adjusted twice | Red lines state "what is allowed", not only "what is forbidden"; workaround needs go through the formal channel — never open private ones |
| L8 | **The source code is right there**: the most important question stays "blocked, need leads" while the answer sits 15 minutes away in local source | The same improvement item sat marked "blocked" for three consecutive sessions | Any "what does ST actually do" question: first action is reading local ST source (`public/scripts/` — macros / openai / regex engine / PromptManager are proven forensics points), evidence rank equal to official docs |

## How to use

- Before planning any task >30 minutes, scan this table and ask: will this task hit one of these?
- If yes → the defense is already in SKILL.md/references; follow it. Defense insufficient → build the defense before starting.
- New lessons: register only what really happened; include "case" and "defense" columns — a lesson without a defense is a lesson not learned.
