# Setting up an Ubuntu host as a Multica runtime

This guide turns a fresh Ubuntu server into a Multica runtime like **dev-ai-1** (Claude Code) or **dev-ai-3** (Codex). It assumes Ubuntu 24.04 LTS on x86_64, a headless server you reach over SSH, and Multica Cloud.

A runtime is two things on one machine: the **Multica daemon**, which connects to Multica and claims runs, and an **AI coding tool**, which the daemon runs for each agent. Everything an agent uses that isn't in its Multica settings also has to be on this machine: MCP servers, `gh`, `bws` and the project's toolchain (see `README.md`).

> **Sources.** Steps marked 📘 follow official documentation: Multica, Claude Code, Codex, GitHub CLI and Bitwarden. Steps marked 🛠 are this repo's recommendations; Multica's docs don't cover them. Those are the dedicated user, running the daemon as a systemd service, and the hardening notes. Check versions and URLs against the linked docs before following them.

## 1. Prepare the host 🛠 + 📘

Run the agents under a dedicated, unprivileged user. Agents run shell commands, so this user must **not** have `sudo`, and must **not** be in the `docker` group (step 6 explains why). Run the commands in this step from an admin account.

**System packages.** Install the basics, plus the general tools coding agents use regularly:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  git curl wget unzip zip ca-certificates gnupg \
  build-essential make python3 python3-venv python3-pip \
  ripgrep fd-find jq tree \
  htop less openssh-client
```

| Group | Packages | Why |
|---|---|---|
| Fetching and archives | `curl`, `wget`, `unzip`, `zip`, `ca-certificates`, `gnupg` | Installers, release downloads, apt signing keys |
| Build and scripting | `build-essential`, `make`, `python3`, `python3-venv`, `python3-pip` | Common in test, lint and build scripts |
| Code search and data | `ripgrep`, `fd-find`, `jq`, `tree` | Searching code, reading JSON output from `multica` and `gh`, listing folders |
| Diagnostics | `htop`, `less`, `openssh-client` | Day-to-day use, and `ssh` for git remotes |

**Node.js LTS** 📘 ([NodeSource](https://github.com/nodesource/distributions)). Many repos and tools need it:

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x -o nodesource_setup.sh
sudo -E bash nodesource_setup.sh
sudo apt install -y nodejs
node --version
```

**The runtime user.** Create the user, then enable linger. Linger lets that user's services (the Multica daemon and rootless Docker) start at boot and keep running when nobody is logged in:

```bash
sudo adduser --disabled-password --gecos "Multica runtime" multica
sudo loginctl enable-linger multica
```

Do everything after this point **as the `multica` user**, except where a step says to use an admin account. Switch with `sudo -iu multica`. Each tool keeps its sign-in in that user's home directory, and the daemon has to find them there.

Make sure `~/.local/bin` is on `PATH`, because several installers below put their binaries there. On Ubuntu, `fd-find` installs its command as `fdfind`, so add the usual `fd` name too:

```bash
mkdir -p ~/.local/bin
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.profile && source ~/.profile
ln -s "$(command -v fdfind)" ~/.local/bin/fd
```

## 2. Install the AI coding tool 📘

Install **one** of these per runtime: Claude Code for a machine like dev-ai-1, Codex for one like dev-ai-3. A machine can run both, and the daemon registers one runtime for each tool it finds. Multica's minimum versions are Claude Code 2.0.0 and Codex 0.100.0; below those, the daemon doesn't register the runtime.

**Claude Code** ([setup docs](https://code.claude.com/docs/en/setup)):

```bash
curl -fsSL https://claude.ai/install.sh | bash
claude --version
```

**Codex** ([Codex CLI](https://developers.openai.com/codex/cli/)):

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
codex --version
```

## 3. Sign the tool in to the subscription 📘

The tool must work on its own in a terminal before the daemon can use it. Multica never sees these credentials; they stay on this machine.

**Claude Code (Claude Max).** On a headless server, use either option:

- **Interactive:** run `claude`, choose to sign in, open the link it prints in a browser on your own computer, and paste the code back in when prompted.
- **Long-lived token:** on a machine where browser sign-in works, run `claude setup-token`. Then make the printed token available to the daemon as `CLAUDE_CODE_OAUTH_TOKEN`; step 8 shows where. This token can only make model requests, which is all an agent run needs.

**Codex (ChatGPT Pro).** Use device-code sign-in, which needs no browser on the server:

```bash
codex login --device-auth      # open the printed URL on your own computer and enter the code
codex login status             # should print "Logged in using ChatGPT"
```

Then check that the tool works without interaction:

```bash
claude -p "Reply OK"      # Claude Code
codex exec "Reply OK"     # Codex
```

## 4. Configure MCP servers

Configure any MCP servers the agents need on the coding agent itself, as the `multica` user. See `README.md` for which agents use which servers. How you do this depends on the tool and your account: servers can be added through the tool's own configuration, or synced from the account the tool is signed in with. Check that they're connected before starting the daemon. Multica's agent settings don't provide them.

## 5. Install `gh` and `bws` 📘

**GitHub CLI** ([install guide](https://github.com/cli/cli/blob/trunk/docs/install_linux.md)):

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install -y gh
```

The `sudo` steps need an admin account, not `multica`. Then, as `multica`, sign in with a GitHub identity that can push branches and open pull requests on the Foodie repo, but can't merge to `main`:

```bash
gh auth login        # choose GitHub.com, HTTPS, and the device/browser code flow
gh auth setup-git    # lets git push over HTTPS using gh's credentials
gh auth status
```

**Bitwarden Secrets Manager CLI** ([releases](https://github.com/bitwarden/sdk-sm/releases)). Pick the latest `bws-v*` release; 2.1.0 is shown here:

```bash
cd /tmp
curl -fsSLO https://github.com/bitwarden/sdk-sm/releases/download/bws-v2.1.0/bws-x86_64-unknown-linux-gnu-2.1.0.zip
unzip bws-x86_64-unknown-linux-gnu-2.1.0.zip && install -m 755 bws ~/.local/bin/bws
bws --version
```

`bws` authenticates with a **machine account access token** in `BWS_ACCESS_TOKEN`. Create a machine account in Bitwarden Secrets Manager with access to only the projects the agents need, then supply its token through the environment file in step 8.

## 6. Install Docker (rootless) 📘

Agents may need containers, for example a database for tests. **Don't add `multica` to the `docker` group:** membership is effectively root access, so any agent could take over the host. Use **rootless Docker** instead, which runs the Docker daemon as the `multica` user ([Docker rootless docs](https://docs.docker.com/engine/security/rootless/)).

**From an admin account**, add Docker's apt repository ([install guide](https://docs.docker.com/engine/install/ubuntu/)):

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update
```

Still as admin, install Docker and the rootless extras, then turn off the system-wide (root) daemon:

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin \
  docker-ce-rootless-extras uidmap dbus-user-session
sudo systemctl disable --now docker.service docker.socket
sudo rm -f /var/run/docker.sock
grep multica /etc/subuid /etc/subgid      # each should show a range of at least 65536
```

Installing `docker-ce-rootless-extras` from apt also provides the AppArmor profile that Ubuntu 24.04 needs for rootless containers. No manual AppArmor step is required.

**As `multica`**, set up and enable the rootless daemon. It starts at boot because linger was enabled in step 1:

```bash
dockerd-rootless-setuptool.sh install
systemctl --user enable --now docker
echo 'export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock' >> ~/.profile && source ~/.profile
docker run --rm hello-world
```

**Limitations to know:**
- Containers can't publish ports below 1024 unless an admin allows it: run `sudo setcap cap_net_bind_service=ep $(which rootlesskit)`, then `systemctl --user restart docker`.
- Resource limits (`--cpus`, `--memory`) need cgroup v2, which Ubuntu 24.04 uses by default.
- Images and containers belong to `multica` and are stored in its home directory, so plan disk space for that.

## 7. Install the project toolchain ❓

Agents run the Foodie project's own build, test and lint commands, so this host needs the same toolchain as the repo: language runtimes, package manager and test tools. Copy it from an existing runtime, or follow the repo's README or contributing guide. *This part is still to be documented.*

## 8. Install and connect the Multica daemon 📘 + 🛠

**Install the CLI** 📘 ([Multica CLI docs](https://multica.ai/docs/cli)):

```bash
curl -fsSL https://raw.githubusercontent.com/multica-ai/multica/main/scripts/install.sh | bash
multica version
```

**Sign in** 📘. A headless server can't use browser sign-in, so use a personal access token (PAT):

1. In Multica on the web: **Settings → API Token → create a token**. Give it a name like `dev-ai-4 runtime` and an expiry. The CLI renews PATs automatically before they expire.
2. On the server: `multica login --token`, then paste the token at the prompt, which keeps it out of shell history.
3. `multica auth status` should show your workspace.

**Run the daemon as a service** 🛠. `multica daemon start` runs in the background but doesn't survive a reboot. Run it as a systemd **user** service in foreground mode instead. Linger, enabled in step 1, keeps it running when nobody is logged in.

As `multica`, create the environment file for secrets, readable only by `multica`:

```bash
mkdir -p ~/.config/multica && touch ~/.config/multica/daemon.env && chmod 600 ~/.config/multica/daemon.env
```

```ini
# ~/.config/multica/daemon.env
BWS_ACCESS_TOKEN=...
# Only if Claude Code uses a setup-token instead of an interactive login:
# CLAUDE_CODE_OAUTH_TOKEN=...
```

```ini
# ~/.config/systemd/user/multica-daemon.service
[Unit]
Description=Multica daemon
After=network-online.target docker.service
Wants=network-online.target docker.service

[Service]
EnvironmentFile=%h/.config/multica/daemon.env
Environment=PATH=%h/.local/bin:/usr/local/bin:/usr/bin:/bin
Environment=DOCKER_HOST=unix:///run/user/%U/docker.sock
ExecStart=%h/.local/bin/multica daemon start --foreground
Restart=on-failure
RestartSec=10

[Install]
WantedBy=default.target
```

Check the `multica` path in `ExecStart` with `command -v multica`. If the installer put it somewhere else, fix the path, and add that directory to `PATH`. The daemon finds AI coding tools through `PATH`. If it can't find one, set `MULTICA_CLAUDE_PATH` or `MULTICA_CODEX_PATH` to an absolute path in the environment file.

```bash
systemctl --user daemon-reload
systemctl --user enable --now multica-daemon
systemctl --user status multica-daemon
multica daemon status
```

Everything in the daemon's environment is passed to agent runs, including `BWS_ACCESS_TOKEN`. That's deliberate, because agents use `bws`. It's also why the machine account should be scoped tightly.

## 9. Verify the runtime 📘

```bash
multica runtime list      # the new machine's tool should show "online"
journalctl --user -u multica-daemon -f   # or: multica daemon logs -f
```

In Multica, open **Runtimes**. The machine and its tool should appear and show as online. If the tool is missing, check, in order:
- `command -v claude` / `command -v codex` as the `multica` user
- that the tool works on its own (step 3)
- `docker info` as `multica`, if agents need containers (step 6)
- the daemon logs
- `systemctl --user restart multica-daemon`

## 10. Put it to work

- **Bind agents** to the new runtime. In each agent's settings, choose this runtime, then select the model and thinking level from `Agents/*/Settings.md`. The model list comes from the runtime, so pick from it rather than typing names in.
- **Runtimes are private** to the person who connected them. Only that owner can create agents on one, or make it public.
- **Concurrency:** one daemon runs at most 20 runs at once by default, and each agent at most its own limit (see `Settings.md`). For a smaller host, lower the machine-wide cap with `MULTICA_DAEMON_MAX_CONCURRENT_TASKS` in the environment file.
- **Update `README.md`** in this folder with the new machine, its tool version and which agents use it.

## Maintenance 🛠

- **Updates:** the Claude Code native install updates itself. For Codex, re-run its installer. For Multica, run `multica update`, then `systemctl --user restart multica-daemon`. Restart the daemon after any tool update or sign-in change, so it re-detects the tools.
- **Expired credentials:** a failed sign-in shows up as failed runs. Check `claude` / `codex login status`, `gh auth status` and `multica auth status`.
- **Network:** the daemon only connects outwards to Multica, so no inbound ports need to be open.
- **Retiring a host:** stop and disable the service, revoke its Multica PAT, GitHub sign-in and Bitwarden machine-account token, then delete the runtime from **Runtimes**. Multica also removes runtimes that have been offline for more than 7 days with no agents bound.
