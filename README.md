# Ansible Infrastructure Automation

An infrastructure automation project based on **Ansible** for Linux system administration, container services, networking, basic security controls, and a small virtualization lab.

Originally created as a hands-on administration project. This public version has been **sanitized for portfolio review**: host addresses, domains, accounts, home-directory paths, and the personal SSH public key have been replaced with demonstration values. The original environment is not included.

## Scope

| Area | What the repository contains |
| --- | --- |
| Linux administration | Package installation, updates, time zone, system information and MOTD |
| Docker | Docker installation, daemon log configuration, n8n via Docker Compose |
| Network & security | Tailscale installation, UFW firewall configuration, SSH hardening, system security checks |
| Reverse proxy | Nginx configuration templates for n8n webhooks and TLS termination |
| Virtualization | KVM/QEMU/libvirt host, storage pools, VM provisioning with cloud-init |
| Kubernetes lab | K3s master and worker installation, kernel and network configuration |
| Developer environment | Shell/Zsh and Vim dotfiles |

## Repository structure

```text
.
├── ansible.cfg
├── inventories/dev/
│   ├── hosts.yml                 # Example inventory (not real infrastructure)
│   └── group_vars/
│       ├── kvm_hosts.yml        # VM host settings
│       └── linux.yml            # Linux group variables
├── playbooks/                   # Administration and deployment playbooks
├── roles/                       # KVM and K3s roles
├── templates/                   # Jinja2 / Docker Compose / Nginx templates
├── files/                       # MOTD and placeholder SSH public key
└── requirements.yml             # Required Ansible collections
```

## Prerequisites and safety

> **Example inventory only.** This repository is intended to be reviewed and adapted, not executed against unknown machines without an environment-specific test. It contains real administration automation, including potentially disruptive changes.

1. Install Ansible in your environment and the required collections:

   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

2. Configure `inventories/dev/hosts.yml` with your own reachable hosts and SSH accounts. Configure `inventories/dev/group_vars/linux.yml` and `inventories/dev/group_vars/kvm_hosts.yml` for your own infrastructure.

3. Before provisioning VMs, replace `files/ssh/id_ed25519.pub` with **your own public SSH key**. The repository only includes a non-functional placeholder. Before deploying n8n, provide a real `.env` on the target host (mode `0600`), using `templates/n8n.env.j2` as an example; use a strong encryption key and a tested, pinned n8n version. The deployment refuses a missing `.env` or a placeholder/`latest` version.

4. Example commands:

   ```bash
   ansible-inventory --graph
   ansible linux -m ping
   ansible-playbook playbooks/server-info.yml
   ansible-playbook playbooks/install-basic-tools.yml --ask-become-pass
   ```

## Security and scope notes

- No actual SSH private keys, cloud tokens, live `.env` files, or real system credentials are intentionally included.
- `n8n.example.com` is an example domain; the public IPs in the sample inventory are documentation-only values. The example Tailscale address must also be replaced.
- `templates/n8n.env.j2` contains placeholders only. Supply a strong encryption key and other credentials through your own secure configuration before using it.
- The n8n Compose example uses the standard secure-cookie behavior; accessing its editor should use HTTPS. Public reverse-proxy routes must be reviewed for each deployment.
- The SSH firewall playbook can **lock you out** if misconfigured. It requires an explicit `-e confirm_tailscale_ssh_restriction=true` after verifying working Tailscale access and out-of-band recovery. That flag is a safety check, **not a guarantee** against lockout.
- KVM and K3s components document a virtualization and Kubernetes lab. The K3s installer is obtained from an external endpoint without a pinned checksum/version and is **not production-hardened**. Adapting these roles for production requires environment-specific versioning, hardening and recovery planning.
- The public version uses example inventory values; it has not been redeployed against live targets following sanitization. Validate changes on isolated hosts before production use.

## Validation before use

Run the following from the repository root on a control node with Ansible and the collections installed:

```bash
ansible-inventory -i inventories/dev/hosts.yml --list
for playbook in playbooks/*.yml; do ansible-playbook --syntax-check "$playbook" || exit 1; done
ansible-lint --project-dir . playbooks/ roles/
```

These commands are suggested checks, not claims that they have been run successfully. `--syntax-check` does **not** validate runtime safety or idempotency. Use an isolated test VM and `--check --diff` when supported before deploying changes to any real host.

## Main tools

Ansible · Linux (Debian/Ubuntu) · Docker / Docker Compose · Tailscale · UFW · Nginx · SSH · KVM/QEMU · cloud-init · K3s · Jinja2
