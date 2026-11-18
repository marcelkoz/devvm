
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
- **`pb_update_mirrors.yml`**: Pacman mirror selection (runs automatically during bootstrap).
- **`pb_arch_backup.yml`**: Adhoc backup of select directories to the control node.


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
ansible-playbook site.yml --tags bootstrap
```

This performs disk partitioning, filesystem setup, base system installation, bootloader configuration, and mirror selection. Reboot after completion to enter phase 2.


## Phase 2: Arch

Update `hosts.yml` with the new IP address obtained by the machine and run the configuration phase:

```bash
ansible-playbook site.yml
```

If the play stops on a stale pacman lock (`/var/lib/pacman/db.lck`), first make sure no pacman process is running. Then remove the lock explicitly:

```bash
ansible-playbook site.yml --tags arch_pacman_unlock
```

That run removes only the lock file. Rerun the normal phase 2 command afterwards.

## Backups

`pb_arch_backup.yml` archives the directories listed in `arch_backup_source_paths` on the target and fetches the archive to `~/Documents/DevVMBackups/` on the control node.

```bash
ansible-playbook pb_arch_backup.yml --limit <inventory-host>
```

Source paths are resolved from `roles/arch/defaults/main/main.yml`, which this playbook loads directly. An `arch_dev_user_*` override in `hosts.yml` therefore does not change the backup path; pass extra vars instead.

## Configuration

Modify `hosts.yml` and role variable files in `roles/*/defaults/main/` to customize disk layout, user accounts, network configuration, and package selection. Template files in `roles/*/templates/` render configurations based on these variables.
