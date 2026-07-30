# Content and balance rules

Standing rules for encounter tables and trainer parties, and a record of what
has been applied. **Hoenn is now complete on both counts.**

## Tooling

All four live in the engine repo and survive between sessions:

```bash
# what is on a map now - encounters aggregated by species, plus every trainer
python3 tools/emerald_plus/encounter_report.py Route102:MAP_ROUTE102

# the twelve land slots individually - needed to choose WHICH slot to overwrite
python3 tools/emerald_plus/encounter_report.py --raw MAP_ROUTE102

# every trainer in Hoenn against the rules below; --list names the misses
python3 tools/emerald_plus/party_audit.py

# which Gen 1-3 families Hoenn can actually produce
python3 tools/emerald_plus/availability.py

# one line per table for the whole region, flagging one-species tables
python3 tools/emerald_plus/table_survey.py land_mons
```

`party_audit.py` and `availability.py` are the regression guards. The audit
exits non-zero if anything is short; availability should read **373/386** with
only the 13 legendary and event families outstanding. If either moves, a table
or a party moved with it.

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
slot 4's species on all 153 maps that have a fishing table. That's the lever,
but it's not worth pulling for one species.

### Rules when editing a table

1. **Totals must stay at 100%.** Every patch script asserts this.
2. **Newcomers pay for themselves out of the most over-represented species**,
   never out of a route's signature or its rarities.
3. **Protect the rarities.** Ralts and Seedot (R102), Skitty (R116), Sableye
   (Granite Cave), Larvitar (Jagged Pass), Nuzleaf (R114), Tropius (R119),
   Kecleon, Absol (R120), Chimecho (Mt. Pyre), Heracross and Pinsir and Pikachu
   (Safari Zone), Snorunt (Shoal Cave), Plusle (R110). Patch scripts assert
   protected slots both before *and* after writing and refuse to save if one
   moved.
4. **A newcomer inherits the level of the slot it replaced**, so no level curve
   shifts as a side effect.
5. **Assert the expected current occupant of every slot** before overwriting. A
   misread must fail loudly rather than silently corrupt the table — this caught
   `route23.inc` having 16 badge references instead of 8, and caught Altering
   Cave owning nine tables under one map name.

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

**The two documented exceptions**, both encoded in `party_audit.py`:

- **Wally's Mauville battle** keeps its single Ralts. The scene is that he has
  just caught it. His later battles carry full parties.
- **The Winstrate family** is exempt from the level floor. Their battles are
  scripted from `Route111/scripts.inc` but fought inside a house, and a living
  room has no grass.

### Rules when filling parties

1. **Fillers come from the map's own wild table** — land *and* water together —
   preferring a species that also fits the trainer class's theme, and falling
   back to the class theme alone where the map has no table.
2. **Only slots worth 5% or more are eligible.** The 4% and 1% slots are where a
   route keeps its rarities; handing a random Bug Catcher a Larvitar both spikes
   the fight and cheapens the encounter.
3. **Leaders' aces stay last.** Additions go *before* the final Pokémon.
4. **Leaders get authored movesets**; rank-and-file get none, so the game
   assigns level-up moves. Same convention as the Johto trainers.
5. `IVs:` lines in `.party` are **inert** — `P_ALL_PERFECT_IVS` overrides them at
   the generation choke point. Don't bother tuning them.

⚠️ **A blank line between Pokémon is required by the format.** Appending a mon
straight after the previous one's `IVs:` line produces a file that a naive
line-walking parser still reads correctly and `trainerproc` does not. Verify
party sizes by splitting on blank lines, never with the same walker that wrote
the file.

---

## Gen 1–3 availability in Hoenn

**373 of 386 reachable.** The 13 outstanding are the Gen 1–2 legendaries plus
the two event mons — Articuno, Zapdos, Moltres, Mewtwo, Mew, Raikou, Entei,
Suicune, Lugia, Ho-Oh, Celebi, Jirachi, Deoxys — deliberately skipped, since
none belongs in a normal encounter table. They are their own job.

The unit that matters is the **evolutionary family**, not the species: Gen 3
lets you walk a family both ways, evolving forward and breeding backward, so one
member in one wild table makes the whole family obtainable. 55 families had zero
Hoenn presence and were placed in 82 slots across 57 maps.

Where a table was one species repeated, the newcomers did double duty:

| Map | Was | Now also has |
|---|---|---|
| Mt. Pyre 1F–6F | 100% / 90% Shuppet, six floors | Gastly, and Cubone upstairs — Shuppet still dominant at 70% |
| Shoal Cave ice room | Spheal + Zubat | Swinub, Sneasel, Jynx, Delibird — an actual ice cave |
| Magma Hideout ×8 | identical Geodude rooms | Magmar |
| Safari Zone | — | Tauros, Kangaskhan, Chansey, Scyther, Farfetch'd, Lickitung, Togepi |
| Route 119 | Zigzagoon + Linoone + Oddish held 90% | Roselia, Venonat, Tangela, Yanma |
| 32 identical sea maps | Tentacool 60 / Wingull 35 / Pelipper 5 | Krabby, Shellder, Slowpoke, Poliwag, Qwilfish, Mantine, Lapras |

**Surskit and Roselia were absent from this build's Emerald tables entirely**
despite reading as Hoenn natives. They're back, on Routes 102 and 119.

### Trade evolutions

Gengar, Alakazam, Machamp and Golem need a trade and there is no second console.
The expansion ships `ITEM_LINKING_CORD` as an `EVO_ITEM` alternative on every
`EVO_TRADE` entry, but nothing sold or gave one — the item existed and was
unreachable. **It is now stocked by the Battle Frontier's vitamin clerk at 1 BP.**

Every other trade evolution was already solvable: Metal Coat, King's Rock,
Dragon Scale, Up-Grade and the Deep Sea items all have in-game sources, and
Feebas reaches Milotic through beauty.

Four places must agree when that menu changes — the item list, the description
list, the vendor's `tNumItems`, and the script's `case` list.

### Pseudo-legendary availability

All four lines reachable pre-Elite Four:

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

Now **387 entries**. Every species catchable in Hoenn is in the Hoenn dex — the
convention since the first Gen 1–2 additions — which as of the availability pass
means essentially the whole Gen 1–3 list.

The 148 added in that pass include species that were always reachable and never
listed: Onix, Ditto, the two starter trios given by Mr. Stone and Steven, and the
entire Gen 2 population of the Safari Zone's expansion areas.

⚠️ **Two traps here, both of which cost a build:**

- **`FOREACH_SPECIES_IN_HOENN_DEX_ORDER` is not the only list in that file.**
  `FOREACH_SPECIES_IN_KANTO_DEX_ORDER` sits further down with its own `F(...)`
  entries. Grepping the whole file for `F(BUTTERFREE)` hits the Kanto list and
  falsely reports the species as already in the Hoenn dex. Extract the Hoenn
  macro body alone.
- **`HOENN_DEX_COUNT` must track the LAST entry** of the macro. Appending species
  without moving it leaves them silently outside the dex.

The real guard is the build: `sHoennToNationalOrder` is sized
`HOENN_DEX_COUNT - 1`, so a count that disagrees with the entry list fails with
*"excess elements in array initializer"*.

**Counting the macro by hand is unreliable** — a multi-line C comment inside it
swallows its own newlines, so the definition continues past it. Naive "stop at
the first line without a trailing backslash" parsing under-reports the list
badly. Strip comments first.

Dex flags are keyed on **national** number, so all of this is display ordering
only and never touches the saveblock. EWRAM held at 226,760 B across the whole
pass, which confirms it.

---

## Still open

- **The 13 legendary and event families.** Wild tables are the wrong home; they
  want one-time statics with a flag each, at designed locations.
- **Kanto.** None of these rules has been applied to the 419 `_Frlg` maps —
  neither encounters nor parties. `party_audit.py` scopes to Hoenn deliberately.
- **The Battle Frontier's own facilities.** Battle Pike, Battle Pyramid and
  Trainer Hill carry their own tables and are untouched.
