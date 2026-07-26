# What the expansion already gives you

A survey of `include/config/` in `expansion/1.16.2`, written because it's easy to
build a feature by hand that was one `#define` away.

**The one thing to understand first:** almost every setting defaults to
`GEN_LATEST`, and `GEN_LATEST` is `GEN_9`. So the expansion is *already* a
modernised Emerald out of the box. The interesting question is not "what QoL can
I add" — it's "what is still off, and what did I inherit without choosing it."

Setting a config to `GEN_3` reverts that one behaviour to Ruby/Sapphire/Emerald.
Changing `GEN_LATEST` itself (in `config/general.h`) reverts *everything* at
once — a blunt instrument, listed here only so nobody discovers it by accident.

---

## 1. Already on — you have these

Worth reading properly. Several are features people hand-implement without
realising they shipped.

### In battle

| Setting | What you get |
|---|---|
| `B_LAST_USED_BALL` | **R** re-throws your last ball. Hold R + D-pad to cycle ball types |
| `B_SHOW_MOVE_DESCRIPTION` | **L** opens move info during the fight |
| `B_SHOW_EFFECTIVENESS` | Effectiveness replaces the PP string — for species you've *seen* |
| `B_SHOW_TARGETS` | Spread moves highlight everything they'll hit before you commit |
| `B_SHOW_CATEGORY_ICON` | Physical/special icon in summary and relearner |
| `B_FAST_HP_DRAIN`, `B_FAST_EXP_GROW`, `B_FAST_INTRO_PKMN_TEXT` | Faster bars and intro |
| `B_CATCH_SWAP_INTO_PARTY` | Swap a fresh catch straight into the party |
| `B_RUN_TRAINER_BATTLE` | You *can* flee trainer battles (counts as a whiteout) |
| `B_RECALCULATE_STATS` | Stats recalculated after each battle |
| `B_MOVE_REARRANGEMENT_IN_BATTLE` | Gen 4+: move slots can't be reordered mid-battle |

### Overworld

| Setting | What you get |
|---|---|
| `OW_RUNNING_INDOORS` | Run indoors |
| `OW_CHOOSE_FROM_PC_AND_PARTY` | Tutors/traders can reach into your PC |
| `OW_PC_HEAL` | Gen 8+: depositing does **not** heal — a rare non-QoL default |
| `OW_WHITEOUT_CUTSCENE` | Gen 4+ whiteout handling |
| `I_REPEL_LURE_MENU` | Pick which Repel to use when one runs out |
| `OW_ENABLE_DNS` | Day/night overworld tinting |
| `P_POKERUS_ENABLED` | Pokérus, with the two vanilla bugs fixed — see below |

**The two Pokérus "bugs"**, both `TRUE` in vanilla Emerald and `FALSE` (fixed)
here. Neither is worth restoring unless you want bug-for-bug fidelity:

- **`P_POKERUS_HERD_IMMUNITY`** — vanilla picks a random party slot to infect
  from *all* slots, including Pokémon that already had Pokérus and are now
  immune. Landing on one wastes the roll entirely. So the more of your party has
  been infected before, the *lower* your odds of a new infection. The fix
  filters immune mons out of the candidate list first
  (`pokerus.c:50`), keeping the rate constant.
- **`P_POKERUS_WEAK_VARIANT`** — Pokérus has 16 strains, numbered from 0.
  Vanilla tests "does this mon have a strain?", and strain **0** reads as
  falsy, so a mon carrying strain 0 counts as uninfected and can be overwritten
  by a spreading strain, resetting its remaining days. The fix tests the
  Pokérus value as a whole instead (`pokerus.c:177`).

### Menus

| Setting | What you get |
|---|---|
| `P_SUMMARY_SCREEN_RENAME` | **Rename from the summary screen** — no Name Rater trip |
| `P_SUMMARY_SCREEN_MOVE_RELEARNER` | Relearn level-up moves from the summary screen |
| `P_SUMMARY_MOVE_RELEARNER_FULL_PP` | …at full PP |
| `P_SUMMARY_SCREEN_NATURE_COLORS` | Nature boosts coloured red/blue |
| `OW_POKEMON_OBJECT_EVENTS` | Every species usable as an overworld sprite |

### Debug (auto-disabled by `make release`)

`DEBUG_OVERWORLD_MENU` (**R + START**), `DEBUG_BATTLE_MENU` (**Select** in battle),
`DEBUG_POKEMON_SPRITE_VISUALIZER` (**Select** in summary), and `ENABLE_QUICKSTART`
(**Select** at the title screen to skip straight into a new game).

---

## 2. Off by default — the actual decision list

Nothing here is enabled. These are the real choices left.

### Strong candidates for an endgame-QoL hack

| Setting | Default | What flipping it does |
|---|---|---|
| `I_REUSABLE_TMS` | `FALSE` | TMs become infinite-use, as in Gen 5–8. Can also be cherry-picked per-TM via item importance |
| `P_ENABLE_MOVE_RELEARNERS` | `FALSE` | Turns on egg / TM / tutor move relearners |
| `P_ENABLE_ALL_LEVEL_UP_MOVES` | `FALSE` | Relearn any level-up move regardless of level |
| `P_PRE_EVO_MOVES` | `FALSE` | Learn moves from the pre-evolution |
| `I_EXP_SHARE_ITEM` | `GEN_5` | ⚠️ Currently the **held item**. `GEN_6` makes it the party-wide toggle key item |
| `OW_FOLLOWERS_ENABLED` | `FALSE` | HGSS follower Pokémon. Needs extra scripting to be fully supported |
| `SKIP_SAVE_CONFIRMATION` | `FALSE` | Drops the "there is already a saved file" prompt |
| `AUTO_SCROLL_TEXT` | `FALSE` | Text advances itself after a delay |
| `TEXT_SPEED_INSTANT` | `FALSE` | Truly instant text, overriding the options menu |

### Bigger systems, still free

| Setting | Default | Notes |
|---|---|---|
| `DEXNAV_ENABLED` | `FALSE` | Full ORAS DexNav. Needs **3 flags + 2 vars** assigned. `USE_DEXNAV_SEARCH_LEVELS` warns it may exceed saveblock space (1 byte/species) |
| `POKEDEX_PLUS_HGSS` | `FALSE` | HGSS-style Pokédex with dark mode and per-species move lists |
| `WE_OW_ENCOUNTERS` | `FALSE` | Overworld wild encounters (LGPE style). Wants `OW_GFX_COMPRESS FALSE` |
| `WE_DOUBLE_WILD_CHANCE` | `0` | % chance of wild double battles |
| `FNPC_ENABLE_NPC_FOLLOWERS` | `FALSE` | DPP-style NPC followers. Grows SaveBlock3 |
| `OW_TIME_OF_DAY_ENCOUNTERS` | `FALSE` | Per-time-of-day encounter tables |
| `B_EXP_CAP_TYPE` / `B_LEVEL_CAP_TYPE` | `NONE` | Level caps by badge flag or var — hard or soft |
| `B_VAR_DIFFICULTY` | `0` | Multiple difficulty versions of every trainer, switched by a var |
| `OW_SHOW_ITEM_DESCRIPTIONS` | `OFF` | ⚠️ `FIRST_TIME` is **save-breaking** (SaveBlock3). `ALWAYS` is not |
| `I_FISHING_MINIGAME` | `GEN_3` | Only Gen 1/2 and Gen 3 minigames are implemented |

### Berry farming

The entire ORAS/Gen 6 berry system is present and fully off:
`OW_BERRY_MUTATIONS`, `OW_BERRY_MOISTURE`, `OW_BERRY_WEEDS`, `OW_BERRY_PESTS`,
`OW_BERRY_MULCH_USAGE`, `OW_BERRY_SIX_STAGES`, `OW_BERRY_IMMORTAL`. Growth and
yield are pinned to `GEN_3`.

✅ **All of it is save-safe.** `struct BerryTree` (`global.berry.h:85`) already
allocates `weeds`, `mulch`, `mutationA`, `mutationB`, `pests`, `moistureLevel`
and `moistureClock` unconditionally — the storage exists and is simply unused
while the configs are off. Enabling them shifts no offsets.

**Enabled here:** `OW_BERRY_IMMORTAL` and `OW_BERRY_MUTATIONS` — the two that add
value without adding upkeep. Both verified independent of `OW_BERRY_MOISTURE`,
which stays off, so there is no watering schedule, no weeds and no pests.

| Setting | What it adds |
|---|---|
| ✅ `OW_BERRY_IMMORTAL` | Grown trees never vanish unpicked. Pure quality win, no upkeep |
| `OW_BERRY_MOISTURE` | Watering becomes about keeping soil moist, not once per stage. The gateway setting — several others only matter with it on |
| `OW_BERRY_ALWAYS_WATERABLE` | With moisture on: `FALSE` = Gen 6 (water when dry, raises yield), `TRUE` = Gen 4 (water freely, drying lowers yield) |
| `OW_BERRY_DRAIN_RATE` | How fast soil dries. `GEN_6_ORAS` = 4 hours, `GEN_6_XY` = 24 hours, `GEN_4` = per-berry |
| ✅ `OW_BERRY_MUTATIONS` | Adjacent plantings can mutate into different berries. `OW_BERRY_MUTATION_CHANCE` defaults to 25% |
| `OW_BERRY_WEEDS` | Weeds appear and need pulling; ignoring them cuts yield |
| `OW_BERRY_PESTS` | Pests appear and need clearing; ignoring them cuts yield |
| `OW_BERRY_MULCH_USAGE` | Mulch becomes usable fertiliser. Without it, Mulch items are inert |
| `OW_BERRY_SIX_STAGES` | XY's six growth stages instead of four. Does **not** change total grow time |
| `OW_BERRY_GROWTH_RATE` / `OW_BERRY_YIELD_RATE` | Currently `GEN_3`. Later presets grow faster and yield more |
| `OW_BERRY_COLORS` | Cosmetic — restores ~19 berries' Gen 6 colours |

The split that matters: `IMMORTAL`, `GROWTH_RATE`, `YIELD_RATE` and `COLORS` are
**pure convenience**. `MOISTURE`, `WEEDS`, `PESTS` and `MULCH` are **upkeep** —
they make berry growing an active minigame with more reward and more chores.
`MUTATIONS` is the one that adds genuine discovery.

---

## 3. Features that are invisible until you assign a flag or var

This is the least obvious category and the easiest to overlook: the code ships,
but is inert until you point a config at a real flag/var ID. A `0` means "off."

**Flags claimed so far** (block continues from the starter gifts at `0x20`–`0x21`):

| Flag | ID | Drives |
|---|---|---|
| `FLAG_EGG_MOVE_RELEARNER` | `0x22` | `P_FLAG_EGG_MOVES` |
| `FLAG_TUTOR_MOVE_RELEARNER` | `0x23` | `P_FLAG_TUTOR_MOVES` |
| `FLAG_POKE_RIDER` | `0x24` | `OW_FLAG_POKE_RIDER` |
| `FLAG_ORAS_DOWSING_ACTIVE` | `0x25` | `I_ORAS_DOWSING_FLAG` |

`FLAG_EGG_MOVE_RELEARNER` and `FLAG_TUTOR_MOVE_RELEARNER` are set in
`NewGameInitData` (after `InitEventData`, which clears every flag), so **they only
apply to a new game** — existing saves need them set via the debug menu.
`FLAG_POKE_RIDER` is set by the Kanto Hall of Fame script instead, as a reward.

⚠️ `FLAG_ORAS_DOWSING_ACTIVE` is **runtime state, not an enable switch**, despite
sitting among the enable flags. The feature turns on by the config being non-zero;
`oras_dowse.c` sets and clears the flag as dowsing starts and stops. Never set it
at new game.

⚠️ **Finding free flags:** a naive grep reports most `FLAG_UNUSED_*` as in use.
Those hits are declarations in `flags_frlg.h`, not real uses. Exclude both flag
headers when checking.

| Config | Gives you |
|---|---|
| `I_EXP_SHARE_FLAG` | Party-wide EXP share toggled by a flag |
| `P_FLAG_FORCE_SHINY` / `P_FLAG_FORCE_NO_SHINY` | Force every encounter shiny — **the fastest way to test shiny palettes** |
| `OW_FLAG_POKE_RIDER` | Fly directly from the region map / Town Map with **R** |
| `OW_FLAG_NO_COLLISION` | Walk through walls (debugging) |
| `OW_FLAG_NO_TRAINER_SEE` | Trainers stop initiating battles |
| `B_FLAG_NO_WHITEOUT` | Player can't lose to trainers (party is *not* auto-healed) |
| `B_FLAG_SLEEP_CLAUSE` | Sleep clause |
| `I_VS_SEEKER_CHARGING` | The VS Seeker (disables Match Call rematches) |
| `I_ORAS_DOWSING_FLAG` | ORAS Dowsing Machine with proximity colours/sounds |
| `P_FLAG_EGG_MOVES` / `P_FLAG_TUTOR_MOVES` | Individual relearners without enabling all |
| `FLAG_TEXT_SPEED_INSTANT` | Toggleable instant text |
| `B_FLAG_AI_VS_AI_BATTLE` | Watch the AI play itself |
| `WE_FLAG_FORCE_DOUBLE_WILD` | All wild battles become doubles |
| `B_VAR_NO_BAG_USE` | Disable items in trainer (1) or all (2) battles |

⚠️ **Budget check before committing to these.** Vars are the scarce resource in
this project — see the note in README.md. Flags are comfortable; vars are not.

---

## 3b. Deliberately reverted to Gen 3

Modern defaults this hack has turned back off. Listed separately so nobody
"helpfully" restores them later.

| Setting | Now | Effect |
|---|---|---|
| `B_SPLIT_EXP` | `GEN_3` | EXP splits among participants instead of full to each |
| `B_SCALED_EXP` | `GEN_3` | Flat EXP. The divisor returns to **7** (vanilla), not 5 |
| `B_EXP_CATCH` | `GEN_3` | No EXP for catching |
| `B_BADGE_BOOST` | `GEN_3` | Badges boost stats again, +10% |
| `B_AFFECTION_MECHANICS` | `FALSE` | No Gen 6 affection effects |
| `OW_POISON_DAMAGE` | `GEN_3` | Poison damages in the overworld and **can faint** |
| `B_PHYSICAL_SPECIAL_SPLIT` | `GEN_3` | Damage class by type |
| `P_LVL_UP_LEARNSETS` | `GEN_3` | RSE movepools |
| `B_TRAINER_EXP_MULTIPLIER` | `GEN_3` | Trainer battles give the vanilla 1.5× bonus |
| `B_UNEVOLVED_EXP_MULTIPLIER` | `GEN_3` | No Gen 6 1.2× for unevolved mons |
| `B_CRITICAL_CAPTURE` | `FALSE` | No Critical Capture |
| `B_CRITICAL_CAPTURE_IF_OWNED` | `GEN_3` | ⚠️ Must be set too — see below |

⚠️ **`B_CRITICAL_CAPTURE FALSE` is not sufficient on its own.**
`B_CRITICAL_CAPTURE_IF_OWNED >= GEN_9` is an **independent `||` branch** in
`FinalizeCapture` (`battle_script_commands.c:9667`) that never consults
`B_CRITICAL_CAPTURE`. Left at `GEN_LATEST` it still plays the one-shake critical
animation on every catch of an already-owned species. Both must be set.

**Exp. Share** is `I_EXP_SHARE_ITEM GEN_5` — the Gen 3–5 *held item*, which is
what we want. Combined with `B_SPLIT_EXP GEN_3` this gives the classic split:
half the EXP to participants, half shared among Exp. Share holders
(`battle_script_commands.c:3928`).

**Badge boost details.** +10%, player side only, excluded from link, e-reader,
recorded and Frontier battles (`battle_util.c:8933`). Flags are vanilla Emerald's:

| Stat | Badge |
|---|---|
| Attack | `BADGE01` — Stone (Roxanne) |
| Speed | `BADGE03` — Dynamo (Wattson) |
| Defense | `BADGE05` — Balance (Norman) |
| Sp. Atk & Sp. Def | `BADGE07` — Mind (Tate & Liza) |

These are **Hoenn** flags only, so the eight Kanto gyms grant no further boosts —
by the time Kanto opens, all five are already active. The boost checks
`IsBattleMovePhysical`/`IsBattleMoveSpecial`, which are now type-based, so it
composes correctly with the no-split rule.

**Net effect on the EXP curve.** Base yield now divides by 7 instead of 5, splits
across participants, and loses both the catch EXP and the 1.2× unevolved bonus —
but regains the 1.5× trainer multiplier. Expect noticeably slower levelling than
the expansion default, with trainer battles carrying much more of the load than
wild grinding. That is the vanilla Emerald pacing.

Still modern and untouched: `B_MAX_LEVEL_EV_GAINS`, `B_RECALCULATE_STATS`,
`B_LEVEL_UP_NOTIFICATION`. None affect the curve.

---

## 4. Where this collides with decisions already made

Interactions worth knowing, found by reading the code rather than guessing.

### Perfect IVs make several systems dead

`P_ALL_PERFECT_IVS TRUE` already renders Hyper Training, the IV judge and Destiny
Knot inheritance meaningless. Two further consequences:

- **Hidden Power needed rescuing — see below.**
- `P_SUMMARY_SCREEN_IV_EV_INFO` is still worth enabling for the **EV** half; EVs
  remain fully in play even though IVs don't.

### Hidden Power and the 30/31 fix

Hidden Power's type comes from **bit 0 of each of the six IVs**
(`battle_main.c:5856`). A flat 31 spread sets every bit, so `typeBits` was always
63 and every Hidden Power in the game — yours and every trainer's — was **Dark**.

Fixed with `P_PERFECT_IVS_VARY_FOR_HIDDEN_POWER TRUE`: each IV is now 30 or 31 at
random. Bit 0 varies again, all 16 types return, and the cost is at most one IV
point per stat — one stat point at level 100, less below.

Applied at the same single choke point as `P_ALL_PERFECT_IVS`, so it covers wild
encounters, gifts, eggs, static legendaries, enemy trainers and Frontier mons.

**The distribution is not uniform**, and this is inherent to the vanilla
`(15 * typeBits) / 63` formula rather than anything we did:

| Type | Chance |
|---|---|
| Fighting, Bug, Grass | 7.8% each |
| Flying, Poison, Ground, Rock, Ghost, Steel, Fire, Water, Electric, Psychic, Ice, Dragon | 6.2% each |
| **Dark** | **1.6%** — only `typeBits == 63` reaches it |

So Dark went from guaranteed to the rarest type.

⚠️ **The type cannot be bred for.** Eggs use this same code path, so it is random
per Pokémon. Re-rolling means catching or hatching again, not chaining parents.

`P_SHOW_DYNAMIC_TYPES` was turned `TRUE` alongside it — without that, Hidden Power
displays as Normal and a randomised type would be invisible, which is strictly
worse than a predictable one. It also affects Weather Ball, Judgment and similar.

### No physical/special split

`B_PHYSICAL_SPECIAL_SPLIT GEN_3` means damage class comes from the move's **type**:

- **Physical:** Normal, Fighting, Flying, Poison, Ground, Rock, Bug, Ghost, Steel
- **Special:** Fire, Water, Grass, Electric, Psychic, Ice, Dragon, Dark, Fairy

Fairy didn't exist in Gen 3; the expansion files it under Special.

The catch: **every Pokémon introduced from Gen 4 onward was designed around the
split.** Their stat spreads and movepools assume it. Expect specific mons to feel
wrong — a physically-built Ghost attacker now uses its Attack (fine, Gen 3 rules),
but a special-built one is stuck using Attack too. This is inherent to the choice,
not a bug, and it's the price of the Gen 3 feel.

`B_UPDATED_MOVE_DATA` is still `GEN_LATEST`, so moves keep their modern power and
accuracy. Only the damage class reverted.

### Critical Capture — why it was turned off

Recorded because the reasoning is not obvious from the config comment.

It is a Gen 5 mechanic: catch odds are multiplied by 0.5× / 1× / 1.5× / 2× / 2.5×
depending on how much of the Pokédex you have **caught**, then divided by 6 and
rolled. On success the ball shakes once and resolves immediately. Below ~4.6% dex
completion it never fires. The Catching Charm doubles the multiplier.

Beyond simply post-dating Gen 3, it was badly scaled here:
`B_CRITICAL_CAPTURE_LOCAL_DEX TRUE` compares your **national** caught count
against the **regional** dex total (`battle_script_commands.c:9976-9994`). With
1,029 species enabled against a Hoenn-sized total, the top 2.5× tier arrives at
roughly 195 species caught — very early for this hack.

If it is ever re-enabled, set `B_CRITICAL_CAPTURE_LOCAL_DEX FALSE` at the same
time to restore the intended curve.

### Two things enabled with wider reach than expected

- **The egg/tutor relearner flags also light up the summary screen.**
  `pokemon_summary_screen.c:4868` shares the same condition, so egg and tutor
  moves are relearnable from the party menu, free, anywhere — not just at the
  Fallarbor NPC with Heart Scales. If that proves too strong, set
  `P_SUMMARY_SCREEN_MOVE_RELEARNER FALSE` to restrict it back to the NPC.
- **Fly-from-region-map does not require Fly.** It checks
  `MAPSECTYPE_CITY_CANFLY` (so only already-visited cities) and the flag, but
  never checks for HM Fly or a flier in the party
  (`pokenav_region_map.c:223`). It is therefore gated purely by when the flag
  gets set. **Resolved:** `FLAG_POKE_RIDER` is now set by the Kanto Hall of Fame
  script, making it a reward for clearing the Kanto league rather than something
  available from gym 2. Oak explains the unlock in a second message.
  The Kanto league is now a real gate: Kanto has its own eight badge flags
  (`FLAG_KANTO_BADGE01_GET`–`08` at `0x26`–`0x2D`), so Route 23's guards and the
  Viridian Gym check Kanto progress rather than Hoenn's.

### Learnsets

`P_LVL_UP_LEARNSETS GEN_3` gives Gen 1–3 species their authentic RSE movepools.
For species introduced later the expansion falls back to their **debut**
generation (Turtwig gets DP, Sprigatito gets SV), per the rule documented at
`config/pokemon.h:15`.

If Gen 4–9 mons later feel undertuned, the alternative is a **hybrid file** —
Gen 1–3 rows from `gen_3.h`, everything else from `gen_9.h`. That's a mechanical
script, not hand work, but it forks a 690 KB data file away from upstream and has
to be regenerated on every expansion update. Not worth it unless play proves it is.

---

## 5. Save-breaking — check before flipping

Changing these shifts saveblock offsets and **invalidates existing saves**:

- Anything changing the **newest enabled generation** in `species_enabled.h`
  (shifts Dex flags)
- `OW_SHOW_ITEM_DESCRIPTIONS` set to `FIRST_TIME` (SaveBlock3)
- `FNPC_ENABLE_NPC_FOLLOWERS` (grows SaveBlock3)
- `USE_DEXNAV_SEARCH_LEVELS` (1 byte per species)
- Any `FREE_*` in `config/save.h` — these *reclaim* space (3,790 bytes available
  across both blocks) but by definition move everything after them

Everything in sections 1–3 above is pure logic or ROM data and is save-safe.

See SETUP.md's Working notes for how a stale save presents — it looks like a
graphics bug, not a save problem.
