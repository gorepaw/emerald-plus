# emerald+

Design notes, build setup and progress tracking for a Pokémon Emerald romhack
built on [pokeemerald-expansion](https://github.com/rh-hideout/pokeemerald-expansion)
`expansion/1.16.2`, targeting the **Miyoo Mini Plus on stock firmware**.

**This repository contains documentation only — no ROMs, no patches, no game
assets.** See [Legal](#legal) below.

## What the hack does

| | |
|---|---|
| **Species** | All 9 generations (1,029), DS-style art |
| **Shiny rate** | 1/512 (≈1/171 with the Shiny Charm) |
| **IVs** | Every Pokémon rolls 30 or 31 in each stat — player, wild, gift, and enemy trainers. The 30s exist only to keep Hidden Power's type varied |
| **Battle style** | No physical/special split — damage class is by type, as in Gen 1–3 |
| **EXP** | Gen 3 rules: split among participants, unscaled, 1.5× from trainers, none for catching, Gen 3–5 held-item Exp. Share |
| **Kept from Gen 3** | Badge stat boosts, overworld poison damage, no affection mechanics, no Critical Capture |
| **Movepools** | Gen 1–3 species use authentic RSE learnsets; later species use their debut generation's |
| **Encounters** | **373 of 386 Gen 1–3 species catchable in Hoenn**, up from 268 — 55 evolutionary families given a home. Only the Gen 1–2 legendaries are missing |
| **Gen 4–9** | **545 of 639 obtainable**, hosted across Kanto — 284 families placed by type affinity. Only legendaries and paradox mons are missing |
| **Kanto scaling** | Postgame content on two bands — wilds L50–89, trainers L60–98 — with species evolved to match |
| **Kanto league** | All 5 Elite Four and Champion rooms are double battles too: Lorelei+Will, Bruno+Karen, Agatha+Koga, Lance+Chuck, Blue+**Red** |
| **Rematches** | Every Kanto leader and Elite Four member is in the rematch table and re-fightable, with both halves of a pair scaling together |
| **Pokédex** | Hoenn dex extended to 387 entries; every species catchable in Hoenn is listed in it |
| **Pseudo-legendaries** | All four lines reachable pre-Elite Four — Beldum, Bagon, Dratini and Larvitar |
| **Trainers** | All 818 Hoenn trainers meet the party minimums (2 / 3 in gyms / 4–6 for leaders), rematches included, and none is below its route's wild level |
| **Trade evolutions** | Solvable solo — the Linking Cord is sold at the Battle Frontier for 1 BP |
| **Starters** | Kanto starter from Mr. Stone; Johto starter from Steven on Route 118 |
| **Kanto** | All 417 FRLG maps compiled into the Emerald ROM, reachable postgame by ferry from Lilycove |
| **Kanto gyms** | All 8 are double battles pairing each Kanto leader with their Johto counterpart, awarding Kanto's own eight badges — 3 Pokémon each, at matched levels |

Johto as a walkable region is **cancelled** — its cast is delivered through the
Kanto double-battle pairings instead, at a fraction of the cost.

## Documents

| File | Contents |
|---|---|
| [PLAN.md](PLAN.md) | The project plan: measured resource constraints, features tiered by real cost, and the Kanto namespace-merge findings |
| [SETUP.md](SETUP.md) | Toolchain, WSL2 environment, build and deploy steps |
| [TODO.md](TODO.md) | Running task list — done, in progress, and parked ideas |
| [TESTING.md](TESTING.md) | mGBA debug-menu recipes, flag/map numbers for this build, and the current test checklist |
| [QOL.md](QOL.md) | Survey of every QoL option the expansion already ships — what's on, what's off, and what's inert until you assign it a flag |
| [BALANCE.md](BALANCE.md) | Encounter and trainer-party rules, the slot-rate tables, and how far through Hoenn they've been applied |

## Notable findings

Things measured against the actual checkout rather than assumed — the parts most
likely to be useful to someone else:

- **ROM space is not the scarce resource.** At 26.5/32 MB the cartridge has ~6.7 MiB
  free, but EWRAM and IWRAM sit at ~86% and only **23 of 256 vars** are unused.
  Budget vars and RAM, not cartridge space.
- **Kanto ships with the expansion.** 140 `*_Frlg` map folders, layouts and encounter
  tables are already in the repo behind the `firered` build target. Getting them into
  an Emerald build is a namespace problem, not a porting problem.
- **The namespace merge fits in the saveblock.** Flags, trainer IDs and vars are
  version-split and numerically overlapping; re-basing FRLG trainers and allocating
  the 714 flags upstream stubs to literal `0` costs ~140 of SaveBlock1's 304 free
  bytes. All 417 maps then compile **unchanged**, because the constant names are
  identical across versions.

## Source

The engine changes live on branch `emerald-plus` of a pokeemerald-expansion fork,
branched from tag `expansion/1.16.2`:

**[github.com/gorepaw/pokeemerald-expansion @ `emerald-plus`](https://github.com/gorepaw/pokeemerald-expansion/tree/emerald-plus)**

Every commit hash referenced in these documents resolves there. The full delta from
upstream is
[47 commits](https://github.com/rh-hideout/pokeemerald-expansion/compare/expansion/1.16.2...gorepaw:pokeemerald-expansion:emerald-plus),
or `git diff expansion/1.16.2` in a local checkout.

## Legal

Pokémon Emerald is copyright Nintendo / Game Freak / The Pokémon Company. This
repository distributes no ROM, no patch and no game asset, and is not affiliated
with or endorsed by any of them. Building the hack requires the pokeemerald-expansion
decompilation and a legally obtained copy of the game.
