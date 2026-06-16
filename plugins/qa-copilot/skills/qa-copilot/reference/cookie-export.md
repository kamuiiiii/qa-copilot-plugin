# A1：导出 / 刷新 cookie

> **何时用**:测试需要登录态、但 `secrets/cookies-all.json` 不存在或已失效时。两种触发,流程相同——
>
> - **首次配置**:空仓库初始化时导出(见 `bootstrap.md` 第 3–4 步)。
> - **运行中途失效**:进站验证发现仍停在登录入口（`SessionId` 是会话级、无持久过期时间，服务端 session 超时即失效）。
>
> 失效时**不要**自己点 `Enter` 或尝试输账号密码,回这里重新导出。

cookie 由用户已登录的 Chrome 导出。导出**全量**（不带 `--url`），一份覆盖所有站、所有环境，无需切 tab、无需逐站导：

```bash
browser-use connect                                   # 接管用户已登录的 Chrome（需先开远程调试，见下）
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

然后等待。导出后起一个 headless 实例注入，**验证真的进站**（dump 正文，出现应用内真实数据 = 成功；仍只剩登录入口 = 仍未成功）再继续。
