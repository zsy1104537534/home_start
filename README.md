# Home Start

A small content mod for **Cataclysm: Dark Days Ahead** that starts your survivor locked inside their own flat on an upper floor of a downtown apartment tower.

* **Version:** 1.0.8
* **Game:** CDDA **0.I** stable (developed and tested on `0.I-1`, build `2026-09-19-2324`, commit `7b2efa5`)
  * **0.H and older are not supported.** The start script uses the `u_run_monster_eocs` effect, which does not exist there (`src/npctalk.cpp` — absent in `0.H`, present in `0.I`), so the scenario fails to load.
  * **On experimental builds** the mod loads and plays normally, and riot damage is handled there too: the patch deletes the new-style `post_process_generators` entry as well as the old flag, in one file — see the note below the installation steps. That experimental path has **not been verified in game yet** (the author develops on 0.I-1).
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

1. Get the mod — both routes work:
   * **Release ZIP:** it contains a single top-level `home_start/` folder. Drop that folder into the game's **user mod directory** (step 2).
   * **Code → Download ZIP** on this repository: you get `home_start-main/`, which contains the mod *and* the documentation. The game searches for `modinfo.json` **recursively**, so dropping the whole `home_start-main` folder in also loads correctly (Chinese translation included). Putting just the inner `home_start/` folder in is tidier.
2. The **user mod directory** is not always the folder next to the executable:

   | Install | Where the mod folder goes |
   |---|---|
   | Windows / Linux official ZIP (portable) | `mods/` next to `cataclysm-tiles.exe` |
   | macOS | `~/Library/Application Support/Cataclysm/mods/` |
   | Linux, package/XDG install | `~/.local/share/cataclysm-dda/mods/` (or `$XDG_DATA_HOME/cataclysm-dda/mods/`); very old builds used `~/.cataclysm-dda/mods/` |

   A freshly unpacked game has **no** `mods/` folder: start the game once and it creates one, or create it yourself.
3. **Never put it in `data/mods/`.** CDDA only loads a third-party mod's own translation files from the user mod directory; a mod in `data/mods/` still runs, but its bundled translation (the Chinese one here) is silently ignored.
4. **When updating, delete the old `home_start` folder first.** Two folders with the same mod id make the game report `there is already a mod with ident home_start`.
5. Create a **new world** and enable **Home Start** in the mod list.
6. Pick the *Home Start* scenario — only the *Homebody Survivor* profession is offered. A custom character works fine; **do not use an old character preset**, since presets carry their own saved inventory.

### One file covers both 0.I and experimental builds

Riot damage was moved upstream from the `PP_GENERATE_RIOT_DAMAGE` overmap flag to a `post_process_generators: [ "riot_damage" ]` entry, so `riot_patch.json` now deletes **both** keys in a single `delete` object: 0.I reads the flag one and ignores the other, experimental builds read the generator one and ignore the flag. A key a given version does not know is silently ignored — CDDA calls `allow_omitted_members()` while reading a `delete` object — so there is nothing extra to install or enable.

Honest caveat: the 0.I side is what the author plays and tests; **the experimental side has not been verified in game** (no experimental install on the development machine). If riot damage still appears on an experimental build, that is a bug worth reporting.
Notes: **new characters**; existing saves keep whatever they were created with.
* Using an existing world is not recommended. If you do add this mod to one, only newly generated map areas get the patch.

## Language

Source text is English. A Simplified Chinese translation ships in `lang/mo/zh_CN/LC_MESSAGES/home_start.mo` and is loaded automatically when the game language is Chinese. The gettext sources (`lang/po/home_start.pot` and `lang/po/zh_CN.po`) are included, so a translation can be edited without touching the compiled `.mo`. Both files contain every `msgid` exactly once — gettext tooling refuses to compile a catalogue that repeats one (`duplicate message definition`) — so strings that appear twice in the mod (the mod name is also the scenario name) share a single entry whose `#:` line lists both sources.

To add another language: copy `lang/po/home_start.pot` to `lang/po/<lang>.po`, fill in the `msgstr` lines, then compile it:

```sh
msgfmt -o home_start/lang/mo/<lang>/LC_MESSAGES/home_start.mo home_start/lang/po/<lang>.po
```

## Compatibility

* **Works alongside:** monster-filtering mods (`classic_zombies`, `Only_Wildlife`), magic/psionic content mods (`Magiclysm`, `Mind Over Matter`, `Xedra Evolved`), difficulty mods (`No Hope`, `Deadly Bites`), QoL mods (`Bombastic Perks`, `Rummaging`, `Tamable Wildlife`).
* **Not compatible with total conversions that remove cities**, such as `innawood`, `The Backrooms`, `Sky Island` or `Defense Mode` — the start location is a downtown apartment tower, which those mods do not generate.
* **Not compatible with CDDA 0.H or older** (see the version notes at the top); on **experimental** builds everything works except the riot-damage patch.
* Mods that rework apartment tower mapgen, or that patch the same overmap terrain entries (`apartments_con_tower_*`) — e.g. **Alternative Map Key** — may override this mod's patch or be overridden by it, depending on load order. The practical effect is cosmetic (the overmap icon/colour of the apartment tower); neither mod errors.

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

**安装**：两种下载方式都行——**Release 的 ZIP** 里是顶层 `home_start/` 文件夹，直接丢进**用户 mod 目录**；在仓库点 **Code → Download ZIP** 得到的是 `home_start-main/`（里面才是 `home_start/`，还夹着文档），因为游戏是**递归**查找 `modinfo.json` 的，整个 `home_start-main` 丢进去也能正常加载（中文照样生效），想干净就只放里面那个 `home_start/`。

**用户 mod 目录**不一定和 exe 同级：Windows / Linux 官方压缩包（便携版）是 exe 旁边的 `mods\`；**macOS** 是 `~/Library/Application Support/Cataclysm/mods/`；**Linux 包管理器安装**是 `~/.local/share/cataclysm-dda/mods/`（或 `$XDG_DATA_HOME/cataclysm-dda/mods/`，很老的版本用 `~/.cataclysm-dda/mods/`）。**刚解压的游戏没有 `mods\` 这个文件夹**，先启动一次游戏它就会自己建，也可以自己新建。**绝对不要放 `data\mods\`** —— 游戏只从用户 mod 目录读第三方 mod 自带的翻译，放 `data\mods\` 虽然能玩，但**中文不会生效**。**升级前先删掉旧的 `home_start` 文件夹**，两个同 id 的文件夹会让游戏报 `there is already a mod with ident home_start`。

**暴乱破坏：一份文件同时兼容 0.I 和实验版** —— 上游把暴乱破坏从 `PP_GENERATE_RIOT_DAMAGE` 这个 flag 改成了 `post_process_generators: ["riot_damage"]`，所以 `riot_patch.json` 在同一个 `delete` 里**两个键都删**：0.I 认 flag 那条、忽略另一条，实验版认生成器那条、忽略 flag 那条；**当前版本不认识的那个键会被静默忽略**（引擎读 `delete` 时调用了 `allow_omitted_members()`），所以不需要额外装或勾任何东西。老实说一句：0.I 这条是作者本机实测的，**实验版那条还没在游戏里实测过**，如果实验版上暴乱破坏依旧那就是 bug，欢迎回报。
**版本要求**：需要 **0.I** 稳定版（`0.I` / `0.I-1`，本 mod 在 `2026-09-19-2324` 上开发验证）；**0.H 及更早不支持**（开局脚本用的 `u_run_monster_eocs` 那些版本里没有，场景会加载失败）；**实验版**能正常游玩，暴乱破坏也由同一份文件处理（把新式的 `post_process_generators` 和旧的 flag 一起删，见安装步骤后的说明）；**实验版这条路还没在游戏里实测过**（作者本机是 0.I-1）。

**兼容性**：可以和 `classic_zombies`、`Only_Wildlife`、`Magiclysm` 等共存；**不能**和删掉城市的完全转换类（`innawood`、`The Backrooms`、`Sky Island`、`Defense Mode`）一起用；和同样修改这些大地图地形（`apartments_con_tower_*`）的 mod（例如 **Alternative Map Key**）一起用时，谁覆盖谁取决于加载顺序，**表现只是大地图图标/颜色**，不会报错。

**注意**：所有改动只对**新角色/新地图**生效；约 7% 的公寓门是带警报的锁门（`t_door_locked_alarm`），撬开会响警报——这是故意保留的。
