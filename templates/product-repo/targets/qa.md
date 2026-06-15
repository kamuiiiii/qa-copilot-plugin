# qa 环境

> 一个文件 = 一个环境 = 这个环境下的一整组被测站。一次测试选定一个环境（读这一个文件），
> 在其中作用于一个或多个站。新增站就往下表加一行。
>
> `id` 用作 AC 里的 `@site` 标注、session 后缀（`$SID-<id>`）、截图前缀。
> 同一个站的 `id` 在各环境文件里应保持一致（`qa.md` 与 `uat.md` 都叫 `portal`），
> 这样跨环境复测时 AC 的 `@site` 标注无需改动。
> 角色（操作端 / 观察端）是每次测试才定的，**不写在这里**，写进该 run 的 `ac.md`。

- **环境**: qa

## 站

| id     | URL                           | 说明     |
| ------ | ----------------------------- | -------- |
| portal | https://portal.example-qa.com | 操作门户 |
| order  | https://order.example-qa.com  | 订单中心 |

<!--
怎么用这份模版：

1. 拷成 targets/<env>.md —— 每个环境一个文件，如 targets/qa.md、targets/uat.md。
   把标题、`- **环境**:` 和表里的占位 URL 换成真实值。

2. 加一个站 → 表里加一行（一个 id / URL / 说明）。
   - id 取短横线小写、语义清晰（portal / order / billing / admin …）。
   - id 一旦定下，跨环境（qa.md / uat.md）必须同名，AC 才能跨环境复用。

3. 加一个环境 → 复制本文件成 targets/<新环境>.md，改标题与 `- **环境**:`，
   各站 id 与已有环境保持一致，URL 换成该环境的。

4. 不要写进这里的：角色（操作端/观察端，每次测试在 ac.md 里定）、登录态
   （在 secrets/cookies-all.json）、任何一次性的测试范围。
-->
