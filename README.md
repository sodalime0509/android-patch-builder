# Android Patch Builder

Private GitHub Actions project for personal Morphe builds.

## Automatic update builder

The repository now has a second workflow: **Automatic patched APK updates** (`.github/workflows/auto-build.yml`).

- Runs every 6 hours and can also be started manually from **Actions**.
- Checks YouTube, Reddit, Google Photos and Instagram independently.
- Asks each configured patch source for its compatible app versions and selects the newest version returned.
- Downloads that exact version using APKPure via apkeep.
- Applies the latest stable patch release, signs with the existing persistent keystore and verifies the APK signature.
- Publishes one release per app only when that app-version/patch-version combination has not already been published.
- A failed app job does not stop the other app jobs. Failed or unverified builds are never released.

### Download source note

The apkeep downloader does **not** support APKMirror. The current automated workflow therefore uses APKPure (apkeep's supported provider). APKMirror scraping/API integration is not included yet. If APKPure cannot provide the exact compatible version, that app's job fails safely rather than silently building a different version.

Split APK bundles are not merged by this first automatic workflow. If a source returns only a split bundle instead of a single APK, the job will fail rather than publish an incomplete package.

### Obtainium

Keep this repository private and use your existing GitHub token in Obtainium. Create or edit one GitHub Releases source per app, all pointing to:

`https://github.com/sodalime0509/android-patch-builder/releases`

Set the release tag/title filter to the matching prefix:

| App | Tag filter |
|---|---|
| YouTube | `^youtube-` |
| Reddit | `^reddit-` |
| Google Photos | `^photos-` |
| Instagram | `^instagram-` |

Use the APK asset from the matching release. The APK is signed with the same keystore on every run, so updates from an earlier build of this builder retain the same signing identity. Obtainium must be configured to use release date as its version (or disable version comparison) because release tags include both app and patch versions.

## Existing manual workflow

The original **Build patched APK** workflow remains unchanged. It still accepts a manually supplied direct APK/APKM/XAPK/APKS URL.

## Signing secrets

The automatic workflow uses the same four repository Actions secrets already used by the manual workflow:

- `ANDROID_KEYSTORE_BASE64`
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_ALIAS`
- `ANDROID_KEY_PASSWORD`

Never commit or share the keystore or these secret values.

## Patch sources

- YouTube and Reddit: https://github.com/MorpheApp/morphe-patches
- Google Photos: https://github.com/rushiranpise/morphe-patches
- Instagram: https://github.com/SysAdminDoc/HushGram

Morphe Desktop CLI documentation: https://github.com/MorpheApp/morphe-desktop/blob/main/docs/documentation.md

## Important limitations and safety

- The automated workflow has been committed but must complete a successful Actions run for each app before treating that app as fully validated.
- APKPure may lag behind the latest release or may not provide the exact compatible version. The workflow will fail closed in that case.
- Experimental patch releases are not intentionally selected; only non-prerelease patch releases are used for release identity.
- Re-signed apps generally cannot update the Play Store-installed original in place. The installed app must already be signed by this builder's keystore. Back up app data before changing signing identities or uninstalling an original app.
- Modified third-party APKs may be subject to copyright, licence and app terms. Use generated builds only where permitted.

## Troubleshooting

Open the failed Actions run and inspect the first failed step. For help, share the non-secret log, app name and input APK version. Never share signing secrets or the keystore.
