# Cross-Platform Parity

If you work on more than one operating system — Mac at home, Linux on servers, Windows at work, WSL on a Windows laptop, or any combination — you'll discover that "the same setup" is far harder than it sounds. Tools that work flawlessly on one platform fail mysteriously on another. Agents that edit code correctly on Mac mangle line endings on Windows. Scripts that run perfectly in CI break on a teammate's machine.

This page is the discipline that prevents most of this.

## The principle

**Treat parity as a product, not an aspiration.** A setup with parity is a setup where:

- The same dotfiles install cleanly on every OS you use
- The same tools are available with the same commands
- The same scripts run with the same behavior
- An agent run on machine A produces the same artifacts as the same run on machine B

If any of those isn't true, you have parity drift, which manifests as "works on my machine" failures and surprise breakage.

## What works the same on every OS (modern toolchain)

The good news first. The modern Unix-like toolchain has gotten genuinely cross-platform:

- **ripgrep, fd, bat, eza, delta, fzf, zoxide, atuin, mise, just, lazygit, starship** — all have native binaries for Linux, macOS, and Windows. Same commands, same flags, same config files.
- **Git** — same on every platform (modulo line-ending defaults; see below).
- **Docker / Podman** — same CLI on every platform; the engine differs but the surface doesn't.
- **Most modern language runtimes** — Node, Python, Go, Rust — work the same on every OS. Compiled outputs differ; the developer surface largely doesn't.
- **MCP servers** — protocol-defined; portable across OSes by design.

If you stick to this layer, parity comes mostly for free. The pain points are everywhere *outside* this layer.

## Where parity breaks (and what to do)

### Line endings (CRLF vs LF)

The classic. Windows uses CRLF; Mac and Linux use LF. Mismatched line endings cause shell scripts to fail with cryptic errors (`bad interpreter: /bin/bash^M`) and cause git diffs to show "every line changed" when nothing actually did.

**Discipline:**

- In your global git config: `core.autocrlf=input` (Linux/Mac) or `core.autocrlf=true` (Windows). Or, better:
- Add a `.gitattributes` file to every repo specifying line endings per file type:

```
* text=auto eol=lf
*.bat text eol=crlf
*.ps1 text eol=crlf
*.sh text eol=lf
```

- Before running an unfamiliar script: `file script.sh` — if it says "with CRLF line terminators," fix it before running (`dos2unix script.sh` or sed equivalent).

### Path separators

Windows uses `\`; everything else uses `/`. Most modern tools accept both, but not all. Bash on Git Bash for Windows handles `/` correctly for most operations but trips on edge cases (e.g., `mise activate` emits Windows-format PATH that bash can't resolve).

**Discipline:**

- In code (Python, Node, etc.): always use `pathlib.Path` / `path.join` rather than string concatenation
- In shell scripts intended to be cross-platform: use forward slashes; both Git Bash and POSIX shells accept them
- For `mise` on Git Bash: use shims mode (`mise.toml` style) instead of `mise activate` hook mode

### Font names

Nerd Fonts use abbreviated metadata names — `JetBrainsMono NFM` is what the system registers, not `JetBrainsMono Nerd Font Mono` as it appears in marketing. Configuring your terminal with the marketing name silently falls back to the system default font; icons and powerline glyphs disappear without error.

**Discipline:**

- After installing a Nerd Font, list the registered families and use the exact registered name in terminal config
- Test by rendering a known glyph (e.g., the Powerline arrow); if you see a placeholder square, the font name is wrong

### Shell defaults

macOS defaults to Zsh. Most Linux defaults to Bash. Windows has PowerShell, plus Git Bash if installed, plus WSL. A script written for Zsh that uses `[[` array syntax may fail on a system where `/bin/sh` is dash (Ubuntu's default).

**Discipline:**

- Pick one shell as your interactive default and configure it the same way on every machine
- For scripts, pin the shell explicitly with the shebang: `#!/usr/bin/env bash` not `#!/bin/sh`
- For shell-portable scripts: stick to POSIX features. If you need bash-isms, declare bash explicitly and accept the dependency

### Aliases and functions

A function defined in `.bashrc` exists only in interactive shells that source `.bashrc`. Scripts run with `bash script.sh` don't see it. Agents that invoke shell commands via a harness usually don't see it either.

**Discipline:**

- Don't assume aliases exist in scripts or agent-invoked contexts
- Define commonly-needed functionality as scripts in `~/bin` or `~/.local/bin`, not as shell aliases
- For convenience aliases that help interactive use, alias a script call rather than embedding the logic in the alias

### Default tools

`grep` on Mac is BSD grep; on Linux it's GNU grep. Different flag support. `sed` differs the same way. `date`, `cp`, `tar`, and many others have BSD vs GNU divergences that bite when scripts move between platforms.

**Discipline:**

- Install GNU coreutils on Mac via Homebrew if you regularly write portable shell (`brew install coreutils`)
- Or, better: use modern replacements (`rg` instead of `grep`, `fd` instead of `find`, `sd` instead of `sed`) — they have the same behavior across platforms

### Notification / desktop integration

Toast notifications, system tray, file dialogs — these are platform-specific. A script that uses `osascript` for Mac notifications won't work on Linux or Windows.

**Discipline:**

- Wrap platform-specific calls behind a single function (`alert "message"`) that detects the platform and dispatches appropriately
- For agent-relevant notifications: prefer email/Slack/file-based output that works the same everywhere over native desktop notifications

### Cloud CLIs

`gcloud`, `aws`, `az` install on every platform, but auth flows differ subtly (browser vs gcloud-cli flow vs config files) and credential storage locations differ.

**Discipline:**

- Use environment variables for credentials in scripts (`AWS_PROFILE`, `GOOGLE_APPLICATION_CREDENTIALS`) rather than relying on platform-specific credential stores
- Document the auth setup once per project so a teammate or agent on a different OS can replicate it

## WSL specifically

Windows Subsystem for Linux is the strongest cross-platform pattern available. WSL 2 gives you a real Linux environment on Windows, which means most cross-platform pain disappears if you treat WSL as your primary development environment on Windows machines.

**Discipline:**

- Treat the Linux side as primary; Windows hosts only the GUI apps (browser, IDE, Slack, etc.) and the terminal emulator
- Project files live inside WSL's filesystem, not on `/mnt/c/...`. Filesystem performance is much better, and there are fewer cross-OS surprises
- Use Windows Terminal as the terminal emulator; configure it with profiles for both PowerShell and WSL bash
- Tools installed in WSL don't see Windows `$PATH` by default, and vice versa, unless you configure `WSLENV`. Treat them as separate machines that share a filesystem

## The dotfiles repo pattern

The single most important parity tool: a version-controlled dotfiles repo with an idempotent installer. A reasonable structure:

```
dotfiles/
├── shell/
│   ├── bashrc.d/        # modular shell config
│   ├── zshrc            # zsh-specific entry
│   └── starship.toml
├── terminal/
│   ├── wezterm.lua
│   └── windows-terminal.json
├── tmux/
│   └── tmux.conf
├── tools/
│   ├── mise.toml
│   └── direnv.toml
├── agent/
│   ├── CLAUDE.md        # global agent persona
│   └── settings.json    # tool permissions, hooks
└── install.sh           # idempotent: safe to run multiple times
```

The installer either symlinks or copies files into the right places per OS. On a fresh machine: `git clone`, `./install.sh`, done.

**Test parity twice a year** by intentionally setting up a fresh VM (or container) and running your installer. If anything fails, fix the installer; that's where the parity bugs accumulate.

## Agent-specific parity concerns

Agents introduce a few additional parity considerations beyond what a developer would notice:

- **Line endings in agent-edited files.** An agent editing files on Windows may insert CRLF; the same file edited on Mac stays LF. PRs end up with mixed line endings. Fix: enforce via `.gitattributes` so the editor (or git on commit) normalizes.
- **Tool availability in agent environments.** An agent that calls `rg` works on a machine where ripgrep is installed; fails on one where it isn't. Make the agent's required tooling part of the project's setup script.
- **Path resolution in agent shells.** Agent harnesses often invoke commands in a non-interactive shell that doesn't source `.bashrc`. PATH may differ from your interactive shell. Use absolute paths in agent-invoked commands when in doubt.
- **Environment variables.** API keys, project IDs, service endpoints — all differ across machines. Use `.env.example` (committed) and `.env` (gitignored) per project so the agent finds the same names everywhere.

## A pragmatic test

If you can do all of these, your parity is good:

1. Pull your repo onto a fresh machine, run the install script, and have a working environment in under an hour
2. Move a half-finished agent task from Mac to Windows (or WSL to Linux) without anything breaking
3. Run a script you wrote three months ago on a different machine without modification
4. Have a teammate clone your project and reproduce your dev environment from documentation alone

If any of these fails consistently, you have parity drift worth fixing.

## What to skip

You don't need:

- A unified GUI experience across OSes (terminal-first work is more portable than GUI-first)
- A single config file format (let each tool use its native format; the dotfiles repo abstracts this)
- The exact same theme everywhere (themes are aesthetic; behavior matters more)
- Identical hardware (parity is software discipline; hardware variation is fine)

The goal is *behavioral* parity. Visual parity is a bonus, not the target.

## See also

- [dev-environment.md](./dev-environment.md) — the abstract patterns; this page is the cross-platform application
- [agentic-os-spine.md](./agentic-os-spine.md) — the project-side files that make agents work consistently across machines
- [08-resources/dev-environment.md](../08-resources/dev-environment.md) — concrete reference setup with working examples (terminal-stack, dotfiles, macOS counterpart)
- [08-resources/shell-and-terminal-tools.md](../08-resources/shell-and-terminal-tools.md) — the modern toolchain that gets you most of the parity for free

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
