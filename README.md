# Ansible Infrastructure Automation

A collection of Ansible playbooks and roles I built to automate recurring Linux administration tasks, service deployments and infrastructure setup.

The project grew out of my own environments: Linux services hosted on a VPS and a local lab for virtualization and K3s. I used Ansible to make configuration and maintenance tasks repeatable instead of performing them manually on each system.

## What's included

| Area | Examples |
| --- | --- |
| Linux administration | Updates, package installation, timezone, system information and MOTD |
| Docker | Installation, logging configuration and n8n deployment with Docker Compose |
| Networking and security | Tailscale, UFW, SSH configuration and basic security checks |
| Web services | Nginx reverse proxy and TLS configuration for n8n |
| Virtualization | KVM/QEMU, libvirt storage pools and VM provisioning with cloud-init |
| Kubernetes lab | K3s server and worker installation and basic node configuration |
| Shell setup | Zsh, Oh My Zsh and Vim configuration |

## Repository layout

```text
ansible.cfg              Ansible settings
inventories/dev/         Example inventory and group variables
playbooks/               Administration and deployment playbooks
roles/                   KVM and K3s roles
templates/               Jinja2 configuration templates
files/                   Static files and example SSH public key
requirements.yml         Required Ansible collections
```

## Getting started

Install the required collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

Update `inventories/dev/hosts.yml` and the files under `inventories/dev/group_vars/` with settings for your own environment. The published inventory contains example values, not reachable hosts.

To inspect the inventory and run a simple playbook:

```bash
ansible-inventory -i inventories/dev/hosts.yml --graph
ansible-playbook -i inventories/dev/hosts.yml playbooks/server-info.yml
```

## Validation

The original playbooks were used and tested in my VPS and local lab environments. For this public version, I replaced infrastructure-specific values and made code-quality fixes.

The published code has passed these local checks:

- Inventory parsing with `ansible-inventory`
- Syntax checks for all 22 playbooks
- `ansible-lint`: 0 violations, `production` profile

The updated public version has not been redeployed end-to-end. Syntax and lint checks do not replace testing against a target system.

## Before running the playbooks

- Replace example hosts, domains, user accounts and SSH public key values with your own configuration.
- Supply n8n credentials and its encryption key outside Git. Review the Docker Compose and reverse-proxy configuration for your environment.
- Be careful with SSH and firewall playbooks: incorrect settings can lock you out of a server. In particular, `secure-ssh-tailscale-only.yml` requires explicit confirmation and a verified recovery path.
- The KVM and K3s roles reflect lab work; adapt and test their versions, security settings and recovery procedures before production use.

## Technologies

Ansible · Debian/Ubuntu · Docker / Docker Compose · Nginx · Tailscale · UFW · SSH · KVM/QEMU · libvirt · cloud-init · K3s · Jinja2
