# Helium for AerynOS, managed by moss

This repository packages Helium's signed upstream `x86_64` binary tarball as
an AerynOS `.stone` package. GitHub Actions checks for a stable Helium release
daily, builds against AerynOS `stream/unstable`, and publishes a static moss
repository as assets of the `repository` GitHub Release.

There is no separate Helium updater installed on the AerynOS machine you install this on. Once the package
is installed, the normal system update is the entire client-side update path:

```bash
sudo moss sync --update
```

`sudo moss sync -u` is the equivalent short form

## Repository layout

- `stone.yaml`: `helium-bin` package recipe.
- `monitoring.yaml`: upstream release metadata for Aeryn tooling.
- `.github/workflows/publish-stone.yml`: remote update, verification, build,
  index, and publication pipeline.
- GitHub Release `repository`: `stone.index`, the current `.stone`, its build
  manifest, and checksums.

The AerynOS tools revision is pinned in the workflow. Upstream Helium tarballs
must pass both their recipe SHA-256 check and detached OpenPGP signature check.
The expected Helium signing-key fingerprint is pinned in the workflow.

## One-time install

For the standard imperative moss configuration:

```bash
sudo moss repo add helium \
  https://github.com/fernbacher/helium-aerynos/releases/download/repository/stone.index \
  -p 50 \
  -c "Helium personal Stone repository"

sudo moss install helium-bin
```

Do not use `moss repo add` if `/etc/moss/system-model.kdl` exists. Add the
equivalent repository and package to that file instead:

```kdl
repositories {
    helium {
        description "Helium personal Stone repository"
        uri "https://github.com/fernbacher/helium-aerynos/releases/download/repository/stone.index"
        priority 50
    }
}
packages {
    helium-bin
}
```

Merge those entries into the existing `repositories` and `packages` blocks;
do not create duplicate top-level blocks. Then run `sudo moss sync --update`.

## Normal operation

```bash
sudo moss sync --update
```

The workflow runs remotely; there is no Helium-specific service or timer on
the AerynOS machine. The workflow deliberately uses stable releases only.

## Rollback and removal

AerynOS retains previous moss system states, so the usual boot/state rollback
remains available after a bad package update. To stop consuming this repository:

```bash
sudo moss repo disable helium
sudo moss sync --update
```

To remove Helium and its repository definition entirely:

```bash
sudo moss remove helium-bin
sudo moss repo remove helium
```

The browser profile in `~/.config/net.imput.helium` is not package-owned and
is intentionally left untouched.

## AI-assisted research

AI was used as a research assistant to study AerynOS packaging, `moss`, Boulder, upstream Helium releases, and related repositories. It also helped cross-reference documentation and troubleshoot the publishing workflow.

This project was not assembled by blindly accepting generated code. Every recipe, workflow step, verification measure, and migration procedure was reviewed and tested by the maintainer on a real AerynOS installation. AI assisted the research; the technical decisions, testing, and responsibility remain human.
