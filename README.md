# Homebrew tap for Xcode Switcher

Formulae and casks for [Xcode Switcher](https://github.com/Monkey0803/xcode-switcher-macos),
a macOS menu bar app for discovering, diagnosing and switching between installed
Xcode versions.

## Install

```bash
brew tap Monkey0803/xcode-switcher
brew trust Monkey0803/xcode-switcher   # Homebrew 6 and later refuse to load untrusted taps

# Prebuilt release — ad-hoc signed and not notarized
brew install --cask --no-quarantine xcode-switcher

# Or build from source; the resulting app carries no quarantine attribute
brew install xcode-switcher
```

## Why the cask needs `--no-quarantine`

Homebrew only distributes macOS apps whose artifacts pass its Gatekeeper checks,
which is why the official `homebrew/cask` accepts notarized apps. Xcode Switcher
is deliberately published ad-hoc signed and not notarized, so:

- this cask lives in this tap instead of `homebrew/cask`, and
- a quarantined copy is blocked by Gatekeeper on first launch. Pass
  `--no-quarantine` to skip that, or approve the app once under
  **System Settings → Privacy & Security** afterwards.

The global shortcut additionally needs Accessibility permission.

## Why the formula needs a recent Xcode

Building from source avoids Gatekeeper entirely, but needs **Xcode 26 or later**:
the app uses macOS 26 APIs (`NSGlassEffectView`), and a runtime `#available`
check cannot help a compiler that cannot see the symbol at all.

The formula is currently only *expected* to work when the active Xcode is 26.
With Xcode 27 the SDK 27 `@State` macro is expanded through `swift-plugin-server`,
which Homebrew's formula build sandbox refuses. See the header comment in
`Formula/xcode-switcher.rb` for the full verification status.

## Bumping after a release

Each release publishes `Xcode-Switcher-<version>-<build>-local.zip` (and `.dmg`)
alongside a `SHA256SUMS` file. Update the cask accordingly:

```ruby
version "1.4.0,2"
sha256 "..."   # from SHA256SUMS, or: curl -sL <zip url> | shasum -a 256
```
