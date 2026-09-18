# geovoya-macos-build

A build recipe that produces unsigned macOS installers for
[Geovoya](https://geovoya.com) from its published GPL-3.0 source tarball.

## What this repo is

* One GitHub Actions workflow (`.github/workflows/build-macos.yml`) and this README.
* A reproducible recipe: it downloads a published release tarball, verifies its
  SHA-256 against the published `SHA256SUMS.txt`, and only then extracts and builds it
  on GitHub's macOS runners.

## What this repo is not

* It contains **no application code**. Nothing here is part of Geovoya itself.
* It is not where Geovoya is developed, and it holds no signing certificates or secrets.

## Why it exists

GitHub gives public repositories free, unlimited Actions minutes, including on the
macOS runners. Building the Mac installers here costs nothing, while the same jobs in a
private repository require a paid plan. This repository exists purely so that the
macOS `.dmg` / `.zip` artifacts can be produced at no cost. The artifacts are then
published from the Geovoya release process, not from here.

## Where the source comes from

Each build fetches, for the version you ask for:

* `https://downloads.geovoya.com/releases/<version>/geovoya-<version>-source.tar.gz`
* `https://downloads.geovoya.com/releases/<version>/SHA256SUMS.txt`

The tarball's SHA-256 is compared with the entry in `SHA256SUMS.txt` **before** it is
extracted. A mismatch, or a missing entry, fails the job immediately; there is no
option to skip this check. The build then runs entirely inside the extracted tree:
`npm ci`, `npm run runtime:prepare`, `npm test`, and
`npm run dist -- --publish never --<arch>`.

## Licensing

Geovoya is free software licensed under **GPL-3.0-or-later**, and its complete
corresponding source for every release is published at the URL above; the `LICENSE`
file inside the tarball is the authoritative licence for the application. The workflow
in *this* repository is a trivial build script and is released under the MIT licence
(see [`LICENSE`](LICENSE)).

## Running a build

Builds are manual only (`workflow_dispatch`):

```sh
gh workflow run build-macos.yml -f version=0.2.1
gh run watch
```

Two jobs run in parallel, one per architecture:

| Runner | Target |
| --- | --- |
| `macos-15` | `darwin-arm64` (Apple silicon) |
| `macos-15-intel` | `darwin-x64` (Intel) |

Each produces an artifact named `Geovoya-<version>-mac-<arch>` containing the `.dmg`
and `.zip` for that architecture. Download them with:

```sh
gh run download <run-id>
```

If a job fails, a `diagnostics-mac-<arch>-<attempt>` artifact with the runtime
provenance files and the effective electron-builder config is uploaded as well.

## The artifacts are unsigned

These builds run with `CSC_IDENTITY_AUTO_DISCOVERY=false` and no Apple Developer ID
certificate, so the apps are **neither signed nor notarised**. macOS Gatekeeper will
refuse to open them on a first double-click; a user must explicitly allow the app
(right-click → Open, or System Settings → Privacy & Security). Signing and notarisation,
if they happen at all, happen outside this repository.
