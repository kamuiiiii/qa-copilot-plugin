# qa-copilot-plugin

浏览器驱动的 QA 测试执行方法论，打包成 Claude Code 插件，供团队共享。

**这个仓库只装「方法论」**——工作流四阶段、AC（Gherkin）纪律、截图与 AC 一一绑定、性能规则、浏览器接入约定。它**不含**任何被测产品的 URL、登录态或测试产物。那些由每个人自己的**产品仓库**维护（见下方「产品仓库」）。

## 结构

```
qa-copilot-plugin/                       ← 本仓库 = marketplace 根
├── .claude-plugin/marketplace.json      ← marketplace 清单
├── plugins/qa-copilot/
│   ├── .claude-plugin/plugin.json
│   └── skills/qa-copilot/SKILL.md       ← 纯方法论
└── README.md
```

## 安装

### 方式 A：随产品仓库自动装（推荐，日常用这个）

在**产品仓库**的 `.claude/settings.json`（提交进 git、全队共享）里声明。队友 clone 该产品仓库 + 信任工作区后，marketplace 自动注册、插件自动装好，无需任何手动命令：

```json
{
  "extraKnownMarketplaces": {
    "qa-tools": { "source": { "source": "github", "repo": "kamuiiiii/qa-copilot-plugin" } }
  },
  "enabledPlugins": { "qa-copilot@qa-tools": true }
}
```

> 放 `.claude/settings.json`（共享），不要放 `settings.local.json`（个人）。新建产品仓库时把这段抄进去即可。

### 方式 B：手动装（无产品仓库 / 想先单独试用时）

当你还没有产品仓库、只想试一下插件，或某个目录没写上面的声明，就手动装：

```text
/plugin marketplace add kamuiiiii/qa-copilot-plugin
/plugin install qa-copilot@qa-tools
```

> 方式 A 本质上就是让 Claude Code 自动替你执行这两条；二选一即可，不必都做。

### 升级后同步

无论哪种方式装的，我升级了方法论并 push 后，大家这样同步到最新：

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

插件只提供方法论。**每个被测产品建一个「瘦」仓库**，提供 skill 运行时要读的东西：

```
<your-product>/
├── CLAUDE.md                 ← 薄壳：声明本仓库测什么 + 本产品特有的坑 + 引用本 skill
├── targets/
│   ├── qa.md                 ← 本环境的站清单（id | URL | 说明）
│   └── uat.md
├── secrets/cookies-all.json  ← 登录态（.gitignore）
└── runs/                     ← 测试产物（.gitignore）
```

`targets/` 由各人自己动态增改——加站就往表里加一行。每个产品仓库的 `CLAUDE.md` 只写产品身份和特异行为，**方法论不重复**，统一走本插件的 skill。

## 升级方法论

直接改 `plugins/qa-copilot/skills/qa-copilot/SKILL.md`，bump `plugin.json` 的 `version`，commit + push。团队 `/plugin marketplace update qa-tools` 即可拿到。
