# qa-copilot-plugin

浏览器驱动的 QA 测试执行方法论，打包成 Claude Code 插件，供团队共享。

**这个仓库只装「方法论」**——工作流四阶段、AC（Gherkin）纪律、截图与 AC 一一绑定、性能规则、浏览器接入约定。它**不含**任何被测产品的 URL、登录态或测试产物。那些由每个人自己的**产品仓库**维护（见下方「产品仓库」）。

## 结构

```
qa-copilot-plugin/                       ← 本仓库 = marketplace 根
├── .claude-plugin/marketplace.json      ← marketplace 清单
├── plugins/qa-copilot/
│   ├── .claude-plugin/plugin.json
│   └── skills/qa-copilot/
│       ├── SKILL.md                     ← 核心方法论
│       └── reference/                   ← 按需深读：cross-site.md / browser-setup.md
└── README.md
```

## 安装

```text
/plugin marketplace add kamuiiiii/qa-copilot-plugin
/plugin install qa-copilot@qa-tools
```

> qa-copilot 还依赖 **atlassian** 插件（Jira / Confluence）。它在 marketplace 里已声明为跨市场依赖，安装 qa-copilot 时会自动一并装上，无需手动操作。

> 可选——想让全队 clone 产品仓库后免手动安装：在**产品仓库**的 `.claude/settings.json`（提交进 git、全队共享）里声明 `extraKnownMarketplaces` 指向本 marketplace，并把 `qa-copilot@qa-tools` 加进 `enabledPlugins`；信任工作区后即自动注册并安装。模板不再预置这份声明，需要的人自行添加。

### 升级后同步

```text
/plugin marketplace update qa-tools
```

## ⚠️ 前置：browser-use（插件管不了，必须各人自备）

本 skill 通过 **`browser-use`** 驱动浏览器。它**不是本插件的一部分、也无法由插件依赖机制自动安装**（plugin 的 `dependencies` 只能依赖 marketplace 里的别的 plugin，而 browser-use 是独立 CLI + skill）。每个使用者**必须自行准备好两样**，安装方式以官方文档为准：<https://docs.browser-use.com/open-source/browser-use-cli>

**前置**：Python 3.11+（Windows 还需 Git for Windows）。

1. **`browser-use` CLI**

   ```bash
   # macOS / Linux（官方推荐）
   curl -fsSL https://browser-use.com/cli/install.sh | bash
   # 或手动
   uv pip install browser-use && browser-use install
   ```

   装完用 `browser-use doctor` 自检。

2. **`browser-use` skill**（给 Claude Code 提供 CLI 用法上下文）

   ```bash
   npx skills add https://github.com/browser-use/browser-use --skill browser-use
   ```

两样缺任一，qa-copilot 跑到浏览器步骤就会失败。命令可能随官方更新而变化，最终以上面文档链接为准。

## 产品仓库（每人 / 每产品各自维护）

插件只提供方法论。**每个被测产品在一个自己的「瘦」仓库里跑 qa-copilot**——这个仓库只放 skill 运行时要读 / 写的东西，不含任何方法论。

**不用拷模板、也不用 clone 本仓库**：在产品仓库里（空仓库也行）直接跑 qa-copilot，skill 会按需把骨架建起来——

```
<your-product>/
├── .gitignore                ← skill 自动落，忽略 secrets/ runs/（兜 cookie 泄漏）
├── secrets/cookies-all.json  ← ⚠️ 唯一要你手动放的：导出全量登录态
├── targets/<env>.md          ← skill 首次缺失时引导你一起填，绝不瞎猜 URL
├── runs/<run-id>/            ← 每次跑自动建（测试产物）
└── notes/<site>.md           ← 有内容时自动建（跨 run 导航 / 定位备忘）
```

所以你**唯一要手动准备的是登录态**：把全量 cookie 导出到 `secrets/cookies-all.json`（`secrets/` 已被自动忽略，绝不提交进 git）。其余 skill 全包：`targets/` 引导填、`.gitignore` / `runs/` / `notes/` 自动建。

> `templates/product-repo/` 里有一份带注释的 `targets/qa.md` 完整版，想照着写格式可作参考；日常**不必拷**，跟着 skill 引导填即可。

### 维护 targets

`targets/<env>.md` 是 skill 运行时读的被测站清单——一个文件 = 一个环境。规则（skill 引导你填时也按这套；`templates/product-repo/targets/qa.md` 的注释头里有完整版）：

- **填表**：每站一行 `| id | URL | 说明 |`。`id` 取短横线小写、语义清晰（`portal` / `order` / `billing` …），它会被用作 AC 里的 `@site` 标注、session 后缀（`$SID-<id>`）、截图前缀。
- **id 跨环境同名**：同一个站在 `qa.md` 和 `uat.md` 里必须叫同一个 `id`，这样跨环境复测时 AC 的 `@site` 无需改动。
- **加站 = 表里加一行；加环境 = 复制一个文件**改环境名（各站 id 与已有环境保持一致，URL 换成该环境的）。
- 不要写进 targets 的：角色（操作端 / 观察端，每次测试在该 run 的 `ac.md` 里定）、登录态、一次性的测试范围。

> 首次在一个还没有 `targets/<env>.md` 的仓库里跑 qa-copilot 时，skill 会发现清单缺失、停下来按这套规则引导你补，不会瞎猜 URL。
