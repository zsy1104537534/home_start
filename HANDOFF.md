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
* **Supported versions:** CDDA **0.I** stable (`0.I` / `0.I-1`; this mod is developed and verified on build `2026-09-19-2324`, commit `7b2efa5`). **0.H and older do not work** — the start script's `u_run_monster_eocs` effect does not exist there (`src/npctalk.cpp`: absent in the `0.H` tag, present in `0.I`), so the scenario fails to load. On **experimental** builds the mod plays normally and riot damage is handled by the same `riot_patch.json`: it deletes the new `post_process_generators: [ "riot_damage" ]` entry as well as the old flag, and the key a given version does not know is ignored (see §5). That experimental path is **not yet verified in game**.

## 2. Repository layout

```
.
├── README.md          # player-facing documentation (English + Chinese)
├── LICENSE            # CC-BY-SA 3.0
├── .gitattributes     # forces LF so JSON stays byte-identical to json_formatter output
├── HANDOFF.md         # this file
└── home_start/        # ← this folder is the mod; copy it into <game>/mods/  (NOT data/mods/ - see §5)
    ├── modinfo.json       # MOD_INFO: id home_start, category content, dependency dda. The version here is the
    │                      #   single source of truth - build-mo.js reads it into the .pot/.po/.mo headers.
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
    ├── riot_patch.json    # overmap_terrain patch for the apartment tower interiors, copy-from
    │                      #   apartments_tower_any (required for see_cost etc.). One `delete` removes
    │                      #   BOTH keys: the flags PP_GENERATE_RIOT_DAMAGE (0.I) and the
    │                      #   post_process_generators entry riot_damage (experimental builds).
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
3. Leaves a **crowbar** on furniture 1-2 tiles from the player (the minimum radius skips the tile they are lying on, so it never lands on the bed); if there is no such furniture, `u_spawn_item` hands it over instead, so it ends up in your **hands** rather than in a pocket (a crowbar is too long to fit one — see §5 on `i_add_or_drop`). Either way a locked door is never a dead end.
4. Clears the player's floor: `u_run_monster_eocs` runs the vanilla `BEGONE_SHADOW` spell on every monster within 60 tiles that is **indoors**, plus everything within 20 tiles regardless. Note that inside `u_run_monster_eocs` the talker `u` is the **monster** being processed, not the player — that is exactly what makes the `{"not": "u_is_outside"}` filter mean "indoor monsters".
5. Does **not** touch hunger or thirst: the character starts in a normal state (an earlier iteration fed the player and it was removed on request).

## 5. Engine findings (verified against `7b2efa5`)

* **Scenario and profession text is looked up with a gettext *context*, not as a plain string.** `scenario::load` calls `to_translation( "scenario_male", name )` and `to_translation( "scenario_female", name )`, plus `scen_desc_male` / `scen_desc_female` for the description and `start_name` for the start-location line (`src/scenario.cpp`, the block starting with the comment "pretty much the same as in profession::load, but different contexts for pgettext"). `profession::load` does the same with `profession_male`, `profession_female`, `prof_desc_male`, `prof_desc_female` (`src/profession.cpp`). The engine's lookup key is `context + "\004" + msgid` (`TranslationManager::Impl::ConstructContextualQuery`), the same encoding GNU gettext uses for `msgctxt` entries. A `.mo` that contains only the bare `msgid` therefore translates **none** of these strings and the game silently shows English — which is exactly what happened in 1.0.4: the mod-list name/description (plain lookups, no context) were Chinese while the scenario name, scenario description, start-location line, profession name and profession description stayed English. **Fix:** `build-mo.js` now emits `msgctxt` entries for those nine lookups. Cross-check on the vanilla catalogue: `lang/mo/zh_CN/LC_MESSAGES/cataclysm-dda.mo` has 87,107 entries, 5,750 of them contextual, and `scenario_male\004Evacuee` resolves to a Chinese name there. Mod names are plain (`Magiclysm` → 大魔法), which is why both forms have to exist in the same `.mo`.
* **Text style check**: CDDA enforces *two spaces after a sentence-ending period* in English strings. Violations show up as `text_style_check_reader.cpp ... insufficient spaces at this location. 2 required, but only 1 found.` and make `--check-mods` exit non-zero. Chinese text is exempt.
* **`u_location_variable` returns the first match, not the nearest one.** The search walks a bounding box (`map::points_in_radius` builds a `tripoint_range(min, max)`, iterated row-major) and breaks on the first hit (`src/npctalk.cpp`, the `search_target` branch). With `target_max_radius: 40` on a 48×48 floor, the player is therefore always sent to the north-west-most bed — possibly in a neighbouring flat or building. **Fix in use:** grow the radius in stages.
* **`target_min_radius` is a skip filter, not a starting distance**: `if( rl_dist( ... ) <= min_target_dist ) continue;`. A value of 1 excludes the tile the player stands on *and* everything adjacent, so "find nearby furniture" with min 1 misses the bedside nightstand.
* **`passable_only` does not apply to furniture or terrain searches** — it only affects the random-offset branch of the location search.
* **`u_spawn_item` works.** It reaches `talker_character::i_add_or_drop( item &, bool force_equip )` (`src/talker_character.cpp`), which wears the item if it is wearable, else wields it when there is no hand conflict, and otherwise falls through to `Character::i_add_or_drop( item & )` with the default count of 1 — i.e. the item simply goes into the inventory. `force_equip: true` means "wear or wield it", **not** "spawn one". (An earlier version of this file wrongly claimed the flag was passed into the item count and that nothing spawned; that was a misreading of the `Character::` overload and has been corrected.)
* **`map_spawn_item` without `loc`** places the item on the talker's own tile — useful when the exact container is unknown.
* `u_run_monster_eocs` evaluates its EOC with the **monster** as the talker, so the banish spell is cast by each monster in turn (and the `{"not": "u_is_outside"}` condition therefore tests the monster, not the player).  `BEGONE_SHADOW` is vanilla, so this mod inherits its limits.
* **A third-party mod's own `.mo` files are only loaded from the user mod directory.** `TranslationManager::Impl::ScanTranslationDocuments()` scans exactly two places: `PATH_INFO::user_moddir()` (= `<gamedir>/mods/` in this portable install) for any `*.mo`, and `lang/mo/` for `cataclysm-dda.mo`. A mod installed under `data/mods/` therefore runs fine but **its bundled translations are silently ignored** — which is why the Chinese translation only works from `mods/`. The running game states this in `config/debug.log`: `[i18n] Scanning mod translations from ./mods/`.
* **`loc` does not accept a `mutator` variable object.** `map_spawn_item` reads its `loc` through `read_var_info`, which accepts **only** `u_val`, `npc_val`, `global_val`, `var_val` and `context_val` (`src/condition.cpp`, `read_var_info`); anything else — including `mutator`/`target`, and also `const_val` / `math` / `distance` — throws `"Invalid variable type."`. Note that `doc/JSON/EFFECT_ON_CONDITION.md` shows the unsupported form in its `map_spawn_item` example. In this build the failure is worse than a JSON error: writing it made every `--check-mods` run **crash with SIGSEGV / fail-fast inside `JsonObject::error_skipped_members`** (`crash.log`), i.e. the checker died while reporting the error, leaving `debug.log` empty and taking an afternoon to trace. Use a searched location variable (`u_location_variable` + `loc: { "u_val": … }`) or `u_spawn_item` instead.
* **Riot damage moved out of the `PP_GENERATE_RIOT_DAMAGE` flag in current master.** The apartment terrains there get riot damage from a `post_process_generators: [ "riot_damage" ]` entry (`data/json/post_process_generators.json`, `data/json/overmap/overmap_terrain/overmap_terrain_residential.json`), while the flag itself still exists in the engine (`src/omdata.h`, `src/overmap_terrain.cpp`). That is why deleting only the flag does nothing on experimental builds — so `riot_patch.json` deletes **both** keys in one `delete` object, and whichever key the running version does not know is ignored (`allow_omitted_members()` is called before a `delete` object is read). One file therefore serves both versions.
* **Why the mod ships no separate "experimental" patch file any more.** 1.0.7 briefly did (`home_start_riot_patch`), and it was **harmful**, not merely redundant. A mod that declares a dependency on `home_start` is loaded **after** it, and redefining the same twelve ids with `copy-from: apartments_tower_any` makes `generic_factory::load` build a **fresh** object from that abstract base (`handle_inheritance` resolves `copy-from` against `map`/`abstracts`) and then `insert()` it, **replacing** the definition `home_start` had patched. On 0.I the abstract base still carries `PP_GENERATE_RIOT_DAMAGE`, so enabling that companion would have brought the riot damage **back**, silently, with no error at all. The rule that follows: **change an existing object in the mod that owns it, or point `copy-from` at the patched object rather than at the abstract base.** Note also that `--check-mods` exiting 0 proves only that nothing errors -- **not** that nothing changed.
* **How mods are found and installed** (`src/mod_manager.cpp`): `mods/` is created automatically if missing (`assure_dir_exist( PATH_INFO::user_moddir() )` at startup), so a freshly unpacked game needs no manual setup; `modinfo.json` is searched **recursively** (`get_files_from_path( MOD_SEARCH_FILE, path, true )` in `load_mods_from`), which is why a nested "Download ZIP" folder such as `home_start-main/home_start/` still loads; and a **second** folder carrying the same `id` triggers `debugmsg( "there is already a mod with ident %s" )` — the old folder must be deleted when updating.
* **The user mod directory is platform dependent** (`PATH_INFO::init_user_dir`): Windows/Linux portable ZIP → `<gamedir>/mods/`; macOS → `~/Library/Application Support/Cataclysm/mods/`; Linux XDG → `$XDG_DATA_HOME/cataclysm-dda/mods/` or `~/.local/share/cataclysm-dda/mods/` (legacy builds: `~/.cataclysm-dda/mods/`).
* General CDDA notes that are **not** used by this mod (kept here so they are not confused with the above): `upgrades.half_life` is multiplied by the world option `EVOLUTION_INVERSE_MULTIPLIER` (larger = slower evolution); `MONSTER_WHITELIST` with `mode: EXCLUSIVE` only takes effect when at least one whitelist entry is non-empty.

## 6. How to verify a checkout

The mod is pure JSON, so verification is mechanical. Run these **from the game directory**, with the game **not running** (the checker starts a game process):

```sh
cataclysm-tiles.exe --check-mods home_start            # exit code 0 = clean
json_formatter.exe mods/home_start/modinfo.json       # run for every .json file; a diff means reformat
```

The first run after editing mod files often prints `Stale game data detected` (flexbuffer cache) — run it a second time.

Real errors appear in `config/debug.log` as `Json error: file …, at line X, character Y:` and the **reason is on the following line**.

Translation coverage: every user-facing English string must exist as a `msgid` in `home_start/lang/mo/zh_CN/LC_MESSAGES/home_start.mo`. The `.po`/`.pot` sources in `home_start/lang/po/` are generated from the JSON by the project's `build-mo.js` helper, which **merges identical strings into a single entry** — `Home Start` is both the mod name and the scenario name, and gettext tooling refuses to compile a catalogue that repeats a `msgid` (`duplicate message definition`); a merged entry lists every source file on one `#:` line and every origin after `#.`.

The shipped `.mo` is reproducible: compiling `zh_CN.po` with the gettext compiler that ships with Python (`Tools/i18n/msgfmt.py`) produces a **byte-identical** file — sha256 `08674631de03993baec33372c703b7d0b80c386d9dc42fa9fc5cef4961a6b9a7`, 4537 bytes, 14 entries (the header plus 13 strings, 9 of them contextual). No GNU `msgfmt` was available on the development machine, so that compiler check was run with the Python tool; note that GNU `msgfmt` writes a hash table by default, so its output can differ byte-wise while staying functionally identical.

**Check translations the way the engine looks them up.** A coverage test that only searches for the bare `msgid` passes even when the game shows English, because scenario and profession strings are queried as `msgctxt + "\004" + msgid` (§5). Every user-facing string in this mod is therefore verified by asking the `.mo` for exactly the key the engine uses: `md5`-level coverage is not enough.

## 7. Known limitations / maintenance risks

* References vanilla ids that could be renamed upstream: `apartments_tower_any`, `PP_GENERATE_RIOT_DAMAGE`, `BEGONE_SHADOW` (the spell used to clear monsters), `f_bed`, `f_bathtub`, and the apartment terrain ids in `start_locations.json`.
* The riot-damage patch only covers `apartments_con_tower_*`. Other apartment mapgens (`mod_tower`, `s_apt`, the apartment-complex set) still generate riot damage.
* `start_locations.json` currently lists six upper-floor terrains (`..._002/102/012/112/013/113`). The half-floors `..._011`/`..._111` look like the same plan and could probably be added, but that has not been verified in game — check the floor you spawn on before adding them.
* The whole mod only affects **new characters and newly generated maps**.
* Incompatible with total conversions that remove cities (`innawood`, `The Backrooms`, `Sky Island`, `Defense Mode`).
* **Version support** (details in §1): 0.I. 0.H and older fail to load the scenario (`u_run_monster_eocs` is missing). Experimental builds run the whole mod, riot patch included, but that path has not been verified in game.
* **Install traps** (details in §5): the mod must go into the *user* mod directory, never `data/mods/` (translations would be ignored); the folder name may not matter but the mod `id` may not exist twice — delete the old folder when updating; a nested "Download ZIP" folder is fine because `modinfo.json` is found recursively.
* Mods that patch the same overmap terrain entries (`apartments_con_tower_*`), such as **Alternative Map Key**, may override this mod's riot patch or be overridden by it depending on load order; the visible effect is cosmetic (overmap icon/colour) and neither mod errors.
* Not accepted into the official CDDA repository as-is: `doc/IN_REPO_MODS.md` requires a long-term curator and forbids mods whose only purpose is to switch off a working vanilla feature; the riot patch would have to be argued as part of the "clean flat" concept.

## 8. Publishing (for whoever maintains it)

```sh
# first time on a machine: make gh the git credential helper
gh auth setup-git

git add -A && git commit -m "…" && git push
gh release create v<version> <zip> --title "Home Start <version>" --notes-file <notes.md>
```

If `git` cannot reach `github.com:443` (it is intermittently blocked on the development machine's network, while `gh` keeps working because it talks to `api.github.com`), publishing still works through the Git Data API: POST blobs → tree (with `base_tree`) → commit → PATCH the `main` ref, then `gh release create`. **Tag every release with the exact commit that was published, and never move an existing tag onto a different commit** — a tag that was re-pointed after its asset had been replaced is exactly how v1.0.3 briefly shipped a tag and a ZIP that disagreed.

The release ZIP contains a single top-level `home_start/` folder, meant to be extracted into the game's **user mod directory** (see §5 for why `data/mods/` breaks translations).

**Build the ZIP with a tool that writes `/` separators.** This project uses bsdtar: `tar -a -c -f home_start-<version>.zip -C <staging> home_start`. PowerShell's `Compress-Archive` writes `home_start\lang\…` with **backslashes**, which Windows extracts happily but macOS/Linux `unzip` turns into one literal filename — every release up to 1.0.5 had this bug. Verify after packaging that no entry name contains a backslash (e.g. list `ZipFile.OpenRead( zip ).Entries | % FullName`). Players who use **Code → Download ZIP** instead get `home_start-main/home_start/…`, which also loads because `modinfo.json` is found recursively (§5).

## 9. 中文速览

* 这是什么：CDDA 的数据 mod，让你在自己家（市中心公寓楼高层）里开局；只有 JSON，没有新增物品/怪物/贴图。
* 文件在哪：仓库根目录放文档，**`home_start/` 子目录就是 mod 本体**，复制进 `<游戏>/mods/`（**不是** `data/mods/`，否则自带的汉化不会加载，见第 5 节）；`lang/po/` 是翻译源文件，`lang/mo/` 是游戏读取的成品。
* 开局脚本做什么：弹一句提示 → 按 6→12→24 格**分阶段**找床（一次性大半径会永远传送到该层最西北角那张床）→ 把撬棍放在床边 1~2 格的家具上（绝不放在床上；床边没家具时直接交到你手上（撬棍塞不进口袋）） → 对 60 格内**室内**的怪 + 20 格内所有怪施放 `BEGONE_SHADOW` 清场；**不改饥饿口渴**。
* 验证方式：`cataclysm-tiles.exe --check-mods home_start`（退出码 0）、`json_formatter.exe`（官方格式）、中文语言包覆盖率。
* 踩过的坑见第 5 节（英文句末必须两个空格、`u_location_variable` 取的是"第一个"不是"最近的"、`target_min_radius` 是"跳过"语义、`passable_only` 对家具搜索无效、`u_spawn_item` 其实是好的）。
