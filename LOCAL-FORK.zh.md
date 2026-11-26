# 本机使用的 ds-harness-remote（Kovisun 维护版）

这个 fork 只做一件事：**让 DSH Go 手机端在中继连接下能真正永久删除会话**。

## 现状

- 基线：上游 `liguobao/ds-harness-remote` **v0.6.8**（2026-11-25，官方从这版起适配 DSH 0.2.3）。
- 唯一补丁：`packages/plugin/src/harness-remote-bridge.ts`
  1. `HARNESS_REMOTE_ALLOWLIST` 放行 `session/delete`、`session/deletion`；
  2. `call()` 在 Gateway 之前用进程级钩子 `globalThis.__dshSessionDeletion` 回答这两个端点。
- 上游仍未支持删除（白名单里没有这两个端点），所以补丁必须留着。

## 本机怎么装（不再走 npm 官方源）

1. 取 npm 包 `ds-harness-remote@0.6.8` 的 tarball（含编译好的 `dist/`）；
2. 对 `dist/index.js` 打上面同一处补丁（源码补丁在本仓库）；
3. 版本号改成 `0.6.8-fork.1`，打成 `ds-harness-remote-0.6.8-fork.1.tgz`，**作为 release 资产挂在仓库里**；
4. DSH 桌面 profile `~/.dsh/profiles/desktop/package.json` 用这个 URL 引用：

```json
"ds-harness-remote": "https://github.com/Kovisun/ds-harness-remote/releases/download/v0.6.8-fork.1/ds-harness-remote-0.6.8-fork.1.tgz"
```

这样插件版本的来源只有本仓库，DSH 插件管理器不会拿官方 npm 的最新版来提示升级（这正是之前显示「Remote ↑ 0.6.8」的原因）。本地副本仍留在 `/Users/admin/Documents/Pi/plugins/ds-harness-remote`，用于离线重装与调试。

## 升级流程

上游发新版时：取新 tarball → 打同一处补丁 → 改版本号（`<上游版本>-fork.N`）→ 建 release、上传 tgz → 改 profile 里的 URL → `pnpm install` → 重启 DSH。
只升级、不重新打补丁，中继就会把删除请求拒掉，手机端会退回「此主机不支持安全永久删除」。
