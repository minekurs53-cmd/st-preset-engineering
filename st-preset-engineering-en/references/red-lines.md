# red-lines.md — S0: Red Lines

> **When to read**: S0; before any operation involving sensitive content or outbound files.
> **Why S0 comes first**: across all real sessions run under this discipline, the only damage that could ever be irreversible was not a technical error but a crossed line (source modified, sensitive content committed). S0's entire cost is a few commands; its entire benefit is keeping "irreversible" out the door.

## Rule 1: Source is read-only + before/after verification

- The source file/directory is **never modified by a single byte**. All changes happen on the working copy.
- If a hash baseline exists (`profile.hash_baseline` or a manifest tool): run `profile.verify_command` **once before and once after every operation**, expecting `[verify] OK`; with no baseline, compute a sha256 first and record it in the profile.
- Verification failed → stop. First find out who changed it. **Do not continue any surgery.**

## Rule 2: Sensitive-content triage

First decide whether the preset contains sensitive content (NSFW guidance / jailbreak-style adversarial instructions / restricted terms). The deciding tool is a lexicon scan (`04_audit_for_git.py` or the sanitizer's `scan`), **never eyeballing** — real measurement shows eyeballing misses (one generic word misclassified 32% of content; see the explain card in toolmap).

| Strategy | When | How |
|---|---|---|
| `placeholder-pipeline` | Sensitive body text + long-term analysis/modification | Sanitizer produces a PUBLIC copy (zero lexicon hits, safe to export) + a quarantined copy; the mapping table lives only in a gitignored directory |
| `quarantine` | Sensitive but no body-level analysis needed | Working copy in a gitignored directory; edit it directly; artifacts never leave the repo |
| `none` | Zero lexicon hits | May be committed normally |

- **Artifacts containing sensitive body text never enter git history** (once in history, they are hard to remove).
- When citing sensitive entry names, use aliases (`«NAME:NNN|category»`) or placeholders (`«BLOCK:NNN»`) — **never hand-copy real names into any document that will be committed**.
- The tool enforces this (the sanitizer refuses to write a mapping table containing original text outside the sensitive directory) — being refused means you put it in the wrong place. **Fix the path; do not bypass the tool.**

## Rule 3: Outbound judgment

- For anything outbound (pasting to others / uploading / posting) use only files matching `outbound_files_pattern`; the criterion is **zero lexicon hits**, not "I don't think it's sensitive".
- Export files (importable JSON containing full body text) always live outside the repository; what gets committed is documentation, tools, and PUBLIC copies.

## Rule 4: Write-to-disk discipline

- Every tool writes files with explicit `newline="\n"` — Windows text mode once made a "byte-identical restore" differ by 4896 bytes.
- Generators write JSON with `ensure_ascii=False, indent=4`, then compute sha256 into the output notes.

## Rule 5: Regex double-False ban (error level)

- A regex with `markdownOnly=false` and `promptOnly=false` and not disabled will **permanently rewrite chat history** (confirmed on disk: markers stripped from message bodies and swipes, irreversibly).
- If the S1 dossier finds such a regex → list it as an **error** in the diagnosis report. Fix = split into a display-side (markdownOnly) + prompt-side (promptOnly) pair, or disable with user consent. **Never leave it in place.**

## Rule 6: Connection-bound fields don't take effect from presets by default (source-code lesson)

Connection-bound fields such as `custom_prompt_post_processing` **have no effect when written into a preset by default** — ST source skips `isConnection` fields on preset switch, **but conditionally**: the code actually reads `if (isConnection && !bind_preset_to_connection) continue` — when the user enables "bind preset to connection" in the UI, they do apply. So: when a specific value is needed, put it in the handover notes for the user to check both the bind checkbox and the field value in the UI — **never write it into the preset JSON pretending it took effect** (fields like media_inlining with `isConnection=false` do travel with the preset).

## Rule 7: Remote dependencies disabled by default

Scripts containing `injectScript` / external JS links (unverifiable domains): register as a risk in S1; default `enabled=false` and note the recovery method in the handover. Never decide on the user's behalf to execute remote code.

## Rule 8: Remote-resource risk grading (used in S3)

Grade every remote reference in scripts/regexes (criterion: static asset → executable code):

| Grade | Form | Example | Handling |
|---|---|---|---|
| I | Static asset | image CDN, CSS link | register |
| W | Read-only API | fetching third-party JSON/API | register domain & purpose, note "content changes with the remote" |
| **E** | **Executable code injection** | `injectScript(…, 'https://third-party/inject.js')` without version lock / checksum | **grade as error**: supply-chain risk. Read-only tasks **do not fetch** (requests need user consent) — register a "content unauditable" declaration + handling advice (quarantine-review or remove before delivery) |

Outbound discipline: never send requests to third parties by default; get user consent first when a fetch-audit is genuinely needed.

## S0 close-out check (30 seconds)

- [ ] Read-only verification passed (before and after)
- [ ] Sensitive strategy chosen and artifact locations match the triage table
- [ ] Files pending commit have zero lexicon hits
- [ ] No generated artifacts were written into the source directory
