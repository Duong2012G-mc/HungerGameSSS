<div align="center">

<img src="https://cdn.modrinth.com/data/hungergamessss/icon.png" width="120" alt="HungerGamesSSS Icon"/>

# HungerGamesSSS

**A full-featured Hunger Games survival plugin for Paper 1.21+**

[![Modrinth](https://img.shields.io/badge/Modrinth-4.5.9-00AF5C?logo=modrinth)](https://modrinth.com/plugin/hungergamesss)
[![Paper](https://img.shields.io/badge/Paper-1.21%2B-blue?logo=papermc)](https://papermc.io)
[![Java](https://img.shields.io/badge/Java-21%2B-orange?logo=openjdk)](https://adoptium.net)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue)](LICENSE)

*45 legendary weapons. Procedural dungeons. Last player standing.*

[📥 Download](https://modrinth.com/plugin/hungergamesss) • [🐛 Report a Bug](https://modrinth.com/plugin/hungergamesss) • [💬 Discord](#)

</div>

---

## 📖 Table of Contents

* ✨ Features
* 📦 Requirements
* 🚀 Installation
* 🏟️ Arena System
* ⏱️ Match Lifecycle
* 🎮 Game Modes
* ⚡ Legendary Weapons
* 🏰 Dungeons
* 🌽 Cornucopia Builder
* 🎁 Feast Event
* 👥 Teams
* 💾 Persistence
* 🌐 Localisation
* ⌨️ Commands & Permissions
* 🔌 Developer API
* 🔍 Troubleshooting
* 📄 Credits

---

# ✨ Features

## 🏟️ Arena System

* **Multi-arena support** — run unlimited concurrent arenas in separate worlds
* **Per-arena toggles** — PvP, lifesteal, hardcore, veinminer, nether, end
* **Glass cage countdown** — players locked in cages at spawn until the match begins
* **Auto-reset** — arena world and all blocks fully restored after each match
* **Legendary spawn points** — holograms float above each marked item location

---

## ⚡ 45 Legendary Weapons

| Category | Count | Examples |
| -------- | ----- | ------- |
| Swords & Melee | 15 | Cloud Sword, Excalibur, Dragon Katana |
| Bows & Ranged | 8 | Artemis Bow, Phantom Longbow, Warden Crossbow |
| Staves & Magic | 8 | Void Staff, Lich Staff, Hypnosis Staff |
| Tridents & Special | 9 | Aiglos, Mjolnir, Ravager Horn |
| Armor | 3 | Hades' Helm, Hermes Boots, Reinforced Elytra |
| Utility / Healing | 2 | Fountain of Youth, Warpick |

✔️ Each weapon has unique active + passive abilities
✔️ All stats tunable in `legend-items/<id>.yml`
✔️ Hot-reload with `/perf reload` — no restart needed

---

## 🏰 Procedural Dungeons

| Type | Terrain |
| ---- | ------- |
| 🕷️ Spider Nest | Cobweb cave |
| 💀 Crypt | Stone brick chamber |
| ⛏️ Gold Mine | Mineshaft tunnels |
| 🔥 Lava Temple | Nether-brick structure |
| 🧊 Ice Temple | Packed-ice biome |
| 🏛️ Surface Temple | Sandstone ruin |
| 🗼 Surface Outpost | Watchtower |
| 🌤️ Sky Island | Floating grass |
| 🏯 Floating Castle | Castle 30 blocks up |

✔️ No external schematics required
✔️ Loot configurable per dungeon type
✔️ All blocks restored after match ends

---

## 📈 Match Lifecycle

```
WAITING → STARTING → PLAYING → DEATHMATCH → ENDED
```

| Phase | Description |
| ----- | ----------- |
| ⏳ **Waiting** | Players queue in lobby |
| 🔢 **Starting** | Countdown; players teleport into cages |
| ⚔️ **Playing** | Grace period → main phase → feast → border shrink |
| 💥 **Deathmatch** | Border closes; final players fight |
| 🏆 **Ended** | Winner announced; arena fully reset |

✔️ Disconnect/reconnect mid-match restores last location, inventory, and scoreboard

---

## 🎮 Game Modes

### ❤️ Lifesteal
Kill a player → they drop a **heart item**. Pick it up to raise your max HP. Convert hearts back with `/withdraw <amount>`.

### 💀 Hardcore
Eliminated players are kicked instead of moved to spectator mode.

### ⛏️ Veinminer
Sneak + break an ore → chain-mines the entire vein. Max vein size is hot-configurable.

---

## 🌽 Cornucopia Themes

13 procedural themes for the arena centrepiece:

`classic` · `ultimate` · `speedsilver` · `ancient` · `void` · `royal` · `industrial` · `modern` · `crystal` · `jungle` · `end` · `nether` · `grand`

---

## 🌐 Localisation

10 built-in language files:

🇬🇧 `en` · 🇻🇳 `vi` · 🇷🇺 `ru` · 🇩🇪 `de` · 🇫🇷 `fr` · 🇪🇸 `es` · 🇵🇹 `pt` · 🇯🇵 `ja` · 🇰🇷 `ko` · 🇨🇳 `zh`

Players change language in-game with `/hg lang <code>`.

---

## 💾 Persistence — SQLite / MySQL

All arena state, player stats, team data, and match history persist across restarts. Switch backends in `config.yml`:

```yaml
database:
  type: sqlite    # or "mysql"
  sqlite-file: database.db
  mysql:
    host: localhost
    port: 3306
    database: hungergames
    username: root
    password: password
```

✔️ All writes are async
✔️ Auto schema migration on startup — no data loss

---

# 📦 Requirements

| Component | Version |
| --------- | ------- |
| Java | 21+ |
| Paper | 1.21+ |
| PlaceholderAPI | Optional |
| Vault | Optional |
| ProtocolLib | Optional |
| FastAsyncWorldEdit | Optional (`.nbt` / `.schem` support) |

---

# 🚀 Installation

1. Download the `.jar`
2. Drop it into `plugins/`
3. Start the server — configs generate automatically
4. Stand at the centre of your arena → `/hg create <name>`
5. `/hg setcornucopia <name> classic`
6. Walk to each spawn point → `/hg addspawn <name>` (repeat 2–24×)
7. Stand in your lobby → `/hg setlobby`
8. `/hg start <name>`

Players join with `/hg join <name>`.

---

# 🏟️ Arena System

**Per-arena toggles** (changed live with `/hg setuparena <name> <flag>`):

| Flag | Default | What it does |
| ---- | ------- | ------------ |
| `pvp` | on | Enable / disable PvP damage |
| `lifesteal` | off | Players drop heart items on death |
| `hardcore` | off | Eliminated players are kicked |
| `veinminer` | off | Sneak + break to chain-mine ores |
| `nether` | on | Isolated Nether world per arena |
| `theend` | on | Access to The End dimension |

**Legendary spawn points:**

```
/hg setlegendary <arena> <itemId>   — mark spawn at your feet
/hg deletelegendary <arena>         — remove nearest spawn (within 5 blocks)
```

---

# ⚡ Legendary Weapons

### 🗡️ Swords & Melee

| Item | Ability |
| ---- | ------- |
| ☁️ **Cloud Sword** | Walk on clouds; fall immunity; armor shred on hit |
| 💛 **Midas Sword** | +1 damage per kill (stacks); golden particle on kill |
| 💎 **Emerald Blade** | Deposit emeralds to upgrade damage; sneak+click GUI |
| 🌑 **Shadow Blade** | Sneak: teleport behind target; click: invisibility + Speed III |
| 💀 **Reaper Scythe** | On hit: steal buffs and heal; right-click: AOE sweep |
| 🐉 **Dragon Katana** | Passive Speed I; right-click: Dragon Fireball dash |
| ✨ **Excalibur** | 5s true invincibility (all damage cancelled); 50% reflect |
| 🪨 **Golem Hammer** | Leap + shockwave on landing; passive knockback resistance |
| 🌋 **Magma Club** | Lava pillar AOE; sneak+click: volcanic eruption |
| ⛏️ **Warpick** | On-hit 3×3 explosion; on critical hit: armor durability shred |
| 🖤 **Obsidian Dagger** | Sneak+click: Resistance V shield; click: meteor strike |
| 🔴 **Crimson Chainsword** | Shred stacks passive; right-click: Chainsaw Rampage burst |
| 👻 **Soul Gauntlet** | Collect souls on hit; right-click: Soul Blast; sneak: Ground Slam |
| 🪝 **Harpoon Launcher** | Fires a harpoon that hooks and pulls the target toward you |

### 🏹 Bows & Ranged

| Item | Ability |
| ---- | ------- |
| 🌙 **Artemis Bow** | 5 homing arrows; lightning proc; Sky Strike sneak ability |
| 👁️ **Phantom Longbow** | Invisible arrow; caster invisible on fire; target gets Darkness |
| 🌀 **Ender Bow** | Sneak: Void Dash; arrow hit: swap positions with target |
| 🧪 **Toxic Crossbow** | High-velocity bolt; poison cloud on impact |
| 🦾 **Warden Crossbow** | Sonic Boom projectile; converts blocks to Sculk on hit |
| 🌑 **Corrupted Crossbow** | Tick-based poison cloud bolt; Multishot enchant support |
| 🔱 **Guardian Cannon** | Passive Dolphins Grace in water; continuous stasis laser |
| 🐡 **Pufferfish Cannon** | Bouncing pufferfish; explodes with Poison + Nausea |

### 🔮 Staves & Magic

| Item | Ability |
| ---- | ------- |
| 🧊 **Lich Staff** | 4 ammo ice balls; 30s reload; passive Frost Walker |
| 🌌 **Void Staff** | Toggle Portal / Shulker Mode; independent cooldowns per mode |
| 🐛 **Hypnosis Staff** | Hypnotise up to 5 mobs; minions target your enemies |
| 💥 **Villager Wand** | Aim at block → explosive slime bomb detonates |
| 📓 **Death Note** | `/dn <player>`: instant kill (3 uses, per-target cooldown 25s) |
| 🌑 **Sculkweaver's Lantern** | Sculk bomb: Darkness + Slowness lingering zone |
| ❄️ **Horn of Winter** | Blizzard: continuous Freeze + Slowness in large radius |
| 🔔 **Ghastly Whistle** | Rideable Ghast companion; auto-fireball every 2s; 20s duration |

### 🌊 Tridents, Launchers & Special

| Item | Ability |
| ---- | ------- |
| 🔱 **Aiglos** | Trident: explosive throw + AOE freeze; melee stats via attributes |
| 🌀 **Wither Sickles** | Auto dual-wield offhand; throw (CD 7s); AOE Wither + lifesteal on hit |
| ⚡ **Mjolnir** | Throw (returns); melee hit: lightning strike |
| 🦏 **Ravager Horn** | Mount a Ravager; sneak+click: Stampede wave |
| 🔬 **Shrink Ray** | Left-click: shrink target; right-click: enlarge target |
| 🔴 **Armadillo Detonator** | Shoot up to 3 armadillos; left-click: detonate all |
| 🐝 **Beehive Blaster** | Launches 4 angry bees that auto-target nearest enemy |
| 🌿 **Ribbit Reel** | Hit block: pull yourself; hit entity: pull them to you |
| 💧 **Fountain of Youth** | Passive +1 HP/s while held; right-click: 30s healing zone for allies |

### 🛡️ Armor

| Item | Slot | Ability |
| ---- | ---- | ------- |
| 🪖 **Hades' Helm** | Head | Night Vision; invisibility while sneaking; fire aura; Wither attacker on hit received |
| 👟 **Hermes Boots** | Feet | Speed II + Jump Boost; Hermes Dash; full fall immunity |
| 🪂 **Reinforced Elytra** | Chest | Particle trail while gliding; Sonic Boom boost; landing explosion |

---

# 🏰 Dungeons

Dungeons spawn procedurally during a match — no schematic files needed.

```yaml
dungeons:
  mob-hp: 60.0
  boss-hp: 100.0
  spawn-chance: 0.1
  lava-temple:
    enabled: true
    mob-hp: 40.0
    fire-ticks: 100
```

---

# 🎁 Feast Event

High-tier chests spawn at the Cornucopia after a configurable timer.

```yaml
arena:
  feast:
    enabled: true
    time-minutes: 20
    chest-count: 8
    announce-minutes: [15, 10, 5, 3, 2, 1]
```

---

# 👥 Teams

| Command | Description |
| ------- | ----------- |
| `/team create <name>` | Create a team (you become leader) |
| `/team invite <player>` | Send an invite (expires 60s) |
| `/team join <name>` | Join an open team |
| `/team leave` | Leave your team |
| `/team open` | Toggle open / invite-only |
| `/team disband` | Disband team (leader only) |
| `/tc <message>` | Team chat |

---

# ⌨️ Commands

### `/hg` — Main Command

| Sub-command | Permission | Description |
| ----------- | ---------- | ----------- |
| `join <arena>` | `hoplitebr.hg` | Join an arena lobby |
| `leave` | `hoplitebr.hg` | Leave the current arena |
| `lang <code>` | `hoplitebr.hg` | Change your language |
| `create <name>` | `hoplitebr.admin` | Create a new arena |
| `start <arena>` | `hoplitebr.admin` | Force-start a match |
| `setcornucopia <arena> <theme>` | `hoplitebr.admin` | Set and build the Cornucopia |
| `addspawn <arena>` | `hoplitebr.admin` | Add player spawn at your feet |
| `setuparena <arena> <flag>` | `hoplitebr.admin` | Toggle per-arena flags |
| `setlegendary <arena> <id>` | `hoplitebr.admin` | Mark legendary spawn |
| `setlobby` | `hoplitebr.admin` | Set post-match lobby location |
| `admin give <player> <item>` | `hoplitebr.admin` | Give a legendary item |
| `reload` | `hoplitebr.admin` | Reload all config files |

### Other Commands

| Command | Permission | Description |
| ------- | ---------- | ----------- |
| `/withdraw <amount>` | `hoplitebr.withdraw` | Convert lifesteal hearts to items |
| `/perf <stats\|optimize\|reload\|gc\|chunks>` | `hoplitebr.admin` | Performance tools |
| `/structure <reset\|stats>` | `hoplitebr.admin` | Structure management |
| `/locate <name>` | `hoplitebr.hg` | Locate a placed structure |
| `/migrate` | `hoplitebr.admin` | Migrate legacy legendary PDC tags |
| `/debug` | `hoplitebr.admin` | Dump debug info to console |
| `/dn <player>` | `hoplitebr.dn` | Death Note kill command |

---

# 🔐 Permissions

| Permission | Default | Description |
| ---------- | ------- | ----------- |
| `hoplitebr.hg` | `true` | Basic game commands |
| `hoplitebr.team` | `true` | Team commands |
| `hoplitebr.tc` | `true` | Team chat |
| `hoplitebr.withdraw` | `true` | Lifesteal heart withdrawal |
| `hoplitebr.dn` | `true` | Death Note kill command |
| `hoplitebr.admin` | `op` | All admin / setup commands |

---

# 🔧 Configuration

Config files:

* `config.yml` — global settings, arena defaults, database, performance
* `loot.yml` — chest loot tiers (common / uncommon / rare)
* `scoreboard.yml` — scoreboard layout and update interval
* `legend-items/<id>.yml` — per-weapon stats, recipe, lore, enchantments
* `dungeons.yml` — dungeon type toggles and mob HP
* `cornucopia.yml` — Cornucopia build presets
* `languages/<code>.yml` — message translations

---

# 🔌 Developer API

```java
// Get the arena a player is in (null if not in any)
Arena arena = HopliteAPI.getPlayerArena(player);

// Access the plugin instance
Main plugin = HopliteAPI.getPlugin();
```

**Custom Events:**

| Event | Fired when |
| ----- | ---------- |
| `ArenaStartEvent` | Match transitions STARTING → PLAYING |
| `ArenaEndEvent` | Match ends (winner declared) |
| `PlayerEliminatedEvent` | A player is eliminated |
| `DungeonSpawnEvent` | A dungeon is generated |

---

# 🔍 Troubleshooting

<details>
<summary><b>Legendary abilities not working</b></summary>

* Confirm the player is in an active arena (`PLAYING` or `DEATHMATCH`)
* Check `hoplitebr.admin` permission — OP always bypasses ability checks
* Run `/debug` and check console for `[Ability]` log lines with `debug: true`

</details>

<details>
<summary><b>Countdown cancels immediately on start</b></summary>

* Fixed in 4.5.5 (BUG-CD-01)
* Update to latest version

</details>

<details>
<summary><b>Feast never spawns in second match</b></summary>

* Fixed in 4.5.5 (BUG-FM-01)
* Update to latest version

</details>

<details>
<summary><b>Stats reset to zero on every join</b></summary>

* Fixed in 4.5.7 (BUG-JOIN-02)
* Update to latest version

</details>

<details>
<summary><b>Ability listeners fire twice</b></summary>

* Fixed in 4.5.7 (BUG-REG-01)
* Was caused by `registerEvents()` in ability constructors

</details>

<details>
<summary><b>Italic text on legendary item names</b></summary>

* Fixed in 4.5.4 (BUG-FONT-01)
* Paper 1.20.5+ changed the default rendering style for custom item names

</details>

<details>
<summary><b>/reload breaks the plugin</b></summary>

* Always restart the server — never use `/reload`

</details>

---

# ❓ FAQ

### Can I run multiple arenas at once?
Yes — each arena is fully isolated in its own world instance.

### Do I need FastAsyncWorldEdit?
No. It is only required if you want to paste custom `.nbt` / `.schem` structures.

### Can players on the same team damage each other?
No — all legendary weapons include friendly-fire checks.

### How do I add a custom language?
Create `languages/<code>.yml` — copy `en.yml` as a base and translate. Players switch with `/hg lang <code>`.

### Does it support Spigot?
No — Paper 1.21+ is required.

---

# 📄 Credits

**Author:** Duong2012G — [Modrinth](https://modrinth.com/plugin/hungergamessss)

Several weapon mechanics were inspired by **DemoRaLegendary Weapons** by **Demora** (Apache 2.0 licence).
No source code was copied — only gameplay concepts were referenced.

> 🔗 DemoRaLegendary Weapons: <https://modrinth.com/plugin/demoralegendaryweapons>

*Open source — contributions welcome.*
