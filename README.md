
# Dev VM Automation

This repository contains ansible playbooks to automate the configuration of an Arch VM for development and testing purposes. The process consists of two phases as shown below:

* `bootstrap`: Configuration when the live Arch ISO is plugged in. This installs a minimal Arch system that can support the second phase. Instructions on how to setup the environment for ansible access are contained below.
* `arch`: Configuration of a functional Arch system that has been installed. This includes setting up a user, installing a desktop environment and configuring software and OS components.

There is also the `mirrors.yml` playbook which selects the best pacman package mirrors based on predefined criteria. This is run automatically during the `bootstrap` phase.


### Templates
The directory `templates` contains config file templates. The playbooks require host information which the templates contain the structure for.


### Phase 1: Bootstrap

Within the live Arch ISO environment, set a password for the root user and ensure ssh server is running.

```bash
passwd root
systemctl enable --now sshd
```

Transfer the public ssh key to the server.

```bash
scp ~/.ssh/{KEY_NAME} root@{ARCH_IP}:/root/.ssh/authorized_keys
```

Then run ansible playbooks.

```bash
ANSIBLE_ARGS='-i hosts.yml'
ansible-playbook $ANSIBLE_ARGS site.yml --tags bootstrap
```


### Phase 2: Arch

Once the Arch linux system has been bootstrapped and rebooted, it should be ready for phase 2. The `bootstrap` phase retains the ssh public key provided for phase 1 so the only detail to change is the `hosts.yml` file with the updated IP address of the system. Once that is done, run the ansible playbooks.

```bash
ANSIBLE_ARGS='-i hosts.yml'
ansible-playbook $ANSIBLE_ARGS site.yml --tags arch
```

Alternatively, the playbooks can be run without installing [AUR](https://aur.archlinux.org) packages. Note this still fetches the yay git repository but no packages are installed (including yay).

```bash
ansible-playbook $ANSIBLE_ARGS site.yml --tags arch --skip-tags aur
```
