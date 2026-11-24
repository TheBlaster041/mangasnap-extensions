# Extension signing

Extensions are trusted by certificate, not by package name, so one key signs every extension the app
will bind. This is the same key that signs the app itself.

| | |
|---|---|
| Key alias | `MANGASNAP_RELEASE_KEY_ALIAS` (not committed) |
| Algorithm | RSA 2048, SHA-256 |
| Fingerprint | printed by the command below once the keystore exists |

```bash
keytool -list -v -keystore release.keystore -alias <alias> | grep SHA256
```

Add the value to the app's allowlist in `app/src/main/res/raw/trusted_extension_signers.txt`. Debug
builds of the app bypass the check; release builds do not.

The keystore itself, `keystore.properties` and the passwords never enter this repository. The
publish workflow takes them from repository secrets.
