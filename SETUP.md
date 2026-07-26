# emerald+ build setup

Base ROM verified clean: `Pokemon - Emerald Version (USA, Europe).gba`
MD5 `605b89b67018abcea91e693a4dd25be3` · SHA1 `f3ae088181bf583e55daf962a92bb46f4f1d07b7`

Target: **32 MB ROM**, Miyoo Mini Plus on **stock firmware**.
Verified 2026-07-24: stock gpSP loads a 32 MiB ROM fine. Final build will be ~26 MB.

---

## Environment decision (revised)

Originally planned WSL1 so the project could live on `C:\`. After the reboot,
WSL1 turned out to need the legacy "Windows Subsystem for Linux" optional
component — which meant *another* reboot. WSL2 was already working, so:

**Using WSL2, project on the Linux filesystem.**

- Builds are much faster (native Linux FS, no `/mnt/c` overhead)
- Porymap and Explorer reach the project via `\\wsl.localhost\Ubuntu\home\<user>\decomps\...`

---

## ✅ Step 1 — DONE

`wsl --install` ran; WSL core is installed (v2.7.11, kernel 6.18.33). Rebooted.

## ✅ Step 2 — DONE

Ubuntu distro installed via `wsl --install -d Ubuntu --no-launch`.

---

## Step 3 — Create your Ubuntu user (interactive, do this yourself)

Open **Ubuntu** from the Start menu. On first launch it asks for a username and
password.

> When typing the password there is **no visible feedback** — no dots, no
> asterisks. That's normal. Type it and press Enter.

Then close it and come back to Claude. Everything after this is automatable.

---

## Step 4 — Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential binutils-arm-none-eabi gcc-arm-none-eabi libnewlib-arm-none-eabi git libpng-dev python3
```

Toolchain is Ubuntu's `gcc-arm-none-eabi`. **devkitARM is not needed**, despite
what the expansion's INSTALL.md implies.

---

## Step 5 — Clone and build

```bash
mkdir -p ~/decomps && cd ~/decomps
git clone https://github.com/rh-hideout/pokeemerald-expansion.git
cd pokeemerald-expansion
git checkout expansion/1.16.2
make -j$(nproc)
```

`expansion/1.16.2` is the latest stable tag. Don't build `master` — that's the
development branch. No baserom is needed; all assets ship in the repo.

**Verified working 2026-07-24.** Build took 116s on 16 cores.

Output is padded to **exactly 32 MiB** (33,554,432 bytes) — the Makefile's last
step is `gbafix pokeemerald.gba -p`, and `-p` pads to the next power of two.
Actual data ends at `0x1941613`; the rest is 0xFF padding.

```
Memory region         Used Size  Region Size  %age Used
           EWRAM:      226584 B       256 KB     86.43%
           IWRAM:       28392 B        32 KB     86.65%
             ROM:    26482200 B        32 MB     78.92%
```

⚠️ Note which resource is actually scarce: **ROM is at 79%, but EWRAM and IWRAM
are both at ~86%.** RAM is the binding constraint for new features, not ROM
space. Roughly 6.7 MiB of ROM headroom remains; far less RAM headroom.

## Rebuilding later

```bash
cd ~/decomps/pokeemerald-expansion
make -j$(nproc)
cp pokeemerald.gba "/mnt/c/Users/lando/Desktop/emerald+/emerald-plus.gba"
```

---

## Config changes

### ✅ Shiny rate → 1/512 — APPLIED

`include/constants/pokemon.h:104`, commit `4ecadc5146` on branch `emerald-plus`:

```c
#define SHINY_ODDS 128   // was 8 (1/8192). 128/65536 = 1/512
```

Safe to change any time — it's pure logic, not saveblock layout, so it will
**not** invalidate saves.

Work is on branch `emerald-plus`, branched from tag `expansion/1.16.2`. Run
`git diff expansion/1.16.2` any time to see exactly what differs from upstream —
keeps future expansion updates manageable.

### ✅ Shiny Charm → keep default 2 rolls — DECIDED, no change needed

`include/config/item.h:5` — `I_SHINY_CHARM_ADDITIONAL_ROLLS` stays at `2`.
Rerolls stack as `1-(1-p)^N`, so at 1/512 base the Charm gives roughly **1/171**.

Testing aid, if wanted later:
- `include/config/pokemon.h:72-73` — `P_FLAG_FORCE_SHINY` / `P_FLAG_FORCE_NO_SHINY`.
  Assign a spare flag number to force every encounter shiny; fastest way to
  eyeball shiny palettes.

### ✅ Species scope → all 9 gens, DS-style art — DECIDED, no change needed

`P_GEN_1_POKEMON`…`P_GEN_9_POKEMON` all stay `TRUE` (the default), and
`P_GBA_STYLE_SPECIES_GFX` stays `FALSE`. 1,029 species, uniform Gen 4/5-style art.

⚠️ Changing the **newest enabled generation** later shifts Dex flags in the
saveblock and **forces a fresh save file**. Now that this is settled, avoid
touching it once real playtime starts.

**Why not GBA-style art:** it only exists for 385 species (Gen 1–3). Zero
post-Gen-3 species have `_gba` assets — verified by counting `back_gba.png`
across `graphics/pokemon/`. Creating it would mean hand-spriting 644 Pokémon.
Turning `P_GBA_STYLE_SPECIES_GFX` on *does* still build (Gen 4+ sprites are
declared unconditionally), but yields mixed art styles in the same battle.

---

## Deployment notes

- Copy the built `.gba` to the SD card's `GBA` folder — **raw, never zipped**.
  gpSP pages the ROM in on demand and needs it uncompressed.
- Distribute finished work as **BPS or xdelta** patches, never IPS/UPS (those
  can't move data and produce ~14 MB patches).
- Keep the `FLASH1M_V103` save-type string intact — gpSP sniffs it to pick the
  save backend.
- If accuracy bugs appear, stock also ships RetroArch; switching GBA to the
  **mGBA** core is a config change, not a redesign.

---

# Working notes

Hard-won gotchas. Each of these cost real time at least once.

## ⚠️ A stale save looks exactly like a graphics bug

Garbage tiles, missing NPCs, a player that can't move, being unable to leave a
building — all of it is what a save written against an older saveblock layout
looks like. It is **not** a rendering or map-integration problem.

Anything touching `TRAINER_FLAGS_END`, `FLAGS_COUNT` or `VARS_COUNT` shifts every
offset after it. **Delete `emerald-plus.sav` and start fresh.** Cost most of a
session before being identified.

Conversely, pure script/data work (e.g. the gym conversions, the door merge)
leaves SaveBlock1 untouched and saves stay valid.

⚠️ **Don't use `nm ... | grep gSaveblock1` to check this.** That symbol is
`SaveBlock1ASLR`, a fixed-size container, so it always reports `0x3e00` (15872)
no matter what the layout does — a check that cannot fail. What you actually
want is whether the change touched RAM at all:

```bash
arm-none-eabi-size -A build/emerald/src/<file>.o | grep -E '\.data|\.bss'
```

`.data = 0` and `.bss = 0` means the change is pure `.rodata` in ROM, so the
saveblock is untouched by construction. Confirm the data landed in ROM with
`nm pokeemerald.elf | grep <symbol>` — an `0x08...` address is cartridge.

Only `flags[]`, `vars[]` and friends move offsets, i.e. anything touching
`TRAINER_FLAGS_END`, `FLAGS_COUNT` or `VARS_COUNT`.

## The Bash tool is Git Bash, not WSL

`arm-none-eabi-*`, `make` and the repo itself live in WSL. Running them through
the Bash tool **fails silently** — `nm` returned "no symbols" for tilesets that
were present, which sent one investigation down a false path. Always:

```
wsl -d Ubuntu -u lando --cd "~/decomps/pokeemerald-expansion" -- <cmd>
```

For anything with quoting, write a `.sh` to the scratchpad and run that instead.

## Editing over `\\wsl.localhost\...` changes file ownership to root

The Read/Edit tools work fine over the UNC path, but files they write become
`root:root`. `make` still builds (it only reads), but Python/shell scripts
running as `lando` get `PermissionError`. Fix:

```bash
wsl -d Ubuntu -u root -- chown -R lando:lando /home/lando/decomps/pokeemerald-expansion
```

## mGBA holds a lock on the ROM

Copying a new build over `emerald-plus.gba` while mGBA has it open fails with
`cp: Invalid argument`. Without `set -e` the script carries on and reports
success. **Close mGBA, copy, then relaunch** — and verify with an MD5 compare.

## Two repos, on opposite sides of the WSL boundary

| | Location | Remote |
|---|---|---|
| Engine | `~/decomps/pokeemerald-expansion` (WSL) | push `fork`, pull `origin` (rh-hideout) |
| Docs | `C:\Users\lando\Desktop\emerald+` (Windows) | `origin` → gorepaw/emerald-plus |

Both are **public**; pushes authenticate silently, so treat `git push` as
publishing. After engine commits, bump the "N commits" compare link in the
README so it stays honest.

## File-format traps

- **`.party` files have no comment syntax.** `@` is the held-item separator
  (`Metagross @ Sitrus Berry`), so a comment line parses as species-@-item. The
  file says so at line 72.
- **`#pragma` cannot appear inside a brace initializer.** For per-file warning
  suppression use the Makefile idiom already in the tree:
  `$(C_BUILDDIR)/data.o: override CFLAGS += -Wno-override-init`

## The recurring bug class: version-gated FRLG data

FRLG data is gated behind `#if IS_FRLG` all over the tree, so it simply doesn't
exist in an Emerald build. **This is the single most productive thing to check
when anything Kanto misbehaves** — it has now been the cause seven times:
tilesets, trainer tables, map scripts, object-event graphics, object-event
palettes, and door animations. Symptoms range from link errors to NULL derefs to
wrong colours to "the feature silently does nothing."

```bash
grep -rn "#if IS_FRLG" src/ include/ | grep -i <subsystem>
```

Two shapes, two fixes:

- **Additive block** (no `#else`) — just enable it. Most are this.
- **Either/or** (`#if !IS_FRLG ... #else ... #endif`) — the two halves must be
  concatenated, not swapped. Before doing that, check the halves share no symbol
  names, and check what disambiguates entries at runtime.

House idiom for both: replace the condition with `#if 1` and leave a `// emerald+:`
comment rather than deleting the directive, so the block boundaries still line up
against upstream in a diff.

**The runtime is often already region-aware even when the data isn't.** Before
writing any branching logic, grep for `isFrlg` — `field_door.c` needed *zero*
code changes because every draw path already branched on
`gMapHeader.mapLayout->isFrlg`. Only the table was gated. `fieldmap.c`, `shop.c`
and `fldeff_escalator.c` are region-aware the same way.
