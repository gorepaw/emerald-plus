# Content and balance rules

Standing rules for encounter tables and trainer parties, plus a record of which
routes have been signed off. Applied **route by route in story order**, not
game-wide — "retroactive" means back over the routes already covered, not all
1,491 trainers in both regions.

## Tooling

Both live in the engine repo and survive between sessions:

```bash
# what is on a map now - encounters aggregated by species, plus every trainer
python3 tools/emerald_plus/encounter_report.py Route102:MAP_ROUTE102

# the twelve land slots individually - needed to choose WHICH slot to overwrite
python3 tools/emerald_plus/encounter_report.py --raw MAP_ROUTE102

# check every signed-off route against the rules below
python3 tools/emerald_plus/party_audit.py --done
```

`party_audit.py` exits non-zero if anything is short. It reports; it never edits.

---

## Encounter slot rates

Rates are fixed **by slot position** — you don't choose a percentage, you choose
which slot to overwrite.

| Table | Slot rates |
|---|---|
| Land (12 slots) | 20 · 20 · 10 · 10 · 10 · 10 · 5 · 5 · 4 · 4 · 1 · 1 |
| Water (5) | 60 · 30 · 5 · 4 · 1 |
| Rock Smash (5) | 60 · 30 · 5 · 4 · 1 |
| Fishing — Old Rod | 70 · 30 |
| Fishing — Good Rod | 60 · 20 · 20 |
| Fishing — Super Rod | 40 · 40 · 15 · 4 · 1 |

⚠️ **The Good Rod's rarest slot is 20%.** The only sub-15% slots in the entire
fishing structure are Super Rod's 4% and 1%. A "1% on the Good Rod" encounter is
not expressible without editing the global rate constants — which would move
slot 4's species on all 153 maps that have a fishing table (slot 3 and slot 4
differ on 145 of them). Not worth it for one species, but that's the lever.

### Rules when editing a table

1. **Totals must stay at 100%.** Every patch script asserts this.
2. **Newcomers pay for themselves out of the most over-represented species**,
   never out of a route's signature or its rarity.
3. **Protect the rarities.** Ralts and Seedot (R102), Skitty (R116), Sableye
   (Granite Cave), Mawile (Victory Road). Patch scripts assert protected slots
   both before *and* after writing and refuse to save if one moved.
4. **A newcomer inherits the level of the slot it replaced**, so no level curve
   shifts as a side effect.
5. **Assert the expected current occupant of every slot** before overwriting. A
   misread must fail loudly rather than silently corrupt the table — this caught
   `route23.inc` having 16 badge references instead of 8.

---

## Trainer party rules

| Who | Party size |
|---|---|
| Every trainer | **≥ 2** |
| Trainers inside a gym | **≥ 3** |
| Roxanne | **4** |
| Brawly | **5** |
| All remaining leaders + Elite Four | **6** |

**Level floor:** no trainer Pokémon may be below the minimum wild level of the
map it stands on.

### Rules when filling parties

1. **Fillers come from the route's own wild table** where it has one, and from
   the trainer class's theme where it doesn't (Magnemite for a Guitarist,
   Machop for a Black Belt, Zubat for an Aqua grunt, Aron in a Rock gym).
2. **Leaders' aces stay last.** Additions go *before* the final Pokémon — the
   fill scripts have an insert-before-last mode for exactly this.
3. **Leaders get authored movesets**; rank-and-file get none, so the game
   assigns level-up moves. Same convention as the Johto trainers.
4. `IVs:` lines in `.party` are **inert** — `P_ALL_PERFECT_IVS` overrides them at
   the generation choke point. Don't bother tuning them.

---

## Signed off so far

| Map | Encounters added | Party work |
|---|---|---|
| Route 101 | Rattata 5%, Sentret 5% | — |
| Route 102 | Caterpie 5%, Weedle 5%, Pidgey 1% | Calvin +1 |
| Route 103 | Spearow 5%, Hoothoot 5%; Poochyena 60→50 | 6 rivals +1 each (Zigzagoon L4 as **lead**), Miguel/Marcos/Rhett/Pete/Isabelle +1 |
| Route 104 | — | Winston/Cindy/Darian +1 |
| Petalburg Woods | Metapod 5%, Kakuna 5% | Aqua grunt +1; **Lyle L3 → L5** (level floor) |
| Rustboro Gym | — | Josh +2, Tommy +1, Marc +1, **Roxanne 3→4** (+Aron 12) |
| Route 116 | Houndour 5%, Hoothoot 5% | Joey/Jerry/Clark/Janice/Karen +1 |
| Rusturf Tunnel | Geodude 16%, Meditite 4% (was **100% Whismur**) | Aqua grunt +1 |
| Route 106 | — | Kyla +1, Ned +1 |
| Dewford Gym | — | 6 trainers 1→3 each, **Brawly 3→5** |
| Granite Cave B2F | Onix 9%, **Beldum 1%** | — |
| Meteor Falls ×4 | **Dratini** Good Rod 20% L15–25, Super Rod 15%; **Dragonair** Super Rod 1% L25–35 | — |
| Jagged Pass | **Larvitar 1%** L20 | — |
| Victory Road B1F/B2F | **Pupitar 1%** L42 / L44 | — |
| Route 107 | *deferred* | Tony +Wailmer, Beth +Wingull, Camron +Goldeen |
| Route 108 | *deferred* | Jerome +Wailmer, Matthew +Tentacool, Missy +Horsea |
| Route 109 | *deferred* | Austina +Goldeen, Gwen +Wingull, Edmond +Machop, Ricky +Marill, Hailey +Azurill |
| Seashore House | — | Johanna +Marill |

**Next up:** Route 110 → Mauville City, where **Wattson goes 4→6** and the
six-Pokémon leader rule starts biting.

### Deferred, deliberately

- **The Slateport approach's water and rod tables.** Route 107, Route 108,
  Route 109 and Slateport City share **one identical table** — Tentacool 60 /
  Wingull 35 / Pelipper 5 on the surface, and a Super Rod that is five slots of
  Wailmer. Editing them one route at a time would mean writing the same patch
  four times, so all the region's water goes in one pass later. Their *parties*
  are done.
- **Rematch parties.** Trainers with Match Call rematches (`_2` … `_5`) have
  never been touched — `RICKY_1` now has two Pokémon while `RICKY_2` still has
  one Linoone. Every signed-off route has this. One sweep, once the rules stop
  moving.

⚠️ Most of this stretch is **Surf-gated**: only Route 109's beach and the
Seashore House are reachable when the ferry drops you off, which is why their
trainers sit at L11–13 while Routes 107, 108 and Route 109's water are L24–27.
The Abandoned Ship on Route 108 needs Dive and is a much later stop.

### Pseudo-legendary availability

Deliberately opened up early. All four lines reachable pre-Elite Four:

| Line | Where | Rate |
|---|---|---|
| Beldum → Metagross | Granite Cave B2F | 1% |
| Bagon → Salamence | Meteor Falls B1F_2R | 25% (vanilla) |
| Dratini → Dragonite | Meteor Falls, Good Rod / Super Rod | 20% / 15% |
| Larvitar → Tyranitar | Jagged Pass (Larvitar), Victory Road B1F/B2F (Pupitar) | 1% |

Beldum and Dratini were normally post-game only. Pupitar at L42–44 is still well
short of its **L55** Tyranitar evolution.

Meteor Falls copies the shape FRLG's Safari Zone already uses for this line:
Dratini on the Super Rod's 15% slot and **Dragonair on its 1% slot**, in all four
fishing rooms. Dratini itself stays likelier on the Good Rod (20% vs 15%) because
the Super Rod's rates are 40/40/15/4/1 and there is nothing between 15 and 40 —
what the better rod buys you is the evolved form, which no other rod can produce.

---

## Hoenn Pokédex

Now **239 entries**, 25 past vanilla's 214. Every species made catchable in
Hoenn has been added.

⚠️ **Two traps here, both of which cost a build:**

- **`FOREACH_SPECIES_IN_HOENN_DEX_ORDER` is not the only list in that file.**
  `FOREACH_SPECIES_IN_KANTO_DEX_ORDER` sits further down with its own `F(...)`
  entries. Grepping the whole file for `F(BUTTERFREE)` hits the Kanto list and
  falsely reports the species as already in the Hoenn dex. Extract the Hoenn
  macro body alone.
- **`HOENN_DEX_COUNT` must track the LAST entry** of the macro. It was
  `HOENN_DEX_DEOXYS + 1`; it's now `HOENN_DEX_TYRANITAR + 1`. Appending species
  without moving it leaves them silently outside the dex.

The real guard is the build: `sHoennToNationalOrder` is sized
`HOENN_DEX_COUNT - 1`, so a count that disagrees with the entry list fails with
*"excess elements in array initializer"*.

**Counting the macro by hand is unreliable** — a multi-line C comment inside it
swallows its own newlines, so the definition continues past it. Naive "stop at
the first line without a trailing backslash" parsing under-reports the list
badly. Strip comments first.

Dex flags are keyed on **national** number, so all of this is display ordering
only and never touches the saveblock.
