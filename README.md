# Factorio Ansible Package

This package sets up a Fedora-based headless Factorio server roughly matching the configuration we worked through:

- non-root `factorio` service user
- binaries under `/opt/factorio`
- mutable data under `/var/lib/factorio`
- config under `/etc/factorio`
- systemd unit with sandboxing
- firewalld rule for UDP 34197 in the chosen zone
- optional Tailscale install + trusted `tailscale0`
- optional mod management
- startup wrapper that can boot the newest `_autosave*.zip`

## Layout

- `playbooks/site.yml` — example playbook
- `roles/factorio_server` — reusable role

## Prerequisites

Install Ansible collections (needed for `ansible.posix.firewalld`):

```bash
ansible-galaxy collection install -r requirements.yml
```

## Quick start

Inventory example:

```ini
[factorio_servers]
game-server ansible_host=<ansible_host> ansible_user=<ansible_user>
```

Basic run:

```bash
ansible-playbook -i inventory.ini playbooks/site.yml \
  -e factorio_archive_src=files/factorio_headless_x64_2.0.73.tar.xz
```

## Important variables

A few you will probably want to override:

```yaml
factorio_archive_src: files/factorio_headless_x64_2.0.73.tar.xz
factorio_firewalld_zone: FedoraServer
factorio_server_name: Factorio Server
factorio_server_description: Private Factorio server
factorio_game_password: "set-a-real-password"
factorio_admins:
  - your_factorio_username
factorio_enable_tailscale: true
factorio_manage_mods: true
factorio_mod_zip_files:
  - files/mods/AutoDeconstruct_1.0.10.zip
  - files/mods/BottleneckLite_1.3.4.zip
  - files/mods/flib_0.16.5.zip
  - files/mods/mining-patch-planner_1.7.15.zip
  - files/mods/RateCalculator_3.3.8.zip
  - files/mods/squeak-through-2_0.1.4.zip
factorio_mods:
  - { name: base, enabled: true }
  - { name: squeak-through-2, enabled: true }
  - { name: AutoDeconstruct, enabled: true }
  - { name: elevated-rails, enabled: true }
  - { name: flib, enabled: true }
  - { name: mining-patch-planner, enabled: true }
  - { name: quality, enabled: true }
  - { name: BottleneckLite, enabled: true }
  - { name: RateCalculator, enabled: true }
  - { name: space-age, enabled: true }
```

## Notes

- The role assumes you already have the headless Factorio tarball on the Ansible control node.
- Tailscale installation is handled, but authentication still needs an interactive `tailscale up` or an auth key strategy.
- SELinux relabeling is included because moving binaries/data around on Fedora often needs `restorecon`.
- The startup wrapper uses the latest autosave if present, otherwise it falls back to `world.zip`.

## Example group vars

Put this in `group_vars/factorio_servers.yml` if you want a cleaner inventory-driven setup:

```yaml
factorio_archive_src: files/factorio_headless_x64_2.0.73.tar.xz
factorio_enable_tailscale: true
factorio_manage_mods: true
factorio_game_password: "replace-me"
factorio_admins:
  - your_factorio_username
```
