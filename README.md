# MangaSnap extensions

Signed release APKs for MangaSnap's definition-driven sources. This repository holds releases and
their documentation only; the source of each extension lives in its own repository, scaffolded from
`mangasnap-extension-template` (private).

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

| Source id | Name | Domain | Release |
|---|---|---|---|
| `ehentai` | E-Hentai | e-hentai.org | [ehentai-v1.0.0](https://github.com/mangasnap/mangasnap-extensions/releases/tag/ehentai-v1.0.0) |

The app ships no site code, so this APK is the only thing that provides `ehentai`: install it and the
site appears in Browse, uninstall it and the site is gone.

## In-app download (`extensions.json`)

`extensions.json` is the index MangaSnap's **Extensions** screen reads (Profile → Extensions, or the
"Browse as" sheet → Manage extensions). Each entry carries the release asset URL, its `sizeBytes`
and the APK's `sha256`; the app verifies that digest before it hands the file to the system
installer, so a truncated or replaced download installs nothing.

Add an entry when a release is published, and bump `versionCode` in the extension repo at the same
time — Android rejects an install that does not raise it.

The index and the release assets are public, so a phone fetches both without credentials: this is
the default URL the app ships with. The screen also offers **Install from file**, and its list URL
is editable, so a mirror of this index works without an app update.

## Why the app is separate

- `MangaSnap` — app, nhentai API, library, reader, downloads, the extension client.
- `mangasnap-extension-sdk` — the versioned contract both sides build against.
- `mangasnap-extension-template` — the per-site repository template.
- this repository — the built artifacts.

Only this repository is public, because an installed app must fetch the index and the APKs without a
token. The app, the SDK and each extension's source stay private.
