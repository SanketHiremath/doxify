---
name: doxify-generate
description: >
  Runs the Doxygen binary to generate HTML (and optionally XML) documentation
  from an existing C/C++ codebase. Use when the user says "/doxify-generate",
  "generate doxygen docs", "run doxygen", "build the docs", or "generate documentation".
---

# `doxify-generate` — Doxygen Doc Generator

Runs the `doxygen` binary against the target codebase and opens the output.
Does **not** add or modify source comments — use `/doxify` for that first.

---

## Invocation

```
/doxify-generate
/doxify-generate path/to/project
/doxify-generate path/to/Doxyfile
```

- No argument — run from the current working directory.
- Directory path — treat that directory as the project root.
- Doxyfile path — use that exact Doxyfile.

---

## Execution Steps

### 1. Check for Doxygen — Auto-Install if Missing

```bash
doxygen --version
```

If found, proceed to Step 2.

If not found, detect the OS and install automatically — **do not ask the user**:

**macOS:**
```bash
# Prefer Homebrew
if command -v brew &>/dev/null; then
  brew install doxygen
else
  # Homebrew itself is missing — install it first, then doxygen
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  brew install doxygen
fi
```

**Linux (Debian/Ubuntu):**
```bash
sudo apt-get update -qq && sudo apt-get install -y doxygen
```

**Linux (Fedora/RHEL/CentOS):**
```bash
sudo dnf install -y doxygen
```

**Linux (Arch):**
```bash
sudo pacman -S --noconfirm doxygen
```

Detect Linux distro via:
```bash
. /etc/os-release 2>/dev/null && echo "$ID"
```

**Windows:** Cannot auto-install. Stop and tell the user:
> Windows detected. Install Doxygen manually: https://www.doxygen.nl/download.html

After installing, verify with `doxygen --version`. If still not found, stop and report the install error verbatim.

### 2. Locate or Create a Doxyfile

**Case A — Doxyfile already exists** at the project root (or user-supplied path): use it as-is.

**Case B — No Doxyfile exists**: generate a default one then patch the critical settings:

```bash
cd <project_root>
doxygen -g Doxyfile
```

Then edit `Doxyfile` with these values (use the Edit tool — do not run `sed`):

| Setting | Value |
|---|---|
| `PROJECT_NAME` | Directory basename, quoted |
| `INPUT` | `.` |
| `RECURSIVE` | `YES` |
| `FILE_PATTERNS` | `*.c *.cpp *.h *.hpp` |
| `GENERATE_HTML` | `YES` |
| `HTML_OUTPUT` | `docs/html` |
| `GENERATE_LATEX` | `NO` |
| `GENERATE_XML` | `NO` |
| `EXTRACT_ALL` | `YES` |
| `EXTRACT_STATIC` | `YES` |
| `QUIET` | `YES` |
| `WARN_IF_UNDOCUMENTED` | `NO` |

### 3. Run Doxygen

```bash
cd <project_root> && doxygen Doxyfile 2>&1
```

Capture stdout + stderr. If exit code is non-zero, print the error output verbatim and stop.

### 4. Locate Output

Look for `HTML_OUTPUT` in the Doxyfile to find the output directory (default: `html` relative to `OUTPUT_DIRECTORY`, which defaults to the project root). The entry point is `index.html` inside that directory.

### 5. Report Result

Print:

```
Docs generated: <absolute_path_to>/index.html
```

Then offer to open it:

> Run `open <path>/index.html` (macOS) or `xdg-open <path>/index.html` (Linux) to view in browser.

If the user confirms, run the open command.

---

## Error Handling

| Symptom | Action |
|---|---|
| `doxygen: command not found` | Auto-install per Step 1. Stop only if install fails. |
| Non-zero exit, output has `Error:` lines | Print the error lines verbatim. Stop. |
| `HTML_OUTPUT` dir missing after run | Print last 20 lines of doxygen output. Stop. |
| Doxyfile exists but has parse errors | Print the error. Offer to regenerate with `doxygen -g`. |

---

## Constraints

- Never modify source `.h`/`.c`/`.hpp`/`.cpp` files.
- Never commit generated docs. `docs/html/` is build output.
- `sudo` is allowed only for package manager install commands (`apt`, `dnf`, `pacman`).
- If a Doxyfile already exists, prefer using it unchanged unless the user asks to reconfigure.
