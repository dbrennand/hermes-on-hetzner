# Hermes on Hetzner

🚀 Provision a Hetzner Cloud VPS for [Hermes Agent](https://hermes-agent.nousresearch.com/docs) with Ansible, then deploy Hermes onto it.

## 📋 Requirements

- [`uv`](https://docs.astral.sh/uv/)
- Python 3.13
- A Hetzner Cloud API token exported as `HCLOUD_TOKEN`

## 🛠️ Setup

1. Setup the Python environment:

    ```sh
    uv sync
    ```

2. Install the required Ansible collection:

    ```sh
    uv run ansible-galaxy collection install -r collections/requirements.yml -p ./collections
    ```

## ⚙️ Configuration

### Hetzner variables

- Edit [`vars/hetzner.yml`](vars/hetzner.yml) to set the server details, bootstrap username, timezone, and SSH public key.

> [!NOTE]
> The Hetzner `user_data` payload is only applied when the server is created or rebuilt. Changing `hcloud_server_username`, `hcloud_server_timezone`, `hcloud_ssh_public_key`, or other variables inside `hcloud_server_user_data` will not update an already existing server on a normal rerun.

### Hermes variables

- Edit [`vars/hermes.yml`](vars/hermes.yml) to set the Hermes branch and any optional runtime configuration you want Ansible to manage.
- `hermes_install_branch` controls which upstream Hermes branch the installer uses.
- `hermes_extra_system_packages` lets you install extra Ubuntu packages before Hermes is installed.
- `hermes_env_block` lets Ansible manage a marked block inside `~/.hermes/.env` for your own `KEY=value` settings while leaving any content outside that block untouched.
- `hermes_soul_content` lets Ansible template `~/.hermes/SOUL.md` when you want to define Hermes's primary identity from this repo. If it is empty, the playbook leaves any existing `SOUL.md` alone.

## 📘 Playbooks

- [`playbook.yml`](playbook.yml) is the full workflow entrypoint. It provisions the Hetzner VPS first and then deploys Hermes Agent onto it.
- [`playbooks/provision.yml`](playbooks/provision.yml) provisions the Hetzner infrastructure. It resolves your current public IP with [ipify](https://www.ipify.org/), creates the SSH firewall rule for port `22`, and creates the VPS with `cloud-init` `user_data` to bootstrap the non-root sudo user, disable password SSH auth, disable root login, update and upgrade packages, and set the server timezone.
- [`playbooks/hermes-deploy.yml`](playbooks/hermes-deploy.yml) deploys Hermes Agent onto an existing provisioned VPS. It waits for SSH login as the configured user, installs the required system dependencies with Ansible tasks, downloads the upstream Hermes installer as a file, executes it with `--skip-setup` using the Ansible `command` module, and optionally manages `~/.hermes/.env` and `~/.hermes/SOUL.md`.

## ☁️ Run The Full Workflow

```sh
export HCLOUD_TOKEN="Your Token"
uv run ansible-playbook playbook.yml
```

## 🔧 Run Individual Playbooks

```sh
uv run ansible-playbook playbooks/provision.yml
uv run ansible-playbook playbooks/hermes-deploy.yml
```

## AI Attribution

AIA HAb SeCeNc Hin R Codex (gpt-5.4) v1.0
