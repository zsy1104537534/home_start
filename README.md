# Home Start

A small content mod for **Cataclysm: Dark Days Ahead** that starts your survivor locked inside their own flat on an upper floor of a downtown apartment tower.

* **Version:** 1.0.3
* **Game:** CDDA 0.I and newer experimental builds (developed and tested on build `2026-09-19-2324`, commit `7b2efa5`)
* **Dependencies:** `dda` only
* **License:** CC-BY-SA 3.0 (same as the game's content license)

---

## What it adds

| Piece | Description |
|---|---|
| Scenario **Home Start** | Starts inside an upstairs flat of a downtown apartment tower, on foot, alone, five days into the Cataclysm (default season length: spring, day 61, 08:00 — the engine derives the dates from the `SEASON_LENGTH` world option, so they follow your settings). |
| Start location **Apartment Tower (upstairs flat)** | Restricts the start to upper-floor apartment interiors, so you never spawn on a burnt-out ground floor. |
| Profession **Homebody Survivor** | A civilian in a fitting set of clothes (`dress_shirt`, `jeans`, `socks`, `sneakers`), with no bonus items. |
| Start script | Wakes you up on a bed in the flat (searching nearby first, then further out), leaves a **crowbar** on a piece of furniture 1-2 tiles away — never on the bed you wake on — so a locked door is never a dead end, and clears monsters from your own floor. |
| Riot damage patch | Apartment tower interiors (`apartments_con_tower_*`) no longer generate with `PP_GENERATE_RIOT_DAMAGE` — intact windows, furniture not smashed. |

**Design intent:** a quiet, self-contained opening. Your floor is clean and lootable, the streets outside are not.

## Installation

1. Download the release ZIP (or clone this repository — the mod itself is the `home_start/` folder inside it) and put the folder named `home_start` into your game's **`mods/`** directory — the one next to `cataclysm-tiles.exe`, **not** `data/mods/`. CDDA only loads a third-party mod's own translation files from `mods/`; a mod in `data/mods/` still runs, but its bundled translation (the Chinese one here) is never loaded.
2. Create a **new world** and enable **Home Start** in the mod list.
3. Pick the *Home Start* scenario — only the *Homebody Survivor* profession is offered. A custom character works fine; **do not use an old character preset**, since presets carry their own saved inventory.

Notes:

* Mods and world settings only affect **new characters**; existing saves keep whatever they were created with.
* Using an existing world is not recommended. If you do add this mod to one, only newly generated map areas get the patch.

## Language

Source text is English. A Simplified Chinese translation ships in `lang/mo/zh_CN/LC_MESSAGES/home_start.mo` and is loaded automatically when the game language is Chinese. The gettext sources (`lang/po/home_start.pot` and `lang/po/zh_CN.po`) are included, so a translation can be edited without touching the compiled `.mo`.

To add another language: copy `lang/po/home_start.pot` to `lang/po/<lang>.po`, fill in the `msgstr` lines, then compile it:

```sh
msgfmt -o home_start/lang/mo/<lang>/LC_MESSAGES/home_start.mo home_start/lang/po/<lang>.po
```

## Compatibility

* **Works alongside:** monster-filtering mods (`classic_zombies`, `Only_Wildlife`), magic/psionic content mods (`Magiclysm`, `Mind Over Matter`, `Xedra Evolved`), difficulty mods (`No Hope`, `Deadly Bites`), QoL mods (`Bombastic Perks`, `Rummaging`, `Tamable Wildlife`).
* **Not compatible with total conversions that remove cities**, such as `innawood`, `The Backrooms`, `Sky Island` or `Defense Mode` — the start location is a downtown apartment tower, which those mods do not generate.
* Mods that rework apartment tower mapgen may conflict with the riot damage patch.

## Known limitations / maintenance notes

* The mod references vanilla ids: `apartments_tower_any`, `PP_GENERATE_RIOT_DAMAGE`, `BEGONE_SHADOW`, `f_bed`, `f_bathtub`, `sloc_*` apartment terrains. If upstream renames or removes any of them, the corresponding piece stops working — re-run the check below after game updates.
* The riot damage patch covers `apartments_con_tower_*` only; other apartment mapgens (`mod_tower`, `s_apt`, the apartment-complex set) still generate riot damage.
* The patch only affects **newly generated** apartment towers, and the mod as a whole only affects **new characters**.
* About 7 % of apartment doors generate as `t_door_locked_alarm`; prying one open with the crowbar sets off an alarm. Left in on purpose.
* The **bed** search teleports you onto a bed, not necessarily the nearest one: the engine's location search returns the first match in scan order, so the script deliberately searches 6 → 12 → 24 tiles instead of using one large radius.
* Monster clearing uses the vanilla `BEGONE_SHADOW` spell, so it respects that spell's own limits.

## Checking a checkout

```sh
# from the game directory (the game must not be running)
cataclysm-tiles.exe --check-mods home_start                    # exit code 0 = clean
json_formatter.exe mods/home_start/modinfo.json               # run for every .json file in home_start/
```

Every JSON file must be style-clean, and every user-facing English string must have a `msgid` in the Chinese `.mo`.

---

# 家中开局（中文说明）

一个给 **《大灾变：黑暗之日》** 用的小型内容 mod：让你在自己位于市中心公寓楼**高层**的家里开局。

| 内容 | 说明 |
|---|---|
| 场景「家中开局」 | 在公寓楼楼上的住家里醒来，独自一人。默认季节长度（91 天）下是春季第 61 天 08:00，即大灾变后第 5 天；日期由引擎按你的「赛季长度」设置自动推算。 |
| 开局地点「公寓楼（楼上住家）」 | 限定只在高层的公寓室内开局，不会落在被烧毁的一楼。 |
| 职业「居家幸存者」 | 普通市民，开局穿着**合身的一套衣服**（衬衫/牛仔裤/袜子/运动鞋），没有额外奖励物品。 |
| 开局脚本 | 在床上醒来（床是**分阶段**就近找的：先 6 格、再 12 格、再 24 格）；**撬棍放在床边 1~2 格内的家具上**（绝不放在你躺的那张床上；床边没有家具时直接交到你手上——撬棍塞不进口袋），免得被锁在门里；清空你所在**楼层**的怪物。 |
| 暴乱破坏补丁 | 公寓塔楼室内（`apartments_con_tower_*`）不再生成暴乱破坏（窗户完整、家具没被砸）。 |

**安装**：把 `home_start` 文件夹放进游戏目录的 **`mods\`**（和 `cataclysm-tiles.exe` 同级那个），**不要**放 `data\mods\` —— 游戏只从 `mods\` 读取第三方 mod 自带的翻译，放 `data\mods\` 虽然能玩，但**中文不会生效**。然后**开新世界**时勾选它；职业只有「居家幸存者」，用「自定义角色」创建即可（**不要用旧的角色预设**，预设会带自己的存档物品）。

**兼容性**：可以和 `classic_zombies`、`Only_Wildlife`、`Magiclysm` 等共存；**不能**和删掉城市的完全转换类（`innawood`、`The Backrooms`、`Sky Island`、`Defense Mode`）一起用。

**注意**：所有改动只对**新角色/新地图**生效；约 7% 的公寓门是带警报的锁门（`t_door_locked_alarm`），撬开会响警报——这是故意保留的。
