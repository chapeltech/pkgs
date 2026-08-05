# Repository signing

The ChapelTech package repositories use this OpenPGP identity:

```text
ChapelTech Package Repository <repo@chapeltech.uk>
```

The offline certifying primary key is:

```text
DBDE D585 C6A4 CC17 9E45 77BE 80B0 DB25 BE16 C7EF
```

It was created on 2026-08-05 and expires on 2036-08-02.

The automated signing subkey is:

```text
4A29 65F1 4BC8 AFDD 0E16 0543 07AE E50E F54E 2DF2
```

It expires on 2029-08-04. Rotate the signing subkey before that date and
update the signing environment without changing the client trust anchor.

APT trusts signed `InRelease` metadata. EL9 clients verify both embedded RPM
signatures and the detached signature on `repodata/repomd.xml`.

## GitHub Actions setup

The `publish` job uses the `release-signing` GitHub Actions environment. Protect
that environment with required reviewers and restrict it to the publication
branch before adding these environment-scoped values:

* Secret `ARCHIVE_SIGNING_KEY`: ASCII-armored secret signing-subkey export.
* Secret `ARCHIVE_SIGNING_PASSPHRASE`: passphrase for that subkey.
* Variable `ARCHIVE_SIGNING_SUBKEY_FINGERPRINT`: full 40-hex-character signing
  subkey fingerprint, without spaces.

Create the secret export on the offline primary-key system:

```sh
gpg --batch --armor --export-secret-subkeys '4A2965F14BC8AFDD0E16054307AEE50EF54E2DF2!' \
  >archive-signing-subkey.asc
```

The export contains the public primary key, a primary-secret-key stub, and the
protected signing subkey. It must not contain usable primary secret-key
material. Store the export only in the protected Actions secret, securely
remove the temporary file after configuring the secret, and never add it to
this repository.

The workflow imports the export into an ephemeral keyring. Before publication,
`scripts/check-signing-key` rejects a usable primary secret key, requires the
exact signing subkey, and proves that the passphrase can unlock it.
