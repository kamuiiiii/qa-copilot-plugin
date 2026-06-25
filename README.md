# qa-copilot-plugin

浏览器驱动的 QA 测试执行方法论，打包成 Claude Code 插件，供团队共享。

**这个仓库只装「方法论」**——工作流四阶段、AC（Gherkin）纪律、截图与 AC 一一绑定、性能规则、浏览器接入约定。它**不含**任何被测产品的 URL、登录态或测试产物。那些由每个人自己的**产品仓库**维护（见下方「产品仓库」）。

## 设计哲学：记忆由用户维护，插件不持有状态

qa-copilot **本身是无状态的**——它不维护任何记忆层，不偷偷记你的偏好、不积累跨会话的历史。所有需要被「记住」的东西（被测站清单、登录态、导航备忘、测试产物）都落在**你自己的产品仓库**里，是你能看、能改、能 diff、能进 git、能随时删的纯文本。

这样设计是刻意的：

- **所有权归你**——记忆是你仓库里的 markdown，不是插件的私有黑盒。插件升级或卸载，你的记忆不受影响、也不会变成孤儿。
- **可见、可控、可信**——你对「记了什么」始终有完全的可见性和否决权，不存在插件背着你攒数据。
- **插件保持简单**——它只负责按约定**读取**这些文件作为上下文（被测站读 `targets/`、跨 run 导航备忘读 `notes/<site>.md`、可复用用例读 `specs/<flow>.md`），不负责管理存储、迁移、清理。
- **提议而非自作主张**——当一次测试发现了值得长期留存的东西（比如某个站的稳定定位方式、一份验证有效的用例），插件会**建议**你把它记进 `notes/` 或晋升进 `specs/`，但写什么、写不写、写到哪，决定权和动作都在你这边。

一句话:**记忆的所有权与持久化归用户,插件只持有「读取这层记忆」的约定。** 下方「产品仓库」一节就是这个哲学的具体落地。

## 结构

```
qa-copilot-plugin/                       ← 本仓库 = marketplace 根
├── .claude-plugin/marketplace.json      ← marketplace 清单
├── plugins/qa-copilot/
│   ├── .claude-plugin/plugin.json
│   └── skills/qa-copilot/
│       ├── SKILL.md                     ← 核心方法论
│       └── reference/                   ← 按需深读：bootstrap.md（首次初始化）/ cookie-export.md（导出登录态）/ cross-site.md（跨站）
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

## ⚠️ 并发跑多个测试：冲突要你自己把关

skill 用 run-id 做 session 名，**浏览器实例之间是隔离的**（多个终端 / 多个 run 各开各的浏览器，互不抢）——这层 skill 已经替你做好。但**隔离只到浏览器为止**：

- 所有 run 共用同一份 `secrets/cookies-all.json`，即**同一个账号、同一套后端数据**。两个并发测试一旦碰到重叠的数据 / 状态（一个改 setting、另一个正在读它；一个下单消耗库存、另一个在断言库存；一个测删除、另一个依赖那条数据），会在**后端层面互相污染**。
- **skill 的视野只限当前这一个 run**，它看不到别的终端 / 别的 run 在干什么，因此**无法察觉、也无法替你避开**这种冲突。报告里读到的"异常"，可能其实是另一个并发 run 造成的，skill 不会（也没法）提醒你这一点——你可能据此误判产品有 bug。

所以并发跑多个测试时，**保证它们作用在不相交的数据 / 站 / 账号上是你的责任**：要么让各 run 测互不重叠的范围，要么干脆错开串行跑。**凡涉及破坏性 / 写操作的测试，尤其不要并发。**

## 产品仓库（每人 / 每产品各自维护）

插件只提供方法论。**每个被测产品在一个自己的「瘦」仓库里跑 qa-copilot**——这个仓库只放 skill 运行时要读 / 写的东西，不含任何方法论。

**不用拷模板、也不用 clone 本仓库**：在产品仓库里（空仓库也行）直接跑 qa-copilot，skill 会按需把骨架建起来——

```
<your-product>/
├── CLAUDE.md                 ← skill 首次跑时缺则生成（薄指引，已有则不动）
├── .gitignore                ← skill 自动落，忽略 secrets/ runs/（兜 cookie 泄漏）
├── secrets/cookies-all.json  ← ⚠️ 唯一要你手动处理的：测试需要登录态，配合 skill 导出全量 cookie
├── targets/<env>.md          ← skill 首次缺失时引导你一起填，绝不瞎猜 URL
├── runs/<run-id>/            ← 每次跑自动建（测试产物）
├── notes/<site>.md           ← 有内容时自动建（跨 run 导航 / 定位备忘）
└── specs/<flow>.md           ← 有内容时自动建（跨 run 可复用的正式 AC / 用例母本）
```

所以你**唯一要手动准备的是登录态**：把全量 cookie 导出到 `secrets/cookies-all.json`（`secrets/` 已被自动忽略，绝不提交进 git）。其余 skill 全包：`targets/` 引导填、`CLAUDE.md` / `.gitignore` / `runs/` / `notes/` / `specs/` 自动建。

> `templates/product-repo/` 里有一份带注释的 `targets/qa.md` 完整版，想照着写格式可作参考；日常**不必拷**，跟着 skill 引导填即可。

### 维护 targets

`targets/<env>.md` 是 skill 运行时读的被测站清单——一个文件 = 一个环境。规则：

- **填表**：每站一行 `| id | URL | 说明 |`。`id` 取短横线小写、语义清晰（`portal` / `order` / `billing` …），它会被用作 AC 里的 `@site` 标注、session 后缀（`$SID-<id>`）、截图前缀。
- **id 跨环境同名**：同一个站在 `qa.md` 和 `uat.md` 里必须叫同一个 `id`，这样跨环境复测时 AC 的 `@site` 无需改动。
- **加站 = 表里加一行；加环境 = 复制一个文件**改环境名（各站 id 与已有环境保持一致，URL 换成该环境的）。
- 不要写进 targets 的：角色（操作端 / 观察端，每次测试在该 run 的 `ac.md` 里定）、登录态、一次性的测试范围。

> 首次在一个还没有 `targets/<env>.md` 的仓库里跑 qa-copilot 时，skill 会发现清单缺失、停下来按这套规则引导你补，不会瞎猜 URL。
