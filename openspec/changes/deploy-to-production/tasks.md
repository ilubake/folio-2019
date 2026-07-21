## 1. 本地代码改造（替换身份 & 构建优化）

- [x] 1.1 在 [src/index.html](src/index.html) 替换 `<title>` 为自己的品牌名
- [x] 1.2 替换 `meta[name=description]`、`meta[itemprop=name]`、`meta[itemprop=description]` 为自己的简介
- [x] 1.3 替换 `meta[property=og:url]` 为最终域名（暂无则先用 `xxx.vercel.app` 占位）
- [x] 1.4 替换 `meta[property=og:title]`、`meta[property=og:description]`、`meta[property=og:image]`
- [x] 1.5 替换 `meta[name=twitter:title]`、`meta[name=twitter:description]`、`meta[name=twitter:image]`
- [x] 1.6 替换 `meta[name=apple-mobile-web-app-title]`、`meta[name=application-name]`
- [x] 1.7 删除 [src/index.html:88-94](src/index.html#L88-L94) 的 GA `<script>` 两段（或替换为自己的 G-ID）
- [x] 1.8 修改 [vite.config.js:16](vite.config.js#L16) 把 `sourcemap: true` 改为 `sourcemap: false`
- [x] 1.9 全局搜索 `bruno-simon` 确认无残留（除 license / readme 引用原作者外）

## 2. Git 仓库初始化

> 已存在：仓库已初始化并推送到 `git@github.com:ilubake/folio-2019.git`，`.gitignore` 齐全。

- [x] 2.1 确认项目根有 `.gitignore`，包含 `node_modules/`、`dist/`、`.env*`、`.DS_Store`、`*.log`
- [x] 2.2 执行 `git init`（若未初始化）
- [x] 2.3 `git add .` 后用 `git status` 确认没有误加 `node_modules/` 或 `dist/`
- [x] 2.4 创建首次 commit：`git commit -m ":tada: Initial commit"`
- [x] 2.5 在 GitHub 新建空仓库（建议先 private，验证无误后改 public）
- [x] 2.6 添加 remote 并推送：`git remote add origin <url> && git push -u origin master`

## 3. 本地构建验证

- [x] 3.1 执行 `npm install`（首次）或确认依赖已装
- [x] 3.2 执行 `npm run build`
- [x] 3.3 确认 `dist/` 在项目根目录生成，而非 `src/dist/`
- [x] 3.4 确认 `dist/` 中无 `.map` 文件（`find dist -name "*.map"` 应为空）
- [x] 3.5 用 `du -sh dist` 记录总体积，确认单文件最大者 < 5 MB
- [ ] 3.6 执行 `npx vite preview --outDir dist` 本地预览
- [ ] 3.7 浏览器逐项核对验证清单：3D 场景、`.glb` 模型无 404、`.mp3` 可播放
- [ ] 3.8 核对：控制台无 `draco_decoder` 404、无 CORS / 模块加载错误
- [ ] 3.9 核对：favicon、分享图无 404
- [ ] 3.10 浏览器标签页标题、描述显示为自己的品牌

## 4. Vercel 部署

> ⚠️ 需用户操作：涉及账号登录、Dashboard 配置，Claude 无法代办。

- [ ] 4.1 注册 / 登录 [vercel.com](https://vercel.com)（建议用 GitHub 账号）
- [ ] 4.2 Dashboard → `Add New Project` → 导入 `ilubake/folio-2019` 仓库
- [ ] 4.3 确认 Framework Preset 选 **Vite**
- [ ] 4.4 确认 Build Command 为 `npm run build`
- [ ] 4.5 确认 Output Directory 为 `dist`（**不要填 `src/dist`**）
- [ ] 4.6 确认 Install Command 为 `npm install`
- [ ] 4.7 点 `Deploy`，等待 1-3 分钟构建完成
- [ ] 4.8 访问 `xxx.vercel.app`，确认 3D 场景加载正常

## 5. 上线后线上验证

> ⚠️ 依赖 4.x 完成，需用户在浏览器中核验。

- [ ] 5.1 HTTPS 正常（地址栏小锁）
- [ ] 5.2 HTTP 自动重定向 HTTPS
- [ ] 5.3 移动端可访问（性能差是预期，能加载即可）
- [ ] 5.4 控制台无 404 / CORS 报错
- [ ] 5.5 favicon 显示正确
- [ ] 5.6 把线上 URL 粘到微信 / Twitter 预览，确认 OG 卡片正确
- [ ] 5.7 搜索引擎视角：页面标题与描述已替换为自有品牌
- [ ] 5.8 检查浏览器网络请求，确认无 `googletagmanager.com/gtag/js?id=UA-3966601-5`
- [ ] 5.9 Lighthouse 跑一次，记录 Performance / Accessibility 分数作为基线

## 6. 文档与知识沉淀

- [ ] 6.1 把实际使用的域名、Vercel 项目名补回 [DEPLOY.md](DEPLOY.md)（或新增 `ops/README.md`）
- [x] 6.2 在仓库根新增 `ops/deploy-log.md`，记录首次部署日期、Vercel 项目 URL、临时域名
- [ ] 6.3 确认 `git push` 后 Vercel 自动重新部署，无需手动触发

## 7. 第二阶段：自定义域名（非阻塞，可延后）

- [ ] 7.1 选定域名并完成注册（`.com` 推荐，约 75 元/年）
- [ ] 7.2 在 Vercel Dashboard → Settings → Domains 添加自定义域名
- [ ] 7.3 在域名服务商添加 CNAME 记录指向 `cname.vercel-dns.com`
- [ ] 7.4 等 SSL 证书自动签发（几分钟到几小时）
- [ ] 7.5 回到 [src/index.html](src/index.html)，把 `og:url` 从 `xxx.vercel.app` 改为最终域名
- [ ] 7.6 触发一次新部署让 OG 更新生效
- [ ] 7.7 验证 `https://<自定义域名>` 可访问且与默认域名一致
