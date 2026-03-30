# Ansible - Deploy Apache Container

Ansible playbook to deploy an Apache (`httpd`) Docker container on a remote host with a bind-mounted, read-only `index.html` served from a Jinja2 template.

## Project Structure

```
.
├── deploy.yml              # Main playbook
├── hosts.yml               # Inventory file
├── group_vars/
│   └── prod.yml            # Group variables for prod (uses Ansible Vault)
├── secret/
│   └── cred.yml            # Ansible Vault encrypted credentials
└── templates/
    └── index.html.j2       # Jinja2 template for the web page
```

## Requirements

- Ansible >= 2.10
- `community.docker` collection
- Target host running Ubuntu (Debian-based)
- Ansible Vault password to decrypt `secret/cred.yml`

Install the required collection:

```bash
ansible-galaxy collection install community.docker
```

## Inventory

The inventory defines a single `prod` group with one host:

| Host    | IP              |
|---------|-----------------|
| client1 | 192.168.56.22   |

## Variables

`group_vars/prod.yml` sets SSH connection options and uses a vaulted password:

```yaml
ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
ansible_user: vagrant
ansible_password: "{{ vault_ansible_password }}"
```

`vault_ansible_password` is stored encrypted in `secret/cred.yml` using Ansible Vault.

## What the Playbook Does

1. Updates apt cache and installs prerequisite packages
2. Adds the Docker GPG key and repository
3. Installs Docker CE and related packages
4. Ensures the Docker service is running
5. Removes any existing `apache` container
6. Creates `/opt/apache-web/` directory on the remote host
7. Renders `index.html.j2` and copies it to `/opt/apache-web/index.html`
8. Deploys an `httpd:latest` container named `apache` with:
   - Port `80:80` exposed
   - `/opt/apache-web/index.html` bind-mounted read-only into the container

## Usage

Run the playbook with your Vault password:

```bash
ansible-playbook -i hosts.yml deploy.yml --ask-vault-pass
```

Or using a vault password file:

```bash
ansible-playbook -i hosts.yml deploy.yml --vault-password-file ~/.vault_pass
```

## Vault

Credentials are encrypted with Ansible Vault. To edit:

```bash
ansible-vault edit secret/cred.yml
```

To encrypt a new value:

```bash
ansible-vault encrypt_string 'your_password' --name 'vault_ansible_password'
```
