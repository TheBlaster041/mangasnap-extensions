# Extension signing

MangaSnap trusts signers by SHA-256 fingerprint, not by package name or file name. During the APK-to-bundle
migration there are two distinct first-party keys:

| Carrier | Key | Where the fingerprint is checked |
|---|---|---|
| Transition extension APK | MangaSnap release key | APK discovery in `ExtensionTrust` |
| App-owned `.msext` bundle | MangaSnap bundle key | `BundleDiscovery` and `BundleHostTrust` in the app |

Both fingerprints are listed in the app's `app/src/main/res/raw/trusted_extension_signers.txt`. Debug builds
skip the checks so locally built carriers can be tested; release builds do not.

## Transition APK key

The APK key is the same key that signs the app itself. Its alias and passwords come from
`MANGASNAP_RELEASE_*` Gradle properties, environment variables, or `keystore.properties`.

| | |
|---|---|
| Key alias | `MANGASNAP_RELEASE_KEY_ALIAS` (not committed) |
| Algorithm | RSA 2048, SHA-256 |
| Fingerprint | `91:77:3F:9C:62:E6:81:14:E2:96:B3:19:93:DB:73:58:AE:E7:55:73:76:EE:AE:ED:36:C7:12:87:52:77:8D:E5` |

```bash
keytool -list -v -keystore release.keystore -alias <alias> | grep SHA256
```

The keystore itself, `keystore.properties` and the passwords never enter this repository.

## Bundle key

Bundles are signed with a separate PKCS#8 RSA private key. The private half lives outside every
repository, currently at `~/.mangasnap/bundle-signing/mangasnap-bundle-signing.pem`; it must never be
committed, attached to a release, or passed on a command line where a process listing can expose it.

| | |
|---|---|
| Algorithm | RSA 2048, SHA-256 |
| Fingerprint | `99:F6:12:37:36:00:5C:D8:53:6D:2E:A5:87:FD:55:D2:26:CF:E3:00:98:8A:0A:9E:75:35:14:67:29:9D:A0:BC` |

Build and verify a bundle from an extension repository with:

```bash
./gradlew :extension:buildBundle :extension:verifyBundle \
  -PMANGASNAP_BUNDLE_SIGNING_KEY_FILE=/absolute/path/to/mangasnap-bundle-signing.pem
```

`MANGASNAP_BUNDLE_SIGNING_KEY_FILE` can also be an environment variable. For CI, put the PEM contents in
`MANGASNAP_BUNDLE_SIGNING_KEY` as a secret instead of writing the key to disk. The public key is embedded in
each signed bundle so the app can verify the signature; the private key is never embedded.
