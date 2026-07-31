# emerald+ — running TODO

## ▶ RESUME HERE

**State as of 2026-07-30:** engine fork at **53 commits** since
`expansion/1.16.2`. ROM 84.89%, built ROM MD5 `f57eb97df42a0a5881c34eeda7cb69ee`.
EWRAM is **down** to 223,344 B (85.20%) after reclaiming 3,420 bytes of
link-only save data. **Start a new game** — the rematch enum moved
`FLAG_REGISTERED_*`, and the save layout was deliberately rebuilt on top.

**The Pokédex is completable: 576 of 576 evolutionary families are obtainable in
a single save.** That is every Pokémon in the game.

**Save-layout changes were batched on purpose** while a new game was already
required — see the memory note or the commit. Flag block grown to 256 (227
free), var range +64, and nine link-only `FREE_*` toggles flipped. Anything that
moves `FLAGS_COUNT`, `VARS_COUNT`, saveblock field order, or the item/move/
species enums is save-breaking, so batch it before a real playthrough starts.
`FREE_MATCH_CALL` and `FREE_EXTRA_SEEN_FLAGS_SAVEBLOCK1` must stay FALSE.

Kanto is walkable postgame with its own eight badge flags, all 8 Kanto gyms are
Kanto+Johto double battles, doors animate, and the ruleset is settled — read
**QOL.md** for every config decision and **BALANCE.md** for the encounter and
trainer-party rules.

## ▶ All content is done. What's left is playing it.

**Hoenn content is complete**, in one autonomous pass on 2026-07-30:

- **373 of 386 Gen 1–3 species reachable**, up from 268. 55 evolutionary
  families placed in 82 slots across 57 maps.
- **Hoenn dex 239 → 387 entries.**
- **818 trainers audited, 0 short and 0 under the level floor** — 290 Pokémon
  added and 51 levels raised, rematch parties included.
- **Linking Cord sold at the Battle Frontier for 1 BP**, which is what makes
  Gengar, Alakazam, Machamp and Golem obtainable without a second console.

**Kanto content is complete too**, same day:

- **Kanto had no wild Pokémon at all** — 264 tables compiled out behind
  `#ifdef FIRERED` / `#ifdef LEAFGREEN`. Fixed; FireRed is canonical Kanto.
- **Two level bands: wilds L50–89, every trainer L60–98**, with 1,384 wild
  entries and 677 trainer Pokémon evolved to match.
- **Gen 4–9 at 545/639**, 284 families placed by type affinity.
- **All 8 gyms and all 5 league rooms are double battles**, 3 + 3 per side —
  not the 8-vs-6 Giovanni and Clair used to bring. Koga needed a second entry,
  `TRAINER_JOHTO_KOGA`, because he is both a Kanto gym leader and a Johto
  Elite Four member.

Guard rails: `party_audit.py` at 0/0, `availability.py` at 373/386 and 545/639,
and its completability section at **576/576 families in one save**.
`legendary_audit.py` at 19 slots / 90 placed / 90 unique.

All nine Gen 1–3 starter families are catchable in Kanto as their fully-evolved
forms, so picking one per trio no longer costs you the other six.

**Track A — DONE. The legendaries are 19 rotating slots.**

Clear a spot — catch or defeat — and it stays empty until Kanto's Champion next
falls, then refills with the next Gen 4–9 legendary. Slots advance
**independently**: nothing waits on any other legendary, so refills come free
with the E4 money runs you were doing anyway. All 103 legendaries at **L70**.
5 Champion wins is the fewest to see all 90 successors, not a gate.

Species are a **fixed authored table, never a random draw** — that is what makes
them huntable. `CreateScriptedWildMon` rerolls personality and IVs every call,
so nature and shininess reset on re-entry while the species stays put.

Only five needed new homes: Raikou → New Mauville, Entei → Fiery Path, Suicune →
Shoal Cave, Celebi → Viridian Forest, Jirachi → Meteor Falls. Four already had
Kanto statics; four more sat on the event islands, which the Champion's ticket
rotation (Mystic → Aurora → Old Sea Map → Eon) now opens.

Built additively — no existing legendary script was rewritten. Each slot got a
*second* object on the same tile, with the original's resolved-flag as its
prerequisite. Guard rail: `legendary_audit.py` → 19 slots, 90 placed, 90 unique.

**Two blockers found and fixed along the way:** the Sevii Islands were
unreachable (one FRLG story var), and **Kanto's league could only be run once** —
nothing cleared its flags, so the Champion's room stopped triggering forever,
killing E4 money rematches and the Blue+Red rematch pairing alike.

**Track B — DONE.** All 8 gyms and all 5 league rooms are double battles. The
rematch table holds 90 of 100 entries with Kanto's leaders correctly placed
above `REMATCH_SIDNEY`. The Johto half of a league rematch scales with its Kanto
counterpart. All 8 Kanto leaders are re-fightable and in the Match Call
rotation. Mechanics are written up in BALANCE.md.

⚠️ **`PokemonLeague_HallOfFame_Frlg/scripts.inc` sets `FLAG_POKE_RIDER`**
(Fly-from-region-map). It lives there rather than in the Champion's Room so
league work can't disturb it — **don't drop it.**

### Untested, and only a playthrough can settle it

- **7 of 8 Kanto gyms** have never been played (only Pewter).
- **The Kanto Hall of Fame has never run.** It uses FRLG's
  `FLDEFF_HALL_OF_FAME_RECORD_FRLG` and `special EnterHallOfFame`; both symbols
  link, but that sequence has never executed in an Emerald build.
- **None of the Hoenn content has been played** — not the early routes added
  from 2026-07-27, and not one of the 82 new encounter slots, 290 added trainer
  Pokémon or 148 dex entries from the 2026-07-30 pass. The audits prove the
  rules hold; they say nothing about whether it's *fun*.
- **The Battle Frontier's Linking Cord** has never been bought. The vendor menu
  needs four separate lists to agree, and only the game can confirm they do.

---

The catch-all for the "scattered and ever-growing list of small things."
Write it down here the moment it occurs to you; batch the work later.
Scope discipline is what decides whether this project ships.

---

## ✅ Done

- [x] Shiny base rate → 1/512 (`SHINY_ODDS 128`) — commit `4ecadc5146`
- [x] Shiny Charm stays at 2 extra rolls (≈1/171 with Charm) — default, no change
- [x] All 9 generations enabled, DS-style art — default, no change
- [x] **All Pokémon have perfect 31 IVs** — commit `51ca3949d8`
- [x] **Zangoose + Lunatone added to wild encounters** — commit `1f03a77cd8`.
      Route 114 is now Seviper 2 / Zangoose 1 slots (~5%/4%); Meteor Falls is
      Solrock 18 / Lunatone 12 across five maps.
- [x] **Kanto starter gift** — Mr. Stone, Devon Corp 3F, after the Devon favour.
      Commit `39f924a8ba`. Re-offered if declined.
- [x] **Johto starter gift** — Steven, Route 118 (just east of Mauville), folded
      into his existing cutscene. Commit `39f924a8ba`. Non-cancellable menu.

---

## In progress / next

- [ ] **Kanto postgame + Kanto/Johto double battles** — the headline feature.
      See PLAN.md for the pairing table and the measured namespace findings.

      ⚠️ **Do NOT try to compile the `*_Frlg` maps into the Emerald build.**
      Measured and failed on branch `spike/kanto-in-emerald`: FRLG flags, trainer
      IDs and vars are separate, overlapping namespaces. Merging them is
      save-breaking and doesn't fit the free flag/var budget.

      **✅ NAMESPACE MERGE DONE — all 417 Kanto maps compile into the Emerald ROM.**
      75 map groups (was 34), 935 MAP_ constants (was ~518), ROM 83.53%.
      Verified with a full clean rebuild. Commits `087bca9898`, `150eab015e`,
      `8419383f3f`.
      - [x] Re-base FRLG trainer IDs; MAX_TRAINERS_COUNT_EMERALD 864 → 1536
      - [x] Allocate the 714 FRLG flags that upstream stubbed to literal 0
      - [x] Relax the three mapjson region/layout filters
      - [x] Assemble FRLG map scripts in Emerald builds
      - [x] Compile both regions' tilesets
      - [x] **Kanto is reachable** — VERMILION CITY added as an SS Tidal
            destination at Lilycove Harbor. Commits `805e999e74`, `87aa6d187e`.
            Postgame-gated for free: the ferry attendant already required
            `FLAG_SYS_GAME_CLEAR`. Arrival is `MAP_VERMILION_CITY (23, 33)`.
      - [ ] ⚠️ **Untested in-game.** Verify on the Miyoo: the ferry menu shows
            VERMILION CITY, the crossing works, and Vermilion is walkable.
            The arrival tile (23, 33) was inferred from the S.S. Anne dock warps
            at y=34, not visually confirmed — adjust if it lands somewhere odd.
      - [ ] No return ferry from Kanto yet — you can get there but not back
            without escape rope / Fly.
      - [x] **Return ferry** Vermilion → Lilycove, on the existing sailor NPC.
            Commit `118d44ae4d`.
      - [x] **All 12 Johto trainers created** with teams and placeholder sprites
            (Red already had a real pic). Commit `4499e2b2cd`. Movesets omitted
            deliberately so the game assigns level-up moves — tune later.
      - [x] **Kanto trainers now have party data.** `gTrainers` was initialised
            from `trainers.h` *or* `trainers_frlg.h`, never both, so the Kanto
            gym leaders had NO teams in an Emerald build. Fixed in `4499e2b2cd`.
      - [x] **Pewter Gym = Brock + Falkner double battle.** Commit `bfe810d733`.
            ✅ **VERIFIED IN-GAME 2026-07-26** — battle runs and the badge awards.
            The badge reroute through `DefeatedBrock` works, so the remaining
            gyms can copy this pattern with confidence.
      - [x] **All 8 gyms are now double battles.** Commit `cfecc01551`.
            Cerulean/Bugsy, Vermilion/Whitney, Celadon/Morty, Fuchsia/Janine,
            Saffron/Jasmine, Cinnabar/Pryce, Viridian/Clair. Clean rebuild
            verified; SaveBlock1 unchanged, so existing saves stay valid.
      - [ ] ⚠️ Only Pewter has been played. The other 7 are built and verified to
            compile but **untested in-game** — worth a pass through each.
      - [x] **Kanto doors animate.** Commit `271d00bdf9`. `sDoorAnimGraphicsTable`
            was `#if !IS_FRLG / #else`, so an Emerald build carried only Hoenn's
            53 door entries and every Kanto door fell out of `GetDoorGraphics`
            as NULL. Table is now 86 entries (53 + 32 + terminator). No code
            changes needed — every draw path already branched on
            `mapLayout->isFrlg`. ROM-only (`.data`/`.bss` both 0), saves valid.
      - [x] **Kanto has its own eight badge flags.** Commit `13417d5afd`.
            `FLAG_KANTO_BADGE01_GET`–`08` at `0x26`–`0x2D`; 24 references
            repointed across 10 files. Kanto progression is now real: Route 23's
            guards gate on Kanto badges, and the Viridian Gym needs Kanto 2–7.
            Obedience, level caps, the trainer card and the catch malus stay
            Hoenn-only on purpose — the player has all 8 Hoenn badges before
            Kanto opens, so Kanto badges are pure gating.
            Save-safe: reused already-allocated `FLAG_UNUSED` ids, so
            `FLAGS_COUNT` is unchanged.
            ⚠️ Consequence: the **trainer card still shows Hoenn's eight and
            never Kanto's**. Known, deferred, low priority.
      - [ ] Elite Four (4 pairs) + Champion (Blue + Red)
            ⚠️ The Kanto Hall of Fame script sets `FLAG_POKE_RIDER`
            (Fly-from-region-map). It lives there rather than in the Champion's
            Room precisely so this conversion won't disturb it — **don't drop
            it** if you touch `PokemonLeague_HallOfFame_Frlg`.
      - [ ] Region map graphics
      - [ ] Replace the 11 placeholder trainer sprites with real GBA-style art
      - [ ] **Trainer card still shows the Hoenn badge list** while earning Kanto
            badges. Cosmetic; a Kanto badge display is wanted eventually but is
            explicitly low priority.
- [ ] **Stock Kanto with the miscellaneous wild Pokémon** (Gen 4–9 etc.), so
      everything is catchable without cluttering Hoenn.

**Decided:** no removal from Hoenn's existing encounter tables. Re-gating is
additive only. The earlier "remove non-Hoenn species pre-E4" idea is dropped —
Kanto absorbs the overflow instead.
- [ ] **Post-E4: everything available** — mechanism not yet chosen. Options:
      flag-gated encounter table swap (cheapest) vs. new post-game areas vs.
      folding into the hidden-area system below.
- [ ] **Endgame QoL batch** — most wants are already expansion config flags in
      `include/config/`. Collect them here as they come up, flip in one pass.
      See QOL.md for the full survey of what exists.
- [ ] **Add Kanto's leaders + E4 to the rematch table** — do this alongside the
      Kanto E4 conversion. `gRematchTable` holds **78 of `MAX_REMATCH_ENTRIES`
      (100)**, and the saveblock array is already `u8 trainerRematches[100]`, so
      there are **22 free save-safe slots** — enough for 8 leaders + 4 E4 +
      champion = 13. Going past 100 *is* save-breaking.
      **Decided: Match Call, not Vs. Seeker.** Both are bounded by the same
      table — `GetRematchTrainerIdVSSeeker` (`vs_seeker.c:567`) returns 0 for any
      trainer with no table entry, so the Vs. Seeker does not work on arbitrary
      trainers either, *and* it excludes Elite Four entries. Match Call is
      strictly wider. `I_VS_SEEKER_CHARGING` stays `0`.
- [ ] **Infinite TMs via vendors, NOT reusability.** Decided 2026-07-26.
      `I_REUSABLE_TMS` stays `FALSE` and the TM *relearner* stays off — both are
      back doors to the same thing. Instead, TM shops should stock every TM in
      unlimited quantity. To be done when we reach that part of the game.
      ⚠️ Don't "helpfully" set `P_ENABLE_MOVE_RELEARNERS TRUE` — it switches the
      TM relearner on as a side effect.

## Later

- [ ] Hidden-area rare encounters (Gen 4–9 + legendaries). Build on Hidden
      Grottoes or DexNav — **not** `src/secret_base.c`. See PLAN.md.
- [ ] Safari Zone expansion
- [ ] Repurpose or remove now-dead IV content: Hyper Training, the IV judge NPC,
      Destiny Knot / IV inheritance breeding. These still exist in-game and now
      do nothing useful.

## Someday / big

- [ ] **Kanto region — already in the repo.** 140 `*_Frlg` map folders + layouts +
      encounter tables ship with the expansion, behind the `firered` build target.
      Work is *integration* (include in the Emerald map set, wire connections,
      adapt FRLG scripts, region map), not map creation.
- [ ] Johto region — must be hand-built. No decomp source exists in any portable
      format. This is the genuinely expensive one.

⚠️ Before committing: prototype **wiring one Kanto map into the Emerald build**
end-to-end. That measures the real integration cost, which is currently unknown.
See PLAN.md.

---

## Known deliberate exceptions

- `src/script_pokemon_util.c:395-404` still honours explicitly-scripted IV
  spreads. Left intact on purpose — it's an authoring escape hatch for giving a
  specific Pokémon non-31 IVs from a script. Nothing in the base game uses it, so
  it has no effect today. Remove it if you want the 31s rule to be absolute.

## Ideas not yet committed to

_(park loose thoughts here rather than acting on them immediately)_
