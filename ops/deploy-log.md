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

## 2026-07-23：首次部署到 Cloudflare Workers（Static Assets）

- **平台**：Cloudflare Workers（Static Assets 模式）
- **项目 URL（Dashboard）**：https://dash.cloudflare.com/90319a173cc213619138038dad6cfbca/workers/services/folio-2019
- **默认域名**：https://folio-2019.i-lubake.workers.dev
- **自定义域名**：无（第二阶段，可后续绑定）
- **Git 仓库**：git@github.com:ilubake/folio-2019.git
- **构建产物体积**：19 MB（`du -sh dist`）
- **单文件最大**：1.1 MB（`dist/assets/index-*.js`）
- **构建耗时**：约 4 秒（Vite build）+ 约 30 秒（wrangler deploy）
- **Lighthouse**：_待测_
- **关键配置**：
  - Framework preset：None（重要：不要选 Vite，否则 wrangler 会按 Worker 自动配置）
  - Build command：`npm run build`
  - Deploy command：`npx wrangler deploy`
  - Root directory：`/`
  - Production branch：`master`
- **配置文件**：[wrangler.jsonc](../wrangler.jsonc) 声明静态资源 + SPA 回退
- **API Token 权限**：Cloudflare Pages Edit + Workers Scripts Edit（Custom Token，不是 "Edit Cloudflare Workers" 模板）

### 踩坑记录（避免下次重蹈）

1. **Vercel 国内无法注册** → 切换到 Cloudflare。
2. **Framework preset 选 "Vite" 会被识别成 Worker，但要求 Vite ≥ 6.0.0** → 选 None + 让 wrangler.jsonc 接管。
3. **`npx wrangler deploy` 在 CI 中报 "Missing Pages project name"** → 必须用 `wrangler.jsonc` 声明，不能靠交互式 prompt。
4. **"Edit Cloudflare Workers" 模板不包含 Pages Edit 权限** → 用 Custom Token 显式加 `Cloudflare Pages → Edit`。
5. **`wrangler pages deploy` 和 `wrangler deploy` 不能混用** → Pages 项目用前者，Workers Static Assets 项目用后者。本项目是后者。

### 下次部署只需

```bash
git add .
git commit -m "<message>"
git push origin master
```

push 即自动触发 CF Workers Builds，无需手动操作。
