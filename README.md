# Android Patch Builder

Private GitHub Actions project for personal Morphe builds.

## Current stage: manual APK input

This first workflow builds one selected app when you manually provide a direct download URL for a clean APK/APKM/XAPK/APKS file. It then:
1. Selects the configured patch repository.
2. Prints the compatible app versions reported by that patch source.
3. Runs Morphe Desktop CLI using that patch source.
4. Signs with your persistent keystore.
5. Verifies the output APK signature.
6. Publishes a GitHub Release only after a successful build.

Apps:
- YouTube — official Morphe patches
- Reddit — official Morphe patches
- Google Photos — Doom's Morphe patches
- Instagram — HushGram

## One-time setup: persistent signing key

A persistent signing key allows future builds to update the same patched app without a signing-certificate mismatch.

Create a keystore on your computer (Java's `keytool` is included with a JDK):

```bash
keytool -genkeypair -v -keystore android-patch-builder.jks -alias android-patch-builder -keyalg RSA -keysize 2048 -validity 10000
```

Remember the passwords and alias. Convert the keystore to Base64:

- Linux/macOS: `base64 -w 0 android-patch-builder.jks`
- Windows PowerShell: `[Convert]::ToBase64String([IO.File]::ReadAllBytes("android-patch-builder.jks"))`

In this repository go to **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret name | Value |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | Base64 text of your JKS file |
| `ANDROID_KEYSTORE_PASSWORD` | Keystore password |
| `ANDROID_KEY_ALIAS` | `android-patch-builder` (or the alias you chose) |
| `ANDROID_KEY_PASSWORD` | Key password |

Never commit the keystore or any of these values.

## Run a build

1. Open **Actions** in this repository.
2. Select **Build patched APK**.
3. Choose the app.
4. Paste a **direct download URL** for the clean APK/APKM/XAPK/APKS file. A webpage URL is not a direct download.
5. Run the workflow.
6. Open the run log and read the **Compatible APK versions** step. Check that the APK you supplied matches a version listed there.
7. If the workflow succeeds, download the APK from the generated **Releases** entry.

The first run will fail early if signing secrets are missing. It will not publish a release after a failed patch or failed signature verification.

## Patch sources

- Official Morphe patches: https://github.com/MorpheApp/morphe-patches
- Doom's Morphe patches: https://github.com/rushiranpise/morphe-patches
- HushGram: https://github.com/SysAdminDoc/HushGram

Morphe Desktop CLI documentation: https://github.com/MorpheApp/morphe-desktop/blob/main/docs/documentation.md

## Current limitations

- APK selection and download URL are manual. Automated APK version discovery and download adapters are not included yet.
- This workflow uses the latest stable Morphe Desktop release and the selected source's latest stable patch bundle.
- It uses patch defaults. App-specific option profiles will be added only after their current option schemas have been inspected.
- A patch source may stop supporting an older APK version. Always compare the supplied APK's version with the compatible versions printed in the run log.
- HushGram's published changelog currently documents specific Instagram builds and a minimum Morphe Desktop version; check its latest release notes before selecting an Instagram APK.
- Keep this repository private. Modified third-party APKs may be subject to copyright, licence and app terms. Use the generated builds only where permitted.
- Re-signed apps generally cannot update the Play Store-installed original in place. Back up app data before uninstalling an original app.

## Troubleshooting

Open the failed Actions run and inspect the first failed step. For help, share the non-secret log, app name and input APK version. Never share signing secrets or the keystore.
