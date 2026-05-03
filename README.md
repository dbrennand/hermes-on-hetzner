# Hermes on Hetzner

🚀 Provision a Hetzner Cloud VPS for [Hermes Agent](https://hermes-agent.nousresearch.com/docs) with Ansible, then deploy Hermes onto it.

Feel free to fork this project and adapt it to your own infrastructure, Hermes Agent configuration, and operational preferences.

## ⚡ Quick Start


Install `uv` with Homebrew:

```sh
brew install uv
```

Clone the project, sync the environment, and install the required Ansible collections:

```sh
git clone https://github.com/dbrennand/hermes-on-hetzner.git
cd hermes-on-hetzner
uv sync
uv run ansible-galaxy collection install -r collections/requirements.yml -p ./collections
```

Edit [`vars/hetzner.yml`](vars/hetzner.yml) and [`vars/hermes.yml`](vars/hermes.yml) for your target server and Hermes configuration, then export your Hetzner Cloud API token and run the full workflow:

> [!TIP]
> Need a Hetzner Cloud API token? Follow Hetzner's guide for [generating an API token](https://docs.hetzner.com/cloud/api/getting-started/generating-api-token/).

```sh
export HCLOUD_TOKEN="your-token"
uv run ansible-playbook playbook.yml
```

## 📋 Requirements

- [`uv`](https://docs.astral.sh/uv/)
- Python 3.13
- A Hetzner Cloud API token exported as `HCLOUD_TOKEN`

## ⚙️ Configuration

### Hetzner variables

Edit [`vars/hetzner.yml`](vars/hetzner.yml) to set the Hetzner Cloud VPS details, bootstrap username, timezone, and SSH public key.

> [!NOTE]
> The Hetzner `user_data` payload is only applied when the server is created or rebuilt. Changing `hcloud_server_username`, `hcloud_server_timezone`, `hcloud_ssh_public_key`, or other variables inside `hcloud_server_user_data` will not update an already existing server on a normal rerun.

### Hermes variables

Edit [`vars/hermes.yml`](vars/hermes.yml) to configure the Hermes Agent version, uv version, extra OS packages and other settings.

| Variable                     | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hermes_agent_custom_skills` | Controls optional Hermes skills installed with `hermes skills install`. Define a list of objects with a required `url` pointing to a raw `SKILL.md` file and an optional `name` to install the skill under a custom name. Only install skills you trust. Leave it empty if you do not want Ansible to install any custom skills.                                                                                                                  |
| `hermes_agent_env_block`     | Manages a marked block inside `~/.hermes/.env` for your own `KEY=value` settings, such as API keys, model configuration, or other runtime overrides that should be applied during deployment. If this contains sensitive values, encrypt them with `ansible-vault` or another secret-management mechanism instead of storing them in plaintext. Leave it empty if you want Hermes Agent to use its existing or default environment configuration. |
| `hermes_agent_npm_packages`  | Controls optional npm packages installed for Hermes Agent to use. Define a list of package objects with `name`, `enabled`, and `version`, for example `@googleworkspace/cli` at `0.22.5`. Set `enabled: true` to install a package, or leave entries disabled if you do not want them installed.                                                                                                                                                  |
| `hermes_agent_soul_content`  | Manages the contents of `~/.hermes/SOUL.md` when you want to define Hermes Agent's identity, behavior, or operating instructions from this repo. Leave it empty if you want Hermes Agent to use the default behavior or keep any existing `SOUL.md` unchanged.                                                                                                                                                                                    |

## 📘 Playbooks

### [`playbook.yml`](playbook.yml)

This playbook triggers the playbooks below in sequence.

### [`playbooks/hetzner-deploy.yml`](playbooks/hetzner-deploy.yml)

This playbook deploys a Hetzner Cloud VPS.

At a high level it performs the following steps:

- Resolves your current public IP with [ipify](https://www.ipify.org/) and creates the SSH firewall rule for port `22`.
- Creates the VPS with `cloud-init` `user_data` to bootstrap the non-root sudo user, disable password SSH auth, disable root login, update and upgrade packages, and set the server timezone.

### [`playbooks/hermes-agent-deploy.yml`](playbooks/hermes-agent-deploy.yml)

This playbook deploys Hermes Agent onto the Hetzner Cloud VPS and can install optional extra tooling and skills for Hermes Agent to use.

At a high level it performs the following steps:

- Waits for SSH login as the configured user and installs the required system dependencies with Ansible tasks.
- Downloads the upstream Hermes installer as a file, executes it with `--skip-setup` using the Ansible `command` module, and optionally manages `~/.hermes/.env` and `~/.hermes/SOUL.md`.
- Installs any enabled packages from `hermes_agent_npm_packages`, such as `@googleworkspace/cli`, with the `community.general.npm` module.
- Installs any entries listed in `hermes_agent_custom_skills` with `hermes skills install <url> --force --yes` and appends `--name <name>` when a custom skill name is provided. Only install skills you trust.

### 🔧 Run Individual Playbooks

```sh
uv run ansible-playbook playbooks/hetzner-deploy.yml
uv run ansible-playbook playbooks/hermes-agent-deploy.yml
```

## AI Attribution

AIA HAb SeCeNc Hin R Codex (gpt-5.4) v1.0
