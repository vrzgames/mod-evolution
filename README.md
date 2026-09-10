# mod-evolution

**AzerothCore module for phase-based server progression.**

Created by **iCore**.

`mod-evolution` turns a WotLK AzerothCore realm into a controlled progression server where the entire world evolves through predefined phases — from the first Vanilla raids all the way to Ruby Sanctum.

Instead of opening every expansion and endgame system from day one, the active phase controls the maximum level, accessible expansion content, PvE tier, PvP season and restricted areas for both real players and Playerbots.

---

## Features

- **Global server progression** controlled by one active phase.
- **13 progression phases** from Vanilla to the end of WotLK.
- **Automatic level caps**: 60 → 70 → 80.
- **Expansion locking**:
  - Vanilla phases: Outland and Northrend locked.
  - TBC phases: Northrend locked.
  - WotLK phases: Northrend unlocked.
- **Restricted-area handling**: players entering content that is not available in the current phase can be returned to an allowed location / innkeeper.
- **Playerbot support**: progression restrictions also apply to bots.
- **PvE progression** based on the active raid tier.
- **PvP progression** based on the active Arena season.
- **Death Knight / expansion restrictions** can follow the active phase.
- **Server announcements** can show the current phase, level cap, PvE tier and PvP season.
- Designed for **AzerothCore WotLK**.

---

## Progression Phases

| Phase | Expansion | Max Level | PvE Progression | PvP Progression |
|---|---|---:|---|---|
| **Phase 1** | Vanilla | 60 | Molten Core / Onyxia — Tier 1 | Vanilla PvP |
| **Phase 2** | Vanilla | 60 | Blackwing Lair — Tier 2 | Vanilla PvP |
| **Phase 3** | Vanilla | 60 | Zul'Gurub / Ahn'Qiraj — Tier 2.5 | Vanilla PvP |
| **Phase 4** | Vanilla | 60 | Naxxramas 40 — Tier 3 | Vanilla PvP |
| **Phase 5** | The Burning Crusade | 70 | Karazhan / Gruul / Magtheridon — Tier 4 | Season 1 |
| **Phase 6** | The Burning Crusade | 70 | Serpentshrine Cavern / Tempest Keep — Tier 5 | Season 2 |
| **Phase 7** | The Burning Crusade | 70 | Mount Hyjal / Black Temple — Tier 6 | Season 3 |
| **Phase 8** | The Burning Crusade | 70 | Sunwell Plateau — Tier 6 / Sunwell | Season 4 |
| **Phase 9** | Wrath of the Lich King | 80 | Naxxramas / Obsidian Sanctum / Eye of Eternity — Tier 7 | Season 5 |
| **Phase 10** | Wrath of the Lich King | 80 | Ulduar — Tier 8 | Season 6 |
| **Phase 11** | Wrath of the Lich King | 80 | Trial of the Crusader — Tier 9 | Season 7 |
| **Phase 12** | Wrath of the Lich King | 80 | Icecrown Citadel — Tier 10 | Season 8 |
| **Phase 13** | Wrath of the Lich King | 80 | Ruby Sanctum / Final WotLK Phase | Season 8 |

The phase numbering is intentionally continuous across all three expansions so the realm has one simple progression state.

---

## Example

If the server is currently running:

```text
Phase 7
Max Level: 70
PvE: Tier 6
PvP: Season 3
Content: Mount Hyjal / Black Temple
Northrend: Locked
```

players and Playerbots remain inside the TBC progression rules.

When the server advances to the next phase, the available content changes globally.

---

## Server Announcements

The module can inform players about the currently active progression state.

Example:

```text
[EVOLUTION] Current Phase: 7 | Max Level: 70 | PvE: Tier 6 | PvP: Season 3
```

This makes it immediately clear what content is currently available on the realm.

---

## Installation

Clone the module into your AzerothCore `modules` directory:

```bash
cd azerothcore-wotlk/modules
git clone https://github.com/vrzgames/mod-evolution.git
```

Then re-run CMake and rebuild the server.

The final structure should look like:

```text
azerothcore-wotlk/
└── modules/
    └── mod-evolution/
```

After compilation, copy the generated `.conf.dist` file to `.conf` if your setup does not do this automatically, then configure the active phase.

---

## Updating

To update an existing installation:

```bash
cd azerothcore-wotlk/modules/mod-evolution
git pull
```

After source-code changes, rebuild the server.

---

## Configuration

The module is designed around a single global active phase.

Example:

```ini
Evolution.Enable = 1
Evolution.Phase = 1
Evolution.Announce = 1
```

Change the phase value to move the entire server forward in progression.

Example:

```ini
Evolution.Phase = 7
```

would represent the Tier 6 / TBC progression period.

> Configuration names may evolve as the module is developed. Always check the included `.conf.dist` file for the current available options.

---

## Playerbot Support

`mod-evolution` is designed to keep Playerbots under the same progression rules as real players.

This prevents bots from bypassing the active server phase by:

- leveling beyond the current cap,
- entering expansion areas that are not yet unlocked,
- accessing progression content ahead of the realm,
- using expansion-specific systems before they become available.

The goal is to make the whole realm — players and bots — progress together.

---

## Philosophy

The purpose of `mod-evolution` is not individual character progression.

It controls the **evolution of the entire server**.

A realm can begin as a Vanilla-style Level 60 server, move through TBC, and eventually unlock the complete WotLK endgame without changing cores or creating a new realm.

One server. One timeline. One active phase.

---

## Compatibility

- AzerothCore WotLK
- Designed to work with Playerbot-enabled AzerothCore setups
- C++17-compatible build environment used by current AzerothCore

---

## License

This project is licensed under the **GNU General Public License v2.0**.

See the `LICENSE` file for details.

---

## Author

Created and maintained by **iCore**.

GitHub: [vrzgames](https://github.com/vrzgames)
