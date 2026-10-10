# QLMarkdown Personal

Personal build based on upstream QLMarkdown 1.5.7 (`3a0cdcd`). This branch is intended for use on the account owner's Mac. It is not a notarized distribution release or a confirmed fix for all extension-registration failures.

## Differences from upstream

- Uses Apple Development signing with Personal Team `BS66N84A2F`.
- Uses bundle prefix `dev.danielgreenlaw.qlmarkdown` and shared app group `BS66N84A2F.dev.danielgreenlaw.qlmarkdown`. Settings are separate from the original app and the earlier `.test` build.
- Names the app and Quick Look provider “QLMarkdown Personal” and “QLMarkdown Personal Preview”.
- Omits `com.apple.security.inherit` from the main app. The app retains its sandbox and shared-group entitlements. This configuration worked in an earlier development build, but the entitlement change alone has not been proven to explain the upstream registration failure.
- Restores the saved base-font-size checkbox and value when loading settings. Previously the app saved a value such as 18 but reopened with the control disabled at 12; another save could then remove the override.
- Uses the existing filename-based Markdown UTType lookup on all supported systems, allowing compilation with the installed SDK 26.5.
- Does not start Sparkle and removes the upstream update feed from the app. Update this build by rebuilding the fork.
- Adds a `QLMarkdown Personal` scheme that uses Release for building/running.

No automatic `pluginkit -a` or explicit registration API call is added.

## Build

1. Clone this fork and check out `personal-build`. Initialize the pinned submodules with `git submodule update --init --recursive`.
2. Install the build tools if missing: `brew install autoconf automake libtool cmake`.
3. Open `QLMarkdown.xcodeproj` in Xcode. Sign in to the account owning the configured Personal Team and use an existing Apple Development certificate. Other developers need to replace the team, bundle identifiers, and team-prefixed shared-group identifier consistently in all targets and `Settings.swift`.
4. Select **QLMarkdown Personal → My Mac**, then **Product → Build**. Allow Xcode to resolve Swift packages.
5. Locate the built `QLMarkdown.app` through Xcode's Products group, **Show in Finder**. Copy the complete app to `/Applications/QLMarkdown Personal.app`. Renaming the outer app folder leaves its signed contents intact. Keep the original app separate.
6. Open the installed personal app once. In System Settings → General → Login Items & Extensions → Quick Look, enable **QLMarkdown Personal Preview**. Disable other Markdown preview providers while checking which provider renders your file.

macOS team-prefixed app groups do not require a provisioning profile; `REGISTER_APP_GROUPS` is off. See [Apple's app-group container documentation](https://developer.apple.com/documentation/xcode/accessing-app-group-containers). This build uses a development certificate for local use, not Developer ID signing/notarization for distribution. If the certificate expires or is revoked, rebuild with a valid certificate.

## Check the installed build

- Open a Markdown file in Finder and press Space; verify formatted content, including a table or fenced code block.
- Set Base font size to 18, save, quit and reopen. Confirm the checkbox and 18 both remain. Save an unrelated option, reopen again, and confirm 18 remains.
- Disable the font override, save and reopen. Confirm it stays disabled. Re-enable 18 if desired.
- Verify registration before and after opening the installed app with this read-only command:

  ```sh
  /usr/bin/pluginkit -m -A -D -v -i dev.danielgreenlaw.qlmarkdown.QLExtension
  ```

  Confirm the result points inside `/Applications/QLMarkdown Personal.app`. A leading `-` means registered but disabled; no output and a successful command exit means no matching registration. A connection error is inconclusive.

A successful Xcode build/discovery workflow does not reproduce a fresh Homebrew installation of the upstream Developer ID release. Finder invocation and settings persistence should be checked separately from build/signing success.

## Verification on 2026-10-10

Xcode 26.6 / SDK 26.5 successfully built both Debug and the personal Release scheme. Release contains arm64 and x86_64 executables for the app and both extensions. Xcode signed them with the configured Apple Development identity. The first Release attempt hit an internal Xcode build-service error; retry succeeded with existing C/C++ warnings.

The installed `/Applications/QLMarkdown Personal.app` matches the Release product across 628 files/symlinks. The original application was preserved. Embedded CodeDirectory code hashes and XML-entitlement hashes match for both architectures in the app and both extensions, all with team `BS66N84A2F` and the personal shared group. This is an integrity check, not certificate-chain/Gatekeeper validation; the restricted automation shell could not complete a system trust check. Apple Development signing also injects `get-task-allow`, even with Release optimization; this is a local development build.

The account owner confirmed that Base font size remains checked at 18 after saving, quitting and reopening the installed app. Finder-provider invocation and registration checks are recorded separately when confirmed.
