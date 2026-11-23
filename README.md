# Ansible-Users

Minimal role that does exactly three things:

1. Creates user accounts (`adduser` equivalent, with home dir and bash shell).
2. Installs their SSH public key(s) into `~/.ssh/authorized_keys`.
3. Grants passwordless sudo to the `sudo` group via `/etc/sudoers.d/91-sudo-users`
   (same pattern as AWS cloud-init's `90-cloud-init-users`).

Nothing else: no password management, no user removal, no dotfiles.

## Requirements

The `ansible.posix` collection (for the `authorized_key` module):

    ansible-galaxy collection install ansible.posix

## Variables

| Variable | Default | Description |
|---|---|---|
| `users` | `[]` | List of user dicts (see below). |
| `users_sudo_nopasswd` | `true` | Install `/etc/sudoers.d/91-sudo-users` with `%sudo ALL=(ALL) NOPASSWD:ALL`. |

Each entry in `users`:

| Key | Required | Description |
|---|---|---|
| `name` | yes | Username. |
| `ssh_key` | no | Public key string; multiple keys can be separated by newlines. |
| `groups` | no | Extra groups (created if missing). Put `sudo` here to grant sudo. |

## Example (host_vars/myhost.yml)

```yaml
users:
  - name: alice
    ssh_key: "ssh-ed25519 AAAAC3Nza... alice@laptop"
    groups: [sudo]
  - name: bob
    ssh_key: "ssh-rsa AAAAB3Nza... bob@desktop"
    groups: []

users_sudo_nopasswd: true
```

## Example playbook

```yaml
- hosts: all
  become: true
  roles:
    - Ansible-Users
```

## Notes

- Sudo access is controlled per-user by membership in the `sudo` group
  (via the `groups` key); the `91-sudo-users` file only changes *how*
  the sudo group works (NOPASSWD), not *who* is in it.
- `groups` uses `append: true`, so the role never removes a user from
  groups it doesn't know about.
- The sudoers file is validated with `visudo -cf` before being installed,
  so a syntax error can never lock you out of sudo.
