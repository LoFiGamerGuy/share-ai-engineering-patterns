# Shell and Terminal Tools

A reference catalog of shell and terminal tooling that meaningfully helps agentic work. Deeper than what's in [02-platform/dev-environment.md](../02-platform/dev-environment.md) and [dev-environment.md](./dev-environment.md) — this page is "given that you've decided to invest in this layer, here's what to actually adopt and why."

The picks are opinionated and current as of May 2026. The categories age more slowly than the specific tools.

## How to use this page

Read top to bottom if you're setting up a fresh environment. Skim by category if you're filling specific gaps. Don't install everything at once — each tool added is a tool to maintain. Add one, use it for a week, decide if it earned the slot.

---

## Terminal emulator

The host process that renders everything. Picking and configuring one well is a one-time investment that pays back daily.

| Tool | Platform | Why |
|------|----------|-----|
| **WezTerm** | Mac / Linux / Windows | Cross-platform, scriptable in Lua, fast, sane defaults |
| **Windows Terminal** | Windows | Proper Unicode, tabs, profiles, JSONC config — the only mature non-embarrassing option on Windows |
| **iTerm2** | macOS | Best-in-class multiplexer integration if you're staying on Mac |
| **Kitty / Ghostty** | Linux / macOS | GPU-accelerated, Nerd Font glyphs without patching |
| **mintty** | Windows (Git Bash) | OSC escape code support; useful as a secondary standalone window when WT multi-tab overhead is unwanted |

**Cross-platform note:** font names are the first parity bug you'll hit. Nerd Fonts register as abbreviated metadata names (e.g., `JetBrainsMono NFM`, not `JetBrainsMono Nerd Font Mono`). Using the wrong name silently falls back to a system font — icons break without error. Test by rendering a known glyph after install.

---

## Shell

**Bash** remains the safest default for scripting. Zsh is standard on macOS. For Windows, Git Bash provides a POSIX environment good enough for most tooling. Fish is user-friendly but non-POSIX — a trap for script portability.

For interactive use, install:
- `bash-completion` (or zsh's built-in completion)
- `zsh-autosuggestions` and `zsh-syntax-highlighting` if on Zsh
- A history searcher (see `atuin` below) — replaces Ctrl-R

One concrete caveat: `mise activate bash` emits Windows-format PATH on Git Bash that bash can't resolve. Use shims-only activation on Windows. WSL doesn't have this issue.

---

## Prompt

**Starship** — single binary, cross-shell, cross-platform, single config file. It reads `.mise.toml`, `.python-version`, git state, and cloud contexts without per-shell plugins. The correct answer regardless of OS.

Configuration tip: embed all palette variants inside one `starship.toml` and swap via a `palette=` field. Live theme switching without juggling config files.

The prompt should show: current directory, git branch, last command status, and (if relevant) Python venv / Node version / cloud context. Don't overload it — every additional element is visual noise on every command.

---

## Multiplexer

Multiple panes in one terminal session. Indispensable when an agent runs in one pane and you watch logs or run adjacent commands in another.

- **Zellij** — modern, friendlier learning curve, sane defaults. Good first pick for new setups.
- **tmux** — universal, available on almost every Linux server without installation. Decades-stable. Pick if you SSH into many remote hosts.

Configure split-pane layouts (a "quad" 2×2, a "vertical triple", etc.) as one-keystroke macros. Setup time: 30 minutes; payoff: every day for years.

**Caveat:** theme palette swaps via OSC escape codes do not automatically propagate to pre-existing panes in either multiplexer. Pre-existing panes cache their palette at spawn. The fix is a `themepush` command (or whatever you name it) that re-emits OSC for the current persisted theme on demand.

---

## Modern Unix tool replacements

The classic Unix utilities have modern replacements that are faster, prettier, and produce output that agents (and humans) parse more reliably. Worth swapping the defaults:

| Replaces | Tool | Why this one |
|----------|------|--------------|
| `ls` | **eza** | Colors, icons, git status integration, tree view |
| `cat` | **bat** | Syntax highlighting, line numbers, paging, git diff markers |
| `grep` | **ripgrep (`rg`)** | Respects `.gitignore`; columnar `file:line:match` output that LLMs parse reliably |
| `find` | **fd** | Sane defaults, `.gitignore`-aware, much faster |
| `diff` (git) | **delta** | Side-by-side, syntax-highlighted; plugs into `git diff`, `git show`, `git log -p` |
| `top` / `htop` | **btm (bottom)** | Cross-platform, composable panes |
| `du` | **dust** | Visual treemap output, human-readable by default |
| `man` | **tldr** | Community-maintained practical examples; faster than reading man pages |
| `make` | **just** | Simpler syntax, no tab-sensitivity bugs, per-project justfiles |

**Agent-specific reason these matter:** ripgrep returns columnar output (`file:line:match`) that LLMs parse far more reliably than grep's variable-format output. fd respects `.gitignore` so agents don't accidentally search `node_modules`. bat's syntax highlighting helps when piping into agent contexts. The modern toolchain produces structured-text output that's downstream-friendly; the classic toolchain produces free-form text that requires parsing heuristics.

---

## Runtime version managers

If you work in more than one language, runtime version managers are non-negotiable. The pain saved is enormous.

- **mise** (formerly rtx) — modern multi-language version manager. One tool for Node, Python, Ruby, Go, Java, and 400+ others. `.mise.toml` per project pins versions.
- **asdf** — older, plugin-based, stable. Same idea. Pick if you need a plugin mise doesn't have.
- **uv** for Python specifically — much faster than pip/venv/poetry; increasingly standard. Use alongside mise (mise installs Python; uv manages packages).
- **rustup** for Rust — `mise` doesn't replicate rustup's component/target management. Keep rustup for Rust toolchains.

The `.mise.toml` (or `.tool-versions`) file in a project means agents and humans both pick up the right versions automatically. Skips the "works on my machine" class of failures.

**Avoid the multi-tool trap:** pyenv + nvm + rbenv + sdkman = four shell hooks, four config syntaxes, four startup-latency taxes. Consolidate to one polyglot manager.

---

## fzf-based interactive widgets

**fzf** is the composable selection primitive. Everything else is a layer on top.

The pattern: any command that emits line-delimited text can be made interactive in ~10 lines of shell. Useful widgets to build (or adopt):

- **Git branch picker** with commit-log preview
- **Stash picker** with pop/drop as multi-action keys
- **Docker container picker** with exec/logs/stop as actions
- **Process killer** (`fkill`)
- **Cloud instance picker** (cloud provider list piped through fzf)
- **Version install picker** over `mise registry`
- **GitHub PR/issue/workflow picker** via `gh --json` piped to fzf

The key insight: fzf isn't a tool, it's a pattern. Once you internalize it, you'll build new widgets in minutes rather than reaching for purpose-built tools.

---

## Project navigation

The useful pattern is a **three-source merge**: a curated static list, a frecency database, and auto-discovered git repos under defined root directories. Present all three through a single fzf picker (`p`), with preview showing recent commits and git status.

- **zoxide** — the frecency layer; replaces `cd` for directories you visit repeatedly. Install early. `z <partial>` is faster than any manual navigation.
- **A simple project picker script** combining: hand-curated favorites, zoxide's frecency, and `find ~/code -maxdepth 3 -name .git -printf '%h\n'`

Single hotkey to launch the picker; instant navigation across dozens of projects. One of the highest daily-value tools in the setup.

---

## Shell history

**atuin** — cross-machine shell history sync with structured storage. Replaces Ctrl-R. History is queryable as a database rather than a flat file.

Worth it on day one; becomes essential once you work across multiple machines. The cross-machine sync alone justifies the install.

Configuration tip: opt out of recording sensitive commands (anything matching `*token*`, `*key*`, `*password*` patterns) at the atuin config level. Default-on capture-everything is too aggressive.

---

## Shell scripting discipline

For any script running unattended (CI/CD, agent runners, cron, container init), use a security-first header. The pattern:

```bash
#!/bin/bash -
# trailing - prevents $0 option injection

\unalias -a       # backslash bypasses potential alias on unalias itself
hash -r           # clear command lookup cache
ulimit -H -c 0 -- # disable core dumps (prevents credential leakage)
IFS=$' \t\n'      # safe IFS: space, tab, newline only
CDPATH=''

PATH='/usr/local/bin:/bin:/usr/bin'
export PATH

TMPDIR=$(mktemp -d "/tmp/${0##*/}.XXXXXXXX") || exit 1
readonly TMPDIR
cleanup() { rm -rf "$TMPDIR"; }
trap cleanup ABRT EXIT HUP INT QUIT TERM
```

Why each line matters:

- `#!/bin/bash -` — the trailing `-` stops the shell from treating the first arg as option flags
- `\unalias -a` — the backslash bypasses any alias on `unalias` itself
- `hash -r` — clears command lookup cache; prevents a trojan earlier on PATH from staying cached
- `ulimit -H -c 0` — disables core dumps, which can contain in-memory credentials
- `IFS=$' \t\n'` — explicit safe IFS prevents word-splitting bugs if IFS was modified by a caller
- Hardcoded `PATH` — never inherit the caller's PATH in unattended scripts
- `mktemp -d` + `trap cleanup` — secure temp directory with guaranteed cleanup on any exit

Skip this for interactive one-liners and throwaway scripts. Use it for everything that runs without a human watching.

### Five most common silent bugs in shell scripts

1. **Pipe to `while read` runs in a subshell** — variable increments are lost in the parent. Fix: redirect instead of piping (`while read; done < "$file"`).
2. **Spaces around `=` in assignment** (`COUNT = 5` runs a program named `COUNT`).
3. **`$*` instead of `"$@"`** with filenames containing spaces.
4. **Unquoted variable in destructive command** (`rm -rf $TMPDIR/` when `TMPDIR` is empty → `rm -rf /`).
5. **`#!/bin/sh` on Ubuntu** where `sh` is dash — bash-specific syntax silently fails.

### PATH audit for untrusted environments

```bash
IFS=:
for dir in $PATH; do
    if [ -d "$dir" ] && [ -w "$dir" ]; then
        printf "WARNING: %s is world-writable\n" "$dir"
    fi
done
unset IFS
```

A world-writable PATH directory is an attacker plant point — anyone can drop a binary there that shadows a system command. Run this on any unfamiliar host before running scripts on it.

---

## Patterns that specifically support agentic work

Beyond raw productivity, several patterns make scripts and tooling more agent-friendly:

- **Prefer JSON output where available.** `gh --json`, `jq`, `mise registry --json`. Agents parse structured output without text-parsing heuristics.
- **Where JSON isn't available, prefer columnar output** (rg's `file:line:match`, fd's plain paths). Better than free-form text.
- **`"$@"` when forwarding agent-generated filenames.** Filenames from agents may contain spaces. `"$@"` preserves them; `$*` doesn't.
- **`while read; done < <(...)` for agent-generated loops.** Process substitution avoids the subshell trap that loses state from piped loops.
- **`direnv` for per-project environment isolation.** Agents working across projects need isolated env vars (API keys, project IDs, endpoints). `.envrc` per project gives automatic isolation on `cd`.
- **`just` for agent-invokable tasks.** A `justfile` at project root gives agents a stable interface to project operations without parsing Makefiles.
- **Health check commands.** A `doctor` command per project that verifies tool versions, config presence, auth state. Agents (and humans) get a fast pass/fail signal at session start.

---

## Anti-patterns

- **Over-aliasing.** Every alias is a hidden contract. Aliases break scripts that don't source `.bashrc`, break agent-invoked commands, and create confusion when a name is rebound. Alias sparingly; prefer functions with explicit names or scripts in `~/bin`.
- **Configuration drift.** Configs accumulated inline in `.bashrc` are not modular, not testable, and hard to audit. Use a modular loader: feature files in a config directory, auto-loaded by a small loader script. Disable a feature by renaming the file.
- **Theme over function.** A visually impressive prompt that adds 300ms to shell startup is a tax paid on every command. Measure startup time; lazy-load heavy tools (atuin, direnv) when latency matters.
- **Multiplexer broadcast to interactive panes.** Sending a shell command to all panes via `send-keys` is unsafe if any pane is running an interactive program (vim, less, a Python REPL). The command gets injected into that program. Per-pane push is the safe path.
- **Inheriting caller PATH in scripts.** The most common security gap in unattended scripts. A compromised PATH can shadow system binaries. Hardcode PATH in any script that runs without supervision.
- **Lights-on always.** Every tool initialized eagerly is startup time. Every plugin loaded by default is dependency surface. Trim.

---

## Setup time and payoff

Realistic numbers:

- **Bare functional setup** (shell, prompt, multiplexer, modern Unix tools, runtime manager): 2–3 hours to install, 1–2 weeks of light tweaking before stabilization.
- **Full setup** (above + fzf widgets, atuin, direnv, project picker, custom aliases/functions, theme system): 1–2 days of focused work, then ongoing trim/maintain.
- **Recovery from a wiped machine** (config in git): 30–60 minutes.

The ROI compounds. Every agent run that doesn't fail because of an env quirk, every minute saved by `rg` over `grep`, every command found by fuzzy history — individually small, collectively the difference between flow and friction.

## See also

- [02-platform/dev-environment.md](../02-platform/dev-environment.md) — the abstract patterns; this page is one concrete instance
- [02-platform/cross-platform-parity.md](../02-platform/cross-platform-parity.md) — applying these tools consistently across Mac/Linux/Windows/WSL
- [dev-environment.md](./dev-environment.md) — the section 08 page; broader scope, links to public reference repos
- [cloud-and-deployment-tools.md](./cloud-and-deployment-tools.md) — the same discipline applied to cloud/deployment tooling

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
