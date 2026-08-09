# ChapelTech package repositories

This repository builds the ChapelTech package repositories served from
`https://repo.chapeltech.uk`.

The current package set is:

* prefork
* lnetd
* knc
* kharon
* krb5_admin
* krb5_keytab

Debian packages are published for Debian 13 (Trixie) on `amd64` and `arm64`.
RPM packages are published for EL9 on `x86_64`. Alpine 3.24 packages are
published for `x86_64` and `aarch64`. Debian and RPM package payloads remain
in the projects' GitHub releases. Alpine packages and their signed indexes are
published together in repository snapshot releases in this repository.

Publication requires every stable latest release to contain `amd64` and
`arm64` Debian packages, signed `x86_64` RPM packages, and signed `x86_64`
and `aarch64` Alpine packages. The fetch step fails closed while any of those
release prerequisites are unavailable.

This repository also owns the reusable package-release workflow used by each
project. It provides both selected-release source and current packaging to the
package-specific commands in `packaging/ci/build-debian` and
`packaging/ci/build-rpm`, and `packaging/ci/build-alpine`; their release workflow
only selects a release tag and calls `.github/workflows/release-packages.yml`
here. The shared workflow builds on native `amd64` and `arm64` runners, signs
RPM and Alpine packages, uploads release assets, and dispatches repository
publication.

Producer repositories must expose `ALPINE_SIGNING_KEY`, `ARCHIVE_SIGNING_KEY`,
`ARCHIVE_SIGNING_PASSPHRASE`, and `PKGS_DISPATCH_TOKEN` to the called workflow,
and define `ARCHIVE_SIGNING_FINGERPRINT`. A missing dispatch token is reported
after package assets have been built and uploaded rather than blocking builds.

## Build

The build requires `apk`, `apt-ftparchive`, `createrepo_c`, `curl`,
`dpkg-scanpackages`, `gh`, `gpg`, `jq`, `openssl`, `rpm`, and `rpmkeys`.

```sh
scripts/fetch-releases artifacts
scripts/build-repositories artifacts public alpine-release
scripts/validate-output public alpine-release
```

`ARCHIVE_SIGNING_SUBKEY_FINGERPRINT` must identify the available OpenPGP
signing subkey and `ARCHIVE_SIGNING_PASSPHRASE` must unlock it. The primary
secret key must not be present in the build keyring. `ALPINE_SIGNING_KEY` must
contain the RSA private key used for APK package and index signatures. See
[SIGNING.md](SIGNING.md) for the environment-secret setup.

## Alpine clients

Install the repository key and add the Alpine 3.24 repository:

```sh
wget -O /etc/apk/keys/chapeltech-alpine.rsa.pub \
	https://repo.chapeltech.uk/keys/chapeltech-alpine.rsa.pub
echo https://repo.chapeltech.uk/alpine/v3.24/main \
	>>/etc/apk/repositories
apk update
```

The generated `public` directory is committed to the `repository` branch by
GitHub Actions. Netlify's `published` branch contains only permanent HTTP 302
rules. Repository metadata and configuration redirect to the raw `repository`
branch, Debian and RPM payloads redirect to their project releases, and Alpine
requests redirect to the latest repository snapshot release. Package
publication never changes the `published` branch and therefore does not cause
a Netlify deployment.

## Tests

`tests/static` and `tests/fetch-releases` run without package assets.
`tests/build-repositories` creates disposable packages and a protected signing
subkey in a clean Trixie container, then exercises the complete build and
validation path.
