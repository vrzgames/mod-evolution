# mod-evolution V1.1
This module is started hard tested by 2026.09.10 19:00 , published when the module is so stable untill this not availabled.
`mod-evolution` is a global realm-era controller for AzerothCore 3.3.5a. This revision targets the
Playerbot checkout at commit `47960183bb03b83e8943eb2f0f39c16df9710c9d`.

The module controls a single realm-wide phase. It enforces level caps, expansion and instance map
access.

## Phase table

| Phase | Cap | Content label |
|---:|---:|---|
| 1 | 60 | Molten Core |
| 2 | 60 | Blackwing Lair |
| 3 | 60 | Zul'Gurub / Ruins of Ahn'Qiraj |
| 4 | 60 | Temple of Ahn'Qiraj |
| 5 | 70 | Karazhan / Gruul / Magtheridon |
| 6 | 70 | Serpentshrine Cavern / Tempest Keep |
| 7 | 70 | Hyjal Summit / Black Temple |
| 8 | 70 | Zul'Aman / Sunwell / Magisters' Terrace / Quel'Danas |
| 9 | 80 | Naxxramas / OS / EoE / Archavon |
| 10 | 80 | Ulduar / Emalon |
| 11 | 80 | Trial raids/dungeon / WotLK Onyxia / Koralon |
| 12 | 80 | ICC / Frozen Halls / Toravon |
| 13 | 80 | Ruby Sanctum |

The labels describe intended progression. Actual protection is provided by the rules below, not by
the label text.

## Built-in access rules

- Outland map 530 opens at phase 5. Northrend map 571 opens at phase 9. Other TBC/WotLK maps not
  explicitly listed inherit their expansion's opening phase from `Map.dbc`.
- The major raid maps follow the table. Patch-tier exceptions are explicit: Magisters' Terrace 585
  at phase 8; Trial of the Champion 650 at phase 11; Forge of Souls 632, Pit of Saron 658, and Halls
  of Reflection 668 at phase 12.
- Vault of Archavon map 624 opens at phase 9. Its later bosses share that same map, so a map gate
  cannot separately lock Emalon, Koralon, and Toravon. The phase labels are accurate, but encounter
  isolation requires DB conditions, boss-specific scripts, or explicit loot/item rules.
- Stock map 249 is the level-80 WotLK Onyxia encounter and opens at phase 11. It is not presented as
  Vanilla level-60 Onyxia.
- Stock map 533 is WotLK Naxxramas. AzerothCore 3.3.5a has no stock Naxx40 map. A restoration module
  must use a unique map/content rule configured through `Evolution.MapRules.PhaseN`.
- Isle of Quel'Danas area 4080 opens at phase 8 through an area-change safety gate. The underlying
  Sun's Reach world-state progression remains the core's 3.3.5 implementation.

Map gates do not recreate removed Vanilla/TBC attunements. If attunements are part of the desired
ruleset, implement and test them separately with AzerothCore's access requirements/conditions.

## Item and quest rule scope

V1.1 intentionally uses explicit IDs only. It does not guess an item's phase from item level,
required level, quality, name, or expansion. In `mod_evolution.conf`, populate:

- `Evolution.ItemRules.PhaseN` with item IDs requiring phase N;
- `Evolution.ItemRules.AlwaysDeny` for items that must remain unavailable;
- `Evolution.ItemRules.AlwaysAllow` for reviewed overrides;
- `Evolution.QuestRules.PhaseN` with reward-source quest IDs that must not complete early.

For listed items, hooks deny use/open, equip/save-equip, vendor purchase and sale, player trade, mail,
and auction bids/buyouts. Loot rolls are rejected before item generation. If a restricted item is in
an equal-chance loot group, the whole group is conservatively suppressed because this core hook
cannot safely remove one candidate from that group.

Important limits:

- Unlisted items are allowed. The shipped lists are empty, so this is an enforcement framework, not
  a complete 3.3.5 item catalog.
- The available hook blocks player auction purchases, but this commit has no pre-listing player hook.
  A restricted item can still be listed, although players cannot bid or buy it through the normal
  handler.
- Quest rules block quest completion by quest ID. They do not hide quests, and item rules alone do
  not prevent the reward item from first being created by core code.
- Already-equipped restricted items are preserved and their existing stats are not stripped. New
  use/equip/transfer operations are blocked. No automatic item destruction occurs.
- Direct database edits or third-party code paths that bypass AzerothCore hooks are outside scope.

These limitations are why `.evolution validate` warns when the item catalog is empty and why the
module does not claim complete gear/vendor coverage.

## Existing characters and safe downgrade behavior

Progression is forward-only by default. `.evolution set <phase>` cannot downgrade. A downgrade needs
both `Evolution.AllowForcedDowngrade = 1` and `.evolution force <phase>`. Config reload downgrades are
separately controlled by `Evolution.AllowConfigDowngrade`.

`Evolution.ExistingLevelPolicy` controls characters above the cap:

- `0`: preserve and allow;
- `1`: explicitly clamp their stored level with `GiveLevel`;
- `2` (default): preserve the level, but quarantine the character from instances and LFG.

`Evolution.ExistingDeathKnightPolicy=1` similarly quarantines existing DKs before phase 9 without
deleting them. New DK creation remains blocked. Quarantine is deliberately non-destructive and does
not promise a complete open-world freeze.

On config reload or runtime phase change, all online players receive the new max-level field, invalid
LFG queues are cancelled, newly forbidden maps/areas are evacuated, and the change is broadcast and
logged. Offline characters are evaluated at their next login. With
`Evolution.RequireValidConfiguration=1`, a validation error rejects both runtime and config-reload
phase changes; startup loads the configured phase and logs the error so it can be repaired.

## Commands

- `.evolution` or `.evolution status`: current phase and active rule counts/states.
- `.evolution validate`: validate current rules, IDs, core cap, and Playerbot settings.
- `.evolution preview <1-13>`: preview metadata, locked-rule counts, direction, and validation.
- `.evolution set <1-13>`: forward-only runtime phase change.
- `.evolution force <1-13>`: downgrade-capable runtime change when explicitly enabled.
- `.evolution reload`: run the core's real configuration reload path. This reloads all worldserver
  and module configs and then invokes the normal module config hooks.

Runtime changes are not persisted. Edit `Evolution.CurrentPhase` for the next restart/reload.

## Installation

1. Copy the `mod-evolution` directory into `azerothcore/modules/`.
2. Copy `conf/mod_evolution.conf.dist` to the module configuration location used by the build and
   remove `.dist` if the deployment does not do that automatically.
3. Review the compatibility settings below and populate the explicit item/quest catalog.
4. Re-run CMake and compile in the normal deployment workflow.
5. On a staging realm, run `.evolution validate`, then the test plan in `TESTING.md`.

The core repository used for this revision was not configured or built, in accordance with its local
agent rules.

## Installed-module compatibility

See `COMPATIBILITY.md` for exact settings and caveats for mod-playerbots, mod-ah-bot-plus,
mod-dynamic-xp, the NPC/profession modules, and the DK skip module.

## Operational warning

Keep a database backup before enabling level clamping or any downgrade. V1.1 performs no automatic
deletions, but a deliberate `ExistingLevelPolicy=1` changes character levels. Treat every downgrade
as a maintenance event and test it on a copy of the realm first.
