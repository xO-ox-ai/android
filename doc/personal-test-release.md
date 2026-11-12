<!--
SPDX-FileCopyrightText: 2026 Nextcloud GmbH and Nextcloud contributors
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Personal integration test release

This personal build combines the current upstream master with these unmerged PRs:

| PR | Head commit | Change |
| --- | --- | --- |
| [18004](https://github.com/nextcloud/android/pull/18004) | `39a3af54197edb42e7dd2eabac7fe94430b7940b` | E2EE certificate validation |
| [18005](https://github.com/nextcloud/android/pull/18005) | `578759088f0537eb4bcfbd4f6e396c3dff820ae0` | Settings log export shortcut |
| [18028](https://github.com/nextcloud/android/pull/18028) | `c6c5300b506366d1104c4ae1fbe21cd61f75327d` | E2EE automatic upload |

Upstream master was fetched on 2026-11-12 and resolved to
`bf053a29aa8eb5b1b5cb0fdc79aeceab5f7256a9`. Each PR was merged without rewriting
its commits. This branch does not merge the separate Chinese translation branches.

## Installation and data

The APK uses the existing QA flavor: `com.nextcloud.android.qa`, account type
`nextcloud.qa`, and display name **Nextcloud QA**. It can coexist with the official
app. The release version is `35.2.0-test.20261112.1` / `2026111201`.

The release APK is optimized and signed with a personal signing key. Keep that
key and its password locally for future updates; neither belongs in Git or release
assets. Future builds with the same package and signing key can update this QA
installation and retain its data, provided their version codes permit upgrading.

An official installation can retain its data through an in-place update only when
the new APK has the same package and a compatible signing identity. A personal key
cannot replace the official signing identity. Renaming the package or signing an
APK with another key does not transfer access to the original app's private data.

The app declares `android:allowBackup="false"`; ordinary ADB backup is not a complete
migration method for account tokens, encryption keys, preferences, upload rules,
or Android Keystore entries. Do not uninstall the official app to install this APK.

Sign in separately in QA. Restore access to existing E2EE files using the original
mnemonic in the encryption setup; do not reset the account's encryption keys.
Reconfigure QA automatic uploads and permissions separately. Local cached files
and pending uploads are not automatically transferred. Select a small test source
and destination first to avoid overlapping automatic upload rules in both apps.

## Build and debug

Build the release variant with JDK 21 and Android SDK 37:

```text
gradlew.bat assembleQaRelease -Pksp.incremental=false
```

The output under `app/build/outputs/apk/qa/release/` must be aligned and signed using
Android SDK `zipalign` and `apksigner` before installation. `ksp.incremental=false`
avoids the Windows cross-drive KSP cache failure in this environment.

USB debugging must be enabled and the computer authorized on the phone.

```text
adb devices -l
adb install -r nextcloud-qa-35.2.0-test.20261112.1.apk
adb shell dumpsys package com.nextcloud.android.qa
adb logcat -v threadtime AndroidRuntime:E '*:S'
```

For application logs, use **Settings > Export logs** in QA. Logs can contain
private server and account details; inspect them before sharing.

## Device test checklist

1. Sign in and open existing E2EE folders after entering the original mnemonic.
2. Confirm the original client can still decrypt files created by QA and vice versa.
3. Export logs using the new settings shortcut.
4. Configure automatic upload to an encrypted folder; test direct files, nested
   directories, the selected conflict policy, and retry after interruption.
5. Remove a disposable destination remotely and verify subsequent uploads follow
   its current server state, including ordinary uploads if encryption was removed.
6. Test background uploads with the phone locked and its normal battery settings.

Device and server integration checks require a connected phone and a live server;
building and passing unit tests alone do not establish these results.
