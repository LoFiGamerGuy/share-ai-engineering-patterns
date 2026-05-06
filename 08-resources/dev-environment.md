# Dev Environment

A productive environment for agentic engineering has more in common with a well-tuned Linux workstation than with a "modern" IDE setup. The agent does most of the editing; you spend most of your time directing, reviewing, and running things. The terminal is the primary surface, the filesystem is the primary substrate, and the IDE plays a supporting role.

This page describes the patterns. Working examples are linked at the bottom.

## The principle

Optimize for **legibility, parity, and recoverability**, in that order:

- **Legibility.** You should be able to look at the screen and tell what's running, where, and why. A noisy or cluttered surface hides the agent's behavior — especially when something goes wrong.
- **Parity.** The setup should look and behave the same on every machine you touch (laptop, desktop, cloud VM, WSL). When environments differ, agents trained on one fail mysteriously on another.
- **Recoverability.** A wiped machine should be back to baseline in under an hour. Configuration as code, version-controlled, idempotent install scripts.

Most engineers have one of these. Few have all three.

## Layers of the setup

A complete dev environment for agentic work has six layers. You can have a productive setup with fewer; six is the inflection point where things start compounding.

### 1. Shell and prompt

A shell that supports modern features (history search, completion, themeable prompt). The specific shell matters less than the discipline of having one config you actually maintain.

- **Bash** with `bash-completion`, `fzf` for history search, and a themeable prompt (Starship, oh-my-bash). Universal availability.
- **Zsh** with `zsh-autosuggestions`, `zsh-syntax-highlighting`, and Starship or oh-my-zsh. Default on macOS; popular on Linux.
- **PowerShell 7+** on Windows when you're running Windows-native tools; otherwise WSL bash.

The prompt should show: current directory, git branch, last command status, and (if relevant) Python venv / Node version / cloud context. Don't overload it.

### 2. Terminal emulator

Pick one and configure it well. Switching emulators monthly is a sign you don't know what you actually want.

- **WezTerm** — cross-platform, fast, scriptable in Lua, sane defaults. Good first pick.
- **Alacritty** — minimal, GPU-accelerated, no built-in tabs (pair with a multiplexer).
- **Windows Terminal** on Windows for the Windows side; WezTerm or similar on the Mac/Linux side.
- **iTerm2** on macOS if you want a more visual configuration surface.

The key feature: a true-color terminal that respects your theme. 256-color terminals make modern color schemes look wrong.

### 3. Multiplexer

Multiple panes in one terminal session. Indispensable when an agent is running in one pane and you're inspecting logs or running adjacent commands in another.

- **tmux** — the universal default. Powerful, ubiquitous, decades-stable.
- **zellij** — modern alternative with a friendlier learning curve. Good if you're new to multiplexers.

Configure split-pane layouts (a "quad" 2x2, a "vertical triple", etc.) as one-keystroke macros. Setup time: 30 minutes; payoff: every day for years.

### 4. Modern Unix tooling

The classic Unix utilities (`grep`, `find`, `cat`, `ls`) have modern replacements that are faster, prettier, and produce output that's easier to scan. Worth swapping the defaults:

- `ripgrep` (`rg`) — replaces `grep`. Respects `.gitignore`, much faster.
- `fd` — replaces `find`. Sane defaults, much faster.
- `bat` — replaces `cat`. Syntax highlighting, line numbers, paging.
- `eza` — replaces `ls`. Colors, icons, git status integration.
- `zoxide` — replaces `cd`. Frecency-based directory jumping.
- `fzf` — fuzzy finder. Wires into shell history, file search, git operations.
- `delta` — replaces `git diff`. Side-by-side, syntax-highlighted.

Agents also benefit from these. A `Bash` tool call to `rg` returns better-shaped output for the model than `grep`.

### 5. Runtime version management

If you work in more than one language, runtime version managers are non-negotiable:

- **mise** (formerly rtx) — modern multi-language version manager. One tool for Python, Node, Ruby, Go, Rust, etc. `.mise.toml` per project pins versions.
- **asdf** — older, plugin-based, stable. Same idea.
- **uv** for Python specifically — much faster than pip/venv/poetry; increasingly standard.

The `.mise.toml` (or `.tool-versions`) file in a project means agents and humans both pick up the right versions automatically. Skips the "works on my machine" class of failures.

### 6. AI tooling layer

The agent harness itself, plus the supporting utilities:

- **Claude Code** (CLI) — the primary harness for agentic work in this repo's voice.
- **Cursor / VS Code + Continue / Zed** — IDE integrations for inline editing alongside agent loops.
- **Ollama** or **LM Studio** for local model serving. See [05-local-llms](../05-local-llms/).
- **MCP servers** for the tools your work needs. See [07-tools-and-mcp/mcp-server-patterns.md](../07-tools-and-mcp/mcp-server-patterns.md).
- **Token-saving proxies** like rtk-ai for high-volume work. See [tool-evaluations.md](./tool-evaluations.md).

Don't install everything at once. Add a tool when you actually need it; uninstall it if it goes unused for a month.

## The configuration-as-code pattern

Every layer above should be config-driven. Concrete pattern:

```
~/dotfiles/                 # version-controlled
├── shell/
│   ├── bashrc.d/           # modular shell config
│   └── starship.toml
├── terminal/
│   ├── wezterm.lua
│   └── alacritty.toml
├── tmux/
│   └── tmux.conf
├── tools/
│   └── mise.toml
├── claude/
│   ├── CLAUDE.md           # global agent persona
│   └── settings.json       # tool permissions, hooks
└── install.sh              # idempotent installer
```

Then on a fresh machine: `git clone`, `./install.sh`, done. The whole environment recreates from the repo. Test it twice a year by intentionally setting up a fresh VM.

## Cross-platform parity

If you work across Mac, Linux, and Windows, you'll discover that "the same setup" is harder than it sounds.

Practical strategies:

- **WSL on Windows.** Treat the WSL Linux environment as your primary; the Windows side handles GUI apps and IDE only. Most cross-platform pain disappears.
- **Symlink-based dotfiles.** The same files install to `~/.bashrc` on Linux and `C:\Users\<you>\.bashrc` on Windows. Don't fork per-platform unless you must.
- **mise / asdf for runtime versions.** Same `.mise.toml` works on every OS. No more "Python 3.10 vs 3.11" platform skews.
- **Test agentic flows on both platforms.** An agent that flawlessly edits files on Mac may misbehave on Windows due to line-ending or path-separator differences. Catch this early.

## Common pitfalls

- **Theme over function.** A beautiful prompt that hides errors is worse than an ugly one that surfaces them.
- **Too many tools.** Every tool added is a tool to maintain, document, and explain to the agent. Trim aggressively.
- **No backup of config.** A laptop dies and the setup is gone. If your dotfiles aren't in git, they don't really exist.
- **Configuration drift between machines.** "Why does this work on my desktop but not my laptop?" — almost always a config that exists in only one place.
- **Mixing system Python and your project Python.** Use a runtime manager. The pain saved is enormous.
- **Hand-edited config files.** If you can't remember why a line is in your `.bashrc`, delete it. Lines you can't explain age into bugs.

## Working examples

These are the public reference implementations of the patterns above. All are installable on Mac, Linux, or WSL.

- **[github.com/LoFiGamerGuy/dotfiles](https://github.com/LoFiGamerGuy/dotfiles)** — personal dotfiles. Shell config, prompt, tool integrations. Idempotent installer.
- **[github.com/LoFiGamerGuy/terminal-stack](https://github.com/LoFiGamerGuy/terminal-stack)** — opinionated terminal kit for Git Bash on Windows. 24 themes with one-command live swap, modern Unix tools wired in, fzf widgets for git/AWS/GCP/Docker, multi-pane launchers, health check (`doctor`), idempotent installer. Cross-platform parity with the Mac/Linux side.
- **[github.com/LoFiGamerGuy/macstyle](https://github.com/LoFiGamerGuy/macstyle)** — macOS-side counterpart for visual and behavioral parity with the Windows terminal stack.
- **[github.com/LoFiGamerGuy/temp-env-synth-macbook](https://github.com/LoFiGamerGuy/temp-env-synth-macbook)** — environment-synthesis scratch repo for a fresh MacBook setup. Useful as a "what does cold-start look like" reference.

These aren't templates to copy verbatim — they're worked examples to learn from. Read the install scripts and config files; adopt patterns; don't fork unless you intend to maintain.

## Setup time and ROI

Realistic numbers:

- **Bare functional setup** (shell, prompt, multiplexer, modern Unix tools, runtime manager): 2–3 hours to install, 1–2 weeks of light tweaking before it stabilizes.
- **Full setup with AI tooling, MCP servers, custom hooks, and cross-platform parity:** 1–2 days of focused work, then ongoing trim/maintain.
- **Recovery from wiped machine** (with config in git): 30–60 minutes.

The ROI compounds: every agent run that doesn't fail because of an env quirk, every minute saved by `rg` over `grep`, every command found by fuzzy history search. Individually small, collectively the difference between flow and friction.

## See also

- [02-platform/dev-environment.md](../02-platform/dev-environment.md) — the abstract patterns; this page is the concrete instance
- [02-platform/agentic-os-spine.md](../02-platform/agentic-os-spine.md) — what makes a *project* agent-ready (this page is what makes a *machine* agent-ready)
- [tool-evaluations.md](./tool-evaluations.md) — picks for the AI tooling layer specifically

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
