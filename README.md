# Hermes on Hetzner

🚀 Provision a Hetzner Cloud VPS for [Hermes Agent](https://hermes-agent.nousresearch.com/docs) with Ansible.

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

Edit [`vars/hetzner.yml`](vars/hetzner.yml) and set the server details, bootstrap username, timezone, and your SSH public key.

The playbook resolves your current public IP at runtime using [ipify](https://www.ipify.org/) and uses it
to create a firewall rule allowing SSH on port 22 only from that IP as `/32`.
It also uses cloud-init `user_data` to create the default sudo user, disable password SSH auth, disable root login, update packages, upgrade packages, and set the server timezone.
The SSH public key is injected directly into that non-root user via cloud-init instead of creating a separate Hetzner SSH key resource.

> [!NOTE]
> The Hetzner `user_data` payload is only applied when the server is created or rebuilt. Changing `hcloud_server_username`, `hcloud_server_timezone`, `hcloud_ssh_public_key`, or other variables inside `hcloud_server_user_data` will not update an already existing server on a normal rerun.

## ☁️ Provision

```sh
export HCLOUD_TOKEN="Your Token"
uv run ansible-playbook playbook.yml
```
