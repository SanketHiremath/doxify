# Contributing to doxify — Key Guidelines

This project is a multi-platform AI skill that adds Doxygen-style comments to C/C++ codebases via a build-review agent pipeline.

## Source-of-Truth Files

**Edit only `doxify.md`.** Everything else is either a thin wrapper or a platform manifest:

| File | Role |
|---|---|
| `doxify.md` | Single source of truth — all skill logic lives here |
| `skills/doxify/SKILL.md` | Claude Code entry point — `@`-includes `doxify.md` |
| `plugins/doxify/skills/doxify/SKILL.md` | Codex entry point — `@`-includes `doxify.md` |
| `GEMINI.md` | Gemini CLI entry point — `@`-includes `skills/doxify/SKILL.md` |

Manifests (`.claude-plugin/`, `gemini-extension.json`, `plugins/doxify/.codex-plugin/`) contain metadata only — update them when bumping version or changing author info, not when changing skill behavior.

## Three Main Contribution Types

1. **Skill edits** — change Doxygen rules, tag behavior, agent prompts, or review checklists
2. **Platform support** — wire doxify into a new agent or IDE
3. **Bug fixes** — fix incorrect placement rules, wrong tag guidance, or broken agent dispatch flow

## Editing the Skill

All behavior lives in `doxify.md`. Sections and what they control:

| Section | Controls |
|---|---|
| `## Scope` | Which files get documented; which to skip |
| `## Doxygen Comment Format` | Canonical block templates and tag rules |
| `## Placement Rules` | Where blocks go (prototype vs. definition, forward decls) |
| `## Return Value Accuracy` | How to derive `@return` text from the function body |
| `## Execution Process` | Agent pipeline order (builder → spec review → quality review → commit) |
| `## Agent Rules` | Embedded prompts prepended when dispatching builder/reviewer agents |
| `## Spec Compliance Review Checklist` | What the spec reviewer checks |
| `## Quality Review Checklist` | What the quality reviewer checks |
| `## Commit Format` | Commit message convention |
| `## Final Quality Gate` | Per-file-pair completion checklist |

**Critical rules that must not be weakened:**
- Builder agents must never run git — the controller owns all git operations
- Forward declarations in `.c`/`.cpp` must never be commented — only definitions
- Existing non-Doxygen blocks (`/*****...*/`, `/*!`) must be replaced entirely, not amended
- `@return` must use the actual constants from the function body, not assumptions

## Adding Platform Support

To support a new agent or IDE:

1. Check how that platform loads skill context (rules file, SKILL.md, context file, etc.)
2. Add a thin entry-point file that `@`-includes `doxify.md` or references `skills/doxify/SKILL.md`
3. Add a manifest if the platform requires one
4. Add install instructions to `README.md`

Do not copy-paste `doxify.md` content into platform-specific files — always `@`-include or reference it so there is one place to edit.

## Code Standards

- No runtime dependencies — this is a pure skill, no build step required
- Do not add hooks that modify project files outside the user's codebase
- Keep agent role names in the embedded prompts consistent (`doxify-builder`, `doxify-reviewer`)
- Commit messages follow `docs: add Doxygen to <file>` — do not change this convention

## Testing

There is no automated test suite. Before submitting a PR:

- Run `/doxify` on a real C/C++ file pair and verify output matches the Doxygen format spec in `doxify.md`
- Confirm forward declarations were not commented
- Confirm auto-generated files were not touched
- Confirm the commit message matches the format in `## Commit Format`

Describe your test case in the PR — which files you ran it on and what the output looked like.
