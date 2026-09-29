# pod-sshd

The `sshd` candy of the OpenCharly candy library, as a standalone repo (the
candy de-submodule cutover, kind-prefixed naming). It provides an OpenSSH server
with key-based login and passwordless sudo for the uid-1000 account.

## What it provides

Installs the OpenSSH server (`openssh-server`, or `openssh` on Arch) plus the
`sudo` package, publishes the SSH port, copies a `sshd-wrapper` launcher into
`/usr/local/bin`, and drops a NOPASSWD sudoers rule for the build's uid-1000
account at `/etc/sudoers.d/charly-user`.

| Property | Value |
|---|---|
| Port | `tcp:2222` |
| Service | `sshd` (`/usr/local/bin/sshd-wrapper`, `restart: always`, priority 3) |
| Env (accepted) | `SSH_AUTHORIZED_KEYS` — public key(s) for authorized access |
| Packages | `openssh-server` + `openssh-clients` (RPM/deb), `openssh` (pac), `sudo` |

The sudoers drop-in targets the **actual uid-1000 account**, whatever it is named
on the running base — discovered at build time via `getent passwd 1000` — so the
same candy works under both create mode (`user`) and adopt mode (`ubuntu`).

Every claim is verifiable: the daemon binary, the wrapper script, the installed
package, the effective sudo grant, and — at deploy scope — the running `sshd`
service and the reachable published port.

## How to use it

```yaml
my-image:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/pod-sshd:<tag>'
```

Set the authorized key(s) at deploy time:

```bash
charly config my-image --ssh-key auto
```

## Verification

The candy's `check:` plan asserts `/usr/sbin/sshd`, the `sshd-wrapper` launcher,
the resolved `openssh-server` package (via `package_map:`), the effective
NOPASSWD sudo grant, and — at deploy scope — the running `sshd` service and a
reachable `127.0.0.1:${HOST_PORT:2222}`.

## Layout

- `charly.yml` — the `sshd:` candy entity (description, `env_accept`, `package`,
  per-distro sections, `port`, `service`, `plan`) plus its `skill:` entity.
- `sshd-wrapper` — the launcher copied to `/usr/local/bin`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-coder:sshd` — the candy properties, the cross-distro
  `package_map:` pattern, and the dual-mode sudo check.
- `/charly-distros:cloud-init` — depends on sshd for VM provisioning.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
