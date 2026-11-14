# AGENTS.md

Operating guide for agents working in this repository.

## Purpose

This repository provisions and configures an Arch Linux workstation for programming
and IT work. It is a two-phase Ansible deployment: bootstrap from the Arch ISO, then
configure the installed system.

Agents change this repository. Do not hand-edit files on a provisioned machine and
then copy them back; change the role variable, template, or task that renders them.

## Hard constraints

- **Target OS is Arch Linux only.** Do not add Debian, Ubuntu, Fedora, or
  distribution-detection branches unless explicitly asked.
- **Never use the AUR.** No AUR helpers (`yay`, `paru`, `pikaur`, and similar), no
  AUR packages, and no building packages from the AUR. Use official pacman
  repositories, npm global packages, pipx, or flatpak instead. If a tool is only
  available from the AUR, stop and ask.
- **Editor is VS Code OSS** (pacman package `code`). Do not reinstall packages that
  are in an "absent" list variable such as `arch_packages_absent`.
- The desktop is KDE Plasma. Core OS components are systemd based including the
  bootloader, DNS and network interface configuration.
- Every task must be idempotent. A second run must report no changes unless a
  change is genuinely pending.
- **Bootstrap is destructive.** `roles/bootstrap/templates/partition-disk.sh.j2`
  runs `wipefs -a` against the primary disk. Never run `--tags bootstrap` against a
  non-ISO host. The role asserts `ansible_facts['nodename'] == 'archiso'` and UEFI
  mode; do not weaken those guards.

## Repository layout

```text
ansible.cfg                  inventory path, interpreter, tags skip = bootstrap
site.yml                     master playbook: bootstrap phase, then arch phase
pb_*.yml                     standalone ansible playbooks for adhoc tasks
hosts.yml                    real inventory, always in gitignore
hosts.yml.example            inventory template
group_vars/all.yml           shared vars
roles/*                      ansible roles
```

The `arch` role includes `common_bootloader`, `common_network`, then its task files
in order: `general.yml`, `git.yml`, `kde.yml`, `dev.yml`, `ai.yml`,
`migrations.yml`. `bootstrap` includes `common_mirrors`, `common_bootloader`, and
`common_network` with chroot-adjusted paths.

## Commands

Run from the repository root:

```bash
ansible-playbook site.yml --syntax-check
ansible-lint
ansible-playbook site.yml --limit <inventory-host>
ansible-playbook site.yml --tags bootstrap --limit <inventory-host>
ansible-playbook pb_update_mirrors.yml --limit <inventory-host>
```

`ansible.cfg` sets `[tags] skip = bootstrap`, so a plain `site.yml` run performs
phase 2 only. Always pass `--limit` when more than one host could match.

Both `ansible-playbook --syntax-check` and `ansible-lint` must pass before declaring
an Ansible change complete. `--check` is useful for phase 2 review but is not a
substitute for lint. Never claim a change was tested on a live host unless it was.

## Role conventions

- **Variable names are role-prefixed:** `arch_*`, `bootstrap_*`, `common_network_*`,
  `common_bootloader_*`, `common_mirrors_*`. Do not introduce unprefixed role
  variables.
- **Generic knobs live in `roles/<role>/defaults/main/`,** split by domain. Ansible
  loads every file in that directory, so keep names unique across the files. Put
  anything a user may reasonably change here: paths, file modes, package lists,
  policy values, site settings, tool configuration objects. Role defaults are the
  lowest precedence layer, so inventory, group vars and extra vars can override
  them.
- **`roles/<role>/vars/main/` holds values that must win over those overrides:**
  personal identity (developer account, git author, personal repository URLs),
  bootstrap machine layout and idempotency markers, template-to-filename maps, and
  JSON serialization kwargs. Role vars beat inventory vars, so use this directory
  only when overriding from the inventory would be wrong.
- When adding a variable, default to `defaults/main/`. Reach for `vars/main/` only
  for the categories above, and say why in the change description.
- Use fully qualified collection names for modules and lookups, for example
  `ansible.builtin.template` and `community.general.pacman`.
- Split task files by concern and include them from `tasks/main.yml` with
  `ansible.builtin.include_tasks`.
- Developer-facing tasks run as the developer account through
  `args: apply: become_user: '{{ arch_dev_user_name }}'`. Follow that pattern; do
  not switch to `become_user` on individual tasks when the include already sets it.
- Use handlers for service restarts. Name the handler; notify it from the task that
  changes configuration. Do not restart services inline.
- No hardcoded user names, home paths, host names, IP addresses, timezones,
  keymaps, or file modes in tasks. Reference the existing variables.
- File and directory modes come from `arch_standard_file_permissions` and
  `arch_standard_directory_permissions` unless a tighter mode is required.
- Package lists belong in `roles/arch/defaults/main/packages.yml` under the existing
  groups. Add removals to the matching `*_absent` list instead of deleting a
  package silently.
- npm global packages use `NPM_CONFIG_PREFIX` pointing at the user's `.local`.
  pipx and lola modules run as the developer user.

## Templates

- Render configuration through templates; do not copy literal JSON or shell from
  the host back into the repository.
- Add the ansible-managed header where the file format supports comments:
  `{{ custom_ansible_managed | comment('c') }}` for JSON, `| comment` for shell.
  Keep `group_vars/all.yml`'s `custom_ansible_managed` value unchanged.
- Generate JSON with `to_nice_json(**arch_json_config)` so ordering and indentation
  stay consistent.
- Generated files are outputs. Edit the source variable map or template, never the
  rendered file on the target.

## Personal values and secrets

Use placeholders (`<dev-user>`, `<git-user-name>`, `<git-email>`, `<timezone>`,
`<keymap>`, `<inventory-host>`, `<host-ip>`, `<dns-server>`) in documentation,
examples, and comments. The real values stay in ansible variable files, and the gitignored
`hosts.yml`.

`hosts.yml` and `secrets.*` are gitignored. Never commit them, never print their
contents in a report, and never move credentials into tracked files.

## Change workflow

1. Read the relevant role, its vars, and its templates before editing.
2. Prefer changing a variable or template over adding a task.
3. Keep task names descriptive and consistent with surrounding tasks.
4. Run `ansible-playbook site.yml --syntax-check` and `ansible-lint`.
5. Report exactly what was verified and what still needs a live-host run.

Git policy for agents: local commits are allowed with conventional commit messages;
**never push**, never rewrite history, and never commit ignored files. Leave the
working tree clean or explain what remains uncommitted.

## Review checklist

- [ ] No AUR helper or AUR-only package introduced.
- [ ] Target remains Arch Linux; no other distributions added.
- [ ] New tasks are idempotent and use FQCN modules.
- [ ] New variables are role-prefixed and stored in `defaults/main/` unless they are
      role internals, which belong in `vars/main/`.
- [ ] No secrets or real personal values added to tracked files.
- [ ] Service changes use a handler.
- [ ] Templates keep the ansible-managed header and JSON generation pattern.
- [ ] `ansible-playbook site.yml --syntax-check` passes.
- [ ] `ansible-lint` passes for the touched files.
- [ ] Bootstrap tasks untouched unless the task explicitly concerns bootstrap.

## Known gaps

- There is no CI, no Molecule scenario, and no committed lint configuration. Do not
  invent test commands or claim coverage that does not exist.
- `README.md` documents the two phases; `TODO.txt` lists pending work.
- Bootstrap completion markers live under `/tmp/ansible-bootstrap` and do not
  survive a reboot.
- The `arch` role clones the dotfiles repository and runs its symlink script from a
  handler. Changes to dotfiles belong in that repository, not here. Inform the user of
  required dotfile changes.
