# 这个 fork 与本地安装

`Kovisun/ds-harness-remote` 是本仓库（`liguobao/ds-harness-remote`，MIT）的 fork，只多一个提交：

> `feat(bridge): carry permanent session deletion to a local Host plugin`

它让中继通道能承载「永久删除会话」：

- 白名单放行 `session/deletion`、`session/delete`；
- 在调用 Gateway 之前用进程级钩子 `globalThis.__dshSessionDeletion` 作答（Gateway 自己没有删除端点）；
- 钩子由桌面端插件 `dsh-session-delete` 公布，转发到它注册的本机回环路由
  `POST /api-ext/session.delete`，因此手机与桌面共用同一条删除路径；
- 没有该插件的主机回答 `supported: false`，客户端退回「不支持、可用归档」，不会报错。

## Mac 上是怎么装的（**不跟随 npm 上游**）

profile 里这个包不是 registry 版本，而是一个 `file:` 本地副本：

```jsonc
// ~/.dsh/profiles/desktop/package.json
"dependencies": {
  "ds-harness-remote": "file:/Users/admin/Documents/Pi/plugins/ds-harness-remote"
}
```

那份副本就是本仓库 `packages/plugin` 构建产物（`dist/`）加本提交的补丁。因此：

- 插件管理器里不会出现 npm 上游 0.6.5/0.7 之类的更新，**不会被上游覆盖**；
- 要更新，只有两条路：改 `~/Documents/Pi/plugins/ds-harness-remote/` 这份副本，或者在本仓库
  `pnpm install && pnpm -C packages/plugin build` 重新产出 `dist/` 后替换副本；
- 升级本仓库后记得重启宿主（运行中的进程不会重新加载 Host 插件）。

## 回到 npm 上游

把 profile 的依赖改回 `"ds-harness-remote": "0.6.4"` 后 `pnpm install` 即可，代价是丢失上面的删除端点
（客户端会显示「此主机不支持安全永久删除」）。
