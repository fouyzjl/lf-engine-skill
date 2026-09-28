# lf-engine（nuphus 技能包）

翎风引擎（LF 引擎）的事实与脚本开发规范 + 850 页官方说明书的提炼索引。

## 这是什么

由 `mir-gm` 工作区的生成脚本编译而来，源是那边的 DSH 技能目录（`mir-gm` 仓库根下的隐藏技能目录）。
**不要在本仓库里直接改内容**——改了会在下次生成时被覆盖；要改去 `mir-gm` 的源里改，然后重新生成并推送。

## 安装

nuphus → 技能页 → 用 git URL 安装，或把本仓库根目录整个拷进 nuphus 的
plugin 根：`<nuphus 安装目录>\plugin\skills\community\lf-engine\`。
（nuphus 的 `skill_install_git` 把**仓库根**整体装成一个技能，所以一个仓库只能放一个技能。）

## 内容

- `skill.json` — nuphus SkillManifest
- `SKILL.md` — 正文（用 `skill_read` 读；`skill_query` 返回命中文件全文）
- `data/` — 说明书提炼索引分片（`skill_query` 只搜这里）
- `manual/` — 说明书原文镜像（**不入库**，另经 SSH 放置；不参与 `skill_query`）

## 更新流程

源在 `mir-gm` 工作区（那边是 DSH 技能目录），本仓库只是它的 nuphus 版产物：

1. 改 `mir-gm` 里的源；
2. 跑生成器 `build-nuphus-skills.py`；
3. 跑发布脚本 `publish-nuphus-skills.sh`（同步 + 提交 + 推送本仓库）；
4. 在装了本技能的机器上拉取同步：`.150` 上是 `C:\Users\9900k\nuphus-skills\sync-skills.ps1`；
5. nuphus 的技能页/新会话会实时扫描，**不需要重启**。

生成日期：2026-09-28
