# 浏览器接入：例外路径

> **何时读这份**：默认路径（headless + cookie 注入）见 SKILL.md「浏览器接入」。只有撞到下面这些情况才读这份——
>
> - **cookie 失效 / 不存在**（进站验证发现仍停在登录入口）→ 见「导出 / 刷新 cookie」。
> - **用户要实时介入**（手动过验证码、临时人工操作、想盯着看）→ 见「备选路径：connect 接管」。
> - **多开终端并行跑多个 run** → 见「多终端并行」。

## 导出 / 刷新 cookie（A1 流程，cookie 失效时重做）

cookie 由用户已登录的 Chrome 导出，只在初次配置或 cookie 过期时做一次。导出**全量**（不带 `--url`），一份覆盖所有站、所有环境，无需切 tab、无需逐站导：

```bash
browser-use connect                                   # 接管用户的 Chrome（需先开远程调试，见下）
browser-use cookies export secrets/cookies-all.json   # 不带 --url = 全量，含所有站 / 环境
```

`connect` 需要用户的 Chrome 开了远程调试。如果 `connect` 失败，告诉用户：

> 没有检测到开了远程调试的 Chrome。请先关掉当前 Chrome，然后用这个命令重新启动：
>
> ```
> open -a "Google Chrome" --args --remote-debugging-port=9222
> ```
>
> 启动后告诉我，我重试 connect。

然后等待。

## 备选路径：直接 connect 接管用户 Chrome

当 **用户需要实时介入**（手动过验证码、临时人工操作、或就是想盯着看）时，可以放弃 headless，直接 `browser-use connect` 接管用户当前 Chrome。connect 成功后执行一次 `browser-use state` 确认活跃 tab；用户开了多个 tab 时主动问哪个是被测对象，**不要** 自行假设。

除非用户明确要求，否则 **不要** 尝试自动化登录流程（输账号密码等）—— 纯 cookie 注入已经够用。
