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

## 安装（团队成员）

```text
/plugin marketplace add <这个仓库的 git 地址或 owner/repo>
/plugin install qa-copilot@qa-tools
```

之后我升级了方法论、push 上来，大家同步：

```text
/plugin marketplace update qa-tools
```

> 依赖：本 skill 通过 `browser-use` skill 驱动浏览器，请确保该 skill 在环境中可用。

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
