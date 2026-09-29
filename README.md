# direnv

Automatic per-directory environment loading from `.envrc` files.

The `direnv` candy installs the `direnv` binary (one package name across
Fedora/Arch/Debian) and declaratively wires the per-shell hook that makes
`.envrc` autoload work. At image build the `shell:` schema bakes the hook into
`/etc/profile.d/charly-direnv-<shell>.sh` for bash/zsh (the generic body
`eval "$(direnv hook <shell>)"` with `${SHELL_NAME}` substituted) and the
fish-specific hook into `/etc/fish/conf.d/charly-direnv.fish`
(`direnv hook fish | source`). `sh` is intentionally opted out (`sh: {}`):
upstream `direnv hook` supports bash/zsh/fish/tcsh but not `sh`, so `direnv hook
sh` errors on every shell that sources it. The `plan:` checks assert both the
installed/runnable binary and the baked hook drop-ins, so they fail if either the
package or the shell wiring is missing.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `direnv` |
| Binary | `/usr/bin/direnv` |
| Shell hooks | `/etc/profile.d/charly-direnv-bash.sh`, `/etc/profile.d/charly-direnv-zsh.sh` (guarded per shell), `/etc/fish/conf.d/charly-direnv.fish` |
| Service / port | none |

The primary use case in OpenCharly is the `.secrets` workflow: `.envrc` calls
`eval "$(charly secrets gpg env)"`, which decrypts a GPG-encrypted `.secrets`
file in memory and exports the variables — no plaintext on disk.

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list:

```yaml
my-dev-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-direnv:v2026.267.2235'
```

Then, inside the built image:

```bash
direnv version                       # 2.x
ls /etc/profile.d/charly-direnv-*.sh # bash + zsh drop-ins
ls /etc/fish/conf.d/charly-direnv.fish
```

`sh` has no drop-in by design — `test ! -e /etc/profile.d/charly-direnv-sh.sh`
must succeed.

## Layout

- `charly.yml` — the `direnv:` candy entity (the package, the `shell:` block,
  the `check:` assertions) and the embedded `direnv-skill:` skill entity.
- `CHANGELOG/` — per-CalVer release notes.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-coder:direnv`
- `.secrets` workflow: `/charly-build:secrets`
- Composition: `/charly-distros:agent-forwarding` (gnupg + direnv + ssh-client)
- Also available in: `/charly-coder:dev-tools`
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
