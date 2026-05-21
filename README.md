# doxify

Add Doxygen-style function comments to C/C++ codebases — and generate the full HTML documentation site.

Runs a **builder agent + spec reviewer + quality reviewer** per file pair, then commits. Works on any C/C++ project — no hardcoded paths or project-specific assumptions.

## What it does

### `/doxify` — Add Doxygen comments

```
/doxify                        # document every undocumented file pair
/doxify src/foo                # document foo.h and foo.c only
/doxify src/foo src/bar        # document multiple pairs
```

- Adds `/** @brief ... @param ... @return ... */` blocks to every public prototype in `.h`/`.hpp` and every static definition in `.c`/`.cpp`
- Replaces stale auto-generated comment blocks; leaves all other comments untouched
- Never touches auto-generated or third-party files
- Commits each file pair with `docs: add Doxygen to foo.h and foo.c`

### `/doxify-generate` — Generate HTML docs

```
/doxify-generate               # run from current directory
/doxify-generate path/to/proj  # specify project root
/doxify-generate path/Doxyfile # use an existing Doxyfile
```

- Installs `doxygen` automatically if not present (macOS via Homebrew, Linux via `apt`/`dnf`/`pacman`)
- Generates or reuses a `Doxyfile` with sensible defaults
- Outputs HTML docs to `docs/html/index.html`
- Reports the output path and offers to open it in the browser

## Install

### Claude Code

Add the plugin source to `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "doxify": {
      "source": {
        "source": "git",
        "url": "https://github.com/sankethiremath/doxify.git"
      }
    }
  }
}
```

Then in Claude Code run:

```
/plugins install doxify
```

### Codex

```bash
git clone https://github.com/sankethiremath/doxify.git ~/.codex/plugins/doxify
```

Restart Codex. The skill loads automatically from `~/.codex/plugins/`.

### Gemini CLI

```bash
gemini extensions install https://github.com/sankethiremath/doxify
```

Or clone manually:

```bash
git clone https://github.com/sankethiremath/doxify.git ~/.gemini/extensions/doxify
```

## License

MIT
