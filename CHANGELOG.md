# Changelog — HungerGamesSSS

All notable changes are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [4.5.9] — Bug Fix Pass 5 (2026-04-24)

### Fixed — 3 bugs across 3 files

#### High

- **BUG-BASE-01 · `BaseAbility` — `validateCanUse()` uses wrong permission node `hg.admin` instead of `hoplitebr.admin`**
  (`BaseAbility.java`)
  `validateCanUse()` granted the admin bypass with `player.hasPermission("hg.admin")`.
  The correct node — defined in `plugin.yml` and fixed for commands in 4.1 (bug M-07) — is
  `hoplitebr.admin`. Any operator who granted the `hoplitebr.admin` permission explicitly
  (without OP) received no bypass; conversely, `hg.admin` grants that had been assigned in
  a permissions plugin incorrectly bypassed ability checks.
  Fix: changed check to `player.hasPermission("hoplitebr.admin")`. The `|| player.isOp()`
  fallback is unchanged so vanilla OP still bypasses.

#### Medium

- **BUG-CTX-01 · `AbilityContext` — no `isCancelled()` / `setCancelled()` API**
  (`AbilityContext.java`)
  The `AbilityContext` object was fully immutable after construction. Abilities that need to
  signal downstream handlers to skip processing (e.g. two abilities sharing the same trigger
  chain) had no standard way to do so — each ability reimplemented ad-hoc boolean flags.
  Fix: added a mutable `boolean cancelled` field with `isCancelled()` and
  `setCancelled(boolean)` accessors. The field defaults to `false`. Abilities can now call
  `ctx.setCancelled(true)` to mark a context as consumed, and other abilities/listeners can
  check `ctx.isCancelled()` before acting.

- **BUG-OD-02 · `ObsidianDagger` — deprecated `getTargetBlock(null, range)` + 1-tick meteor trail task + hardcoded `sendMessage` calls**
  (`ObsidianDagger.java`)
  Three independent issues in `performRightClick`:
  1. `player.getTargetBlock(null, TARGET_DISTANCE)` — passing `null` for the transparent-block
     set is deprecated in Paper 1.21 and emits a compile warning. Replaced with
     `player.getTargetBlockExact(distance, FluidCollisionMode.NEVER)` with a `rayTraceBlocks`
     primary path for accurate targeting.
  2. The meteor particle trail `BukkitRunnable` ran at period `1L` (every tick). A falling
     meteor moves slowly enough that `2L` (every 2 ticks) produces visually identical results
     at half the server-side task overhead.
  3. Two `player.sendMessage("§...")` calls bypassed the Adventure component API (italic
     rendering issue) and the i18n system. Replaced with `player.sendActionBar(Component)`
     using `LegacyComponentSerializer`, consistent with the pattern used throughout other
     abilities since 4.5.5.

### Changed
- `pom.xml` — version `4.5.8` → `4.5.9`.
- `plugin.yml` — version `4.5.8` → `4.5.9`; description updated.

---

## [4.5.8] — Legend Rewrite Pass (2026-03-22)

> Continued from 4.5.7. See 4.5.7 entry below for full bug fix notes.

### Changed
- Version bump: 4.5.7 → 4.5.8
- All legend rewrites from the rewrite pass are now packaged under this version.

---

## [4.5.7] — Full Bug Sweep Pass 4 + Legend Rewrite (2026-03-20 — CLOSED)

### Legend Rewrites (ongoing)

#### Removed

- **ChainsawSword** — removed entirely. `ChainsawSword.java`, `chainsaw_sword.yml`, and
  `AbilityRegistry` entry deleted. Plugin now has 42 legendary abilities.

- **DeathScythe** — removed entirely. `DeathScythe.java`, `death_scythe.yml`, and
  `AbilityRegistry` entry deleted. Plugin now has 42 legendary abilities (down from 44 after
  both removals).

#### Rewritten — Ability Logic

- **Warpick** — complete rewrite.
  Old: SeismicShatter wave + broken mine (mine cancelled on tick 0 due to wrong `isAir()` check).
  New: Two fully independent abilities with two separate cooldown maps.
  - *Ability 1 · Mine Explosion* (BlockBreakEvent, CD 5s): breaks a 3×3×3 zone around the
    broken block, deals 6 HP blast damage to nearby entities. No terrain damage. Uses
    `EventPriority.HIGH` so arena-state guard runs first.
  - *Ability 2 · Critical Hit* (on-hit, CD 20s): triggers only on `event.isCritical()` vs
    Player. Applies light knockback + degrades a random armour piece by 15% max durability.
  - Bug fixes applied: BUG-WP-01 (shared cooldown), BUG-WP-02 (mine never worked),
    BUG-WP-03 (detonate_damage ignored), BUG-WP-04 (Player reference in closure),
    BUG-WP-05 (shatter wave origin), BUG-WP-06 (lore said 5s, was 10s).

- **Dragon Katana** — complete rewrite.
  - *Passive · Speed Run*: permanent Speed I (amplifier read from `passive_speed_amplifier`
    config) via `passiveTick()`. Respects stronger Speed potions. No particle swirl.
  - *Active · Dragon Dash* (right-click, CD 12s): precise rayTrace to block or player up to
    32 blocks. Flanks aimed players (appears 1.5 blocks behind them). `findSafeLanding()`
    searches ±5 blocks vertically for a safe 2-block-tall standing spot. DRAGON_BREATH
    trail + arrival burst.
  - Fix: `passive_speed_amplifier` config key is now read in `passiveTick()` (was hardcoded 0).

- **Artemis Bow** — base enchant changed Power V → Power III, Infinity removed.
  - New `AnvilListener` added: Artemis Bow can be upgraded via anvil up to Power VII
    (configured via `max_power_level: 7` in YAML). Power above VII is clamped automatically.
    All other legendary items: anvil operations that would strip the `hg_ability` PDC tag
    are cancelled (prevents ability removal via rename+combine).
  - `AnvilListener` registered in `Main.registerListeners()`.

- **Aiglos** — complete rewrite.
  Old: frost AOE on trident hit (freeze, slow, ice particles).
  New: explosive spear with attribute-based melee stats.
  - *Passive*: melee stats via `aiglos.yml` `attributes:` block — Attack Damage 7, Attack Speed
    1.6/s, Reach 3.5 blocks. Handled by `AbilityManager.createItem()` attribute parser (new).
  - *Active · Explosive Throw*: on trident hit — visual-only explosion (power 3, no block
    break), AoE damage in 3×3×3 zone. Base 12 HP + 1.5 HP per Sharpness level. Loyalty III
    enchant handles auto-return. `inFlight` map tracks per-player trident for action bar hint.

- **Armadillo Detonator** — complete rewrite.
  Old: single-bolt launch, auto-explode on entity contact.
  New: shoot/detonate two-button system.
  - *Right-click · Shoot* (CD 20s): spawns up to 3 Armadillo entities. Each bounces on
    terrain (max 3 bounces), expires after 400 ticks. CRIT + SMOKE trail.
  - *Left-click · Detonate*: instantly detonates all active armadillos. Each blast: 30 HP
    raw with 40% armour bypass, radius 2.0 blocks (4×4×4), no terrain damage, radial knockback.
  - `cleanupPlayer()` removes all tracked Armadillo entities on quit/death.

- **Beehive Blaster** — complete rewrite.
  Old: ball projectile that spawned swarm on contact, no Poison mechanic.
  New: right-click crossbow that directly spawns angry Bee entities.
  - *Right-click · Bee Swarm* (CD 22s): spawns 4 bees with spread ±1.5 blocks,
    `setAnger(MAX_VALUE)`, `setTarget(nearestEnemy)` within 16 blocks at spawn.
  - *Poison sting*: `attachStingEffect()` BukkitRunnable polls every 4 ticks; applies
    Poison I (8s) when bee is within sting range (~1.5 blocks). Stackable across 4 bees.
  - Bees auto-remove after 30s with POOF particle. `cleanupPlayer()` removes bees on quit/death.

- **Crimson Chainsword** — complete rewrite.
  Old: rev-up mechanic + bleed DoT.
  New: Shred Stacks passive + Chainsaw Rampage active.
  - *Passive · Shred Stacks*: each melee hit on a Player enemy adds 1 stack (max 5, 8s each).
    Each stack: +10% effective armour penetration on all damage. Stack expiry uses per-stack
    `Deque<Long>` timestamps — each stack has its own timer.
  - *Right-click · Chainsaw Rampage* (CD 10s, 3s burst): 12 ticks × 0.25s. Per tick in
    radius 3.5: 6 HP (50% armour bypass) + random 15-25% durability shred on random armour
    piece + 2s fire + 0.5-block radial knockback. BLOCK crack particles + red CRIT glow.
  - `rampageActive` Set prevents overlapping rampage sessions.

- **Corrupted Crossbow** — complete rewrite.
  Old: spread multi-bolt arrows with Poison+Weakness on hit entity.
  New: tick-based poison cloud projectile with AoE splash.
  - *Right-click · Poison Cloud Shot* (CD 4s): tick-based bolt (not Arrow entity). Purple WITCH
    + DUST trail. Mild gravity curve. Detonates on block contact or entity collision (direct
    hit: 12 HP + AoE splash).
  - *On impact*: `detonate()` — Poison I (8s) splashed to all enemies in radius 2.0 (4×4×4).
    WITCH + DUST burst + LARGE_SMOKE.
  - *Multishot*: if Multishot enchant on crossbow, fires 3 bolts at ±0.18 rad spread.
  - CD is 4s (spam-friendly as per spec).

- **Ghastly Whistle** — complete rewrite.
  Old: summon AI-off ghast 10 blocks above, fire fixed number of fireballs in a loop.
  New: rideable ghast companion with manual steering and auto-targeting.
  - *Right-click · Summon Happy Ghast* (CD 45s, 20s duration): spawns a Ghast with
    `setAI(false)`, `setGravity(false)`, `setGlowing(true)`. Player is immediately mounted
    via `ghast.addPassenger(player)`. Every tick, the ghast is teleported toward the player's
    look direction (speed 0.4 b/t) — simulates steering while ridden.
  - *Auto-fireball*: every 40 ticks (2s), scans for nearest enemy within 25 blocks and
    fires a `Fireball` with `setShooter(player)`, yield 1.5 (no block griefing), `setIsIncendiary(false)`.
  - *Happy visuals*: HAPPY_VILLAGER particles every 5 ticks + HEART burst on spawn.
    GHAST_WARN + GHAST_AMBIENT on summon.
  - *Dismiss*: on duration end, quit, or death — `dismissGhast()` ejects passengers,
    spawns POOF, plays GHAST_DEATH, removes entity.
  - **Note**: Happy Ghast entity (`HAPPY_GHAST`) requires Minecraft 1.21.5+.
    On Paper 1.21.1 the plugin uses a standard `GHAST` entity simulating the same
    behaviour. No code change needed when upgrading to 1.21.5 — only swap
    `EntityType.GHAST` → `EntityType.HAPPY_GHAST` in the spawn call.


  Old: right-click healing zone with Regeneration + Saturation potion effects.
  New: passive auto-regen + active healing fountain.
  - *Passive · Auto Regen*: `passiveTick()` every 20 ticks applies +1 HP directly while
    holding. No food, no potion — works in combat. Capped at max health.
  - *Right-click · Healing Fountain* (CD 60s): spawns a 30-second 8×8 healing zone
    centred on the player. Every second, heals +1 HP to self and all teammates within
    radius 4. Rotating DRIPPING_WATER ring particle effect + scattered GLOW dots.
    AMBIENT_UNDERWATER_ENTER sound on spawn. WATER_AMBIENT pulse every 5s.
  - Action bar: shows passive regen status + fountain cooldown while held.


  Old: right-click true-invincibility window (5s) + passive reflect + crit armor pierce.
  New: passive 3-hit immunity charge system.
  - *Passive · 3-Hit Immunity Charges*: `@EventHandler(HIGHEST, ignoreCancelled=false)` on
    `EntityDamageEvent`. When player holds Excalibur and has ≥1 charge, the damage is
    cancelled (`setCancelled(true)`) and one charge is consumed via `AtomicInteger.compareAndSet`.
    Blocks ALL damage sources — sword, arrow, explosion, fall.
  - *Recharge*: BukkitRunnable ticks every 20t (1s). Once `System.currentTimeMillis() - lastGainMs`
    exceeds `recharge_ms` (45s), increments charge if < max. Beacon ambient sound + action bar
    on recharge.
  - *Action bar*: `§e◈◈◈ (3/3)` shield bar + countdown to next recharge.
  - *Death*: charges reset to 0 (fair — no permanent free shields).
  - Implements `Listener` — registered by `AbilityRegistry.register()`.
  - `cleanup()` cancels rechargeTask.

- **`AbilityManager.createItem()` — `unbreakable: true` YAML support**: any legend item with
  `unbreakable: true` in its YAML now has `ItemMeta.setUnbreakable(true)` applied.
  Used by Excalibur and Reaper Scythe.


  Old: right-click dash + arrow hit swap positions with target.
  New: three-ability void mobility system.
  - *Ability 1 · Right-click Ender Pearl* (CD 30s): tick-based virtual pearl with gravity curve
    travels up to 40 blocks. On block collision, `teleportSafe()` finds a 2-block-tall air gap
    above the impact. Purple REVERSE_PORTAL trail.
  - *Ability 2 · Arrow → Floating Pearl*: on `performProjectileLaunch` the arrow is tagged.
    `performProjectileHit` spawns a glowing, gravity-less `Item(ENDER_PEARL)` entity at impact
    (block hits only, not entity hits). Max 3 active per player. Pearl floats with sin-wave
    orbit + REVERSE_PORTAL particles. Auto-despawns after 60s.
  - *Ability 3 · Pearl Pickup → Instant Teleport* (no cooldown): `@EventHandler` on
    `EntityPickupItemEvent`. Only the pearl's owner can trigger the teleport; all other
    players' pickup attempts are cancelled. Teleport is instant with no cooldown.
  - Pearl chain: Arrow → Pearl → Walk in → Teleport → repeat indefinitely.
  - Implements `Listener` — registered by `AbilityRegistry.register()`.
  - `cleanupPlayer()` removes all floating pearl entities on quit/death.


  Old: sweep damage + Wither DoT / melee steal on hit.
  New: standalone Soul Harvest ability via right-click.
  - *Right-click · Soul Harvest* (CD 45s): 360° AoE in 5-block radius.
    - *Potion Steal*: removes ALL beneficial effects (Speed, Strength, Regeneration,
      Resistance, Haste, Jump Boost, Fire Resistance, Water Breathing) from every enemy
      hit. Full duration transferred to caster. Debuffs are never stolen.
      If caster already has the effect, only upgrades if stolen version is stronger/longer.
    - *Lifesteal*: heals 25% of each target's max HP (configurable `lifesteal_pct`).
      Example: 40 HP enemy → +5 hearts healed per target.
  - Visual: 50 SOUL particles in sweeping ring + SOUL_FIRE_FLAME burst + ENTITY_WITHER_HURT sound.
  - Action bar feedback: shows effects stolen count + total hearts healed + targets hit.
  - Base stats via `attributes:` YAML block: Attack Damage 7 HP, Attack Speed 1.0/s.

- **Death Note** — complete redesign.
  Old: passive raycast → progress bar → summon Warden Shinigami after 40s stare.
  New: chat-command kill with uses and per-target cooldown.
  - *Usage*: Hold Death Note → type `/dn <playerName>` in chat.
  - *Effect*: `target.setAbsorptionAmount(0); target.setHealth(0)` — instant kill bypassing
    armor, potions, and Totem of Undying. 2-tick delay for dramatic effect.
  - *Cooldown*: 25s per target (per-target `Map<UUID, Map<UUID, Long>>` — different targets
    have independent cooldowns).
  - *Uses*: 3 total, stored in item PDC (`death_note_uses`). On last use: book removed from
    inventory with ASH particles + PAGE_TURN sound. Lore updates dynamically after each use.
  - *New command* `/dn`: registered in `plugin.yml` + `Main.java`. `DeathNoteCommand.java` and
    `DeathNoteTabCompleter.java` created. Tab-complete shows enemy arena players only.

#### Added — Systems

- **`AbilityManager.createItem()` — attribute modifier parser**: YAML `attributes:` list now
  supported for all legendary items. Format:
  ```yaml
  attributes:
    - attribute: "GENERIC_ATTACK_DAMAGE"
      value: 6.0
      operation: "ADD_NUMBER"
      slot: "MAINHAND"
  ```
  Used by Aiglos to set Attack Damage 7, Speed 1.6/s, Reach 3.5 via item meta without code.

- **`AnvilListener`** (`listener/AnvilListener.java`): custom anvil rules.
  Artemis Bow Power is allowed up to VII (lifts vanilla cap of V). All other legendary items:
  anvil combinations that would strip the `hg_ability` PDC tag are cancelled.

- **`PlayerDeathListener` — player head drop**: on PvP kill, drops victim's `PLAYER_HEAD`
  (with correct skin via `SkullMeta.setOwningPlayer()`). Configurable:
  `game.death.drop-head: true` and `game.death.drop-chance: 1.0` in `config.yml`.
  Head always drops even if `drop-items: false` (used as craft ingredient for Warpick/Aiglos).

- **`DeathNoteCommand` + `DeathNoteTabCompleter`**: `/dn <name>` command and tab-complete.

#### Recipe Updates

All recipe changes made during legend rewrite pass:

| Weapon | Change |
|--------|--------|
| Warpick | New recipe: Diamond Pickaxe + Player Head + Redstone Block × 6 + Deepslate Diamond Ore × 12 + Netherrack × 8 + TNT × 5 + Enchanted Golden Apple + Fire Charge + Echo Shard |
| Dragon Katana | New recipe: Netherite Sword + Ender Eye × 8 + Obsidian × 16 + Blaze Rod × 12 + Enchanted Golden Apple + Dragon Egg |
| Artemis Bow | New recipe: Bow + Amethyst Shard × 16 + Echo Shard × 8 + Diamond × 32 + Prismarine Shard × 6 + Enchanted Golden Apple + Lightning Rod |
| Aiglos | New recipe: Trident + Ice × 8 + Packed Ice × 3 + Diamond Ore × 16 + Snow Block × 8 + Gold Block × 3 + Enchanted Golden Apple + Player Head + Echo Shard |
| Armadillo Detonator | New recipe: Crossbow + Armadillo Scute × 8 + Echo Shard × 3 + Sandstone × 16 + Gunpowder × 8 + Redstone × 6 + Golden Apple × 8 + Bee Nest + Beehive + Player Head |
| Beehive Blaster | New recipe: Crossbow + Honey Block × 4 + Bee Nest × 3 + Honey Bottle × 2 + Iron Ingot × 12 + Gunpowder × 16 + Echo Shard × 5 + Enchanted Golden Apple + Player Head |
| Crimson Chainsword | New recipe: Diamond Sword + Iron Block × 8 + Redstone Block × 6 + Echo Shard × 4 + Blaze Rod × 8 + Fire Charge × 4 + Diamond × 9 + Player Head + Enchanted Golden Apple + Netherite Scrap × 2 + Iron Ingot × 4 |
| Corrupted Crossbow | New recipe: Crossbow + Fermented Spider Eye × 4 + Diamond × 3 + Nether Wart × 4 + Spider Eye × 12 + Brown Mushroom × 12 + Sugar Cane × 10 + Enchanted Golden Apple + Echo Shard + Player Head |
| Ghastly Whistle | New recipe: Dried Ghast × 5 + Diamond × 4 + Saddle + Blaze Rod × 4 + Fire Charge × 4 + Leather × 3 + Ghast Tear × 3 |
| Fountain of Youth | New recipe: Water Bucket × 4 + Gold Block × 4 + Diamond + Golden Apple × 8 + Glistering Melon × 8 + Blaze Powder × 3 |
| Excalibur | New recipe: Gold Block × 36 + Emerald Block × 4 + Diamond Sword + Enchanted Golden Apple |
| Ender Bow | New recipe: Diamond × 28 + Clock + Ender Pearl × 16 + End Stone × 8 + Purple Dye × 6 + Chorus Fruit × 4 + Shulker Shell × 2 + Echo Shard × 2 + Blaze Powder × 2 + Slime Block |
| Reaper Scythe | New recipe: Wither Skeleton Skull × 3 + Candle + Bone × 3 + Stick × 4 + Bone Block × 8 + Soul Sand × 8 + Wither Rose × 4 + Enchanted Golden Apple + Echo Shard × 2 + Player Head + Rotten Flesh × 3 |
| Death Note | New recipe: Book + Enchanted Book + Paper × 8 + Ink Sac × 15 + Wither Skeleton Skull × 3 + Player Head × 3 + Rotten Flesh × 12 + Bone × 12 + Enchanted Golden Apple × 2 + Echo Shard × 3 + Nether Star + Shulker Shell × 3 |

---



### Fixed — 13 issues across 14 files

#### High

- **BUG-REG-01 · 8 ability classes — `registerEvents()` in constructor causes double event-handling on reload**
  (`CloudSword.java`, `VoidStaff.java`, `WitherSickles.java`, `Excalibur.java`, `EmeraldBlade.java`,
  `MidasSword.java`, `VillagerWand.java`, `ChainsawSword.java`, `HypnosisStaff.java`)
  Each of these abilities called `Bukkit.getPluginManager().registerEvents(this, plugin)` inside
  their constructor. On plugin reload, `AbilityRegistry` instantiates fresh objects, but the old
  instances' handlers are never unregistered — both the old and new instances fire for every
  event, causing every hit, click, and projectile event to be processed twice.
  Fix: moved all `registerEvents()` calls out of constructors and into
  `AbilityRegistry.register()`, which is called exactly once per ability instance.
  `AbilityRegistry.register()` now checks `ability instanceof Listener` and registers
  automatically, so abilities only need to implement `Listener` — no manual registration needed.

- **BUG-OD-01 · `ObsidianDagger` — standalone passive `BukkitRunnable` leaks on reload**
  (`ObsidianDagger.java`)
  `startPassiveTask()` spawned a new `BukkitRunnable` on every class instantiation. After N
  reloads there were N concurrent tasks all granting Resistance to low-health players.
  Same root cause as BUG-AB-01 (fixed in 4.5.1 for ArtemisBow).
  Fix: removed `startPassiveTask()` entirely. Low-health Resistance passive migrated to
  `passiveTick()`, driven by the single `UnifiedPassiveTicker` in `AbilityManager`.

- **BUG-HS-01 · `HypnosisStaff` — minionTask has no reference, cannot be cancelled on shutdown**
  (`HypnosisStaff.java`)
  `startMinionTask()` spawned a `BukkitRunnable` but discarded the returned `BukkitTask`
  reference. On plugin shutdown, `AbilityManager.shutdown()` cancelled the unified passive
  ticker but had no way to reach the minion task — it kept running indefinitely after disable,
  pathfinding mobs toward offline players.
  Fix: stored the task as `private BukkitTask minionTask`. Added `cleanup()` override that
  cancels the task and calls `cleanupMobs()` for every tracked owner, removing all controlled
  mobs from the world.

- **BUG-GH-01 · `GolemHammer` — `disable()` is never called, passiveTask never cancelled**
  (`GolemHammer.java`)
  `GolemHammer` stored its passive task in a field and provided a `disable()` method to cancel
  it, but nothing ever called `disable()` — not `AbilityManager.shutdown()`, not
  `Main.onDisable()`. The task ran forever after plugin disable, checking and modifying
  knockback-resistance attributes on every online player.
  Fix: renamed `disable()` to `cleanup()` (implementing the new `Ability.cleanup()` contract)
  so that `AbilityRegistry.shutdownAll()` picks it up automatically. `disable()` is kept as a
  `@Deprecated` delegate for any external callers.

- **BUG-JOIN-01 · `PlayerJoinListener` — `loadPlayerStats()` blocks main thread on every join**
  (`PlayerJoinListener.java`)
  `loadPlayerStats()` executed a JDBC query synchronously on the server's main thread. With a
  remote MySQL database even a 20 ms query introduces a measurable tick-time spike and can
  cause lag for all players when multiple players join simultaneously.
  Fix: wrapped the call in `Bukkit.getScheduler().runTaskAsynchronously()`.

- **BUG-JOIN-02 · `PlayerJoinListener` — `savePlayerStats()` immediately after `loadPlayerStats()` overwrites DB stats with zeroes**
  (`PlayerJoinListener.java`)
  `savePlayerStats()` was called one line after `loadPlayerStats()` with the comment
  "Ensure record exists". Because `loadPlayerStats()` is now async (and was previously also
  a blocking call that hadn't finished before the save began), `PlayerData` still held its
  default values (kills=0, deaths=0, wins=0…) at save time. For any returning player, their
  stats were reset to zero on every server join.
  Fix: removed the erroneous `savePlayerStats()` call. New player records are created
  correctly by the `ON CONFLICT DO UPDATE` / `INSERT OR REPLACE` upsert in `DatabaseManager`
  the first time a real stat change is saved (on death, win, or disconnect).

#### Medium

- **BUG-SHUT-01 · `AbilityManager.shutdown()` — only cancels the unified ticker; standalone tasks in GolemHammer, HypnosisStaff, ObsidianDagger, DeathNote are not cleaned up**
  (`AbilityManager.java`, `AbilityRegistry.java`, `Ability.java`)
  Three abilities (GolemHammer, HypnosisStaff, ObsidianDagger) owned standalone
  `BukkitRunnable` tasks that nothing cancelled on shutdown. `DeathNote` had a working
  `cleanup()` method but it required a specific cast in `Main.onDisable()` — easy to break
  during refactors.
  Fix: added `default void cleanup() {}` to the `Ability` interface. Added
  `AbilityRegistry.shutdownAll()` which iterates all registered abilities and calls
  `cleanup()` on each. `AbilityManager.shutdown()` now delegates to `shutdownAll()` after
  cancelling the unified ticker. `Main.onDisable()` no longer needs any per-ability special
  cases.

- **BUG-ARENA-01 · `Arena` — 14 public fields bypass thread-safe getters/setters**
  (`Arena.java`)
  `name`, `world`, `cornucopia`, `spawns`, `hologramMap`, `holograms`, `originalBlocks`,
  `blockQueue`, `activeLegendaries`, `configuredLegendaries`, `temporaryLegendaries`,
  `pvpEnabled`, `netherEnabled`, `endEnabled`, `lifestealEnabled`, `hardcoreEnabled`,
  `veinminerEnabled`, `enabled`, `netherWorld`, and `maxPlayers` were all `public`.
  Any class could modify them without going through the synchronized setters, silently
  breaking thread-safety guarantees on state, timer, and gameEnded.
  Fix: changed all fields to `private`. Added missing accessor methods
  (`getHologramMap()`, `getHolograms()`, `getOriginalBlocks()`, `getBlockQueue()`,
  `getTemporaryLegendaries()`, `getActiveLegendaries()`, `setActiveLegendaries()`,
  `getNetherWorld()`, `setNetherWorld()`). Updated all call-sites in `ArenaManager`,
  `ArenaListener`, `DungeonManager`, and `MatchService` to use the new accessors.

- **BUG-UM-01 · `UpdateManager` — missing connection timeout causes async thread to hang indefinitely**
  (`UpdateManager.java`)
  `HttpURLConnection` was opened with no `connectTimeout` or `readTimeout`. If Modrinth's
  API was unreachable (DNS failure, network outage, server down), the async thread blocked
  forever, leaking a thread for the entire server uptime.
  Fix: added `conn.setConnectTimeout(5000)` and `conn.setReadTimeout(5000)` — the check
  fails gracefully within 5 s and logs a warning instead of hanging.

#### Low

- **DEAD-CODE-01 · `ArenaManager` — three `buildStyle*` placeholder methods are unreachable dead code**
  (`ArenaManager.java`)
  `buildStyleClassic()`, `buildStyleUltimate()`, and `buildStyleSpeedSilver()` were never
  called from anywhere in the codebase. They placed simple flat platforms without integrating
  into the Cornucopia pipeline (no chest scanning, no legendary hologram placement, no queue
  processing). They existed as early prototypes and were never completed.
  Fix: removed all three methods. The Cornucopia pipeline (`buildCornucopia()` →
  `startCornucopiaPhase2()` → `scanForChests()` → `startPhase3()` → `finalizeArena()`)
  is the sole code path for arena construction.

### Changed
- `pom.xml` — version `4.5.6` → `4.5.7`.
- `plugin.yml` — version `4.5.6` → `4.5.7`; description updated.

---


### Fixed — 5 issues across 5 files

#### High

- **BUG-PM-01 · `PlayerManager` / `MatchService` / `WithdrawCommand` — stolen heart item is unclickable after Component-API migration**
  (`PlayerManager.java`, `MatchService.java`, `WithdrawCommand.java`)
  `PlayerManager.onPlayerInteract` identified stolen hearts by comparing `meta.getDisplayName()` to
  the language-file string. After the 4.5.4 fix migrated item names to `meta.displayName(Component)`,
  the legacy `getDisplayName()` call returns `""` — the comparison always failed and hearts could
  never be consumed. Fix: a `NamespacedKey` PDC tag `stolen_heart` is now written to every heart
  item on creation (`MatchService` and `WithdrawCommand`). `PlayerManager` reads the PDC tag
  instead of the display name.

- **BUG-EB-01 · `EnderBow` — position swap has no team check**
  (`EnderBow.java`)
  `performProjectileHit` swapped the shooter and any hit `LivingEntity` unconditionally.
  Shooting a teammate teleported them to the shooter's position. Fix: added
  `isFriendlyFire` check before performing the swap; returns early for friendly targets.

#### Medium

- **BUG-CB-01 · `CornucopiaBuilder` / `ArenaManager` — build progress spams all online players**
  (`CornucopiaBuilder.java`, `ArenaManager.java`)
  `clearArea()` used `Bukkit.broadcastMessage()` for three status messages
  ("Pre-loading chunks", "Clearing area", "Clearing N%"). These messages were delivered
  to every player on the server, not just players in the arena being built.
  Fix: replaced all four `broadcastMessage` calls with `plugin.getLogger().info()`.

#### Low

- **DUPE-IMPORT · Duplicate `import` statements across 8 manager/model files**
  (`ScoreboardManager.java`, `ArenaManager.java`, `DungeonManager.java`, `Arena.java`,
  `LegendaryRecipe.java`, `HologramManager.java`, `LobbyManager.java`, `MessageManager.java`)
  A previous tooling pass accidentally injected the `ColorUtil` import statement 9–27 times
  per file. While harmless at runtime, it produced compiler noise and inflated source size.
  Fix: de-duplicated all import blocks; one import per class per file.

### Changed
- `pom.xml` — version `4.5.5` → `4.5.6`.
- `plugin.yml` — version `4.5.5` → `4.5.6`.

---

## [4.5.5] — Bug Sweep Pass 2 (2026-03-20)

### Fixed — 7 issues across 6 files

#### High

- **BUG-GT-01 · `GameTask` — world border Phase 1 and Phase 2 execute simultaneously for 60 seconds**
  (`GameTask.java`)
  Phase 1 ran while `timer ∈ (grace, main]` and Phase 2 while `timer ∈ (main−60, main]`.
  During the last 60 s of the main phase both conditions were true, so Phase 2 called
  `border.setSize()` immediately after Phase 1 — resetting the border back to `middleSize`
  on every tick and then fast-shrinking to `dmSize`. Players saw the border snap suddenly
  rather than transitioning smoothly.
  Fix: made the ranges mutually exclusive — Phase 1 covers `(grace, main−60]` and
  Phase 2 covers `(main−60, main]` (else-if chain).

- **BUG-FM-01 · `MatchService` — feast never spawns from the second match onward**
  (`MatchService.java`)
  `resetArena()` called `feastManager.cleanup(arenaName)` which removed feast chests but
  did **not** clear the `feastSpawned` guard set in `FeastManager`. On the next match the
  guard was still set, so `spawnFeast()` was never called again for the lifetime of the server.
  Fix: changed the call to `feastManager.resetArena(arenaName)` which calls both
  `cleanup()` and `feastSpawned.remove()`.

- **BUG-CD-01 · `MatchService` — countdown cancels immediately because `getGamePlayers()` is empty during `STARTING` state**
  (`MatchService.java`)
  The not-enough-players check in `startCountdown` used:
  `arena.getGamePlayers().values().stream().filter(GamePlayer::isAlive).count() < 2`
  `gamePlayers` is not populated until the match transitions to `PLAYING`, so `aliveCount`
  was always 0 → the countdown broadcast "not_enough_players" and cancelled on the very
  first tick.
  Fix: replaced the alive-count check with `arena.getPlayers().size() < 2`, which counts
  the raw joined-player list that is populated during `STARTING`.

#### Medium

- **BUG-AB-04 · `DeathNote` — `sendActionBar(String)` deprecated on Paper 1.21**
  (`DeathNote.java`)
  The progress bar was sent via `player.sendActionBar(String)`, a deprecated overload
  removed in newer Paper builds.
  Fix: wrapped the string with `LegacyComponentSerializer.legacySection().deserialize()`.

#### Low

- **BUG-AB-05 · `VoidStaff`, `SoulGauntlet`, `DeathScythe`, `WitherSickles` — `sendActionBar(String)` deprecated (13 call-sites)**
  (`VoidStaff.java`, `SoulGauntlet.java`, `DeathScythe.java`, `WitherSickles.java`)
  Same issue as BUG-AB-04. All 13 remaining String-based action-bar calls replaced with
  `LegacyComponentSerializer.legacySection().deserialize()`.

### Changed
- `pom.xml` — version `4.5.4` → `4.5.5`.
- `plugin.yml` — version `4.5.4` → `4.5.5`.

---

## [4.5.4] — Item Font Fix Pass (2026-03-20)

### Fixed — 9 issues across 9 files

#### High

- **BUG-FONT-01 · All legend weapon names and lore appear in italic (Minecraft 1.20.5+)**
  (`AbilityManager.java`, `LegendaryRecipe.java`, `MidasSword.java`, `EmeraldBlade.java`,
  `CloudSword.java`, `LobbyManager.java`, `MatchService.java`, `WithdrawCommand.java`)
  Minecraft 1.20.5 changed the default rendering style for custom item display names to
  `italic=true`. The legacy `meta.setDisplayName(String)` and `meta.setLore(List<String>)`
  APIs do not override this default, so every legend weapon name and every lore line was
  rendered in italic regardless of `&l` / `&r` codes.
  Fix: `ColorUtil` gained two new static helpers — `component(String legacySection)` and
  `componentOf(String ampStr)` — that deserialize a §-coded string into an Adventure
  `Component` with `TextDecoration.ITALIC` explicitly set to `false`. All eight call-sites
  were updated to use `meta.displayName(Component)` and `meta.lore(List<Component>)`.

#### Medium

- **COMPILE-01 · `ColorUtil.java` had methods placed outside the class body**
  (`ColorUtil.java`)
  A Python append script closed the class with `}` on line 35, then continued writing
  the new `component()` and `componentOf()` methods outside the class. The compiler
  treated them as implicit class declarations (a Java 25 preview feature) and raised
  three errors under `-source 21`. Fix: rewrote `ColorUtil.java` cleanly with all four
  methods (`colorize`, `strip`, `component`, `componentOf`) inside a single class body.

### Changed
- `pom.xml` — version `4.5.3` → `4.5.4`.
- `plugin.yml` — version `4.5.3` → `4.5.4`.

---

## [4.5.3] — Missing Messages Fix Pass (2026-03-20)

### Fixed — 3 issues across 5 files

#### High

- **BUG-CS-01 · `CloudSword` — cloud blocks placed synchronously inside `PlayerMoveEvent` cause constant stutter**
  (`CloudSword.java`)
  `performMove` called `block.setType(WHITE_STAINED_GLASS)` directly inside the
  `PlayerMoveEvent` handler. The server physics engine snapped the player up ~0.1 blocks
  on the same tick the block appeared beneath their feet, producing continuous jitter while
  walking.
  Fix: deferred all block placements to the next server tick via `Bukkit.getScheduler().runTask()`.
  Added an airborne check — clouds are only placed when the block directly below the player
  is `AIR` or an existing cloud block, preventing unnecessary glass floors on flat terrain.

- **BUG-MSG-01 · 8+ message keys missing from all language files cause "§cMessage not found:" spam**
  (`en.yml`, `vi.yml`, `de.yml`, `es.yml`, `fr.yml`, `ja.yml`, `ko.yml`, `pt.yml`, `ru.yml`, `zh.yml`)
  The following keys were used in code but absent from every language file:
  `not_enough_players`, `countdown_cancelled`, `border_shrinking_main`, `border_shrinking_dm`,
  `cooldown_ready_actionbar`, `feast_announcement`, `feast_spawned`, `dragon_katana_shoot`,
  `scoreboard_state_waiting/starting/playing/deathmatch/ending`, `scoreboard_pvp_enabled/protected`,
  and ~10 additional gameplay keys. Every trigger printed the red "Message not found:" fallback
  to all arena players.
  Fix: added all missing keys with appropriate English text to `en.yml` and `vi.yml` with
  translated strings; remaining 8 language files received English fallback values.

#### Medium

- **BUG-CS-02 · `CloudSword` — `cloud_walk_enabled` / `cloud_walk_disabled` keys missing → "Message not found:" on toggle**
  (`CloudSword.java`)
  `performRightClick` called `plugin.getMessageManager().getMessage("cloud_walk_enabled")` and
  `getMessage("cloud_walk_disabled")`, neither of which existed in any language file.
  Fix: replaced both `sendMessage` calls with `player.sendActionBar(Component)` using inline
  text — no language key dependency.

- **BUG-PT-01 · `PoseidonsTrident` — `startPassive()` `BukkitRunnable` leaks on reload**
  (`PoseidonsTrident.java`)
  The constructor called `startPassive()` which spawned a global `BukkitRunnable` iterating
  all arenas every 20 ticks. Identical root cause to the passive-task leaks fixed in 4.5.2.
  Fix: removed `startPassive()`; conduit-power and dolphins-grace logic migrated to `passiveTick()`.

- **BUG-PT-02 · `PoseidonsTrident` — lightning bonus damage never fires for thrown trident hits**
  (`PoseidonsTrident.java`)
  When the trident is thrown, `EntityDamageByEntityEvent.getDamager()` is the `Trident` entity,
  not the `Player`. The existing `performHit` override checked only for a `Player` damager, so
  the water lightning bonus never triggered for thrown trident contacts.
  Fix: added `performProjectileHit` override that checks if the hit entity is in water and
  applies the bonus damage directly.

### Changed
- `pom.xml` — version `4.5.2` → `4.5.3`.
- `plugin.yml` — version `4.5.2` → `4.5.3`.

---

## [4.5.2] — Artemis Bow & Passive Ticker Fix Pass (2026-03-18)

### Fixed — 8 issues across 5 files

#### High

- **AB-OPEN-01 · `ArtemisBow` — memory leak: unbounded homing-arrow runnables accumulate on rapid fire**
  (`ArtemisBow.java`)
  Each call to `performProjectileLaunch` spawned 5 independent `BukkitRunnable` instances
  (one per arrow). Players firing continuously in rapid succession could accumulate dozens of
  concurrent runnables with no upper bound, gradually degrading server TPS.
  Fix: added a per-player `AtomicInteger` counter (`activeArrows` map, capped at 15 =
  3 simultaneous volleys). Each runnable increments on start and decrements on cancel/expire.
  New volleys are silently skipped if the cap is reached. Counter is cleaned up on
  `performQuit` and `performDeath`.

- **AB-OPEN-02 · `ArtemisBow` — `performProjectileHit` deals lightning damage without team check**
  (`ArtemisBow.java`)
  After BUG-10 was fixed (switched to `strikeLightningEffect` + manual `target.damage()`),
  the new manual damage call had no `isFriendlyFire` guard. Result: Artemis Bow arrows could
  deal lightning damage to teammates.
  Fix: added `if (target instanceof Player tp && isFriendlyFire(shooter, tp)) return;` before
  calling `target.damage()`. Also added a null-guard: if `projectile.getShooter()` is not a
  `Player`, the method returns early.

- **AB-OPEN-03/05 · `ArtemisBow` — Sky Strike targets first entity in list, not closest**
  (`ArtemisBow.java`)
  `getNearbyEntities(20, 20, 20)` returns entities in no guaranteed order. Sky Strike broke
  out of the loop on the first non-team entity — sometimes picking a distant mob over an
  enemy player standing 2 blocks away.
  Fix: replaced early-break loop with a full `distanceSquared` scan to find the closest
  valid target. Search radius is now configurable via `sky_strike_radius` (default 20).

- **AB-OPEN-04 · `ArtemisBow` — fan arrows use default Minecraft damage (~0.5 hearts) instead of bow charge**
  (`ArtemisBow.java`)
  `player.launchProjectile(Arrow.class)` spawns arrows with base damage 2.0 — they ignore
  the actual bow charge the player applied. Only the centre (original) arrow inherited the
  real charge damage.
  Fix: added `extraArrow.setDamage(baseArrow.getDamage())` when spawning each fan arrow so
  all 5 arrows deal equal damage consistent with bow charge.

#### Medium

- **BUG-HH-02 · `HadesHelm` — standalone passive `BukkitRunnable` leaks on reload**
  (`HadesHelm.java`)
  `startPassive()` spawned a new `BukkitRunnable` on every class instantiation. On plugin
  reload the old instance was discarded but the runnable kept running → N tasks after N reloads.
  Same root cause as BUG-AB-01 (fixed in 4.5.1 for ArtemisBow).
  Fix: removed `startPassive()` entirely. Night Vision, Invisibility-while-sneaking, and
  Fire Aura logic migrated to `passiveTick()` — driven by the single `UnifiedPassiveTicker`.

- **BUG-GC-01 · `GuardianCannon` — standalone passive `BukkitRunnable` leaks on reload**
  (`GuardianCannon.java`)
  Same root cause as BUG-HH-02. `startPassiveTask()` ran on instantiation → N tasks
  after N reloads.
  Fix: Dolphins Grace water passive migrated to `passiveTick()`.

- **BUG-LS-01 · `LichStaff` — standalone passive `BukkitRunnable` leaks on reload**
  (`LichStaff.java`)
  Same root cause as BUG-HH-02. `startPassiveTask()` ran on instantiation → N tasks.
  Fix: Frost Walker passive migrated to `passiveTick()`. The ammo / shoot system is
  unaffected — it correctly uses per-use `BukkitRunnable` instances that self-cancel.

#### Low

- **plugin.yml — `api-version` set to `4.4.1` (invalid format)**
  (`plugin.yml`)
  Paper requires `api-version` to be a major.minor string (`1.21`), not a plugin version
  string. An invalid value causes a warning on every server start.
  Fix: changed `api-version: 4.4.1` → `api-version: 1.21`.

### Changed
- `pom.xml` — version `4.5.1` → `4.5.2`.
- `plugin.yml` — version `4.5.1` → `4.5.2`; `api-version` corrected to `1.21`; description updated.

---

## [4.5.1] — Logic Bug Fix Pass (2026-03-16)

### Fixed — 12 logic bugs across 11 weapon files

#### Critical

- **BUG-PL-01 · `PhantomLongbow` — ability completely non-functional** (`PhantomLongbow.java`)
  `arrow.remove()` was called immediately in `performProjectileLaunch`, then a `BukkitRunnable`
  was scheduled 1 tick later. By the time the runnable ran, `projectile.isValid()` was `false` →
  instant cancel → ability only granted invisibility, nothing else happened.
  Fix: stop calling `remove()`. Instead set `arrow.setPickupStatus(DISALLOWED)`,
  `setGravity(false)`, `setDamage(0)` so the arrow flies invisibly. The tracking runnable
  now checks `isValid() || isOnGround() || isDead()` and calls `arrow.remove()` itself.

#### High

- **BUG-FY-02 · `FountainOfYouth` — inverted team check heals enemies, skips teammates**
  (`FountainOfYouth.java`)
  `isFriendlyFire` returns `true` for teammates. The condition
  `if (isFriendlyFire(player, target)) continue` was therefore *skipping* teammates and
  *healing* enemies — the exact opposite of the comment.
  Fix: added `!` → `if (!isFriendlyFire(player, target)) continue`.

- **BUG-VS-01 · `VoidStaff` — portal CD starts on first portal, blocks second placement**
  (`VoidStaff.java`)
  `portalCDs.put(...)` ran unconditionally after any portal placement. Player placed portal #1,
  40 s CD started, clicking again to place portal #2 hit the CD guard and was blocked — so a
  connected portal pair could never be created.
  Fix: moved `portalCDs.put(...)` inside the `if (connected)` branch only.

- **BUG-CC-03 · `CorruptedCrossbow` — extra bolts missing `hg_ability` metadata, no hit effects**
  (`CorruptedCrossbow.java`)
  Extra bolts were tagged `corrupted_extra` and `corrupted_bolt` but not `hg_ability`.
  `AbilityManager.handleProjectileHit` only dispatches `performProjectileHit` when
  `hg_ability` is present. Result: extra bolts dealt only vanilla arrow damage with no
  Poison or Weakness effects.
  Fix: added `extra.setMetadata("hg_ability", ...)` when spawning each extra bolt.

#### Medium

- **BUG-CSW-01 · `CrimsonChainsword` — bleed only fires 1 damage tick out of 5**
  (`CrimsonChainsword.java`)
  `runTaskTimer(10L, 10L)` increments `ticks` 0→1→2→3→4 then cancels. Damage condition was
  `ticks % 20 == 0` — only `ticks=0` satisfies this → 1 damage tick instead of 5.
  Fix: removed `% 20 == 0` check. Period is already the interval; every call = one bleed tick.

- **BUG-SB-01 · `ShadowBlade` — Shadow Step has no team check, teleports behind teammates**
  (`ShadowBlade.java`)
  Shadow Step's target loop had no `isFriendlyFire` guard, allowing teleport behind allies.
  Fix: added `if (e instanceof Player tp && isFriendlyFire(player, tp)) continue;`.

- **BUG-RH-01/02 · `RavagerHorn` — lazy location capture + no Ravager cleanup on quit/death**
  (`RavagerHorn.java`)
  (1) Stampede lambdas called `player.getLocation()` lazily — waves diverged if player moved
  during the 16-tick animation. Fixed: capture `castLoc = player.getLocation().clone()` before
  the loop.
  (2) No `performQuit`/`performDeath` hook — Ravager persisted in the world after the player
  disconnected or died. Fixed: added `activeRavagers` map + `dismountAndRemove()` called on
  both events.

- **BUG-DN-01 · `DeathNote` — passive task runs every tick (20×/s), expensive raytracing**
  (`DeathNote.java`)
  `runTaskTimer(1L, 1L)` = 20 `rayTrace(range:50)` calls per player per second. With 10 players
  = 200 raytrace/s. Changed `TASK_PERIOD` from `1L` to `3L` (~7 raytrace/s/player), still
  provides smooth progress bar updates.

#### Low

- **BUG-VW-01 · `VillagerWand` — `taggedForReward` map memory leak** (`VillagerWand.java`)
  Entries were only removed on entity death. Entities that fled, despawned, or were removed by
  arena reset left their UUID in the map indefinitely. Fixed: value now stores an expiry
  timestamp (`now + rewardWindow`). `onEntityDeath` checks `now <= expiry`. A cleanup task
  runs every 20 s to evict stale entries.

- **BUG-SG-01 · `SoulGauntlet` — action bar hardcodes `-3` instead of `blastCost`** (`SoulGauntlet.java`)
  If `soul_blast_souls` was changed in config, the displayed soul count was wrong.
  Fix: `(charges - 3)` → `(charges - blastCost)`.

- **BUG-WS-02 · `WitherSickles` — `dualWieldActive` desync when offhand is overwritten**
  (`WitherSickles.java`)
  If a player manually placed another item into the offhand slot, `dualWieldActive` kept the
  UUID entry, blocking `equipOffhand` from re-running. Fixed `onInventoryClick` to detect this
  state and call `dualWieldActive.remove(uuid)`.

- **BUG-AB-01 · `ArtemisBow` — standalone passive `BukkitRunnable` leaks on reload**
  (`ArtemisBow.java`)
  `startPassive()` spawned a new task on every class instantiation. On plugin reload the old
  instance was discarded but the runnable kept running → N tasks after N reloads.
  Migrated to `passiveTick()` hook consumed by the single `UnifiedPassiveTicker` in
  `AbilityManager`. `startPassive()` and its runnable removed entirely.

### Changed
- `pom.xml` — version `4.5.0` → `4.5.1`.
- `plugin.yml` — version `4.5.0` → `4.5.1`; description updated.

---

## [4.5.0] — Weapon Overhaul (2026-03-16)

### Changed — 4 legendary weapons redesigned

> Mechanic ideas for the 4 weapons below were inspired by
> **DemoRaHoplite Weapons** by Demora (MIT licence).
> Plugin page: <https://modrinth.com/plugin/demorahopliteweapons>

#### `VoidStaff.java` — Dual-mode system
- **Before:** Single sneak+click portal / click shulker logic with no UI feedback.
- **After:** Full mode-switching system (ported concept from DemoRaHoplite):
  - `LEFT-CLICK` toggles between **Portal Mode** and **Shulker Mode**.
  - Each mode has its **own independent cooldown** (portal 40 s, shulker 30 s).
  - **Action bar** shows current mode + remaining cooldown, refreshes on move
    and on item-switch (`PlayerItemHeldEvent`).
  - Cleanup on quit/death prevents state leaks.
  - `StaffMode` enum added for extensibility.

#### `Excalibur.java` — True invincibility
- **Before:** `PotionEffect RESISTANCE V` — very strong but not true invincibility
  (high enough burst damage could still kill the player).
- **After:** `event.setCancelled(true)` at `EventPriority.HIGHEST` (concept from
  DemoRaHoplite) — **100 % damage blocked** for the entire 5 s duration.
  - Attacker hears `IRON_GOLEM_HURT` sound when a hit is deflected.
  - Reflection + projectile deflect mechanics **preserved** from the old version.
  - `activeInvincibility` map with expiry timestamp replaces potion effect check.
  - `isInvincible()` guard added so `performDamaged()` (reflect) never fires
    during the invincibility window.
  - Cleanup on quit/death.

#### `WitherSickles.java` — Dual-wield + throw mechanic
- **Before:** Only AOE on-hit wither + lifesteal. No right-click action.
  No `amount: 2` usage in gameplay.
- **After:** Full dual-wield system (mechanic inspired by DemoRaHoplite):
  - **Auto-equip offhand:** when a player holds the Wither Sickles in main-hand,
    a protected copy is placed in the off-hand slot automatically.
  - **Off-hand copy is locked** — `InventoryClickEvent`, `InventoryDragEvent`,
    `PlayerDropItemEvent`, `PlayerSwapHandItemsEvent` all cancelled for it.
    Copy is removed when the player switches away.
  - **Right-click throw** (CD: 7 s, dmg 12, Wither II × duration × 2):
    projectile travels as a tick-based raycast with SOUL + WITCH particle trail.
  - `passiveTick()` shows throw cooldown in action bar while holding.
  - Legacy AOE wither + lifesteal **preserved** on melee hit.

#### `LichStaff.java` — Balance pass
- **Before:** Reload time `7 000 ms` — could re-fire 4 ice-balls every 7 s, far
  too strong in arena PvP.
- **After:** Reload time raised to **`30 000 ms`** (30 s), matching DemoRaHoplite's
  `cooldown_seconds: 30` for the equivalent weapon. This makes ammo management
  meaningful — players must choose when to spend their 4 shots.
  - Between-shot cooldown tightened to `800 ms` (from 500 ms) to prevent
    accidental double-fire while staying responsive.
  - Reload-complete message now includes countdown (`"Recharging in 30s..."`).
  - Reload-out sound changed to `WITHER_DEATH` (thematic) instead of
    `BEACON_ACTIVATE`.

### Updated
- `legend-items/void_staff.yml` — lore updated to describe mode-switching.
- `legend-items/excalibur.yml` — lore updated to say "TRUE invincibility".
- `legend-items/wither_sickles.yml` — lore updated to describe dual-wield + throw.
- `legend-items/lich_staff.yml` — lore updated to show `30s reload` and ammo dots.
- `pom.xml` — version `4.4.1` → `4.5.0`.
- `plugin.yml` — version `4.4.1` → `4.5.0`; description updated.

---

## [4.4.1] — Hotfix patch



### Fixed

#### Low

- **L-01 · BungeeCord API deprecated in `CooldownManager`** (`CooldownManager.java`)
  Two action-bar sends used `player.spigot().sendMessage(ChatMessageType.ACTION_BAR, new TextComponent(…))` —
  BungeeCord/Spigot legacy chat API that is deprecated on Paper 1.21 and may be removed
  in a future Paper version. Replaced with Paper-native `player.sendActionBar(Component)`.
  BungeeCord imports (`net.md_5.bungee.*`) removed. Adventure `LegacyComponentSerializer`
  used for §-code → Component conversion to keep the existing colour format intact.

- **L-02 · `ChatColor.translateAlternateColorCodes()` deprecated across 9 files**
  (`ColorUtil.java`, `LegendaryRecipe.java`, `Arena.java`, `DungeonManager.java`,
  `HologramManager.java`, `MessageManager.java`, `ScoreboardManager.java`,
  `AbilityManager.java`, `LobbyManager.java`, `ArenaManager.java`)
  `ChatColor.translateAlternateColorCodes('&', s)` is a Spigot legacy API marked
  deprecated on Paper 1.21. All 9 call-sites were replaced with `ColorUtil.colorize(s)`.
  `ColorUtil` was rewritten to use `LegacyComponentSerializer.legacyAmpersand()` internally
  while still returning a §-coded `String` so all downstream `sendMessage` / `setDisplayName`
  call-sites work unchanged. A `ColorUtil.strip()` convenience method was also added using
  Adventure's `PlainTextComponentSerializer`.

- **L-03 · `Player.setResourcePack()` deprecated on Paper 1.21** (`ConnectionListener.java`)
  Both `setResourcePack(url)` and `setResourcePack(url, hash)` overloads are deprecated.
  The replacement is Adventure's `ResourcePackRequest` / `ResourcePackInfo` API, sent via
  `player.sendResourcePacks(request)`. ConnectionListener now builds a `ResourcePackRequest`
  with an optional `required` flag and configurable prompt text (see config keys
  `resource-pack.required` and `resource-pack.prompt`).

- **L-04 · Loot tables hardcoded in `LootTableManager.java`** (`LootTableManager.java`, `loot.yml`)
  All 30 loot entries (11 common, 11 uncommon, 8 rare) were hardcoded — impossible
  to change without recompiling. Extracted to a new bundled resource `loot.yml`.
  `LootTableManager` now calls `plugin.saveResource("loot.yml", false)` on first run
  then loads via `YamlConfiguration`. A `reload()` method is exposed so changes take
  effect with `/perf reload` (or equivalent) without restarting the server.
  Malformed entries log a warning and are skipped; the remaining valid entries load
  normally.

- **L-05 · 4 dungeon types are invisible aliases** (`DungeonManager.java`)
  `buildUndergroundDungeon` and `buildAncientMineshaft` were one-line delegates to
  `buildCrypt` and `buildGoldMine` with no documentation. Server operators had no
  way to know they were paying the performance cost of spawning a dungeon that was
  identical to another one. Both methods are now annotated with deprecation Javadoc
  explaining the alias relationship, and a startup INFO message tells operators which
  config keys to set to `false` to remove the duplicates:
  `dungeons.underground.enabled: false` and `dungeons.mineshaft.enabled: false`.

- **L-08 · `ConnectionListener` — reconnect during active match loses game state** (`ConnectionListener.java`, `ArenaListener.java`, `GamePlayer.java`)
  When a player disconnected and reconnected during a PLAYING or DEATHMATCH phase,
  the `PlayerJoinEvent` handler only sent the resource pack. The player spawned at
  world spawn (or last logout point), in the wrong game mode, with no scoreboard.
  Three changes:
  1. `GamePlayer` gains a `lastLocation` field updated by `ArenaListener.onPlayerMove()`
     on every block-level movement.
  2. `ConnectionListener.onPlayerJoin()` now calls `MatchService.restoreSession()`
     one tick after join to re-apply scoreboard, bossbar, and game mode.
  3. If the rejoining player was alive, they are teleported to `gp.getLastLocation()`
     so they return exactly where they were before the disconnect.

### Changed

- `config.yml` — added `resource-pack.required` and `resource-pack.prompt` keys.
- `plugin.yml` — version bumped `4.3` → `4.4`; description updated.
- `pom.xml` — version bumped `4.3` → `4.4`.
- New file: `loot.yml` — fully configurable loot tables for all three tiers.

---

## [4.3] — Stabilization Patch 3

### Fixed
- **H-04** `FeastManager` double-announce + double-spawn at match start / boundary tick.
- **H-06** `DungeonManager` mob HP fallback `40` → `60`.
- **M-04** `Excalibur` reflected mob damage onto players; bypassed team checks.
- **M-05** `Mjolnir` `rotationAngle` float overflow after ~18 h.
- **M-06** `VeinMinerListener` `maxVeinSize` cached — not hot-reloadable.
- **M-08** `ArenaListener` dead no-arg constructor using static `Main.getInstance()`.
- **M-09** `DatabaseManager.getPlayerStats()` blocked main thread — now async.
- **M-10** `PerformanceOptimizer.toggleMonitoring()` always returned `true`.
- **L-06** `AbilityDispatcher.LEGEND_KEY` static init before `onEnable()`.
- **L-07** Hardcoded test-item blacklist moved to `legendary.excluded-ids` config.
- **L-09** `buildCornucopia()` cleared block-restore map — dungeons/cages never restored.
- **L-10** `GameTask.run()` silently allowed double-tick if ever scheduled independently.

---

## [4.2] — Stabilization Patch 2

### Fixed
- **C-01** `Arena.pvpEnabled` default `false` → `true`.
- **C-06** Kill credit lost on border/mob/knockback deaths.
- **H-03** `DragonKatana` `createExplosion()` caused double damage + bypassed team checks.
- **H-01** Grace period fallback `120 s` → `30 s`.
- **M-01** 9 passive `BukkitRunnable` tasks leaked on reload → unified cancellable ticker.
- **M-02** `HashMap` → `ConcurrentHashMap` in 5 ability classes.
- **M-03** `PortalManager` `Location` map key → `String` block-coordinate key.

---

## [4.1] — Stabilization Patch 1

### Fixed
- **C-03** `max-players` default `1000` → `24`; feast announce-minutes deduped.
- **C-02** `ArenaStartEvent` fired twice per match.
- **C-05** `DeathNote` `HashMap` race → `ConcurrentHashMap`.
- **C-04** `Arena.spectators` `ArrayList` → `Collections.synchronizedList`.
- **H-05** Server reload ended matches instantly.
- **H-02** `ShadowBlade.revealPlayer()` scoped to arena players only.
- **M-07** Wrong permission node `hg.admin` → `hoplitebr.admin`.

---

## [4.0] — Initial Release

- 46 Legendary abilities (melee, ranged, staff, armor, utility).
- 11 dungeon types with custom boss mobs and themed loot.
- Cornucopia procedural builder with optional FAWE `.nbt` / `.schem` support.
- Lifesteal, Hardcore, Veinminer, and per-arena Nether world modes.
- HikariCP SQLite / MySQL persistence with crash recovery.
- Multi-language: English, French, Russian, Vietnamese.
- Soft-dependencies: PlaceholderAPI, Vault, ProtocolLib, FastAsyncWorldEdit.

## [v4.4.1] - 2026-03-13
### Added
- **10 language support** — plugin now ships with full translations for:
  | Code | Language |
  |------|----------|
  | `en` | English ✅ (default) |
  | `vi` | Tiếng Việt |
  | `fr` | Français |
  | `ru` | Русский |
  | `es` | Español |
  | `de` | Deutsch |
  | `pt` | Português (Brasil) |
  | `zh` | 简体中文 |
  | `ja` | 日本語 |
  | `ko` | 한국어 |
### Fixed
- `LangSubCommand` — hardcoded whitelist `en/vi/ru/fr` expanded to all 10 languages
- `fr.yml` / `ru.yml` — hundreds of untranslated English strings now fully translated
- All language files — added new keys: `feast_announcement`, `feast_spawned`, `cooldown_ready_actionbar`
