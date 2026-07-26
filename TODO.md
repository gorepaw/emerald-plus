# emerald+ — running TODO

## ▶ RESUME HERE

**State as of 2026-07-26:** engine fork at `271d00bdf9`, **16 commits** since
`expansion/1.16.2`, both repos pushed and clean. ROM 84.81%, SaveBlock1
15,744/15,872 (128 free). Kanto is walkable postgame via the Lilycove ferry,
all 8 gyms are Kanto+Johto double battles, and Kanto doors now animate.

**Next task: the Elite Four + Champion.** Same restructure as the gyms — see
"Kanto postgame" below and PLAN.md for the pairing table.

| Slot | Kanto | Johto |
|---|---|---|
| E4 1 | Lorelei | Will |
| E4 2 | Bruno | Karen |
| E4 3 | Agatha | Koga |
| E4 4 | Lance | Chuck |
| Champion | Blue | **Red** |

All five Johto-side trainers already exist (`TRAINER_JOHTO_WILL`, `_KAREN`,
`_KOGA`… wait — Koga is a *Kanto* leader trainer, `TRAINER_LEADER_KOGA`, reused
here; that is canonical, he was promoted to the Johto E4 in GSC) plus
`TRAINER_JOHTO_CHUCK` and `TRAINER_JOHTO_RED`. Red already has a real sprite.

⚠️ E4 rooms differ from gyms — they likely use `trainerbattle_continue_script`
or map-script triggers rather than a plain `trainerbattle_single`, and the
champion sequence has a long post-battle cutscene. **Survey the scripts first**,
as was done for the gyms; do not assume the gym pattern transfers unchanged.

Also still unplayed: 7 of the 8 gyms (only Pewter has been tested in-game).

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
