# 跨站测试（多站场景）

> **何时读这份**：当用户在意图 / AC 阶段声明本次测试要在 **同一环境下的多个站**（≥2 站）之间联动——在 A 操作、在 B / C 观察效果。**只在涉及 ≥2 个站时启用**；单站测试一切照旧——`ac.md` **不写** `## 站` 块、session 用裸 `$SID`、截图**不带**站前缀，不用读这份。

核心原则：**cookie 这层塌成一份全量，但"站"作为产物维度始终保留**——否则报告说不清"在哪操作、在哪观察"。

- **环境是 run 级的、单一的**：一个跨站测试始终在一个环境内（全程 qa 或全程 uat），不混。开始前读对应的 `targets/<env>.md` 拿到本环境的站清单（`id → URL`）。
- **本次涉及哪几个站 + 各自角色（操作端 / 观察端），由用户在 AC 阶段说清**，记进该 run 的 `ac.md` 一个 `## 站` 块：每行 `@<id> → 本环境 URL + 角色`。这份 `## 站` 块就是本次的**站清单**——起 session、`@site` 标注都按它来。

## 浏览器开法：每站一个 session

单站测试就一个 session（裸 `$SID`，见 SKILL.md「浏览器接入 → 默认路径」）。**跨站测试**时，cookie 不变——还是同一份全量 `secrets/cookies-all.json`，它本来就含所有站。差别只在 **session**：本次涉及的每个站起一个 **独立 session**，名 = `$SID-<site-id>`（如 `$SID-portal` / `$SID-billing`），这样各站页面同时活着，在 A 操作完切到 B 直接看，不丢 B 的 filter / 滚动状态。遍历本次站清单（`ac.md` 的 `## 站` 块）批量起，每个都导入同一份全量 cookie：

```bash
# 对清单里每个涉及的站（id=xxx / ...）：
browser-use --session $SID-portal  open <portal 目标页>      # URL 从 targets/<env>.md 取
browser-use --session $SID-portal  cookies import secrets/cookies-all.json
browser-use --session $SID-portal  open <portal 目标页>      # import 后必须重新 open 才生效
```

**逐站验证进站，不要因为是全量就假设 N 个站全活**：对每个 session 单独 dump 关键文案确认进了应用内（特征见 SKILL.md「浏览器接入 → 默认路径」第 2 步）。任一站仍停在登录入口 → 判定该站 cookie 失效，停下来提示重新导出全量（见 `reference/browser-setup.md`），**不要** 自己瞎点。测完对每个 `$SID-<id>` 都 `close`。

## "站"维度贯穿所有产物

cookie 全量了不代表站可以省。每个产物都带一个 site 标签，数量无关（2 / 3 / N 同一套）：

- **AC 步骤**：每个 step 标 `@<site-id>`，site-id 来自 `targets/<env>.md` 的站清单。一个主 `When`（在某个操作端），`Then` 可以是多站的 `And` 列表（扇出观察），异步生效的观察把超时窗口写进 `Then`：

  ```gherkin
  AC-1: admin 开启 Auto-dispatch 后，portal 与 billing 同步生效
    Given @admin 规则页显示 "Auto-dispatch: OFF"
      And @portal 列表顶部无 "Auto" 标记
    When @admin 把 "Auto-dispatch" 切到 ON 并点 "Save"
    Then @admin 出现保存成功提示 "Saved"
      And @portal 刷新后列表顶部出现 "Auto" 标记
      And @billing 在 60s 内出现一条 type="auto-dispatch" 的计费记录
  ```

- **截图**：站名进 slug，如 `17-portal-auto-badge.png` / `18-billing-record.png`。`ac.md` 的截图清单按 `@site` 分别列。
- **session**：`$SID-<site-id>`，如上。

## 多站扇出观察（一个 When，多个 Then@site）

一个操作端的动作常要在多个观察端分别验证生效。这种 AC 的 `Then` 是 **多站的 And 列表**，每个站 **独立判定、独立截图、独立超时**——不要糊成一段。`report.md` 里这条 AC 的"观察"按站分行：

```
- 观察:
  - @portal: 刷新后列表顶部出现 "Auto" 标记 — 符合
  - @billing: 90s 内未见 type="auto-dispatch" 计费记录 — 不符合（预期 60s 内）
```

任一站不符合，整条 AC 判 ❌ / ⚠️，并指明是哪个站断的。

## 跨站时序链：每跳留证据 + 轮询超时

跨站传播往往 **不是同步的**（中间隔着队列、消息、最终一致性），且常常 **有先后依赖**（admin 配完 portal 才出入口，portal 提交后 billing 才落账）。两条纪律：

1. **每一跳都留证据**。`@admin 操作 → @portal 生效 → @billing 落账` 这条链，每跳都 dump + 截图。否则最后只看到 `@billing` 没数据，根本分不清是 admin 没生效、还是 portal 没提交、还是 billing 还没传到。
2. **异步观察用轮询 + 明确超时，不要盲 sleep**。`Then` 里写清超时窗口（如 "60s 内"），用 `browser-use --session $SID-billing wait text "..."` 配 `--timeout`，或固定间隔轮询直到出现 / 超时。**超时不等于 fail**：构造得出的失败才判 ❌；只是没等到、原因不明的，判 `needs human review` 并在 Open question 里写清等了多久、轮询了几次。
