# Factorio Achievement Enabler for macOS

macOS ARM64 patcher for the native Steam build of Factorio `2.0.77` (`mac-arm64`). It creates a separate patched copy of `factorio.app`, applies verified ARM64 instruction patches to that copy, and ad-hoc signs the copied app.

It does not modify your original Factorio install or any save files.

## Support

- macOS on Apple Silicon.
- Factorio `2.0.77` Steam build.
- Native macOS Factorio only.
- Cheats, console commands, and map-editor-disabled achievements are intentionally not a supported recovery target.

If Factorio updates and any instruction bytes change, the patcher stops with a mismatch instead of guessing.

## Install

```sh
scripts/patch-factorio-app --force
```

Default source:

```text
~/Library/Application Support/Steam/steamapps/common/Factorio/factorio.app
```

Default patched copy:

```text
~/Library/Application Support/fae-macos/factorio-patched.app
```

## Run

Keep Steam open, then run:

```sh
scripts/fae-launch
```

To pass Factorio arguments:

```sh
scripts/fae-launch --load-game /path/to/save.zip
```

## Isolated Smoke Test

This creates a temporary patched app copy, a temporary Factorio write-data directory, a tiny test mod, and a new throwaway save under `/tmp`. It does not read or write the normal Factorio saves directory.

```sh
scripts/smoke-test-temp-save
```

## Verification

Verify that the installed patched copy contains every expected ARM64 replacement instruction, while the Steam app still contains the original bytes:

```sh
scripts/verify-patched-app
```

Verify that the copied app uses the same default write-data path as the Steam app and that the normal saves directory is unchanged by the check:

```sh
scripts/verify-default-write-data
```

On the tested machine, the patched copy resolved:

```text
shared write-data: /Users/theo_primary/Library/Application Support/factorio
normal saves unchanged: /Users/theo_primary/Library/Application Support/factorio/saves
```

This means the duplicate app shares the normal Factorio progress/config location by default. The smoke test still uses an isolated temp write-data directory so it never needs the user's real saves.

## Why Not `DYLD_INSERT_LIBRARIES`?

The tested Factorio build allows dylib loading, but macOS kills the process when execution reaches modified signed text pages. Patching and ad-hoc signing a separate app copy avoids modifying the original app while allowing the patched code to run.

## Patch Scope

The patcher currently modifies the ARM64 slice at verified offsets for:

- `AchievementStats::allowed`
- `SteamContext::setStat`
- `SteamContext::unlockAchievement`
- `SteamContextInterface::unlockAchievementsThatAreOnSteamButArentActivatedLocally`
- `AchievementGui::updateModdedLabel`
- `PlayerData::PlayerData`

These patches remove the modded-save gates in the local achievement checks and Steam achievement/stat sync paths for the supported Factorio build. They do not repair saves where achievements were disabled by cheats, console commands, or the map editor.

## Test Boundary

The automated tests prove:

- the patched app launches and runs the supported Factorio build;
- the copied app uses the same default write-data directory as the Steam app;
- the normal save directory is not changed by verification;
- a throwaway modded save can be created under `/tmp`;
- all expected original bytes and patched replacement bytes are present at the known ARM64 offsets.

They intentionally do not force-unlock a new real Steam achievement, because that would permanently affect the user's Steam account. For normal play, keep Steam open and launch the patched app through the same Steam environment you use for Factorio.
