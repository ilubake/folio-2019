## Why

项目目前只在本地能跑，没有 Git 仓库、没有线上入口，且 [src/index.html](src/index.html) 还残留原作者 Bruno Simon 的 Google Analytics ID、SEO 元数据和社交分享链接——直接部署会把访问数据发给别人、分享卡片显示错误。需要一条从零开始、可复用的上线路径，让站点在我自己的域名下可访问，并保证线上版本展示的是我自己的身份。

## What Changes

- **BREAKING**（仅对原作者代码而言）：移除 / 替换 [src/index.html](src/index.html) 中的 GA 追踪（`UA-3966601-5`）、`og:*`、`twitter:*`、`<title>`、`meta description`、favicon 关联的 apple-mobile-web-app-title 等所有指向 bruno-simon.com 的字段。
- 初始化 Git 仓库并首次推送到 GitHub（为 Vercel 自动部署做准备）。
- 关闭 [vite.config.js](vite.config.js) 的生产 sourcemap，避免源码在线上被还原。
- 在 Vercel 上以 Vite 预设部署，输出目录 `dist`，获得 `xxx.vercel.app` 默认域名。
- 提供上线前本地构建验证清单与上线后线上核对清单，作为可重复执行的流程。
- 预留自定义域名接入步骤（第二阶段，非阻塞）。

## Capabilities

### New Capabilities

- `site-identity`: 站点的标题、描述、社交分享卡（OG/Twitter）、favicon 元数据、数据分析追踪 ID 全部归属于我本人，可一键替换且不残留原作者信息。
- `production-build`: 生产构建产物经过优化（关闭 sourcemap）、可本地预览、且符合静态托管的目录结构要求。
- `static-deployment`: 项目可通过 Git 推送自动部署到 Vercel，获得可访问的 HTTPS 默认域名，并支持后续绑定自定义域名。

### Modified Capabilities

无（`openspec/specs/` 当前为空，全部为新增）。

## Impact

- **代码**：[src/index.html](src/index.html)（SEO/GA/分享元数据）、[vite.config.js](vite.config.js)（sourcemap 开关）。
- **配置文件**：新增 [.gitignore](.gitignore)（排除 `node_modules/`、`dist/`），可能新增 `vercel.json`（如默认预设不够用时）。
- **外部系统**：GitHub 仓库、Vercel 项目、（可选）域名注册商。
- **依赖**：无新增 npm 依赖；构建仍使用 Vite 5。
- **不可逆操作**：删除原作者 GA 追踪后无法恢复其历史数据（本来也不属于我们）；首次推送到 GitHub 后仓库公开可见性需谨慎选择。
