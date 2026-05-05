# Sandbox Environments

Where the agent runs. Different environments have different blast radii. Pick the smallest one that still lets the agent do its job.

## The blast-radius hierarchy

From smallest to largest:

1. **Ephemeral container.** Spun up for one task, destroyed after. No persistent state, no shared files, no network beyond what you allow. The most contained option.
2. **Dedicated VM.** A long-lived virtual machine for agent work. Persistent state across runs, but isolated from your host and from production.
3. **WSL distribution.** A Windows Subsystem for Linux distro dedicated to agent work. Lighter than a full VM, still meaningfully isolated.
4. **Separate user account.** Agent runs as a different user with limited permissions. Contains damage to that user's files.
5. **Your laptop directly.** No isolation. Agent can read/write anything you can. Largest blast radius for interactive work.
6. **Production-adjacent.** Agent has credentials that touch real systems. Largest blast radius full stop. Treat like an employee with admin access.

Every step up the hierarchy adds some friction. Every step down adds risk. Match the environment to the task.

## Containers (Docker, Podman)

The default for any unattended automation. Patterns:

**Throwaway per-task.**
```bash
docker run --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  --network none \
  agent-image:latest \
  agent-cli --task spec.md
```
- `--rm` destroys the container after exit
- `--network none` prevents network access (for tasks that don't need it)
- Volume mount gives access to the working directory only

**Persistent dev container.** A `devcontainer.json` defines a container with the project's tools installed. The agent runs inside it. State persists between runs but is isolated from the host. Useful when setup is expensive.

**Image hygiene.** Don't bake credentials into images. Pass them at runtime via environment variables. Don't run as root inside the container if you can avoid it. Pin base image versions.

## VMs

For longer-running work or work that needs more system access than a container provides comfortably. Patterns:

**Disposable VM per agent run.** Spin up, run task, shut down. Cloud providers make this cheap. High isolation, slower than containers.

**Persistent agent VM.** A long-lived VM that's "the agent's machine." You SSH in, dispatch tasks, the agent runs there. State accumulates, which has both benefits (caching, project state) and risks (credentials, history).

For VM-based agent work, treat it like a junior team member's laptop: it has tools and access to repos, but not production. Use a separate identity, not your own.

## WSL

If your host is Windows, WSL gives you Linux without a full VM. A dedicated WSL distribution can be your "agent zone":

```
wsl --install -d Ubuntu-AgentSandbox
```

Use a separate distribution (not your main dev distro) for agent work. Filesystem is shared with the Windows host but the Linux process tree is isolated. Network access is the host's network.

WSL is convenient. It's not as isolated as a VM. Don't put production credentials there.

## Separate user accounts

A lighter-weight option than VMs: a separate Unix user for agent work.

```bash
useradd -m agent-user
sudo -u agent-user bash
```

The agent runs as `agent-user`. It can read/write that user's home, but not yours. With proper permissions, it can't read your `~/.ssh`, your `~/.aws`, your password manager.

Caveats:
- The agent user can probably still read system files (`/etc`) and run system commands
- If the agent escalates (sudo, suid binaries) it can escape
- Less isolated than a VM but real if configured carefully

## Network isolation

Independent of the environment choice, decide what network access the agent has.

- **No network.** Highest isolation. Possible for many code-only tasks.
- **Whitelist.** Outbound to specific hosts (npm, GitHub, your model provider). Block everything else.
- **Full network.** Agent can reach anywhere. Standard for development, dangerous for production-adjacent work.

For unattended runs, default to whitelist. Tasks that need access to specific systems (a database, an internal API) get those endpoints whitelisted explicitly.

## Credential isolation

The most common mistake: the agent has access to credentials it didn't need.

Practices:
- Use a credential vault, not env files. Pull credentials at task start, scoped to the task.
- Use scoped tokens, not long-lived ones. A GitHub token that can read one repo, not your entire account.
- Rotate aggressively. If the agent crashed and the credential might have been logged, rotate.
- Audit. Periodically review what credentials the agent's environment has, prune what's not needed.

Production credentials in a sandbox is a contradiction. If your agent has them, it's not really sandboxed.

## Filesystem isolation

The agent should be able to read and write only what it needs.

In containers, mount only the relevant directories.
```
docker run -v $(pwd)/src:/workspace/src ... # only mounts src/
```

In VMs and host setups, use a project directory the agent can fully use, and keep sensitive files (your `~/.ssh`, your password files) in directories the agent isn't told about and shouldn't touch.

For unattended runs, consider a read-only mount of source code with a separate writable scratch space the agent uses for outputs. Forces the agent to be explicit about what it's changing.

## Reproducibility

A useful side effect of sandboxing: reproducibility. If the environment is defined by an image, anyone can recreate it. Bug reports become "ran agent on this image with this spec, got this result," which is reproducible.

Without sandboxes, "the agent broke" depends on the host machine, the host's installed tools, the host's environment variables. Hard to debug.

## Recovery

When the agent does something wrong in a sandbox, recovery is:

- Container: throw it away, start a new one.
- VM: revert to a snapshot.
- WSL: `wsl --unregister <distro>` and reinstall.
- Separate user: `userdel -r agent-user`.
- Host: restore from backup (you have backups, right?).

The first three are minutes. The last two are hours-to-days. Plan for recovery; don't assume it won't be needed.

## Practical recommendation

For most teams getting started:
- **Interactive work** on the host with judgment about what the agent touches.
- **Unattended automation** in containers with no network access and a scoped working directory.
- **Production-touching work** with explicit per-task credentials, never long-lived ones.

You don't need a complex infrastructure to start. A `Dockerfile` and a `docker run --rm` invocation gets you 80% of the value of containerization. Build up from there as you find specific gaps.
