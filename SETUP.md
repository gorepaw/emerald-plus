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
