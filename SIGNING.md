# Extension signing

MangaSnap trusts bundle signers by SHA-256 fingerprint, not by package name or file name. There is one
first-party carrier: the app-owned `.msext` bundle, checked by `BundleVerifier` when it is installed
and by `BundleDiscovery` / `BundleHostTrust` when the host binds it. There is no extension APK and no
system installer.

## Bundle key

Bundles are signed with a PKCS#8 RSA private key separate from the app release key. The private half
lives outside every repository, currently at `~/.mangasnap/bundle-signing/mangasnap-bundle-signing.pem`;
it must never be committed, attached to a release, or passed on a command line where a process listing
can expose it.

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

The app's `app/src/main/res/raw/trusted_extension_signers.txt` lists the fingerprints a release build
may bind. A debug build skips the allowlist so a locally built bundle can be tested; a release build
does not.
