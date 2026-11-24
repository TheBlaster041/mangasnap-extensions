# MangaSnap extensions

Signed release APKs for MangaSnap's definition-driven sources. This repository holds releases and
their documentation only; the source of each extension lives in its own repository, scaffolded from
[`mangasnap-extension-template`](https://github.com/TheBlaster041/mangasnap-extension-template).

An extension APK is service-only: no launcher icon, no UI, no storage. MangaSnap binds its service,
asks it in JSON, and streams page images into files the app owns. Installing an extension never
changes the app itself — discovery happens on next launch.

## Installing

```bash
adb install -r <source-id>-v<versionName>-<versionCode>.apk
```

The app refuses to bind an extension whose signing certificate is not in
`app/src/main/res/raw/trusted_extension_signers.txt`. Every APK here is signed with the MangaSnap
release key; the fingerprint is in [`SIGNING.md`](SIGNING.md). An untrusted certificate is shown in
the extension manager as `Untrusted signer <sha256>` so it can be added deliberately.

Upgrading: the APK must have a higher `versionCode` than the installed one, or Android rejects the
install. Definitions without a version bump cannot reach a device.

## Releases

One release per extension version:

- tag `<source-id>-v<versionName>` (for example `ehentai-v1.0.0`)
- asset `<source-id>-v<versionName>-<versionCode>.apk`
- release notes list the source id, `versionCode`/`versionName`, the SDK version it was built
  against, and any definition change that alters what the app receives.

APKs are **not** committed to git history — attach them to the release. `.gitignore` enforces this
so a build output cannot be committed by accident.

## Extensions

| Source id | Name | Domain | Status |
|---|---|---|---|
| `ehentai` | E-Hentai | e-hentai.org | not released yet |

`e-hentai` is the first extension packaged from the template (Phase 4 of the app's `PLAN.md`). Until
it ships, MangaSnap can only use the template's demo source, which exists to prove the transport.

## Why the app is separate

- `MangaSnap` — app, nhentai API, library, reader, downloads, the extension client.
- `mangasnap-extension-sdk` — the versioned contract both sides build against.
- `mangasnap-extension-template` — the per-site repository template.
- this repository — the built artifacts.

All four are private.
