---
title: 翎风引擎（LF 引擎）事实与脚本规范
id: lf-engine
type: skill
name: lf-engine
description: 翎风引擎（LF 引擎）服务端的事实与脚本开发规范。要写或改传奇 LF 服务端脚本、查脚本命令/触发器/变量、查物品怪物技能数据库字段、确认某个文件放哪、`翎风引擎帮助文档.CHM` 里的内容怎么用时读它；给出「先查说明书镜像与真实服务端文件、禁止凭记忆编造引擎事实」的取证流程与本端文件地图。
tags: [翎风引擎, LF, 传奇, Mir2, 服务端脚本, 数据库字段]
---

# 翎风引擎（LF 引擎）

本工作区维护的是一套**翎风引擎**（LF 引擎，传奇 Mir2 商业端）服务端，部署在腾讯云 Windows 服务器上。

本技能规定**怎么查、往哪查、什么绝对不能编**。它不复制说明书正文 —— 正文在
`data/` 分片里，按需读（入口 `data/00-导航.md`）。


## 0. 在 nuphus 里怎么用这个技能

本技能以 **nuphus 技能包**形式安装（`community/lf-engine/`），工具是 nuphus 自带的那两个：

- `skill_read({ skill: "lf-engine" })` —— 读回本文全文（就是你正在看的这份）。
- `skill_query({ query: "..." })` —— 在**本文 + `data/` 下的索引**里按关键词检索。
  ⚠️ **它返回的是命中文件的全文，不截断**。所以问得越具体越好：
  「CHECKBAGSIZE 的语法」可以，「脚本」「命令」这种宽词会把整份索引灌进上下文。
- `data/` **之外**的文件不参与 `skill_query`（nuphus 只扫 `data/`）：
  `manual/` 里的 850 页说明书原文要用 shell 直接搜（见 §0.2）。

## 0.1 本包在这台机器上的实际路径

技能根：`C:\Users\9900k\AppData\Local\Nuphus\plugin\skills\community\lf-engine\`

| 本文里的说法 | 这台机器上的真实位置 |
|---|---|
| `data/…` 各分片索引 | 技能根下的 `data\` |
| `manual/` 说明书原文镜像 | 技能根下的 `manual\`（850 个 `.txt`，**保留原 CHM 目录结构**） |
| 「真实服务端文件」 | `C:\Users\9900k\Desktop\9.28新建\MirServer222\MirServer222\`（**本机拷贝，仅供参考，非权威**） |
| 「云服务器」 | 唯一的最终裁定者。⚠️ **这台机器上有没有到它的通道尚未验证** —— 用之前先自己测，别假设 |

## 0.2 查说明书原文（替代原来 Mac 上的抽取脚本）

Mac 工作区用 `lf-chm-extract.py` 解开 CHM 再 grep；这里镜像已经就位，直接递归搜：

```powershell
$m = 'C:\Users\9900k\AppData\Local\Nuphus\plugin\skills\community\lf-engine\manual'
Get-ChildItem "$m" -Recurse -Filter *.txt |
  Select-String -Pattern 'CHECKBAGSIZE' -SimpleMatch |
  Select-Object -First 20 | ForEach-Object { $_.Filename + ':' + $_.LineNumber + '  ' + $_.Line }
```

- **命中 → 去读命中那一页的原文**，照抄它给的语法与示例。
- **0 命中 → 该命令很可能在说明书里不存在**，不要按记忆补全；改去本机拷贝或云服务器找真实用例。
- 中文匹配串在 PowerShell 里可能因编码对不上，可改用不区分大小写的简短英文/数字串，
  或用 `-Encoding` 显式指定后再搜。

## 0.3 这台机器的「线上端体」要点（2026-09-27 换根后实测）

> 以下**不是说明书内容**，是采集结论（来源：Mac 工作区的端体画像-20260927；这台机器上没有该文档）。下面这份摘要是为了让规则可用；细节存疑时以云服务器实测为准。

| 项 | 值 |
|---|---|
| 服务端根 | `D:\MirServer222`，区名 `原始沉默01区` |
| 引擎 | `M2Server.exe` `FileVersion=4.0.0.0` / 2026-07-07；启动日志自报 `http://www.lfm2.com` |
| 脚本目录 | `Mir200\Envir`（1958 个 txt/ini；**GBK 与 UTF-8 混杂，写回前逐文件实测编码**） |
| 数据层 | `UseSqliteDB=1`：`Mud2\DB\ApexM2.DB` = StdItems/Monster/Magic；`DBServer\FDB\ApexMir.DB` = 角色/账号 |
| 端口 | M2 5000 / DBServer 5100,6000 / LoginSrv 5500,5600 / LoginGate 7000 / SelGate 7100 / RunGate 7200,27201 |
| 备份 | **没有在运行的有效备份**（`BackList.txt` 指向的目录不存在）→ 每次改动前必须自己做文件级备份 |
| 自启 | **没有**（`GetAutoStartServer=0`）→ 整服重启不会自动恢复 |

## 0.4 与 Mac 工作区的差异（别照抄那边的路径）

- Mac 工作区（`~/Projects/mir-gm`）里还有 `docs` 目录 采集结论、采集脚本 mir-inventory.sh 采集器、
  两份说明书 CHM 孤本、本机 Mir 拷贝 ——**这些在这台机器上都不存在**。
- 那边的 `references` 目录在这里改叫 `data/`（nuphus 只检 `data/`），**并按主题切成了小分片**
  （见 `data/00-导航.md`）。
- 遇到本文提到的、这台机器上没有的资源，正确回答是「本机没有」，不是编一个出来。

---

## 0. 边界

| 本技能管 | 本技能不管 |
|---|---|
| 脚本命令是否存在、语法是什么 | 启停服务、发物品、封号等**危险运维动作** → 走 `lf-gm-ops` |
| 触发器叫什么叫、参数是什么、放哪个文件 | 端口/进程/部署拓扑 → **说明书里没有**，只能实测并写进 `本文 §0.3` |
| 变量体系、作用域、是否落库 | 具体某次改动的批准流程 → 走 `lf-gm-ops` 的分级表 |
| 物品/怪物/技能库字段与代码表 | |
| 脚本文件放哪、谁引用谁 | |

---

## 1. 引擎身份（有硬证据，别再叫它 GOM）

| 证据文件 | 字段 | 值 |
|---|---|---|
| **线上** `Mir200\M2Server.exe` | PE 版本资源 / 文件日期 | `FileVersion=4.0.0.0`，**2026-07-07**（本机拷贝是 2024-06-13） |
| **线上引擎启动日志（自报）** | 原文 | `翎风引擎网站：http://www.lfm2.com` |
| 线上 `GameCenter.exe` / `DBServer.exe` / `LoginSrv.exe` | PE 版本资源 | `翎风游戏控制台` / `翎风引擎数据库服务器` / `翎风登录服务器`，公司 `www.haom2.com` |
| 本机拷贝 `GameCenter.exe` | PE 版本资源 | `FileDescription=翎风游戏控制台`、`CompanyName=www.haom2.com`、`FileVersion=2.0.0.8` |
| 本机拷贝 `DBServer.exe` | PE 版本资源 | `FileDescription=翎风引擎数据库服务器`、`FileVersion=2.0.0.54` |
| `翎风引擎帮助文档-2026-07-08.CHM`（Mac 工作区孤本，本机无原件） | 首页 | 「翎风引擎官方网站」`Http://www.LFM2.Com` |

线上端体的完整实测（进程/端口/数据层/编码/备份）见 `本文 §0.3`。

> 世界里的 GOM / HeroM2 / GameOfMir 资料**不是**本引擎的依据。
> 说明书里有独立的「兼容HeroM2」章节，那只说明它**兼容**部分 HeroM2 命令，
> 不等于本端行为等同 HeroM2。

---

## 2. 事实来源层级（写任何引擎事实之前，先确认站在哪一级）

| 级别 | 来源 | 效力 |
|---|---|---|
| ① 最高 | **云服务器上的实际文件** | 最终裁定。用户明确「一切以云服务器为准」 |
| ② 参考 | 本机 Mir 拷贝（路径见 §0.1） | 只是线索，**非权威**；路径/版本可能与线上不一致 |
| ③ 次之 | `翎风引擎帮助文档.CHM`（官方说明书；本包带文本镜像 `manual/`） | 一手官方文档，但通用、多年累积、有笔误 |
| ④ 最低 | 联网资料 | 必须附来源 URL，并标注「一手（官方/源码/发布物）」或「社区二手」 |

**唯一非法的来源是模型记忆。**

- 任何一条引擎事实都要能回答「这出自哪个文件/哪一页」。
- 查不到就写「**未验证**」，**不得**把推测写成结论。这优先于「给出完整答案」。
- 本机 Mir 拷贝里已经能看出它自己就矛盾（`!setup.txt` 同时指向
  `D:\MirServer暗黑\` 与 `D:\MirServer01\`），所以拿拷贝当权威一定会出事。

---

## 3. 查证三步法（本技能的核心用法）

写任何脚本命令/字段/触发器之前，**按顺序**走这三步：

### 第 1 步：查说明书原文镜像

```powershell
$m = 'C:\Users\9900k\AppData\Local\Nuphus\plugin\skills\community\lf-engine\manual'
Get-ChildItem "$m" -Recurse -Filter *.txt |
  Select-String -Pattern 'CHECKBAGSIZE' -SimpleMatch |
  Select-Object -First 20 | ForEach-Object { $_.Filename + ':' + $_.LineNumber + '  ' + $_.Line }
```

- **命中 → 去读命中那一页的原文**，照抄它给的语法与示例，不要"顺手规范化"成你熟悉的样子。
- **0 命中 → 该命令很可能在说明书里不存在**，不要按记忆补全。
  改去本机 Mir 拷贝或云服务器找真实用例；找不到就标「未验证」。
- 中文匹配串在 PowerShell 里可能因编码对不上，改用简短的英文/数字串更稳。

### 第 2 步：查 `data/` 里的提炼索引

索引已按主题整理好、每条都标了来源页面（见第 9 节清单），**比翻整本 850 页快得多**，
而且每份末尾都有「未验证/存疑」章节，会告诉你说明书哪里自相矛盾。

入口是 `data/00-导航.md`；用 `skill_query` 检索时**关键词要具体**（它返回命中文件全文）。

### 第 3 步：查本机 Mir 拷贝里的真实用例

```powershell
$envir = 'C:\Users\9900k\Desktop\9.28新建\MirServer222\MirServer222\Mir200\Envir'
Select-String -Path "$envir\Market_def\*.txt" -Pattern 'GAMEGOLD' |
  Select-Object -First 20 | ForEach-Object { $_.Filename + ':' + $_.LineNumber + '  ' + $_.Line }
```

本端脚本可能用了说明书没写的写法，也可能把说明书里的写法用歪了。
**两者冲突时，先记录冲突，再把云服务器作为最终裁定者。**

⚠️ 这份拷贝是**非权威**的（它自己的 `!setup.txt` 里就有互相矛盾的路径），拿它当结论一定会出事。

---

## 4. 脚本模型速览

细节与原文示例见 `data/01-脚本命令-*.md`（473 页命令页 + 基础语法页）。

### 4.1 文件骨架

```
(@@InPutString @@InPutInteger @storage ...)   ← 用到输入/特殊段时，必须写在文件第一行

[@段名]                                        ← 段用英文或数字表示
#IF
<检测命令...>
#ACT
<执行命令...>
#SAY
<对话文本，行尾用 \ 续行，按钮写作 <文字/@段名>>
#ELSEACT
...
#ELSESAY
...
```

- `#IF` 可多条件叠加（`#if(n)` 多条件、`NOT` 取反见 `data/01-脚本命令-*.md` §1.2）
- 段内可用 `{ }` 包块（本端 `QuestDiary/登录触发/*.txt` 大量使用）
- 循环优先用 `While 变量/值 比较符 变量/值 … EndWhile`；`GOTO` 递归**会栈溢出**，
  说明书原文明确警告 —— 要循环次数用 `Loopgoto @段 次数` + `endloop`
- 定时：`SETONTIMER 序号(0-9) 毫秒` → 触发 `[@OnTimerX]`；`SETOFFTIMER 序号` 关闭；
  一次性延跳用 `DELAYGOTO @段 毫秒`

### 4.2 路径与调用

- 脚本里的相对路径是**相对 `Mir200`** 的反斜杠路径，例：`..\QuestDiary\爵位捐献\捐献排名数据.txt`
- 跨脚本调用：`#CALL` / `#CALLEX`（`data/01-脚本命令-*.md` §1.5）
- 多级脚本（对别人执行命令）前缀：

  | 前缀 | 对象 |
  |---|---|
  | `H.` | 英雄 |
  | `O.` | 主人 |
  | `M.` | 怪物（当前攻击目标） |
  | `P.` | 对面的角色 |
  | `L.` | 当前攻击自己的角色 |
  | `HM.` / `HL.` | 英雄视角的 M./L. |
  | `PET.` | 宠物（单个宠物时） |
  | `BB.` | 宝宝（多个时随机一个；**仅支持执行命令，不支持检测命令**） |
  | `CO.` | 采集类怪物 |

### 4.3 变量体系（原文定义）

| 变量 | 类型 | 作用域 / 存续 |
|---|---|---|
| `P0-P999` | 数字 | 私人；**关闭对话框即归 0** |
| `D0-D999` | 数字 | 私人；下线清空（说明书原文：摇骰子变量） |
| `M0-M999` | 数字 | 私人；下线清空，**切换地图清空** |
| `N0-N999` | 数字 | 私人；下线不保存，小退归 0 |
| `S0-S999` | 字符 | 私人；下线不保存，小退归 0 |
| `I0-I999` | 数字 | **全局**；重启即重置为 0 |
| `G0-G999` / `A0-A999` | 数字 / 字符 | **全局**；可保存 → `Mir200/GlobalVal.ini` |
| `U0-U499` / `T0-T499` | 数字 / 字符 | 私人；**可保存 → 人物数据库** |
| `J0-J499` / `Z…` | 数字 / 字符 | 私人；可保存，**每晚 12 点自动重置** |

- 引用值用 `<$STR(N1)>`、`<$STR(S$名字)>`；直接读环境值用 `<$USERNAME>` `<$LEVEL>` `<$GAMEGOLD>` 之类（全表见 `data/01-脚本命令-*.md` §2 与 `data/06-其它资料-*.md`）
- **命名扩展**：`S$任意字符` / `N$任意字符`（如 `MOV N$技能防御 0`）—— 本端脚本大量使用
- **自定义变量三件套**：`VAR Integer/String HUMAN|GLOBAL 名字` 声明 → `LOADVAR` 读 → `SAVEVAR` 存；
  增减用 `CALCVAR`，比较用 `CHECKVAR`
- `N998` / `N999` 被引擎占用为鼠标地图坐标，**脚本里不许用**
- 自定义变量名**不要以 P、D、M、N、S、I、G、A 开头**
- 数值上限：仅 `G`、`N$`、`U`、`N` 支持到 `9223372036854775807`，其余数字变量 21 亿

### 4.4 分辨率陷阱

说明书里 **`S/N` 下标 0-999 与 0-499 两种说法并存**（不同页面不同），
`Z0-J499` 一行疑似笔误。**以云服务器实测为准**，别照抄下标范围当结论。

---

## 5. 触发器速览

全表（92 行，含文件位置/段名/参数/来源页）在 `data/02-触发功能-*.md`。
常见触发：

| 触发 | 标签 | 备注 |
|---|---|---|
| 登录/启动 | `[@Login]`、`[@Startup]` | 放 `Envir\MapQuest_def\QManage.txt`；`[@Startup]` 只在 M2 启动时加载一次 |
| 定时 | `#AutoRun NPC SEC\|MIN\|HOUR\|DAY\|RUNONWEEK …` + `[@段]` | 配 `Envir\Robot.txt` + `Robot_def\` |
| 升级 | `[@PlayLevelUp]` | |
| 杀怪 | `[@KillMon]`、`[@OnKillMob]`、`[@HeroKillMon]` | `[@OnKillMob]` 需地图参数带 `ONKILLMON` |
| 死亡 | `[@PlayDie]` | |
| 攻击/被击 | `[@Attack]`、`[@MagicAttack]`、`[@Struck]`、`[@MagicStruck]` | |
| 捡取/丢弃 | `[@PickUpItem]`、`[@PickUpItemEX]`、`[@DropItem]`、`[@PickUpItems<IDX>]` | `…EX` 不需物品规则；按 IDX 的需在"列表信息2"配触发 ID |
| 穿/脱 | `[@TakeOn0-12]`、`[@TakeOff0-12]`、`[@BeginTakeOff]` | 英雄 `@Hero…` |
| 小退/大退 | `[@PlayReconnection]`、`[@PlayOffLine]` | |
| 切换地图 | `[@EnterMap]`、`[@HeroEnterMap]` | |
| 查看装备 | `[@QueryUserState]`、`[@UserQueryState]` | |
| 货币改变 | `@GoldChange`、`@GameGoldChange`、`@GamePointChange` … | 原文自注「可能仅限脚本命令操作时触发」 |
| 套装 | `[@GroupItemOn<X>]`、`[@GroupItemOff<X>]`、`[@GroupItemOnEx]` | |
| 物品使用 | `[@StdModeFunc<X>]` | X = 物品库 `AniCount` |
| 技能自身/目标 | `[@MagSelfFunc<ID>]`、`[@MagTagFunc<ID>]`、`[@MagMonFunc<ID>]` | ID = 技能库 MagID |
| 地图事件 | `MapEvent.txt` → 事件类型 1 调用 `QFunction-0.txt` 段 | 需在 M2 里勾「启用地图事件触发」 |
| 外挂 | `[@UsePlugin]` | |
| 对话框按钮 | `[@DlgButtonClick<N>]`、`[@ButtonClick<N>]` | |
| 指定人触发 | `HCALL 人物名 @段` | |
| 地图内广播触发 | `GOTOLABEL 模式 触发字段 范围` | 模式 0-9 |

### 5.1 触发器放哪

| 文件 | 位置 | 装什么 |
|---|---|---|
| `QFunction-0.txt` | `Mir200\Envir\Market_def\` | 绝大多数全局触发（本端实测 236 个 `[@…]`） |
| `QManage.txt` | `Mir200\Envir\MapQuest_def\` | 登录/启动脚本 |
| `QMission-0.txt` | `Mir200\Envir\Market_def\` | 任务脚本（本端实测含 `[@Login]`） |
| `QChatbox-0.txt` / `QBatter-0.txt` | `Mir200\Envir\Market_def\` | 聊天框触发 / 奇经连击触发 |
| `AutoRunRobot.txt` + `RobotManage.txt` | `Mir200\Envir\Robot_def\` | 定时器 |
| `MapEvent.txt` → `QFunction-0.txt` | `Mir200\Envir\` | 地图事件 |

> ⚠️ **引擎内部触发字段不允许被玩家点 NPC 触发**：`@PlayDie`、`@StdModeFunc…`、
> `@GroupItemOn…`、`@MagTagFunc…`、`@TakeOn/@TakeOff` 等，凡是**前缀相同**的
> 加后缀也一样禁用（`@PlayDie1`、`@PlayDie死亡` 都不行）。
> 需要玩家点击时必须 `goto` 中转一次（说明书有专页说明）。

---

## 6. 文件地图：什么放在哪

细节、文件数、索引文件格式：**本包不含旧的 07 实测地图**（它来自旧根/旧拷贝，已标记为未复核）。
以你手上这份 Mir 拷贝的实测为准。

```
C:\Users\9900k\Desktop\9.28新建\MirServer222\MirServer222\
  GameCenter.exe          翎风游戏控制台（启动/配置入口）
  Mir200\                 主引擎（M2Server.exe）
    !setup.txt            核心配置（ServerName / 端口 / UseSqliteDB / 路径）
    Config.ini            控制台配置（GameDirectory / GatePort…）
    Command.ini           GM 命令名映射（可在本文件改命令名）
    GlobalVal.ini         G / A 全局变量的落盘处
    Envir\                所有脚本与配置（**编码混杂：GBK 与 UTF-8 都有，写回前逐文件实测**）
    Map\                  地图文件
    Log\ ConLog\          日志
  DBServer\
    dbsrc.ini             数据库服务配置
    FDB\ApexMir.DB        SQLite：角色/账号/人物变量
  Mud2\DB\ApexM2.DB       SQLite：StdItems / Monster / Magic
  LoginSrv\ LoginGate\ SelGate\ RunGate\ LogServer\   网关与登录
```

`Envir/` 里常用位置：

| 目录/文件 | 放什么 |
|---|---|
| `Market_def/` | NPC 脚本（按地图分子目录）+ `QFunction-0.txt` 等 Q 系列 |
| `QuestDiary/` | 被脚本引用的数据与子脚本（爆率表、充值卡、活动数据…） |
| `MapQuest_def/QManage.txt` | 登录/启动脚本 |
| `MonItems/` | 爆率文件，一个怪物一个文件，文件名 = 怪物名 |
| `Npc_def/` + `Npcs.txt` | 固定 NPC 脚本与刷新表 |
| `MerChant.txt` | 买卖 NPC 表（`脚本路径/脚本名 地图 X Y 显示名 …`） |
| `MonGen.txt` | 刷怪表 |
| `MapInfo.txt` | 地图与地图参数 |
| `StartPoint.txt` | 安全区/复活点 |
| `Robot_def/` + `Robot.txt` | 定时器 |
| `CustomMagic/` `SmartMonster/` `MonUseItems/` `CustomNPC/` `Boxs/` | 自定义技能 / 智能怪 / 人型怪装备 / 自定义 NPC / 宝箱 |

### 6.1 数据层（本端实测：全 SQLite）

| 库 | 内容 | 别称 |
|---|---|---|
| `Mud2/DB/ApexM2.DB` | `StdItems`（物品）、`Monster`（怪物）、`Magic`（技能） | 物品/怪物/技能数据库 |
| `DBServer/FDB/ApexMir.DB` | `Human`、`HumanItems`、`HumanMagic`、`HumanVariableU/T/J/Z` 等 48 张表 | 角色数据库 |

- 字段含义查 `data/03-配置与数据库-*.md`（官方字段表 + `StdMode`/`race`/`racelmg` 代码表）
- **字段名不要按字面猜**，`Stock`、`Reserved`、`Anicount` 在说明书里各有专门语义
- 读写这两个库属于 `lf-gm-ops` 的分级表（要批准 + 备份 + 审计），**只读排查**不受限

---

## 7. 高频踩坑（每一条都会真出事）

1. **编码**：`Envir` 下脚本是 **GBK + CRLF**。用 UTF-8 写回会乱码或让引擎读错。
   改前改后各验一次字节（抽样 `file` 一下）。
2. **服务器上改文件前先备份**：本端 `dbsrc.ini` 里 `AutoBackup=0`（引擎自带备份是关的），
   别指望它。备份命名 `*.bak-YYYYMMDD`。
3. **命令名照抄原文，不要"规范化"**：说明书自己大小写混用（`SENDMSG` / `sendmsg` 都有），
   同义命令也是分开的页。照抄你查到的那个页面的写法。
4. **别把「兼容HeroM2」当成「等同 HeroM2」**：那一章只说明它兼容这些命令。
5. **说明书本身有错**：三份索引末尾的「未验证/存疑」共 260+ 条（含相互冲突的
   下标范围、字段序、位置编号）。引用时把存疑一并带出，别替原文"修正"。
6. **说明书里没有运维内容**。进程、端口、启动顺序、自启方式一律实测，
   不要从说明书推断。线上实测结论见 `本文 §0.3`。
7. **注意说明书是哪个版本**。本包带的是 **2026-07-08 版（850 页）** 的文本镜像（`manual/`），
   与线上引擎（2026-07-07）同期；而 `data/01`–`06` 分片的底本是更早的 2024-06-14 版（835 页），
   两版差异的 70 页已整理进 `data/08-*.md`。**查新功能以 `manual/` 那版为准**，
   别拿 2024 版的内容当"引擎没有这个功能"。
8. **改完脚本 ≠ 生效，重载是独立的一步**。LF 引擎不感知 `Envir\` 的文件变化，
   改完必须重载对应缓存（重载项菜单 ID 与覆盖范围见 `data/09-*.md`）。
   最常漏的一条：**`QuestDiary\` 下的文件不被「所有NPC」覆盖**，要另发 15/16/17/18/19；
   而 `!setup.txt` 这类 `Mir200\` 根配置**重载无效，只能重启 M2**。
9. **`@_@` 特殊字段**是防封包刷脚本用的（只允许 `CALL`/`GOTO`/`DelayCall` 等非点击调用），
   不要拿它去接玩家点击。
10. **`BB.` 前缀只支持执行命令**，不支持检测命令。
11. **说明书原文镜像在 `manual/`**（850 个 `.txt`，一页一个），随包安装、不需要重建。
    它**不参与 `skill_query`**，要用 shell 搜（见 §0.2）。

---

## 8. 写改脚本的标准动作

1. **取证**：按第 3 节三步法确认每条命令/触发/字段真实存在，记录来源。
2. **看引用关系**：这个文件被谁 `CALL`/`goto`，改它会不会影响别人；同名备份文件（`- 副本.txt`）是不是也在被用。
3. **出 diff 或 dry-run**，交人确认（走 `lf-gm-ops` 的分级表与 `lf-gm-ops` 分级）。
4. **改前备份 → 改 → 校验编码与字节 → 重载并验证生效**（重载项怎么选、日志看哪一行见 `data/09-*.md`）。
5. **落盘结论**：把新查到的事实写进 `C:\Users\9900k\Desktop\9.28新建\_gm-notes\`（**服务端树之外**），不要只留在对话里。

---

## 9. 参考资料（本包 `data/` 分片）

`skill_query` 只搜 `data/`，而且**返回命中文件全文**；分片很多，先看 `data/00-导航.md`。

| 来源索引 | 分片数 | 内容 |
|---|---|---|
| 00-手册检索与覆盖边界 | 1 | 说明书是什么、怎么检索、章节页数、**说明书不覆盖什么**、已知错漏 |
| 01-脚本命令 | 10 | 脚本结构 / 变量体系 / **#ACT 执行命令总表** / **#IF 检测命令总表** / 重点命令原文示例 / 存疑清单 |
| 02-触发功能 | 6 | **触发总表**（触发名→文件→段名→参数→来源页）+ 重点触发原文示例 + 命名规律 |
| 03-配置与数据库 | 4 | StdItems/Monster/Magic 字段表与代码表 + 物品位置编号 + 地图参数 + 存疑清单 |
| 04-功能详解 | 5 | 功能详解，按 地图/怪物/物品/NPC界面/人物技能/活动/UI 分组 |
| 05-常见问题 | 1 | 常见问题：症状 → 原因 → 配法 |
| 06-其它资料 | 2 | 其它资料：称号/时装/骑马/国战/首饰盒/物品来源/显示名称/变量大全/GM 命令 |
| 08-2026-07-08增量 | 5 | 2026-07-08 版说明书（850 页）相对上一版的增量：新增/变更页逐条整理 |
| 09-控制台重载与菜单 | 1 | **改完怎么生效**：控制台窗口拓扑、重新加载菜单 ID 全表、每项覆盖哪些文件、生效日志原文 |

这台机器上**没有** CHM 抽取脚本（那是 Mac 工作区的工具）；说明书原文直接用 `manual\` 里的
850 页文本镜像搜（见 §0.2）。

## 10. 尚未核实（滚动更新）

**P0 阶段那批"接入云服务器后收口"的 8 项（引擎版本、`UseSqliteDB` 生效项、改哪个库、
线上 `Envir` 差异、编码、`AdminList.txt` 语义、进程/端口/自启、备份机制）已全部有结论**，
写在 `本文 §0.3`，不再重复。

当前仍未核实、写脚本时会撞到的：

1. **`QuestDiary\` 下被 `#CALL` 的文件是否被「所有NPC」重载覆盖**（缓存语义未查清）→
   保险做法连发 15/16/17/18/19，见 `data/09-*.md` §6
2. **物品/技能/怪物数据库**（菜单 4/5/6）重载的实际效果 —— 未测
3. 控制台「配置向导 → 1-基本设置」的 **`重新加载最新配置(&R)` 按钮**覆盖什么、
   是否包含 `!setup.txt` —— 未测（涉全局配置，需批准后再动）
4. 菜单 **12 参数设置 / 13 物品掉落规则 / 25 怪物爆率** 三项的实际影响面 —— 未测
5. `@PlayDie` 死亡掉落到底由谁决定（地图参数 `NODROPITEM` 等 vs `!setup.txt` 的
   `DieScatterBag` 等全局键 vs 控制台「功能设置」里的同项）—— 未查清，
   且 `!setup.txt` **改后必须重启**才生效
6. `Mir200\M2Data\ApexM2Data.DB`（WAL）用途未查清

新增未核实项请回填本清单，并把已查清的移进 `_gm-notes\` 与对应 `data/` 分片。
