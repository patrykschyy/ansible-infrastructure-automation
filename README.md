# Ansible Infrastructure Automation

Ansible playbooks for managing Linux services on a VPS and a local KVM/K3s lab. The repository covers routine system administration, service configuration, and VM provisioning.

Hostnames, IP addresses, user accounts, and other environment-specific values in this repository are examples. Actual credentials and local configuration are kept separately.

## Repository structure

```text
ansible.cfg               Ansible configuration
requirements.yml          Required collections
inventories/dev/
  hosts.yml               Hosts and groups
  group_vars/             Variables for Linux and KVM hosts
playbooks/                Playbooks for administration and deployment
roles/                    Roles for KVM and K3s
templates/                Jinja2 templates for services and dotfiles
files/                    MOTD and example SSH public key
```

## Managed components

| Component | Playbooks / roles |
| --- | --- |
| Linux | Updates, basic packages, timezone, MOTD, server information |
| Docker | Installation, daemon logging, n8n with Docker Compose |
| Networking | Tailscale, UFW, SSH configuration and security checks |
| Nginx | Reverse proxy and TLS configuration for n8n |
| KVM | Hypervisor setup, storage pools, cloud-init VM provisioning |
| K3s | Common node configuration, server and worker installation |
| Shell | Zsh, Oh My Zsh, Vim and dotfiles |

## Inventory and variables

The default inventory is `inventories/dev/hosts.yml` (configured in `ansible.cfg`). It defines the following groups:

- `linux` — Linux server and services
- `kvm_hosts` — KVM hypervisor
- `k3s_master` — K3s server node
- `k3s_workers` — K3s worker nodes
- `k3s_cluster` — Combined K3s node group

Edit the inventory and the files in `inventories/dev/group_vars/` for the target environment. Check SSH users, keys, host addresses, paths, ports, and service domains before running a playbook.

## Setup

Install Ansible and the collections declared in `requirements.yml` on the control machine:

```bash
ansible-galaxy collection install -r requirements.yml
```

Run these commands from the repository root:

```bash
# Inspect inventory
ansible-inventory -i inventories/dev/hosts.yml --graph

# Check connectivity to configured Linux hosts
ansible -i inventories/dev/hosts.yml linux -m ansible.builtin.ping

# Read system information
ansible-playbook -i inventories/dev/hosts.yml playbooks/server-info.yml

# Install basic packages (requires sudo on target hosts)
ansible-playbook -i inventories/dev/hosts.yml playbooks/install-basic-tools.yml --ask-become-pass
```

## Other playbooks

```bash
# Linux maintenance
ansible-playbook playbooks/update.yml --ask-become-pass

# Docker and n8n
ansible-playbook playbooks/install-docker.yml --ask-become-pass
ansible-playbook playbooks/deploy-n8n.yml --ask-become-pass
ansible-playbook playbooks/deploy-nginx-n8n.yml --ask-become-pass

# KVM / K3s lab
ansible-playbook playbooks/setup-kvm-host.yml --ask-become-pass
ansible-playbook playbooks/create-k8s-vms.yml --ask-become-pass
ansible-playbook playbooks/install-k3s-cluster.yml --ask-become-pass
```

These are example commands. Configure inventory and variables first; some playbooks require additional preparation on the target machine.

## Service notes

### n8n and Nginx

- n8n uses Docker Compose. Deployment files are rendered from templates in `templates/`.
- Create the target host's `.env` file separately before running `deploy-n8n.yml`. `templates/n8n.env.j2` lists the expected values; do not commit real secrets.
- Use a strong `N8N_ENCRYPTION_KEY` and a pinned `N8N_VERSION`.
- Configure the domain and certificate paths in `inventories/dev/group_vars/linux.yml`. `deploy-nginx-n8n.yml` expects the TLS certificate to exist already.

### SSH and UFW

- Review SSH access and firewall rules before applying changes on a remote host.
- `secure-ssh-tailscale-only.yml` restricts SSH to Tailscale and may disconnect the current session. Confirm a working Tailscale route and console/recovery access first.
- The playbook requires an explicit `-e confirm_tailscale_ssh_restriction=true`. Do not use that flag until access has been verified.

### KVM and K3s

- VM definitions and storage settings are in `inventories/dev/group_vars/kvm_hosts.yml`.
- Replace `files/ssh/id_ed25519.pub` with the public key intended for the provisioned VMs.
- The KVM and K3s roles are used for the local lab. Check OS images, network settings, resource limits, and K3s installation options when adapting them to another environment.

## Checks after changes

```bash
ansible-inventory -i inventories/dev/hosts.yml --graph

for playbook in playbooks/*.yml; do
  ansible-playbook -i inventories/dev/hosts.yml --syntax-check "$playbook" || exit 1
done

ansible-lint --project-dir . playbooks/ roles/
```

Inventory parsing, syntax checks, and linting can be run without connecting to the target systems. They do not replace testing playbooks on the relevant hosts.
