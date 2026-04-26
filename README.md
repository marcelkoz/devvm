
# Dev VM Automation

## Overview

This repository contains Ansible playbooks for provisioning and configuring Arch Linux development VMs. The automation uses a two-phase deployment model that separates low-level system installation from higher-level system configuration.

## Architecture

Two-phase deployment:

- **Phase 1 (Bootstrap)**: Disk partitioning, filesystem creation, base system installation, and bootloader configuration from the Arch ISO.
- **Phase 2 (Arch)**: User setup, desktop environment installation, and software configuration post-reboot.

## Playbook Components

### Core Playbooks

- **`site.yml`**: Master playbook orchestrating deployment via tags.
- **`mirrors.yml`**: Pacman mirror selection (runs automatically during bootstrap).


### Phase 1: Bootstrap

Boot into the Arch ISO and set up root access with SSH:

```bash
passwd root
systemctl enable --now sshd
```

Deploy your SSH public key:

```bash
scp ~/.ssh/{KEY_NAME} root@{ARCH_IP}:/root/.ssh/authorized_keys
```

Run the bootstrap phase:

```bash
ANSIBLE_ARGS='-i hosts.yml'
ansible-playbook $ANSIBLE_ARGS site.yml --tags bootstrap
```

This performs disk partitioning, filesystem setup, base system installation, bootloader configuration, and mirror selection. Reboot after completion to enter phase 2.


## Phase 2: Arch

Update `hosts.yml` with the new IP address obtained by the machine and run the configuration phase:

```bash
ANSIBLE_ARGS='-i hosts.yml'
ansible-playbook $ANSIBLE_ARGS site.yml --tags arch
```

To skip AUR packages:

```bash
ansible-playbook $ANSIBLE_ARGS site.yml --tags arch --skip-tags aur
```

## Configuration

Modify `hosts.yml` and role variable files in `roles/*/vars/` to customize disk layout, user accounts, network configuration, and package selection. Template files in `roles/*/templates/` render configurations based on these variables.
