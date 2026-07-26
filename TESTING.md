# emerald+ — testing guide

mGBA 0.10.5 is installed at `C:\Program Files\mGBA\mGBA.exe` and has been launched
with `emerald-plus.gba`. To relaunch:

```powershell
& "C:\Program Files\mGBA\mGBA.exe" "c:\Users\lando\Desktop\emerald+\emerald-plus.gba"
```

## Why the debug menu instead of a doctored save

A hand-built `.sav` is a bad bet on this project: expanding the trainer and flag
space **changed SaveBlock1's layout**, so no reference offsets apply and a subtly
wrong save corrupts rather than fails cleanly.

The expansion's debug menu does the same job in ~2 minutes and keeps working as
the project changes. It is **already compiled into your ROM** — `RELEASE ?= 0` in
a plain `make`, so `DEBUG_OVERWORLD_MENU = DISABLED_ON_RELEASE = TRUE`.

⚠️ It will switch off automatically if you ever build with `make release`.

## Opening it

**Hold `R`, then press `START`** while walking around the overworld.

Default mGBA keys: arrows = D-pad, `X` = A, `Z` = B, `Enter` = START,
`Backspace` = SELECT, `A`/`S` = L/R. Check under Tools → Settings → Keyboard.

## Skipping to the Kanto postgame (~3 minutes)

1. **New game.** Get through the truck/bedroom until you can walk around
   outdoors — about a minute. The debug menu needs you in the overworld.
2. **`R` + `START`** → **Utilities** → **CHEAT START**
   Skips the *entire* Hoenn intro in one action: all three starters at level 20,
   Pokédex, National Dex, running shoes, bike, and every intro state variable.
3. **Give yourself a team that can actually win.** Brock is level 58-59 and
   Falkner 58-59, so the level-20 starters will lose.
   **`R` + `START`** → **Give** → **Pokémon (Simple)** → pick a species →
   **set the level to 100**. Do that 2-3 times.
   *(Pokémon (Complex) also exposes shiny, nature, ability, IVs and EVs.)*
4. **`R` + `START`** → **Flags & Vars** → **Flags** → enter **`2820`**
   (`FLAG_SYS_GAME_CLEAR`) → set **TRUE**. This is the postgame gate the
   Lilycove ferry checks.
5. **`R` + `START`** → **Utilities** → **Warp** → group **13**, map **10**
   (Lilycove Harbor)
6. Talk to the **ferry attendant** → the menu should now list **VERMILION CITY**

### Other handy debug actions

- **Party → Heal Party** — full heal between attempts
- **Give → Max Money**
- **Flags & Vars → Toggle Badge Flags** — all badges at once
- **Utilities → Fly** — quicker than Warp once a map is known

### mGBA conveniences

- **Save states**: `Shift+F1..F9` to save, `F1..F9` to load. Make one *just
  before* talking to Brock so you can retry the battle instantly.
- **Fast-forward**: hold `Tab`.

## Useful numbers for this build

Resolved through the actual preprocessor, not by hand — several of these moved
when the namespaces were merged.

| Flag | Value |
|---|---|
| `FLAG_SYS_GAME_CLEAR` | **2820** (0xB04) |
| `FLAG_SYS_POKEMON_GET` | 2816 |
| `FLAG_SYS_POKEDEX_GET` | 2817 |
| `FLAG_RECEIVED_KANTO_STARTER` | 32 |
| `FLAG_RECEIVED_JOHTO_STARTER` | 33 |
| `FLAG_DEFEATED_BROCK` | 3713 |

| Map | Warp group / map |
|---|---|
| Lilycove City | 0 / 5 |
| Lilycove Harbor | **13 / 10** |
| Vermilion City | **37 / 5** |
| Pewter City | 37 / 2 |
| Pewter City Gym | **40 / 2** |

Layout landmarks: `TRAINER_FLAGS_START` 1280, `SYSTEM_FLAGS` 2816,
`FRLG_FLAGS_START` 3072, `FLAGS_COUNT` 3786.
`TRAINER_LEADER_BROCK` 1169, `TRAINER_JOHTO_FALKNER` 1479.

## What to check

- [ ] **Ferry menu** lists VERMILION CITY (postgame only)
- [ ] **Crossing works** and you arrive in Vermilion City
- [ ] ⚠️ **Arrival tile.** You should land at `(23, 33)`, just north of the S.S.
      Anne dock. This was inferred from the dock warps, never visually checked —
      if you land in a wall or on water, say so; it's a one-line fix.
- [ ] **Kanto is walkable** — routes, buildings, Pokémon Centers all load
- [ ] **Wild encounters** work in Kanto grass
- [ ] **Return ferry** — talk to the Vermilion sailor to sail back to Lilycove
- [ ] **Pewter Gym: Brock + Falkner double battle**
      - both trainers appear, on the same side
      - Falkner's placeholder sprite is a Bird Keeper (expected)
      - winning awards the Boulder Badge and TM39
- [ ] **Shiny rate** — 1/512, so shinies should show up noticeably often
- [ ] **Perfect IVs** — check any caught Pokémon's stats

## Known gaps (not bugs)

- Only **Pewter Gym** is a double battle so far. The other 7 gyms, the Elite Four
  and the champion are still single battles against the Kanto leader alone.
- **11 Johto trainer sprites are placeholders** reusing existing trainer pics.
  Red already has a real one.
- **Johto trainers have no custom movesets** — the game assigns level-up moves.
- **No region map graphics** for Kanto yet.
