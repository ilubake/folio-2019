# Deploy Log

> 本文件记录每次部署的关键信息，便于后续回溯、迁移、故障排查。
> 第一条记录由 `/opsx:apply deploy-to-production` 创建，后续每次重大部署由人工补充。

## 模板（复制后填写）

```
## YYYY-MM-DD：<简要标题，如 "首次部署到 Vercel">

- 平台：
- 项目 URL（Dashboard）：
- 临时域名 / 默认域名：
- 自定义域名：（未绑定填 "无"）
- 本次 commit SHA：
- 构建产物体积：
- Lighthouse Performance / Accessibility：
- 备注：
```

## 待填：首次部署

- 平台：Vercel（计划）
- 项目 URL：_待用户填（部署后从 Dashboard 复制_
- 临时域名：_待填（形如 `xxx.vercel.app`）_
- 自定义域名：无（第二阶段）
- 本次 commit SHA：_待用户 push 后填_
- 构建产物体积：19 MB（`du -sh dist`，2026-07-21 本地构建）
- Lighthouse Performance / Accessibility：_待填_
- 备注：本次提交包含身份替换（ilubake）、关闭 sourcemap、删除原作者 GA 与 Threejs Journey 弹窗。
