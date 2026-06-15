# qa-copilot — <产品名> 测试仓库

本仓库是 **<一句话描述被测产品 + 环境，如「某派单 / 订单系统，qa / uat 环境」>** 的 QA 执行仓库。

测试**方法论**（工作流四阶段、AC 的 Gherkin 纪律、截图与 AC 一一绑定、性能规则、浏览器接入 / cookie 注入 / 跨站约定、报告模板……）**全部由 `qa-copilot` skill 提供**，不在本仓库重复。用自然语言描述测试目标即可触发该 skill，或显式 `/qa-copilot`。

始终用 **中文** 回复。

## 本仓库提供给 skill 的东西

skill 运行时会读这些（都在当前仓库根下）：

- `targets/<env>.md` — 各环境的被测站清单（`qa.md` / `uat.md`），每站一行 `| id | URL | 说明 |`。加站就往表里加一行。
- `secrets/cookies-all.json` — 全量登录态（含所有站 / 环境），cookie 注入用。**已 gitignore，绝不提交。**
- `runs/<run-id>/` — 每次测试的产物（`ac.md` / `screenshots/` / `report.md`）。**已 gitignore。**

## 本产品特有的坑

> 只记录**仅属于本产品、skill 通则覆盖不到**的怪异行为（如某控件点击会意外跳页、某 toast 用 emotion 随机 class 渲染、某入口有隐藏前置）。通用纪律不要写在这里。
>
> 目前无已登记的产品特异坑——执行中发现的，按 skill 里「跨 run 引用纪律」正常踩、正常记录进 report 的 Open question，确认是稳定的产品行为后再回填到这里。

## 前置：安装 qa-copilot 插件

本仓库 `.claude/settings.json` 已声明插件，**clone 后信任工作区即自动安装**，无需手动操作。
若未自动装（或想手动装），见插件 README：

```text
/plugin marketplace add kamuiiiii/qa-copilot-plugin
/plugin install qa-copilot@qa-tools
```

依赖 `browser-use` skill + CLI（各人自备，装法见插件 README）。
