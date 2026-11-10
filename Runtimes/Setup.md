# Setting up an Ubuntu host as a Multica runtime

This guide turns a fresh Ubuntu server into a Multica runtime like **dev-ai-1** (Claude Code) or **dev-ai-3** (Codex). It assumes Ubuntu 24.04 LTS on x86_64, a headless server you reach over SSH, and Multica Cloud.

A runtime is two things on one machine: the **Multica daemon**, which connects to Multica and claims runs, and an **AI coding tool**, which the daemon runs for each agent. Everything an agent uses that isn't in its Multica settings also has to be on this machine: MCP servers, `gh`, the Infisical CLI and Agent, and the project's toolchain (see `README.md`).

> **Sources.** Steps marked 📘 follow official documentation: Multica, Claude Code, Codex, Homebrew, GitHub CLI and Infisical. Steps marked 🛠 are this repo's recommendations; Multica's docs don't cover them. Those are the dedicated user, installing Homebrew for a user without `sudo`, running the daemon and the Infisical Agent as systemd services, and the hardening notes. Check versions and URLs against the linked docs before following them.

## 1. Prepare the host 🛠 + 📘

Run the agents under a dedicated, unprivileged user. Agents run shell commands, so this user must **not** have `sudo`, and must **not** be in the `docker` group (step 6 explains why). Run the commands in this step from an admin account.

**System packages.** Install what Homebrew needs, plus the general tools coding agents use regularly. Most of the other tools come from the Brewfile later in this step:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  git curl unzip zip ca-certificates gnupg \
  build-essential procps file \
  ripgrep fd-find jq tree \
  less openssh-client
```

| Group | Packages | Why |
|---|---|---|
| Fetching and archives | `curl`, `unzip`, `zip`, `ca-certificates`, `gnupg` | Installers, release downloads, apt signing keys |
| Homebrew prerequisites | `git`, `curl`, `build-essential`, `procps`, `file` | Required by Homebrew on Linux; `build-essential` is also common in test and build scripts |
| Code search and data | `ripgrep`, `fd-find`, `jq`, `tree` | Searching code, reading JSON output from `multica` and `gh`, listing folders |
| Diagnostics | `less`, `openssh-client` | Day-to-day use, and `ssh` for git remotes |

**The runtime user.** Create the user, then enable linger. Linger lets that user's services (the Multica daemon and rootless Docker) start at boot and keep running when nobody is logged in:

```bash
sudo adduser --disabled-password --gecos "Multica runtime" multica
sudo loginctl enable-linger multica
```

Do everything after this point **as the `multica` user**, except where a step says to use an admin account. Switch with `sudo -iu multica`. Each tool keeps its sign-in in that user's home directory, and the daemon has to find them there.

> **Never set up the runtime from your own account.** Running `multica setup`, `multica login`, `multica daemon start` or a coding tool's sign-in as yourself makes the runtime run as you, with all your groups. If you are in `sudo`, `lxd`, `docker` or `adm`, agents can get root (`lxd` and `docker` are root-equivalent). Agents also can't read the Infisical token, because only the `multica` group can (step 5). That is how multica-worker-1 broke on 2026-11-09 (NATG-633). Step 9 checks for it.

Make sure `~/.local/bin` is on `PATH`, because several installers below put their binaries there. On Ubuntu, `fd-find` installs its command as `fdfind`, so add the usual `fd` name too:

```bash
mkdir -p ~/.local/bin
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.profile && source ~/.profile
ln -s "$(command -v fdfind)" ~/.local/bin/fd
```

**Homebrew and the Brewfile** 📘 ([Homebrew on Linux](https://docs.brew.sh/Homebrew-on-Linux)) + 🛠. This repo's [`Brewfile`](Brewfile) installs Claude Code, `gh`, Node.js, Python with `uv`, `git-lfs`, the AWS and Kubernetes CLIs, and a few utilities (`wget`, `nmap`, `fzf`, `htop`). The Infisical CLI is commented out of it on purpose: it comes from apt in step 5, because `scripts/secret.sh` runs only `/usr/bin/infisical`.

Homebrew installs to `/home/linuxbrew/.linuxbrew` and normally uses `sudo` to create that folder. `multica` has no `sudo`, so an admin creates the folder and hands it to `multica`:

```bash
# admin account
sudo mkdir -p /home/linuxbrew/.linuxbrew
sudo chown -R multica:multica /home/linuxbrew
```

Then, as `multica`, install Homebrew into it and run the Brewfile:

```bash
NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile && source ~/.profile
curl -fsSL https://raw.githubusercontent.com/natgreenwood/multica-config/main/Runtimes/Brewfile -o ~/Brewfile
brew bundle --file ~/Brewfile
git lfs install
node --version && gh --version && claude --version
```

To add tools later, change the Brewfile in this repo, download it again and re-run `brew bundle --file ~/Brewfile`.

## 2. Install the AI coding tool 📘

Install **one** of these per runtime: Claude Code for a machine like dev-ai-1, Codex for one like dev-ai-3. A machine can run both, and the daemon registers one runtime for each tool it finds. Multica's minimum versions are Claude Code 2.0.0 and Codex 0.100.0; below those, the daemon doesn't register the runtime.

**Claude Code** ([setup docs](https://code.claude.com/docs/en/setup)) is already installed by the Brewfile in step 1 (`cask "claude-code"`). Check it with `claude --version`.

The Brewfile installs it on every host, including Codex-only ones like dev-ai-3, and the daemon registers a Claude runtime wherever it finds `claude`. On a Codex-only host, either bind no agents to that runtime or remove it with `brew uninstall --cask claude-code`.

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

## 5. Install `gh` and Infisical 📘 + 🛠

**GitHub CLI** ([manual](https://cli.github.com/manual/)) is already installed by the Brewfile in step 1. As `multica`, sign in with a GitHub identity that can push branches and open pull requests on the Foodie repo, but can't merge to `main`:

```bash
gh auth login        # choose GitHub.com, HTTPS, and the device/browser code flow
gh auth setup-git    # lets git push over HTTPS using gh's credentials
gh auth status
```

**Git identity** 🛠. Agents commit their work, and git refuses to commit without a name and email. As `multica`, set them once for the user. Use the host name, so a commit shows which runtime made it, and the signed-in GitHub account's no-reply address, so commits link to that account without exposing a real email:

```bash
sudo -iu multica          # from an admin account: a login shell as multica, with its home and PATH
git config --global user.name "$(hostname) (Multica)"
git config --global user.email "$(gh api user --jq '"\(.id)+\(.login)@users.noreply.github.com"')"
git config --global --get-regexp '^user\.'   # check both are set
exit                      # back to the admin account
```

Use `-i`: without it, `git config --global` writes to the admin account's home and `gh` may not be on `PATH`. Sign in to `gh` first (above), or the email line sets an empty address. On a host whose daemon runs as a different user, set the identity as that user instead. To see which user runs the daemon: `ps -o user= -p "$(pgrep -f 'multica daemon' | head -1)"`.

Don't add a `url.<ssh>.insteadOf` rewrite for GitHub. Agents and `multica repo checkout` use HTTPS with `gh`'s credentials. On dev-ai-1 that rewrite sent every checkout over SSH on port 22, which the network blocks, so checkouts hung until they timed out (NATG-536).

**Infisical** gives agents their secrets from our self-hosted Infisical, `https://infisical.natg.co.uk`. Agents never sign in to it themselves. Each repository that uses secrets carries `scripts/secret.sh` and an `.infisical.yaml` naming its Infisical project (see `natgreenwood/containers`). The script reads an access token that the **Infisical Agent** keeps renewed on this host, and sends it only to the server pinned on this host. Its paths and checks are fixed, so this host needs exactly the layout below.

| Path | Owner and mode | Holds |
|---|---|---|
| `/usr/bin/infisical` | root, from apt | The CLI. The script runs only this path |
| `/etc/infisical-agent/multica/domain` | `root:root`, `0644` | The pinned server URL. Only root may write it, or the script refuses |
| `/etc/infisical-agent/multica/auth/` | `infisical-agent`, `0700` | The machine identity's client ID and secret: this host's only long-lived Infisical credential |
| `/var/lib/infisical-agent/multica/token` | `infisical-agent:multica`, `0640` | The access token the Agent renews, which the script reads |

The client secret stays with a separate `infisical-agent` user, so an agent running as `multica` can read the short-lived token but never the credential that creates it.

**Install the CLI** 📘 ([CLI docs](https://infisical.com/docs/cli/overview)), from an admin account. Use apt, not Homebrew, so it lands at `/usr/bin/infisical`:

```bash
curl -1sLf 'https://artifacts-cli.infisical.com/setup.deb.sh' | sudo -E bash
sudo apt-get install -y infisical
command -v infisical      # must print /usr/bin/infisical
```

**Create a machine identity** 📘 ([Universal Auth](https://infisical.com/docs/documentation/platform/identities/universal-auth)). In Infisical, create one identity per runtime host, such as `dev-ai-4`, with Universal Auth. Give it read-only access to only the projects the agents' repositories name in `.infisical.yaml`, then create a client secret. Only Nathaniel adds or changes secrets in Infisical.

**Lay out the files** 🛠, from an admin account:

```bash
sudo adduser --system --group --no-create-home infisical-agent
sudo install -d -m 0755 /etc/infisical-agent /etc/infisical-agent/multica
echo 'https://infisical.natg.co.uk' | sudo tee /etc/infisical-agent/multica/domain > /dev/null
sudo chmod 0644 /etc/infisical-agent/multica/domain
sudo install -d -m 0700 -o infisical-agent -g infisical-agent /etc/infisical-agent/multica/auth
sudo install -m 0600 -o infisical-agent -g infisical-agent /dev/null /etc/infisical-agent/multica/auth/client-id
sudo install -m 0600 -o infisical-agent -g infisical-agent /dev/null /etc/infisical-agent/multica/auth/client-secret
sudoedit /etc/infisical-agent/multica/auth/client-id       # paste the client ID
sudoedit /etc/infisical-agent/multica/auth/client-secret   # paste the client secret
sudo install -d -m 2750 -o infisical-agent -g multica /var/lib/infisical-agent /var/lib/infisical-agent/multica
```

The setgid bit (`2750`) on the token folder gives the token file the `multica` group, so `multica` can read it.

```yaml
# /etc/infisical-agent/multica/agent-config.yml (no secrets, only paths)
infisical:
  address: 'https://infisical.natg.co.uk'
auth:
  type: 'universal-auth'
  config:
    client-id: '/etc/infisical-agent/multica/auth/client-id'
    client-secret: '/etc/infisical-agent/multica/auth/client-secret'
    remove_client_secret_on_read: false
sinks:
  - type: 'file'
    config:
      path: '/var/lib/infisical-agent/multica/token'
```

**Run the Agent as a system service** 🛠 ([Infisical Agent docs](https://infisical.com/docs/integrations/platforms/infisical-agent)):

```ini
# /etc/systemd/system/infisical-agent-multica.service
[Unit]
Description=Infisical Agent for the Multica runtime
After=network-online.target
Wants=network-online.target

[Service]
User=infisical-agent
Group=infisical-agent
UMask=0027
Environment=HOME=/tmp
Environment=INFISICAL_DISABLE_TELEMETRY=1
ExecStart=/usr/bin/infisical agent start --config /etc/infisical-agent/multica/agent-config.yml
Restart=on-failure
RestartSec=10
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/infisical-agent/multica

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now infisical-agent-multica
sudo systemctl status infisical-agent-multica
sudo -u multica test -s /var/lib/infisical-agent/multica/token && echo "token readable by multica"
```

Only members of the `multica` group can read the token, so the user that runs the Multica daemon must be `multica`. A check as `multica` passes even when the daemon runs as someone else; step 9 checks the daemon's real user. Don't fix a failure by adding your own account to the `multica` group: run the daemon as `multica` instead (step 8).

❓ This layout is derived from `scripts/secret.sh` in `natgreenwood/containers`. How dev-ai-1 and dev-ai-3 actually run the Agent (service name, user, unit options) is not yet confirmed. Step 9 checks a real fetch.

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
# No Infisical credential goes here (see below).
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
Environment=PATH=/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:%h/.local/bin:/usr/local/bin:/usr/bin:/bin
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

Everything in the daemon's environment is passed to agent runs. That's why no Infisical credential goes in this file: agents get secrets only through `scripts/secret.sh`, which reads the Agent's token file (step 5). The token can still read whatever the machine identity can, which is why the identity is read-only and scoped to the agents' projects.

## 9. Verify the runtime 📘 + 🛠

First check that the daemon runs as `multica`, and that `multica` has no admin groups. From any account:

```bash
ps -o user= -p "$(pgrep -f 'multica daemon' | head -1)"   # must print: multica
id multica                                                 # groups: multica only; never sudo, lxd, docker or adm
```

If the first command prints any other user, the runtime was set up from the wrong account (see the warning in step 1). Stop that daemon, redo steps 2 to 5 and step 8 as `multica` (tool installs, sign-ins, MCP servers, `gh`, the git identity and the daemon service), and remove the old runtime from **Runtimes** in Multica.

Then, as `multica`:

```bash
multica runtime list      # the new machine's tool should show "online"
journalctl --user -u multica-daemon -f   # or: multica daemon logs -f
```

To check secrets end to end, as `multica`, in a checkout of a repository that has `scripts/secret.sh`, fetch one secret the repository's instructions name, without printing it. Because the daemon runs as `multica`, this is the same access agents get:

```bash
scripts/secret.sh <NAME> > /dev/null && echo "secret fetch OK"
```

In Multica, open **Runtimes**. The machine and its tool should appear and show as online. If the tool is missing, check, in order:
- `command -v claude` / `command -v codex` as the `multica` user
- that the tool works on its own (step 3)
- `docker info` as `multica`, if agents need containers (step 6)
- `systemctl status infisical-agent-multica`, if agents need secrets (step 5)
- the daemon logs
- `systemctl --user restart multica-daemon`

## 10. Put it to work

- **Bind agents** to the new runtime. In each agent's settings, choose this runtime, then select the model and thinking level from `Agents/*/Settings.md`. The model list comes from the runtime, so pick from it rather than typing names in.
- **Runtimes are private** to the person who connected them. Only that owner can create agents on one, or make it public.
- **Concurrency:** one daemon runs at most 20 runs at once by default, and each agent at most its own limit (see `Settings.md`). For a smaller host, lower the machine-wide cap with `MULTICA_DAEMON_MAX_CONCURRENT_TASKS` in the environment file.
- **Update `README.md`** in this folder with the new machine, its tool version and which agents use it.

## Maintenance 🛠

- **Updates:** as `multica`, run `brew update && brew upgrade`. That updates Claude Code, `gh`, Node.js and the other Brewfile tools; a Homebrew install of Claude Code doesn't update itself. For Codex, re-run its installer. For Multica, run `multica update`, then `systemctl --user restart multica-daemon`. For Infisical, an admin runs `sudo apt update && sudo apt install --only-upgrade infisical`, then `sudo systemctl restart infisical-agent-multica`. Restart the daemon after any tool update or sign-in change, so it re-detects the tools.
- **Expired credentials:** a failed sign-in shows up as failed runs. Check `claude` / `codex login status`, `gh auth status`, `multica auth status` and `systemctl status infisical-agent-multica`. A `scripts/secret.sh` failure that names the token file usually means the Agent has stopped or its client secret has expired.
- **Network:** the daemon only connects outwards to Multica, so no inbound ports need to be open.
- **Retiring a host:** stop and disable both services (`multica-daemon` and `infisical-agent-multica`). Revoke the host's Multica PAT and GitHub sign-in, and its Infisical machine identity's client secret (or delete the identity), then delete the runtime from **Runtimes**. Multica also removes runtimes that have been offline for more than 7 days with no agents bound.
