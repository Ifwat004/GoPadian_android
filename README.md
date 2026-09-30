# GoPadian for Android

This folder is a complete Android app project. It wraps the GoPadian prototype (`app/src/main/assets/www/index.html`) in a native app with the GoPadian launcher icon. Everything runs offline except the live map, which needs internet and falls back to the river sketch without it.

## Option A: get the APK with GitHub (nothing to install)

1. Sign in at github.com and create a new repository, e.g. `gopadian-android`.
2. Click **uploading an existing file** and drag in everything inside this folder, including the hidden `.github` folder (on a Mac, press Cmd + Shift + . in Finder to show it). Click **Commit changes**.
3. Open the **Actions** tab. "Build APK" starts by itself and takes about 5 minutes.
4. When it turns green, open this link on your Android phone (replace the two names):
   `https://github.com/YOUR-NAME/gopadian-android/releases/latest/download/GoPadian.apk`
   If the repository is private, sign in to GitHub on the phone first.
5. Open the downloaded file. Android asks you to allow "Install unknown apps" for your browser the first time. Allow it, then tap **Install**.

If the Actions tab says "Get started with GitHub Actions", the `.github` folder was not uploaded. Click **set up a workflow yourself**, paste the workflow at the bottom of this file, and commit.

## Option B: build it in Android Studio

Open this folder in Android Studio and wait for Gradle sync to finish. Then choose **Build > Build App Bundle(s) / APK(s) > Build APK(s)**. The APK is saved to `app/build/outputs/apk/debug/app-debug.apk`.

## Updating the app

Export a new offline copy of the prototype, save it as `app/src/main/assets/www/index.html`, and upload it to the repository. A new build appears on the Releases page and installs over the old one. If Android refuses the update, uninstall GoPadian from the phone and install the new APK.

## Notes

- The app is debug-signed. That is fine for testing and demos. Publishing on Google Play needs a release signing key and an `.aab` file, which is not set up yet.
- Package id `bn.gopadian.app`. Runs on Android 7.0 and newer.
- The phone's back button follows the in-app back arrows. On a main screen it sends the app to the background, so the demo keeps its place.
- The bar at the bottom holds the demo controls: Passenger/Driver switch, jump to screen, fast-forward, weather alert and reset.

## Workflow (copy of `.github/workflows/build-apk.yml`)

```yaml
name: Build APK

on:
  push:
    branches: [main, master]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      # Keep one debug signing key between builds so new APKs install over old ones.
      - uses: actions/cache@v4
        with:
          path: ~/.android/debug.keystore
          key: gopadian-debug-keystore

      - name: Build
        run: gradle assembleDebug --no-daemon

      - name: Rename
        run: cp app/build/outputs/apk/debug/app-debug.apk GoPadian.apk

      - uses: actions/upload-artifact@v4
        with:
          name: GoPadian-apk
          path: GoPadian.apk

      - name: Publish to Releases
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          gh release create "build-${{ github.run_number }}" GoPadian.apk \
            --repo "$GITHUB_REPOSITORY" \
            --target "$GITHUB_SHA" \
            --title "GoPadian build ${{ github.run_number }}" \
            --notes "Open this page on an Android phone and tap GoPadian.apk to install."
```
