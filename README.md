# Homebrew tap for Xcode Switcher

Formulae and casks for [Xcode Switcher](https://github.com/Monkey0803/xcode-switcher-macos),
a macOS menu bar app for discovering, diagnosing and switching between installed
Xcode versions.

## Install

```bash
brew tap Monkey0803/xcode-switcher
brew trust Monkey0803/xcode-switcher        # Homebrew 6 and later refuse to load untrusted taps

brew install --cask xcode-switcher          # prebuilt; add --force to replace an unmanaged app
brew install xcode-switcher                 # or build from source (macOS 26 or earlier)
```

The cask tracks the latest release: currently **v2.1.1**. For history, v1.4.0 was
withdrawn — the artifact published on 2026-09-11 crashed on launch because the
`xcodebuild archive` release path signed the bundle in a way that made dyld
reject the embedded `Sparkle.framework` ("mapping process and mapped file
(non-platform) have different Team IDs"), so the process died with SIGABRT. That
is fixed from v1.4.1 onward, and the release scripts now launch the artifact
before publishing rather than only checking it structurally.

## Gatekeeper

The published builds are ad-hoc signed and deliberately not notarized, which is
why the official `homebrew/cask` cannot carry them and they live in this tap.

Homebrew 6 removed `--no-quarantine`, so a fresh install is quarantined and
Gatekeeper blocks the first launch. Approve the app once under
**System Settings → Privacy & Security**, or clear the attribute yourself after
checking the download's `SHA256SUMS`:

```bash
xattr -dr com.apple.quarantine "/Applications/Xcode Switcher.app"
```

Until the app is approved, the bundled `xcodeswitcher` CLI is blocked too:
Gatekeeper kills every executable inside a quarantined bundle, and all you see is
a silent `Killed: 9`.

The global shortcut additionally needs Accessibility permission.

## Formula (build from source)

Building from source avoids Gatekeeper entirely, but needs **Xcode 26 or later**:
the app uses macOS 26 APIs (`NSGlassEffectView`), and a runtime `#available`
check cannot help a compiler that cannot see the symbol at all.

The formula is limited to **macOS 26 (Tahoe) and earlier**. On macOS 27 either
Xcode fails: with Xcode 27 the SDK 27 `@State` macro is expanded through
`swift-plugin-server`, which Homebrew's formula build sandbox refuses; with Xcode
26 Homebrew itself refuses to build, because on macOS 27 it demands Xcode 27.
See the header comment in `Formula/xcode-switcher.rb`.

## Notes for maintainers

Each release publishes `Xcode-Switcher-<version>-<build>-local.zip` (and `.dmg`)
alongside a `SHA256SUMS` file. Update the cask as follows:

```ruby
version "2.1.1,9"
sha256 "..."   # from SHA256SUMS, or: curl -sL <zip url> | shasum -a 256
```

Then **launch the installed app once** before pushing the bump. The v1.4.0
mistake got through because the artifact was only checked structurally: version,
bundled CLI output and `codesign --verify` all passed while the app itself could
not start.

The cask installs the CLI with `command_wrapper` rather than `binary`. A symlink
in `/opt/homebrew/bin` is not equivalent: Foundation derives `Bundle.main` from
the invocation path, so a symlinked CLI misses the app's String Catalog and
silently falls back to the source language.
