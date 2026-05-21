# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code / Codex / Gemini CLI skill plugin. When invoked as `/doxify`, it adds Doxygen-style comments to `.h`/`.c` (or `.hpp`/`.cpp`) files in any C/C++ codebase. No build system, no runtime dependencies — purely skill + manifest files.

## File Structure

```
doxify.md                          # Single source of truth — all skill logic lives here
skills/doxify/SKILL.md             # Claude Code entry point — @-includes doxify.md
plugins/doxify/                    # Codex plugin subtree
  .codex-plugin/plugin.json        # Codex manifest
  skills/doxify/SKILL.md           # Codex entry point — @-includes doxify.md
.claude-plugin/plugin.json         # Claude Code plugin manifest
.claude-plugin/marketplace.json    # Claude Code marketplace listing
gemini-extension.json              # Gemini CLI extension manifest
GEMINI.md                          # Gemini CLI entry point — @-includes SKILL.md
AGENTS.md                          # Codex/OpenAI agents context
.codex/hooks.json                  # Codex session hook
.agents/plugins/marketplace.json   # Agents marketplace listing
```

**Single source of truth:** `doxify.md`. All platform-specific SKILL.md files `@`-include it — edit only `doxify.md`.

## Skill Architecture

`doxify.md` is a self-contained skill that orchestrates a **multi-agent pipeline**:

1. **Controller** (main Claude session) — reads the skill, selects target files, dispatches agents, runs all git operations
2. **Builder agent** — edits `.h`/`.c` files to insert Doxygen blocks; never runs git
3. **Spec reviewer agent** — checks structural compliance against the comment format rules
4. **Quality reviewer agent** — checks semantic accuracy (correct `@param` directions, correct enum constants in `@return`)

Fix loops run until both reviewers pass, then the controller commits.

## Editing This Skill

The entire skill lives in `doxify.md`. The sections that matter most:

- **Scope** — which files to touch and which to never touch
- **Doxygen Comment Format** — canonical block templates; tag rules are exact
- **Placement Rules** — where blocks go relative to declarations vs. definitions
- **Return Value Accuracy** — read actual return constants from the codebase; never substitute similar-looking ones
- **Agent Rules** — embedded prompts prepended when dispatching builder/reviewer agents via the `Agent` tool

When updating agent rules, keep the `STATUS LINE` / severity markers / output receipt format intact — the controller parses these to detect pass/fail.

## Commit Convention

```
docs: add Doxygen to foo.h and foo.c
docs: add Doxygen to foo.h
```
