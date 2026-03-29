
# Dev VM Automation

This repo contains ansible playbooks to automate the provisioning of a development Arch VM.

### Templates
The directory `templates` contains config file templates.

### Bootstrap

Within the arch live environment set a password for the root user.

`passwd root`

Then run ansible playbooks.

```bash
ANSIBLE_ARGS='-i hosts.yml -e@secrets.yml'
ansible-playbook $ANSIBLE_ARGS site.yml
```
