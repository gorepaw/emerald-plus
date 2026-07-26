# emerald+ — project plan

Drafted 2026-07-25, against `pokeemerald-expansion` @ `expansion/1.16.2`, branch `emerald-plus`.
Baseline build verified working on the Miyoo Mini Plus.

---

## Measured constraints

These were surveyed from the actual checkout, not assumed.

| Resource | State | Verdict |
|---|---|---|
| **Vars** | **23 free of 256** (`0x4000`–`0x40FF`) | ⚠️ **the real wall** |
| Flags | 373 unused | comfortable |
| Trainer flags | 864 slots (`TRAINER_FLAGS_START 0x500`) | adequate, watch it |
| ROM | 26.48 / 32 MB — ~6.7 MiB free | fine |
| EWRAM | 86.43% used (~30 KB free) | ⚠️ tight |
| IWRAM | 86.65% used (~4 KB free) | ⚠️ tight |
| Existing maps | 944 folders, 806 layouts, 887 script files | for scale reference |

**Read this before planning anything big:** vars are nearly exhausted and RAM is at
~86%. ROM space — the thing romhackers usually worry about — is the one resource
you have plenty of. Budget vars and RAM, not cartridge space.

Expanding the var pool past 256 is possible but rewrites saveblock layout, which
breaks saves and is real engineering. Prefer flags (373 free) wherever a boolean
will do.

---

## The list, sorted by actual cost

### Tier 1 — days of work. Config and data only, no new maps.

**1. ✅ Perfect 31 IVs — DONE** (commit `51ca3949d8`)

Decision: **everything gets 31s, enemy trainers included.**

Implemented as `P_ALL_PERFECT_IVS` in `include/config/pokemon.h`. `SetBoxMonIVs`
is the single generation choke point — wild, gift, egg, static legendary, enemy
trainer and Frontier facility mons all route through it, including the `fixedIV`
path trainers use.

Two overrides had to be suppressed or they'd have reintroduced lower IVs:
- `src/trade.c` — in-game trades hardcode their own IV spreads
- `src/battle_pyramid.c` — at streak ≥140, Pyramid mons roll `(Random() % 17) + 15`
  (15–31), which would have made high-streak encounters **weaker** than the global
  standard rather than stronger

Egg inheritance needed no change: `InheritIVs` copies from parents, and every
parent is now 31, so inheritance is self-consistent.

⚠️ **Expect the E4 and Battle Frontier to be harder** than vanilla. That's the
accepted cost of the everyone-gets-31s rule.

📌 **Now-dead content to repurpose or remove:** Hyper Training, the IV judge NPC,
and Destiny Knot / IV-inheritance breeding all still exist and now do nothing.
Tracked in TODO.md.

**2. Pre-E4 availability re-gating**
Target roster: all Hoenn (RSE) + Kanto starter lines + Johto starter lines
(Johto placed mid-game).

Pure data work in `src/data/wild_encounters.json` (1.0 MB, 388 encounter tables).
Tedious, mechanical, zero technical risk. No new maps, no code.

Starters as one-choice-early / all-three-catchable-endgame is a scripting change
plus a post-game encounter or gift placement.

**3. Post-E4: everything available**
Needs a mechanism decision, and the options differ a lot in cost:
- **Flag-gated encounter table swap** — cheapest. One flag, table variants. Needs
  a small amount of code but no new maps.
- **New post-game areas** — more interesting, much more expensive. Overlaps Tier 3.
- **Hidden-area system** — see Tier 2 item 5; this is probably the natural home for it.

**4. Endgame QoL grab-bag**
Most of what people want here already exists as expansion config flags — there are
hundreds across `include/config/`. Cheap wins, done in batches.

📌 **Start a running `TODO.md`.** The "scattered and ever-growing list of small
things" is exactly what kills romhack projects — not difficulty, but unbounded
scope living in someone's head. Write items down as they occur; batch them.

### Tier 2 — weeks. New systems, self-contained.

**5. Hidden-base rare encounters (Gen 4–9 + legendaries)**
Strong idea, and it neatly solves item 3 as well.

⚠️ **Don't build this by modifying `src/secret_base.c`.** That file is 2,076 lines
of largely multiplayer/record-mixing machinery — repurposing it means fighting
code built for a different purpose. Better options:
- **Hidden Grottoes (Gen 5 model)** — self-contained, purpose-built for exactly
  this: rare rotating encounters at scattered map locations.
- **DexNav** — already present in the expansion (`CalculateDexNavShinyRolls`,
  `graphics/dexnav/`). Built for hunting specific species; could be extended.

Either is a cleaner foundation than secret bases, and both reuse the "check
scattered locations for rare spawns" feel you described.

**6. Safari Zone expansion**
Mostly encounter-table data plus modest map work. Low risk.

### Tier 3 — many months. Treat as a separate project.

**7. Kanto as the postgame region, with Kanto+Johto double battles**

⚠️ **Johto as a region is CANCELLED** (decided 2026-07-25) and replaced with the
design below. This removed the single most expensive item on the project.

**The design:**
- **Kanto is the postgame map.** Already in the repo — see the asymmetry note below.
- **Every Kanto gym is a double battle:** the Kanto leader *and* their Johto
  counterpart, fought together. Same for the Elite Four.
- **Kanto hosts the miscellaneous wild Pokémon** — Gen 4–9 and anything else that
  would clutter Hoenn. This is how "everything is catchable" gets satisfied without
  diluting Hoenn's curated roster.

This keeps Johto's entire cast at a fraction of the cost of building Johto.

**Engine support: already present, nothing to invent.**
- `TRAINER_BATTLE_TWO_TRAINERS_NO_INTRO` — two distinct trainers on the opposing side
- `BATTLE_TYPE_TWO_OPPONENTS`, `BATTLE_TYPE_MULTI`
- **Tate & Liza are a working double-battle gym leader precedent** already in Emerald

**What exists vs. what must be authored:**

| | Kanto | Johto |
|---|---|---|
| Trainer constants | ✅ `opponents_frlg.h` | ❌ author |
| Team data | ✅ `trainers_frlg.party` | ❌ author |
| Front sprites | ✅ 8 leaders + 4 E4 + Blue + Red | ❌ author (~11) |
| Gym maps + scripts | ✅ 8 gyms | n/a |

**The real cost of this design is art, not code.** ~11 Johto trainer front sprites
must be sourced or drawn in GBA style. Trainer entries, teams and battle text are
table data and writing.

**Confirmed pairings** (Bruno and Lance appear in *both* regions' E4s, and Lance is
also Johto's Champion — this mapping resolves that without duplicates, using Janine
for Fuchsia and Red as Blue's champion partner):

| Slot | Kanto | Johto |
|---|---|---|
| Gym 1 | Brock (Rock) | Falkner (Flying) |
| Gym 2 | Misty (Water) | Bugsy (Bug) |
| Gym 3 | Lt. Surge (Electric) | Whitney (Normal) |
| Gym 4 | Erika (Grass) | Morty (Ghost) |
| Gym 5 | Koga (Poison) | Janine (Poison) |
| Gym 6 | Sabrina (Psychic) | Jasmine (Steel) |
| Gym 7 | Blaine (Fire) | Pryce (Ice) |
| Gym 8 | Giovanni (Ground) | Clair (Dragon) |
| E4 1 | Lorelei (Ice) | Will (Psychic) |
| E4 2 | Bruno (Fighting) | Karen (Dark) |
| E4 3 | Agatha (Ghost) | Koga (Poison) |
| E4 4 | Lance (Dragon) | Chuck (Fighting) |
| Champion | **Blue** | **Red** |

Pairs are deliberately *not* type-matched — mixed typings make for more interesting
doubles. Koga appearing at both Gym 5 and E4 3 is canonical: he ran Fuchsia in RBY
and was promoted to the Johto E4 in GSC, with Janine taking over the gym.

**Critical asymmetry — Kanto is ALREADY IN THE REPO, Johto does not exist:**

Revised 2026-07-25 after surveying the checkout. This is better than first assessed:

- **Kanto ships with pokeemerald-expansion.** 140 `*_Frlg` map folders exist under
  `data/maps/` (Celadon, Cerulean, Cinnabar, Cerulean Cave, gyms, department
  stores, Pokémon Centers, houses), with matching `data/layouts/` entries and FRLG
  wild encounter tables already in `wild_encounters.json`. The Makefile has a full
  `firered` / `leafgreen` build target (`GAME_VERSION`, `MAP_VERSION`).
  The expansion has merged pokefirered — **Kanto is not a port, it's a build flag.**
- **Johto exists only in GSC (Game Boy) and HGSS (DS).** Neither format is portable
  to this engine, and no decomp provides it. **Johto must be rebuilt by hand, map
  by map, from reference.**

### ✅ MEASURED 2026-07-25 — spike branch `spike/kanto-in-emerald`

An earlier draft of this doc called the integration "one condition in one build
tool." **That was wrong.** The spike relaxed the three region/`layout_version`
filters in `tools/mapjson/mapjson.cpp`; maps then compile in and the build dies at
assembly:

```
asm/macros/map.inc:109: Error: Hidden Item flag 0 is too small.
                        Must be >= FLAG_HIDDEN_ITEMS_START.
```

**Root cause: three namespaces are version-split, mutually exclusive, and
numerically overlapping.**

| Namespace | Emerald | FRLG | Conflict |
|---|---|---|---|
| Flags | `flags.h` `#else` branch | `flags_frlg.h`, `#ifdef FIRERED` | Hidden items `0x1F4` vs `0x3E8` |
| Trainer IDs | 855 / 864 used — **9 free** | 624 / 768 | Combined ~1,479 |
| Vars | `0x4000`–`0x40FF`, 23 free | **same 256 slots** | Both nearly full |

Surface area: FRLG map events reference **503 distinct flags** (183 hidden items),
map scripts a further 325. Emerald has 371 free flags, 23 free vars.

Worse, trainer flags are *derived* from trainer IDs
(`TRAINER_FLAGS_END = TRAINER_FLAGS_START + MAX_TRAINERS_COUNT - 1`), so growing
the trainer space shifts the flag layout — **save-breaking**, and constrained by
the saveblock (14 sectors × 3968 bytes).

### ✅ FEASIBILITY: saveblock has room. Merge the namespaces.

Scope decision 2026-07-25: **Kanto is to be fully walkable**, not gyms-only.
That rules out the "re-author 30–50 maps" workaround — at 417 maps, rewriting is
worse than merging. So: can the namespaces actually be merged?

Measured from the built ELF (`gSaveblock1` is `SaveBlock1ASLR`, so subtract the
128-byte ASLR pad):

| | Used | Capacity | **Free** |
|---|---|---|---|
| SaveBlock1 | 15,568 | 15,872 (sectors 1–4) | **304 bytes** |
| SaveBlock2 | 3,884 | 3,968 (sector 0) | 84 bytes |

Cost of the merge:

| Item | Demand | Saveblock cost |
|---|---|---|
| FRLG trainers | 268 distinct → allocate 300 | **38 bytes** (flag array) |
| FRLG flags | ~800 new | **~100 bytes** |
| Vars, if needed | 2 bytes each | budget from what's left |

**~140 of 304 bytes. It fits, with headroom to spare.**

### The approach

Define the FRLG flag and trainer constants **inside the Emerald namespace at
fresh offsets**. The constant *names* are identical across versions
(`FLAG_HIDDEN_ITEM_VIRIDIAN_FOREST_POTION`, `TRAINER_LEADER_BROCK`), so the 417
FRLG maps compile **unchanged** — no per-map rewriting at all.

### ✅ DONE — Kanto compiles into the Emerald ROM

Completed 2026-07-25. Verified by full clean rebuild (149s).

| | Before | After |
|---|---|---|
| Map groups | 34 | **75** |
| MAP_ constants | ~518 | **935** |
| ROM | 78.93% | **83.53%** |
| SaveBlock1 | 15,568 | 15,744 / 15,872 (**128 free**) |

What it actually took (commits `087bca9898`, `150eab015e`, `8419383f3f`):

1. **Trainer IDs** — `opponents.h` includes `opponents_frlg.h` *unconditionally*
   and both numbered from 1, so every FRLG trainer aliased an Emerald one.
   Re-based to `(FRLG_TRAINER_BASE + n)`; `MAX_TRAINERS_COUNT_EMERALD` 864 → 1536.
2. **Flags** — the real cause of the spike failure. Emerald's branch *declares*
   every FRLG flag name but defines them as literal `0`. 714 stubs allocated a
   fresh contiguous block; 182 were hidden items, which is why
   `bg_hidden_item_event` rejected them.
3. **mapjson filters** — three conditions relaxed so Emerald accepts
   `REGION_KANTO` maps and `frlg` layouts.
4. **Script includes** — `data/event_scripts.s` gated 418 FRLG `scripts.inc`
   behind `.if IS_FRLG` while Emerald's were unconditional. Made symmetric.
5. **Tilesets** — `headers.h`/`metatiles.h` used `#if !IS_FRLG / #else`, so only
   one region's tilesets existed. Both now compile (verified zero symbol overlap).
6. **An upstream typo** — `RocketHideout_B1F_Frlg/scripts.inc` was missing a
   comma. It survived because those 418 scripts had never been assembled.

⚠️ **Vars were never touched.** Both versions share `0x4000`–`0x40FF` and it
built anyway, so no collision surfaced — but that is untested, not proven safe.
Watch for it once Kanto is actually played.

### ⏭️ Next: make Kanto reachable

The maps are in the ROM but **no warp or connection links Hoenn to Kanto**, so
they cannot be visited. A Lilycove ferry is the natural entry point.

So the order you proposed (Johto first, then Kanto) is backwards on cost, and more
sharply than first thought. **Kanto is plausibly a medium-sized integration job.
Johto is a genuine from-scratch region build.** Do Kanto first regardless.

**Resource risk:** two regions will strain vars (23 free) and trainer flags. Gym
badges and trainer defeats are flags (fine — 373 free), but progression counters
are vars (not fine). Requires deliberate budgeting from day one.

---

## Recommended sequencing

1. **Tier 1 complete** → build → play it on the Miyoo → confirm it's the game you want
2. **Tier 2** → hidden-area system, which also delivers post-E4 availability
3. **Decision point on Tier 3** — and before committing, **prototype one Kanto route
   end-to-end** (map, connections, encounters, trainers, scripts). Measure the real
   hours. Multiply by ~250. Then decide.

That prototype step is the single most valuable thing you can do before Tier 3.
It converts a guess into a measurement, and it's the difference between a project
that ships and one that stalls at 40%.

---

## Immediate next step

Decide the perfect-IV scope (player-only vs. everyone), then Tier 1 items 1–2 can
both be done in one pass.
