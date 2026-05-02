# Hermes on Hetzner

🚀 Provision a Hetzner Cloud VPS for [Hermes Agent](https://hermes-agent.nousresearch.com/docs) with Ansible.

## 📋 Requirements

- [`uv`](https://docs.astral.sh/uv/)
- Python 3.13
- A Hetzner Cloud API token exported as `HCLOUD_TOKEN`

## 🛠️ Setup

1. Install the Python environment:

```sh
uv sync
```

2. Install the required Ansible collection:

```sh
uv run ansible-galaxy collection install -r collections/requirements.yml -p ./collections
```

## ⚙️ Configuration

Edit [`vars/hetzner.yml`](vars/hetzner.yml) and set the server details and your SSH public key.

The playbook resolves your current public IP at runtime using [ipify](https://www.ipify.org/) and uses it
to create a firewall rule allowing SSH on port 22 only from that IP as `/32`.

## ☁️ Provision

```sh
export HCLOUD_TOKEN="Your Token"
uv run ansible-playbook playbooks/provision.yml
```
