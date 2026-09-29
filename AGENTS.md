# AGENTS.md — pod-sshd

Standalone candy repo for the `sshd` candy — the OpenSSH server with key-based
login, a `sshd-wrapper` launcher, and a NOPASSWD sudoers rule for the uid-1000
account. The candy lives in `charly.yml` at the repo root plus its wrapper.

Canonical files:

- `charly.yml` — the `sshd:` candy entity (description, `env_accept`, `package`,
  `distro`, `port`, `service`, `plan`) and its `skill:` entity.
- `sshd-wrapper` — the launcher copied to `/usr/local/bin`.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-coder:sshd` — the owning skill: the candy properties, the cross-distro
  `package_map:` pattern, and the dual-mode sudo check. Load before editing,
  building, deploying, or troubleshooting this candy.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `copy:` / `check:`, per-distro sections, service
  declarations).
- `/charly-check:check` — the check/R10 framework (`charly check box`,
  `charly check run <bed>`).
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; ports, services).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert `/usr/sbin/sshd`, the `sshd-wrapper`
  launcher, the resolved `openssh-server` package (via `package_map:`), and the
  effective NOPASSWD sudo grant; at deploy scope they assert the running `sshd`
  service and a reachable published port.

## Modify this repo

- Edit the `sshd:` candy entity in `charly.yml`; the `skill:` entity in the same
  file is the owning skill's source — a candy change and its skill change land
  together.
- The sudoers rule targets the **actual uid-1000 account** via
  `getent passwd 1000`, never a literal `user`; keep the create-mode/adopt-mode
  parity intact.
- The per-distro package names diverge (`openssh-server` on Fedora/Debian,
  `openssh` on Arch) — a package change must update the `distro:` sections and
  the `package_map:` in the package `check:` together.
- The `skill:` entity is the source for `/charly-coder:sshd`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
