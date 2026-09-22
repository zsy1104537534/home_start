# HANDOFF — Home Start (CDDA mod) / 交接说明

This document exists so that **anyone — a human developer or an AI assistant (Claude, ChatGPT, Gemini…) — can pick this project up from the repository alone**, without access to the machine it was developed on.

如果你要把这个项目给别的 AI 看，**直接把仓库链接发给它**，让它先读这份 `HANDOFF.md` 和 `README.md` 即可：
<https://github.com/zsy1104537534/home_start>

> **Note:** the engine findings in §5 were re-verified against the CDDA source at commit `7b2efa5` after an external review (2026-09-22) caught one wrong claim. Anything in this file that is not backed by a file/line reference should be treated as unverified.

---

## 1. What this is

**Home Start** (`home_start`) is a small content mod for **Cataclysm: Dark Days Ahead**. It changes how a new character begins: instead of the default randomised starts, you wake up inside your own flat on an upper floor of a downtown apartment tower, five days into the Cataclysm.

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
    │                      #   prof_homebody, flags CITY_START + LONE_START, eoc [EOC_home_start].
    │                      #   Start dates are left at the vanilla defaults on purpose - the engine
    │                      #   derives them from the world option SEASON_LENGTH (scenario.cpp), so
    │                      #   hardcoding them would break non-default season lengths.
    ├── professions.json   # profession id "prof_homebody" ("Homebody Survivor"): dress_shirt, jeans,
    │                      #   socks, sneakers, NO_BONUS_ITEMS, points -1
    ├── start_locations.json  # start_location id "sloc_apt_interior": upper-floor apartment tower
    │                      #   terrains only (no ALLOW_OUTSIDE, so you never spawn on the ground floor)
    ├── eocs.json          # start script, see §4
    ├── riot_patch.json    # overmap_terrain patch: removes PP_GENERATE_RIOT_DAMAGE from
    │                      #   apartments_con_tower_* interiors (copy-from apartments_tower_any + delete)
    └── lang/
        ├── po/home_start.pot      # gettext template (generated)
        ├── po/zh_CN.po            # Simplified Chinese translation source
        └── mo/zh_CN/LC_MESSAGES/home_start.mo   # compiled translation the game loads
```

## 3. Design intent

A quiet, self-contained opening. Your floor is clean and lootable; the streets outside are not. The tone is "you have been ill for days in your own home and the world ended while you slept" — the scenario description in `scenarios.json` tells that story (the key snapped off in the lock on the day of the Cataclysm).

## 4. How the start script works (`eocs.json`, EOC `EOC_home_start`)

1. Shows a short popup so the player knows where they are.
2. Teleports the player onto a bed in the flat, searching with **staged radii: 6 → 12 → 24 tiles**, then a bathtub within 24 tiles as the last fallback. The staging is not cosmetic — see §5: a single large radius would always pick the north-west-most bed of the floor. Roughly 7 % of apartment doors generate as `t_door_locked_alarm`, which prying will set off; that is left as an intended surprise rather than patched.
3. Gives the player a **crowbar** with `u_spawn_item` (it lands in the inventory, or is wielded if the hands are free), so a locked door is never a dead end, and drops a **sewing kit** at the player's feet with `map_spawn_item` (no `loc` = the tile they woke on, which is inside the flat by definition).
4. Clears the player's floor: `u_run_monster_eocs` runs the vanilla `BEGONE_SHADOW` spell on every monster within 60 tiles that is **indoors**, plus everything within 20 tiles regardless. Note that inside `u_run_monster_eocs` the talker `u` is the **monster** being processed, not the player — that is exactly what makes the `{"not": "u_is_outside"}` filter mean "indoor monsters".
5. Does **not** touch hunger or thirst: the character starts in a normal state (an earlier iteration fed the player and it was removed on request).

## 5. Engine findings (verified against `7b2efa5`)

* **Text style check**: CDDA enforces *two spaces after a sentence-ending period* in English strings. Violations show up as `text_style_check_reader.cpp ... insufficient spaces at this location. 2 required, but only 1 found.` and make `--check-mods` exit non-zero. Chinese text is exempt.
* **`u_location_variable` returns the first match, not the nearest one.** The search walks a bounding box (`map::points_in_radius` builds a `tripoint_range(min, max)`, iterated row-major) and breaks on the first hit (`src/npctalk.cpp`, the `search_target` branch). With `target_max_radius: 40` on a 48×48 floor, the player is therefore always sent to the north-west-most bed — possibly in a neighbouring flat or building. **Fix in use:** grow the radius in stages.
* **`target_min_radius` is a skip filter, not a starting distance**: `if( rl_dist( ... ) <= min_target_dist ) continue;`. A value of 1 excludes the tile the player stands on *and* everything adjacent, so "find nearby furniture" with min 1 misses the bedside nightstand.
* **`passable_only` does not apply to furniture or terrain searches** — it only affects the random-offset branch of the location search.
* **`u_spawn_item` works.** It reaches `talker_character::i_add_or_drop( item &, bool force_equip )` (`src/talker_character.cpp`), which wears the item if it is wearable, else wields it when there is no hand conflict, and otherwise falls through to `Character::i_add_or_drop( item & )` with the default count of 1 — i.e. the item simply goes into the inventory. `force_equip: true` means "wear or wield it", **not** "spawn one". (An earlier version of this file wrongly claimed the flag was passed into the item count and that nothing spawned; that was a misreading of the `Character::` overload and has been corrected.)
* **`map_spawn_item` without `loc`** places the item on the talker's own tile — useful when the exact container is unknown.
* Monster banish EOCs use the avatar-side spell cast; `BEGONE_SHADOW` is vanilla, so this mod inherits its limits.
* General CDDA notes that are **not** used by this mod (kept here so they are not confused with the above): `upgrades.half_life` is multiplied by the world option `EVOLUTION_INVERSE_MULTIPLIER` (larger = slower evolution); `MONSTER_WHITELIST` with `mode: EXCLUSIVE` only takes effect when at least one whitelist entry is non-empty.

## 6. How to verify a checkout

The mod is pure JSON, so verification is mechanical. Run these **from the game directory**, with the game **not running** (the checker starts a game process):

```sh
cataclysm-tiles.exe --check-mods home_start            # exit code 0 = clean
json_formatter.exe data/mods/home_start/modinfo.json   # run for every .json file; a diff means reformat
```

The first run after editing mod files often prints `Stale game data detected` (flexbuffer cache) — run it a second time.

Real errors appear in `config/debug.log` as `Json error: file …, at line X, character Y:` and the **reason is on the following line**.

Translation coverage: every user-facing English string must exist as a `msgid` in `home_start/lang/mo/zh_CN/LC_MESSAGES/home_start.mo`. The `.po`/`.pot` sources in `home_start/lang/po/` are generated from the JSON by the project's `build-mo.js` helper; `msgfmt -o <out.mo> zh_CN.po` produces a byte-equivalent file.

## 7. Known limitations / maintenance risks

* References vanilla ids that could be renamed upstream: `apartments_tower_any`, `PP_GENERATE_RIOT_DAMAGE`, `BEGONE_SHADOW` (the spell used to clear monsters), `f_bed`, `f_bathtub`, and the apartment terrain ids in `start_locations.json`.
* The riot-damage patch only covers `apartments_con_tower_*`. Other apartment mapgens (`mod_tower`, `s_apt`, the apartment-complex set) still generate riot damage.
* `start_locations.json` currently lists six upper-floor terrains (`..._002/102/012/112/013/113`). The half-floors `..._011`/`..._111` look like the same plan and could probably be added, but that has not been verified in game — check the floor you spawn on before adding them.
* The whole mod only affects **new characters and newly generated maps**.
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
* 文件在哪：仓库根目录放文档，**`home_start/` 子目录就是 mod 本体**，复制进 `<游戏>/data/mods/` 即可；`lang/po/` 是翻译源文件，`lang/mo/` 是游戏读取的成品。
* 开局脚本做什么：弹一句提示 → 按 6→12→24 格**分阶段**找床（一次性大半径会永远传送到该层最西北角那张床）→ 把撬棍放进背包（保证不会把自己锁在门里）、针线盒丢在脚下 → 对 60 格内**室内**的怪 + 20 格内所有怪施放 `BEGONE_SHADOW` 清场；**不改饥饿口渴**。
* 验证方式：`cataclysm-tiles.exe --check-mods home_start`（退出码 0）、`json_formatter.exe`（官方格式）、中文语言包覆盖率。
* 踩过的坑见第 5 节（英文句末必须两个空格、`u_location_variable` 取的是"第一个"不是"最近的"、`target_min_radius` 是"跳过"语义、`passable_only` 对家具搜索无效、`u_spawn_item` 其实是好的）。
