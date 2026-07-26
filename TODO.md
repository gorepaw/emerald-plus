# emerald+ — running TODO

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
            Proof of the pattern; clean rebuild verified.
      - [ ] Apply the same pattern to the remaining 7 gyms (Cerulean, Vermilion,
            Celadon, Fuchsia, Saffron, Cinnabar, Viridian). Mechanical now:
            check defeated-flag up front → msgbox intro → `trainerbattle_two_trainers`
            → `goto` the existing Defeated* script so the badge still awards.
      - [ ] Elite Four (4 pairs) + Champion (Blue + Red)
      - [ ] Region map graphics
      - [ ] Replace the 11 placeholder trainer sprites with real GBA-style art
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
