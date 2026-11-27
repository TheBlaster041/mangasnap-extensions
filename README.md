# MangaSnap extensions

Signed `.msext` bundles for MangaSnap's definition-driven sources. This repository holds releases and
their documentation only; the source of each extension lives in its own repository, scaffolded from
`mangasnap-extension-template` (private).

A bundle is a signed zip: a manifest, one or more source definitions, and optionally a compiled Kotlin
backend. MangaSnap downloads it, verifies the signature and digests, unpacks it under its own
`filesDir`, and runs it in the app's `:extensions` process. There is no launcher icon, system installer,
package to uninstall, or external service: installing or removing a bundle is an app-local operation.

## Installing

Open **Profile → Extensions** (or **Browse as → Manage extensions**) in MangaSnap and choose a source.
A release build accepts only bundles signed by the first-party bundle key listed in the app's
`app/src/main/res/raw/trusted_extension_signers.txt`; see [`SIGNING.md`](SIGNING.md). An untrusted
signer is shown with its computed fingerprint so it can be added deliberately. The screen also offers
**Install from file** for a locally built `.msext`.

During the migration the index still carries old transition APK URLs for phones running the previous
app. Those APKs are service-only and remain signed with the MangaSnap release key. They are no longer
the destination: new installs and upgrades use bundles, and the APK carrier is removed from the app
once both sites have been proven.

## Releases

One release per extension version:

- tag `<source-id>-v<versionName>` (for example `ehentai-v1.9.2`)
- asset `<source-id>-<versionName>.msext` (for example `ehentai-1.9.2.msext`)
- release notes list the source id, `versionCode`/`versionName`, the SDK version it was built
  against, and any definition change that alters what the app receives.

Bundles are **not** committed to git history — attach them to the release. `.gitignore` enforces this
so a build output cannot be committed by accident.

## Extensions

| Source id | Name | Domain | Release |
|---|---|---|---|
| `ehentai` | E-Hentai | e-hentai.org | [ehentai-v1.9.2](https://github.com/mangasnap/mangasnap-extensions/releases/tag/ehentai-v1.9.2) |
| `hentaifox` | HentaiFox | hentaifox.com | [hentaifox-v1.18](https://github.com/mangasnap/mangasnap-extensions/releases/tag/hentaifox-v1.18) |

The app ships no site code, so these bundles are the only thing that provides each site: install one
and it appears in Browse, remove it and it is gone. A Kotlin backend is compiled into `backend.dex`
and signed with the same bundle; definitions alone need no code.

## In-app download (`extensions.json`)

`extensions.json` is the version-2 index MangaSnap's **Extensions** screen reads (Profile → Extensions,
or the "Browse as" sheet → Manage extensions). Each entry carries:

- `bundleUrl`, `bundleSha256`, `bundleSizeBytes` for the signed `.msext` the app installs;
- `contractVersion` and `minHostVersion` before the app will bind it;
- `sourceIds` for the stable `scraper://` identities;
- `apkUrl`, `apkSha256`, `apkSizeBytes` only for the transition APK fallback;
- legacy `sha256`/`sizeBytes` pointing at that APK so the previous app release still verifies it.

The app verifies the bundle digest before it verifies the signature and unpacks nothing on a mismatch.
The index and the release assets are public, so a phone fetches both without credentials. The default
URL the app ships with is the raw file:

    https://raw.githubusercontent.com/mangasnap/mangasnap-extensions/main/extensions.json

The `pages` workflow publishes the same repository root at the Pages URL on every push to `main`;
the raw URL is what the app defaults to. The screen also offers **Install from file**, and its list URL
is editable, so a mirror of this index works without an app update.

## Why the app is separate

- `MangaSnap` — app, nhentai API, library, reader, downloads, the bundle installer and client.
- `mangasnap-extension-sdk` — the versioned contract, bundle format, in-app host and extension client.
- `mangasnap-extension-template` — the per-site repository template.
- this repository — the built artifacts.

Only this repository is public, because an installed app must fetch the index and the bundles without
a token. The app, the SDK and each extension's source stay private.
