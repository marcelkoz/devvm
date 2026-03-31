
# Dev VM Automation

This repo contains ansible playbooks to automate the provisioning of a development Arch VM.

### Templates
The directory `templates` contains config file templates.

### Bootstrap

Within the arch live environment set a password for the root user and ensure ssh server is running.

```bash
passwd root
systemctl enable --now sshd
```

Transfer the public ssh key to the server.

`scp ~/.ssh/{KEY_NAME} root@{ARCH_IP}:/root/.ssh/authorized_keys`

Then run ansible playbooks.

```bash
ANSIBLE_ARGS='-i hosts.yml -e@secrets.yml'
ansible-playbook $ANSIBLE_ARGS site.yml --tags bootstrap
```
