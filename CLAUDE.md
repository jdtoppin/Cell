# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cell is a World of Warcraft raid frame addon by enderneko. This fork adds fixes and enhancements for the WoW 12.0 (Midnight) expansion, particularly around secret value handling and new raid/dungeon debuff data.

## Architecture

- **Core.lua** — Addon initialization, event handling, main namespace (`Cell`, `F`, `I`, `P`)
- **Utils.lua** — Utility functions used throughout
- **RaidFrames/UnitButton.lua** — Main unit button logic including aura processing (`HandleBuff`, `HandleDebuff`, `UnitButton_UpdateAuras`)
- **Indicators/Base.lua** — Indicator rendering system (icons, bars, text overlays on unit frames)
- **Defaults/Indicator_DefaultSpells.lua** — Spell definitions for built-in indicators
- **RaidDebuffs/** — Per-expansion raid debuff definitions (e.g., `RaidDebuffs_Midnight.lua`)
- **Widgets/** — UI widget library
- **Comm/** — Addon communication
- **Modules/** — Feature modules (click casting, raid tools, etc.)

## Key Patterns

- Aura iteration: `ForEachAura` (full update via `GetAuraSlots` + `GetAuraDataBySlot`) and `ForEachAuraCache` (partial update from cached auras)
- `UnitButton_UpdateAuras` handles both full and partial updates (UNIT_AURA updateInfo)
- External cooldowns: matched via `I.IsExternalCooldown(name, spellId, source, unit)`
- Dispels: `dispelName` field on debuff auras, rendered via `self.indicators.dispels:SetDispels()`

## WoW 12.0 Secret Values (Critical)

WoW 12.0 introduced "secret values" that crash on boolean tests:
- `secretVal or 0`, `if secretVal then`, `secretVal and x` all crash
- Safe: `issecretvalue()`, C-level APIs (`SetText`, `SetValue`, `SetVertexColor`, `SetMinMaxValues`)
- `rawequal(x, nil)` is safe for nil checks on potentially-secret values
- Non-dispellable debuffs: `dispelName = nil`; dispellable: `dispelName = SECRET`
- Use `issecretvalue(aura.dispelName)` to detect dispellable vs non-dispellable
- `CooldownFrame:SetCooldownFromDurationObject(durObj, clearIfZero)` for secret-safe cooldown display
- `AbbreviateNumbers(value)` is C-level and accepts secrets
- `SetFormattedText` with secrets produces invisible output — don't use for duration text

## Packaging

```bash
# External libs cached at /Users/josiahtoppin/Documents/Projects/Cell-external-libs/
# Required: LibStub, CallbackHandler-1.0, AceComm-3.0, LibSerialize, LibCustomGlow-1.0, LibSharedMedia-3.0, LibDeflate
mkdir -p /private/tmp/Cell-release-build
git archive HEAD | tar -x -C /private/tmp/Cell-release-build/Cell
cp -R /Users/josiahtoppin/Documents/Projects/Cell-external-libs/* /private/tmp/Cell-release-build/Cell/Libs/
cd /private/tmp/Cell-release-build && zip -r <tag>.zip Cell/
```

- Libs go into `Libs/` subdirectory (NOT addon root)
- When merging to master, use `git merge --ff-only` to avoid duplicate merge commits

## Branches

- `master` — main branch
- `jdtoppin-patch-1` — WoW 12.0 secret value fixes

## Midnight Expansion Data

- Released: March 2, 2026
- Raids: The Voidspire (1307, 6 bosses), March on Quel'Danas (1308, 2 bosses)
- Dungeon debuffs: `RaidDebuffs/RaidDebuffs_Midnight.lua` + entry in `LoadRaidDebuffs.xml`
- Detailed encounter/spell data in memory file `midnight-expansion-data.md`
