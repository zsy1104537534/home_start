# Home Start

A small content mod for **Cataclysm: Dark Days Ahead** that starts your survivor locked inside their own flat on an upper floor of a downtown apartment tower.

* **Version:** 1.0.0
* **Game:** CDDA 0.I and newer experimental builds (developed and tested on build `2026-09-19-2324`, commit `7b2efa5`)
* **Dependencies:** `dda` only
* **License:** CC-BY-SA 3.0 (same as the game's content license)

---

## What it adds

| Piece | Description |
|---|---|
| Scenario **Home Start** | Starts inside an upstairs flat of a downtown apartment tower, on foot, alone, five days into the Cataclysm (spring, day 61, 08:00). |
| Start location **Apartment Tower (upstairs flat)** | Restricts the start to upper-floor apartment interiors, so you never spawn on a burnt-out ground floor. |
| Profession **Homebody Survivor** | A civilian in a fitting set of clothes (`dress_shirt`, `jeans`, `socks`, `sneakers`), with no bonus items. |
| Start script | Wakes you up next to the bed, leaves a **sewing kit** and a **crowbar** somewhere in the flat, and clears monsters from your own floor. |
| Riot damage patch | Apartment tower interiors no longer generate with `PP_GENERATE_RIOT_DAMAGE` (intact windows, furniture not smashed). |

**Design intent:** a quiet, self-contained opening. Your floor is clean and lootable, the streets outside are not, and a crowbar in the flat means a locked door is never a dead end.

## Installation

1. Download the repository (or a release ZIP) and put the folder named `home_start` into your game's `data/mods/` directory.
2. Create a **new world** and enable **Home Start** in the mod list.
3. Pick the *Home Start* scenario — only the *Homebody Survivor* profession is offered. A custom character works fine; **do not use an old character preset**, since presets carry their own saved inventory.

Notes:

* Mods and world settings only affect **new characters**; existing saves keep whatever they were created with.
* Using an existing world is not recommended. If you do add this mod to one, only newly generated map areas get the patch.

## Language

Source text is English. A Simplified Chinese translation ships in `lang/mo/zh_CN/LC_MESSAGES/home_start.mo` and is loaded automatically when the game language is Chinese.

To add another language: create `lang/po/<lang>.po` from `lang/po/home_start.pot`, then compile it with `msgfmt -o lang/mo/<lang>/LC_MESSAGES/home_start.mo lang/po/<lang>.po`.

## Compatibility

* **Works alongside:** monster-filtering mods (`classic_zombies`, `Only_Wildlife`), magic/psionic content mods (`Magiclysm`, `Mind Over Matter`, `Xedra Evolved`), difficulty mods (`No Hope`, `Deadly Bites`), QoL mods (`Bombastic Perks`, `Rummaging`, `Tamable Wildlife`).
* **Not compatible with total conversions that remove cities**, such as `innawood`, `The Backrooms`, `Sky Island` or `Defense Mode` — the start location is a downtown apartment tower, which those mods do not generate.
* Mods that rework apartment tower mapgen may conflict with the riot damage patch.

## Known limitations / maintenance notes

* The mod references vanilla ids: `apartments_tower_any`, `PP_GENERATE_RIOT_DAMAGE`, `BEGONE_SHADOW`, `f_bed`, `f_bathtub`, `sloc_*` apartment terrains. If upstream renames or removes any of them, the corresponding piece stops working — re-run the check below after game updates.
* The riot damage patch only affects **newly generated** apartment towers.
* The start script uses `map_spawn_item` rather than `u_spawn_item`: in 0.I the latter adds nothing unless `force_equip` is set, because the engine passes that flag into the item count argument.
* Monster clearing uses the vanilla `BEGONE_SHADOW` spell, so it also respects that spell's own limits.

## Checking a checkout

```sh
# from the game directory (game must not be running)
cataclysm-tiles.exe --check-mods home_start     # exit code 0 = clean
json_formatter.exe modinfo.json                     # every JSON file must be style-clean
```

---

# 家中开局（中文说明）

一个给 **《大灾变：黑暗之日》** 用的小型内容 mod：让你在自己位于市中心公寓楼**高层**的家里开局。

| 内容 | 说明 |
|---|---|
| 场景「家中开局」 | 在大灾变第 5 天（春季第 61 天 08:00）于公寓楼楼上的住家里醒来，独自一人。 |
| 开局地点「公寓楼（楼上住家）」 | 限定只在高层的公寓室内开局，不会落在被烧毁的一楼。 |
| 职业「居家幸存者」 | 普通市民，开局穿着**合身的一套衣服**（衬衫/牛仔裤/袜子/运动鞋），没有额外奖励物品。 |
| 开局脚本 | 在床边醒来，屋里留一个**针线盒**和一根**撬棍**，并清空你所在楼层的怪物。 |
| 暴乱破坏补丁 | 公寓楼室内不再生成暴乱破坏（窗户完整、家具没被砸）。 |

**安装**：把 `home_start` 文件夹放进游戏目录的 `data/mods/`，然后**开新世界**时勾选它；职业只有「居家幸存者」，用「自定义角色」创建即可（**不要用旧的角色预设**，预设会带自己的存档物品）。

**兼容性**：可以和 `classic_zombies`、`Only_Wildlife`、`Magiclysm` 等共存；**不能**和删掉城市的完全转换类（`innawood`、`The Backrooms`、`Sky Island`、`Defense Mode`）一起用。

**注意**：所有改动只对**新角色/新地图**生效。
