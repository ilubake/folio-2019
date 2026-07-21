## Context

项目是 Bruno Simon 的 folio-2019 fork：纯静态 SPA（Three.js + Vite），无后端。当前状态：

- 代码本地可跑，未初始化 Git。
- [src/index.html](src/index.html) 残留原作者 GA (`UA-3966601-5`)、SEO、社交分享元数据。
- [vite.config.js](vite.config.js) 开启 `sourcemap: true`，生产环境会暴露源码。
- 用户首次部署、Windows 环境、面向中文受众但暂不强制国内极速访问。

约束：单文件最大 0.6 MB、静态资源总 16 MB——远低于所有静态托管平台的单文件限制，无需 R2/OSS 拆分。

## Goals / Non-Goals

**Goals:**

- 一条可重复执行的从零到上线流程，任何团队成员按文档走都能在 30 分钟内完成部署。
- 线上版本展示我的身份（不是原作者的）。
- 不引入新的运行时依赖、不改变 3D 场景功能。
- 留好迁移空间：产物是纯静态 `dist/`，随时能换平台。

**Non-Goals:**

- 不做国内 CDN / 备案（第二阶段，等流量真实增长再投入）。
- 不重构代码、不升级依赖。
- 不接入自定义数据分析（GA 删除即可，暂不替换为其他方案）。
- 不做多环境（preview / staging），只用 production。
- 不做 A/B 测试、灰度发布。

## Decisions

### 决策 1：托管平台选 Vercel，不选 Cloudflare Pages

**为什么**：首次部署门槛最低（GitHub 一键导入、自动识别 Vite），不用改代码，构建快。CF Pages 在免费带宽上更优（无限 vs 100 GB），但个人作品集流量远低于此，差异不显著。

**备选**：
- Cloudflare Pages：带宽无限，但首次需要手动配 build；留作第二阶段或流量超限时迁移目标。
- 自有 VPS + Nginx：完全可控，但运维负担大，非必要不引入。

### 决策 2：GA 追踪直接删除，不替换

**为什么**：用户暂未表达数据分析需求；原作者 GA 是 UA 版（已停用），替换成 GA4 需要新建账号、配域名验证，不阻塞上线。直接删 [src/index.html:88-94](src/index.html#L88-L94) 两段 `<script>` 最快。

**备选**：保留 `<script>` 占位换 `G-XXXXXXX`——推迟到后续单独 change。

### 决策 3：关闭生产 sourcemap

**为什么**：[vite.config.js:16](vite.config.js#L16) 的 `sourcemap: true` 会把源码映射一起发布，体积更大、Three.js 着色器可被还原。个人作品集没有线上 debug 需求。

**备选**：保留 sourcemap 但上传到 Sentry 等错误监控——过度工程，跳过。

### 决策 4：自定义域名列为第二阶段，不阻塞首上线

**为什么**：先用 `xxx.vercel.app` 验证部署无误，再绑域名。避免一次性引入 DNS / SSL / 备案等多重变量，出问题好定位。

### 决策 5：不新增 `vercel.json`

**为什么**：Vercel 的 Vite 预设已能正确识别 `dist/` 输出（[vite.config.js:14](vite.config.js#L14)），零配置即可部署。添加 `vercel.json` 反而增加维护面。

**触发新增的条件**：需要 rewrite 规则、自定义 header、或 SPA fallback 时（当前是纯单页，不需要）。

## Risks / Trade-offs

- **[Vercel 政策变动，免费额度收窄]** → 产物是纯静态 `dist/`，迁移到 Cloudflare Pages / Netlify / Nginx 成本 < 1 小时。
- **[vercel.app 默认域名在中国大陆 DNS 污染]** → 作为第一阶段的临时验证 URL 可接受；绑定自有域名后若国内仍慢，再切国内 CDN。
- **[删除原作者 GA 后无法回收历史流量数据]** → 本就不属于我们，无实际损失。
- **[关闭 sourcemap 后线上排错困难]** → 可通过本地复现 + Vite preview 解决；如后续需要可临时开 sourcemap 单次构建。
- **[Git 仓库首次推送包含 .git 历史]** → 提交前检查 `.gitignore` 是否覆盖 `node_modules/`、`dist/`、`.env*`，避免误传密钥或大文件。
- **[Vercel 自动部署被 GitHub 仓库可见性影响]** → 公开仓库免费部署；私有仓库也在 Hobby 免费档内，无差异。

## Migration Plan

1. **本地准备**（无风险）：替换 [src/index.html](src/index.html) 身份信息、关闭 sourcemap、确认 `.gitignore`。
2. **本地验证**：`npm run build` → `npx vite preview --outDir dist` → 控制台无报错。
3. **首次部署**：推 GitHub → Vercel 导入 → 验证 `xxx.vercel.app` 可访问。
4. **回滚策略**：Vercel 每次部署有版本，出问题一键回滚到上一版；代码层面 `git revert`。

## Open Questions

- 用户的最终品牌名 / 域名是什么？（影响 [src/index.html](src/index.html) SEO 字段的替换值，但首阶段用 `xxx.vercel.app` 不阻塞）
- 是否需要保留原作者的 "Threejs Journey" 弹窗推广（[src/index.html:46-85](src/index.html#L46-L85)）？默认保留，待用户决定后单独 change。
