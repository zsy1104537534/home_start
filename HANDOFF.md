# HANDOFF — Home Start (CDDA mod) / 交接说明

This document exists so that **anyone — a human developer or an AI assistant (Claude, ChatGPT, Gemini…) — can pick this project up from the repository alone**, without access to the machine it was developed on.

如果你要把这个项目给别的 AI 看，**直接把仓库链接发给它**，让它先读这份 `HANDOFF.md` 和 `README.md` 即可：
<https://github.com/zsy1104537534/home_start>

---

## 1. What this is

**Home Start** (`home_start`) is a small content mod for **Cataclysm: Dark Days Ahead**, the open-source post-apocalyptic roguelike. It changes how a new character begins: instead of the default randomised starts, you wake up inside your own flat on an upper floor of a downtown apartment tower, five days into the Cataclysm.

* Owner / maintainer: GitHub [@zsy1104537534](https://github.com/zsy1104537534)
* License: **CC-BY-SA 3.0** (the same license as CDDA's own content) — see `LICENSE`
* Developed and verified against CDDA **0.I / build `2026-09-19-2324`** (commit `7b2efa5`)
* Dependency: `dda` only. No new items, monsters or tiles — everything it spawns is vanilla, so **no tileset work is required**.

## 2. Repository layout

```
.
├── README.md          # player-facing documentation (English + Chinese)
├── LICENSE            # CC-BY-SA 3.0
├── .gitattributes     # forces LF so JSON stays byte-identical to json_formatter output
├── HANDOFF.md         # this file
└── home_start/        # ← this folder is the mod; copy it into <game>/data/mods/
    ├── modinfo.json       # MOD_INFO: id home_start, category content, dependency dda, version 1.0.0
    ├── scenarios.json     # scenario id "home_start": allowed_locs [sloc_apt_interior], profession
    │                      #   prof_homebody, start_of_cataclysm day 56 / start_of_game day 61 08:00,
    │                      #   flags CITY_START + LONE_START, eoc [EOC_home_start]
    ├── professions.json   # profession id "prof_homebody" ("Homebody Survivor"): dress_shirt, jeans,
    │                      #   socks, sneakers, NO_BONUS_ITEMS, points -1
    ├── start_locations.json  # start_location id "sloc_apt_interior": upper-floor apartment tower
    │                      #   terrains only (no ALLOW_OUTSIDE, so you never spawn on the ground floor)
    ├── eocs.json          # start script, see §4
    ├── riot_patch.json    # overmap_terrain patch: removes PP_GENERATE_RIOT_DAMAGE from apartment
    │                      #   tower interiors (copy-from apartments_tower_any + delete flags)
    └── lang/mo/zh_CN/LC_MESSAGES/home_start.mo   # Simplified Chinese translation
```

## 3. Design intent

A quiet, self-contained opening. Your floor is clean and lootable; the streets outside are not. The tone is "you have been ill for days in your own home and the world ended while you slept" — the scenario description in `scenarios.json` tells that story (the key snapped off in the lock on the day of the Cataclysm).

## 4. How the start script works (`eocs.json`, EOC `EOC_home_start`)

1. Shows a short popup so the player knows where they are.
2. Teleports the player next to a bed (`f_bed`), falling back to a bathtub (`f_bathtub`), because the engine's own start placement scoring tends to drop the player in a corridor or at the map edge.
3. Leaves a **sewing kit** and a **crowbar** in the flat: on any nearby furniture within 10 tiles if there is some, otherwise at the player's feet. The crowbar is deliberate — apartment doors can spawn as `t_door_locked_interior`.
4. Clears monsters from the player's own floor (radius 60, only where the player counts as indoors) and everything within radius 20 unconditionally.
5. Does **not** touch hunger or thirst: the character starts in a normal state (an earlier iteration fed the player and it was removed on request).

## 5. Engine findings worth knowing before editing

These were established by reading the CDDA source at the pinned commit and by testing; they save a lot of time:

* **Text style check**: CDDA enforces *two spaces after a sentence-ending period* in English strings. Violations show up as `text_style_check_reader.cpp ... insufficient spaces at this location. 2 required, but only 1 found.` and make `--check-mods` exit non-zero. (Chinese text is exempt.)
* **`u_spawn_item` is broken in 0.I**: `receive_item()` passes the `force_equip` bool into `Character::i_add_or_drop(item&, int qty, …)`'s **qty** parameter (`src/npctalk.cpp` → `src/character_inventory.cpp`). With `force_equip: false` the item count is 0, i.e. **nothing is spawned**; with `true` exactly one item is added and **it is not equipped**. This mod therefore uses `map_spawn_item` instead.
* **Monster evolution**: `upgrades.half_life` is "days in which half of the monsters upgrade", and it is multiplied by the world option `EVOLUTION_INVERSE_MULTIPLIER` (`src/monster.cpp`, `scaled_half_life`). A value of 2.00 therefore means evolution is **twice as slow**, not twice as fast.
* **Monster filters**: `MONSTER_WHITELIST` with `mode: EXCLUSIVE` only takes effect when at least one whitelist entry is non-empty (`src/mongroup.cpp`); an empty list blacklists nothing.
* **Formatting**: the release ships `json_formatter.exe`; run it on every JSON file or CI-style checks will fail.

## 6. How to verify a checkout

The mod is a pure JSON data mod, so verification is mechanical. From the game directory (the game must **not** be running — the checker starts a game process):

```sh
cataclysm-tiles.exe --check-mods home_start     # exit code 0 = clean
json_formatter.exe home_start/modinfo.json      # repeat per JSON file; a diff means it needs reformatting
```

The first run after editing mod files often prints `Stale game data detected` (flexbuffer cache) — run it a second time.

Real errors appear in `config/debug.log` as `Json error: file …, at line X, character Y:` and the **reason is on the following line**.

Translation coverage: every user-facing English string must exist as a `msgid` in `home_start/lang/mo/zh_CN/LC_MESSAGES/home_start.mo`.

## 7. Known limitations / maintenance risks

* References vanilla ids that could be renamed upstream: `apartments_tower_any`, `PP_GENERATE_RIOT_DAMAGE`, `BEGONE_SHADOW` (the spell used to clear monsters), `f_bed`, `f_bathtub`, and the apartment terrain ids in `start_locations.json`.
* The riot-damage patch only affects newly generated apartment towers, and the whole mod only affects **new characters**.
* Incompatible with total conversions that remove cities (`innawood`, `The Backrooms`, `Sky Island`, `Defense Mode`).
* Not accepted into the official CDDA repository as-is: `doc/IN_REPO_MODS.md` requires a long-term curator and forbids mods whose only purpose is to switch off a working vanilla feature; the riot patch would have to be argued as part of the "clean flat" concept.

## 8. Publishing (for whoever maintains it)

```sh
# first time on a machine: make gh the git credential helper
gh auth setup-git

git add -A && git commit -m "…" && git push
gh release create v1.0.0 <zip> --title "Home Start 1.0.0" --notes-file <notes.md>
```

The release ZIP must contain a top-level `home_start/` folder so players can extract it straight into `data/mods/`.

## 9. 中文速览

* 这是什么：CDDA 的数据 mod，让你在自己家（市中心公寓楼高层）里开局；只有 JSON，没有新增物品/怪物/贴图。
* 文件在哪：仓库根目录放文档，**`home_start/` 子目录就是 mod 本体**，复制进 `<游戏>/data/mods/` 即可。
* 开局脚本做什么：弹一句提示 → 传送到床边 → 屋里留针线盒 + 撬棍 → 清空本层怪物（60 格）与身边 20 格；**不改饥饿口渴**。
* 验证方式：`cataclysm-tiles.exe --check-mods home_start`（退出码 0）、`json_formatter.exe`（官方格式）、中文语言包覆盖率。
* 踩过的坑见第 5 节（英文句末必须两个空格、0.I 的 `u_spawn_item` 是坏的、进化倍率是"变慢"、白名单空列表不生效）。
