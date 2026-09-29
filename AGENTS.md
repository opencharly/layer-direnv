# AGENTS.md — layer-direnv

Standalone candy repo for the `direnv` layer — the `direnv` binary plus the
declarative per-shell hooks that make `.envrc` autoload work. The candy lives in
`charly.yml` at the repo root: the `package:`, the `shell:` block (bash/zsh
generic + fish override + the `sh: {}` opt-out), the `plan:` `check:`
assertions, and the embedded `skill:` entity projected into the marketplace
corpus as `/charly-coder:direnv`.

Canonical files:

- `charly.yml` — the `direnv:` candy entity and the `direnv-skill:` skill entity.
- `CHANGELOG/` — per-CalVer release history.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-coder:direnv` — the owning skill. The `.envrc`/`.secrets` workflow and
  the per-shell hook model. Load before editing or troubleshooting the layer.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema, the
  `shell:` block, `plan:` step verbs incl. `check:`, package sections). Load
  before editing any entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` `check:` steps are the functional evidence: the
  `/usr/bin/direnv` binary and its version banner, the bash/zsh drop-ins with
  their guard strings, the fish conf.d drop-in, and the negative check that no
  `sh` drop-in exists. They must stay valid on every distro arm they run on.
- The `shell:` block is emitted at image build, so a change to it changes the
  baked `/etc/profile.d/` + `/etc/fish/conf.d/` files the checks assert.

## Modify this repo

- Edit the `direnv:` candy entity AND the `direnv-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a package,
  shell-hook, or behaviour change not mirrored in the skill leaves the corpus
  stale.
- Keep the `sh: {}` opt-out: `direnv hook sh` is not a valid direnv target, so an
  `sh` drop-in would error on every shell that sources it. Each POSIX drop-in
  stays runtime-guarded to its own shell.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
