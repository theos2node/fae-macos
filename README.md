# Factorio Achievement Enabler for macOS

macOS Apple Silicon patcher for the native Steam build of Factorio `2.0.77` (`mac-arm64`).

It removes Factorio's modded-save achievement gates in the supported binary so modded saves can use the normal local and Steam achievement paths. It does not edit save files.

## Status

- Supported: Factorio `2.0.77` Steam, native macOS ARM64.
- Tested on: macOS Apple Silicon, Steam running, Factorio `2.0.77`.
- Tested mod: `qiMenu 0.2.91`.
- Not supported: Intel macOS, non-Steam builds, other Factorio versions, saves where cheats/console/map-editor usage disabled achievements.

If Factorio updates and the expected instruction bytes change, the patcher fails with a mismatch instead of guessing.

## How It Works

The patcher copies or replaces `factorio.app`, modifies verified ARM64 instructions, and ad-hoc signs the result. The patches target:

- `AchievementStats::allowed`
- `SteamContext::setStat`
- `SteamContext::unlockAchievement`
- `SteamContextInterface::unlockAchievementsThatAreOnSteamButArentActivatedLocally`
- `AchievementGui::updateModdedLabel`
- `PlayerData::PlayerData`

Your saves, mods, settings, and player data remain in Factorio's normal write-data directory:

```text
~/Library/Application Support/factorio
```

## Option A: Patched Copy

This keeps the Steam install untouched and creates a patched copy at:

```text
~/Library/Application Support/fae-macos/factorio-patched.app
```

Install:

```sh
scripts/patch-factorio-app --force
scripts/verify-patched-app
```

Launch through Steam with the patched copy:

```sh
scripts/fae-steam-launch
```

Load a specific save:

```sh
scripts/fae-steam-launch --load-game "/path/to/save.zip"
```

`fae-steam-launch` temporarily points Steam's Factorio app path at the patched copy, lets Steam restart into that patched app, waits while Factorio runs, and restores the original Steam app path afterward. Keep the terminal open while playing.

## Option B: Patch Steam Install

This makes the normal Steam Play button launch the patched app. It is more convenient, but Steam updates or "Verify integrity" can overwrite it.

Patch the installed Steam app:

```sh
scripts/patch-steam-install
```

Restore the original installed app:

```sh
scripts/restore-steam-install
```

`patch-steam-install` moves the original Steam `factorio.app` to a backup, writes a patched app in its place, and verifies the patch. It does not touch save files.

## Verification

Verify patch bytes and code signature:

```sh
scripts/verify-patched-app
```

Verify copied app and Steam app use the same default write-data directory, while normal save ZIP files remain unchanged:

```sh
scripts/verify-default-write-data
```

Create a throwaway modded save under `/tmp`:

```sh
scripts/smoke-test-temp-save
```

Run the Steam-backed `qiMenu` validation:

```sh
scripts/steam-test-qimenu
```

The `qiMenu` test downloads `qiMenu 0.2.91` into a temporary mod directory, creates a throwaway save, launches a Steam-backed patched graphical session, verifies `qiMenu` loaded, verifies a local player exists, and calls Factorio's achievement API for:

- a custom non-Steam helper achievement;
- the already-present vanilla general achievement `so-long-and-thanks-for-all-the-fish`.

It restores the original Steam app path and backed-up Steam remote achievement files afterward.

## Normal Mods

Install and manage mods normally in:

```text
~/Library/Application Support/factorio/mods
```

The patched app and the normal Steam app use the same mod list and save directory. There is no separate profile to sync.

## Limits

This project patches the supported Factorio binary's modded-achievement checks. It cannot guarantee every possible mod is stable, compatible, or safe. A broken mod can still crash the game or corrupt gameplay state.

It also does not scrub a modded save back into a vanilla save. If a world has been played with mods, normal unpatched Factorio still treats that save as modded for achievement purposes.

## Why Not `DYLD_INSERT_LIBRARIES`?

The tested Steam build allows dylib loading, but macOS kills the process when execution reaches modified signed text pages. Patching and ad-hoc signing an app bundle avoids modifying signed text pages at runtime.

## Disclaimer

Use at your own risk. Keep backups of important saves. This is an unofficial community tool and is not affiliated with Wube Software, Factorio, Valve, or Steam.
