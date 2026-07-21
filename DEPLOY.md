# Folio 2019 线上部署流程细则

> 本文档面向首次部署该项目的开发者，覆盖从本地准备到线上上线的完整流程。
> 项目类型：纯静态 SPA（Three.js + Vite），无后端、无数据库、无服务端渲染。

---

## 一、项目技术画像（部署前必须了解）

| 维度 | 说明 |
|------|------|
| 构建工具 | Vite 5 |
| 核心依赖 | three.js、cannon（物理）、gsap（动画）、howler（音频） |
| 着色器 | 通过 `vite-plugin-glsl` 引入 `.glsl` 文件 |
| 入口文件 | [src/index.html](src/index.html) |
| 静态资源目录 | [static/](static/)（包含 3D 模型、音频、Draco 解码器、favicon、分享图） |
| 构建产物 | `dist/`（配置见 [vite.config.js](vite.config.js#L12-L17)） |
| 路由形态 | 单页面，无需服务端 rewrite |
| 资源体积 | 较大（包含 .glb 3D 模型 + .mp3 音频），必须上 CDN |

**部署本质**：把 `dist/` 目录内的所有文件托管到任意静态文件服务器即可。

---

## 二、部署前置准备（上线前必做清单）

### 2.1 替换原作者相关的追踪 / SEO 信息

[src/index.html](src/index.html) 中存在以下需要修改的内容，否则线上数据会发给原作者、分享卡片显示错误：

| 位置 | 原值 | 改为 |
|------|------|------|
| `<title>` | `Bruno Simon` | 你的名字 / 站点名 |
| `meta[name=description]` | 原作者简介 | 你的简介 |
| `meta[property=og:url]` | `https://bruno-simon.com` | 你的最终域名 |
| `meta[property=og:image]` | `https://bruno-simon.com/social/...` | 你的域名 + 同路径（图片可沿用 [static/social/](static/social/)） |
| `gtag('config', 'UA-3966601-5')` | 原作者的 GA ID | **你的 GA4 / G-ID，或整段删除** |
| `googletagmanager` 的 `id=UA-3966601-5` | 同上 | 同上 |

> 如暂不做数据分析，可直接删除 [src/index.html:88-94](src/index.html#L88-L94) 的两段 `<script>`，不影响功能。

### 2.2 关闭生产环境 sourcemap（可选但建议）

[vite.config.js:16](vite.config.js#L16) 的 `sourcemap: true` 会把源码映射一起发布，体积更大、代码可被还原。生产环境建议：

```js
sourcemap: false
```

### 2.3 准备 Git 仓库（所有方案都需要）

```bash
git init
git add .
git commit -m ":tada: Initial commit"
```

再推送到 GitHub / GitLab / Gitee 其中之一（后续方案会用到）。

> 提交前确认 [.gitignore](.gitignore) 已排除 `node_modules/` 与 `dist/`。

### 2.4 Node 版本

本地使用 Node.js 18 LTS 或 20 LTS（Vite 5 最低要求 Node 18）。

---

## 三、本地构建验证（上線前最后一道关卡）

```bash
npm install
npm run build
```

构建成功后会在项目根目录生成 `dist/`。

**本地预览 dist 产物**（最接近线上效果）：

```bash
npx serve dist
# 或
npx vite preview --outDir dist
```

浏览器打开后必须逐一确认：

- [ ] 3D 场景能正常加载
- [ ] 3D 模型（.glb）没有 404
- [ ] 音效（.mp3）能播放
- [ ] Draco 解码器路径正确（控制台无 `draco_decoder` 相关 404）
- [ ] favicon、分享图无 404
- [ ] 控制台没有 CORS / 模块加载错误

若控制台报 Draco 相关错误，检查 [src/javascript/Utils/Loader.js](src/javascript/Utils/Loader.js) 中 DRACOLoader 的路径配置，确保指向 `/draco/`。

---

## 四、部署方案选型

| 方案 | 费用 | 全球 CDN | 国内访问 | 自定义域名 | 推荐场景 |
|------|------|---------|---------|-----------|---------|
| **Vercel** | 免费 | 是 | 中等 | 是（免费 SSL） | 首选，开箱即用 |
| **Cloudflare Pages** | 免费 | 是 | 中等偏好 | 是（免费 SSL） | 首选替代，CDN 更广 |
| **Netlify** | 免费 | 是 | 中等 | 是 | 备选 |
| **GitHub Pages** | 免费 | 是 | 较差 | 是 | 仅 demo |
| **自有服务器 + Nginx** | 需购买 VPS | 需自行接 CDN | 看服务器位置 | 是 | 完全自主 |

> 面向中国大陆用户的访问场景，建议选 **Cloudflare Pages + 备案域名** 或 **国内 VPS + Nginx**。

下面给出前四种方案的完整操作步骤，任选其一。

---

## 五、成本估算与预算规划

> 本章把所有隐性成本摆到台面上：免费方案的真实限额、付费方案的年支出、以及三档典型预算方案。

### 5.1 资源体量估算（成本计算的依据）

| 资源类型 | 单文件典型大小 | 数量 | 小计 |
|---------|--------------|------|------|
| 压缩后的 JS bundle | 200–500 KB | 1 | ~400 KB |
| 3D 模型 `.glb`（Draco 压缩） | 0.5–5 MB | 多个 | 5–20 MB |
| 音频 `.mp3` | 50–500 KB | 多个 | 0.5–2 MB |
| 分享图、favicon 等 | < 200 KB | 若干 | < 1 MB |
| **单用户首屏总流量** | | | **约 8–25 MB** |

> 实际数字以 `npm run build` 后的 `dist/` 目录大小为准，可用 `du -sh dist` 查看。

### 5.2 免费方案的真实限额（务必先看）

| 平台 | 免费流量/月 | 免费构建次数 | 单文件大小上限 | 超限后果 |
|------|------------|-------------|--------------|---------|
| **Vercel Hobby** | 100 GB 带宽 | 6000 分钟构建 | 100 MB | 超带宽会限速 / 提示升级 Pro |
| **Cloudflare Pages** | **不限带宽**（公平使用） | 500 次/月 | 25 MB | 超单文件限制直接构建失败 |
| **Netlify Free** | 100 GB 带宽 | 300 分钟构建 | 25 MB | 超限触发计费警告 |
| **GitHub Pages** | 100 GB 带宽 + 10 GB/仓 | 公开仓库不限 | 25 MB | 超限封仓库 |

**临界点测算**（按首屏 15 MB 估算）：

- 100 GB 带宽 ≈ **约 6600 次完整访问/月**，日均 220 次。
- 个人作品集通常远低于此，免费方案完全够用。
- 若日均访问 > 500 次，或被社交平台引爆，建议提前接 CDN 或升级套餐。

### 5.3 域名费用（年付）

| 后缀 | 首年价 | 续费价 | 推荐场景 |
|------|-------|-------|---------|
| `.com` | 60–90 元 | 70–90 元 | 通用首选 |
| `.cn` | 20–35 元 | 35–50 元 | 国内备案必选之一 |
| `.me` / `.io` / `.dev` | 100–300 元 | 150–350 元 | 个人品牌，有逼格 |
| `.top` / `.xyz` | 5–15 元 | 30–60 元 | 便宜但 SEO 偏差 |

> 阿里云、腾讯云、Cloudflare、Namecheap、Porkbun 都可注册，**Cloudflare Registrar 续费零加价**最划算（国际域名）。

### 5.4 VPS 费用（年付）

| 配置 | 国内云（阿里/腾讯） | 海外（Vultr/Linode/Hetzner） | 适合 |
|------|------------------|----------------------------|-----|
| 1 核 1G / 20G SSD | 70–120 元/年（活动价） | 35–60 元/月 | 仅静态站 |
| 2 核 2G / 40G SSD | 200–400 元/年 | 70–120 元/月 | 静态站 + CI |
| 2 核 4G / 60G SSD | 400–800 元/年 | 120–200 元/月 | 多服务并存 |

> **Hetzner**（德国）性价比最高，CX22 约 4.5 欧/月起；国内服务器**必须备案**才能用 80/443 端口。

### 5.5 CDN 费用（按量计费）

如果站点面向国内大量用户、或资源 > 20 MB、或希望全球加速：

| 服务商 | 计费方式 | 单价（中国大陆） | 1 TB 流量包 |
|-------|---------|---------------|-----------|
| 阿里云 CDN | 按流量 / 按带宽 | 0.18–0.24 元/GB | 约 180–240 元 |
| 腾讯云 CDN | 按流量 / 按带宽 | 0.18–0.21 元/GB | 约 180–210 元 |
| 又拍云 / 七牛云 | 按流量 | 0.18–0.29 元/GB | 约 180–290 元 |
| Cloudflare | 免费版无限流量 | 0 | 0（国内回源差） |

**典型场景**：日均 1000 PV × 15 MB = 15 GB/天 ≈ 450 GB/月 → 月成本约 **80–110 元**（国内 CDN）。

### 5.6 对象存储费用（可选，用于大模型分离）

把 3D 模型从 `dist/` 拆出放到 OSS，可大幅减少每次部署的体积、加速回源：

| 服务商 | 存储费 | 流量费 | 典型月费（1 GB 模型 + 100 GB 流量） |
|-------|-------|-------|--------------------------------|
| 阿里云 OSS | 0.12 元/GB/月 | 0.5 元/GB（外网） | 约 50 元 |
| 腾讯云 COS | 0.12 元/GB/月 | 0.5 元/GB（外网） | 约 50 元 |
| Cloudflare R2 | $0.015/GB/月 | **出站免费** | 约 ¥1（仅存储） |

> **强烈推荐 Cloudflare R2** 存放大模型，出站流量免费，配合 Cloudflare Pages 几乎零成本。

### 5.7 备案成本（仅中国大陆服务器）

| 项目 | 成本 | 说明 |
|------|------|------|
| ICP 备案工本费 | 0 元 | 信息服务商免费办理 |
| 域名持有证明 | 已含 | 备案要求域名持有者与主体一致 |
| 时间成本 | 5–20 个工作日 | 各省审核速度不同 |
| 主体性质 | 个人 / 企业 | 个人备案较简单，不能涉及经营性内容 |

> 部分省份要求服务器购买 ≥ 3 个月，且要关闭网站直到备案通过。

### 5.8 SSL 证书费用

| 来源 | 费用 | 有效期 | 推荐度 |
|------|------|-------|-------|
| Let's Encrypt | **免费** | 90 天，自动续期 | ⭐⭐⭐⭐⭐ |
| Vercel / Netlify / Cloudflare 自带 | **免费** | 自动续 | ⭐⭐⭐⭐⭐ |
| 阿里云 / 腾讯云免费 DV | **免费** | 1 年，需手动续 | ⭐⭐⭐ |
| 企业 OV / EV 证书 | 500–5000 元/年 | 1–2 年 | 仅企业需要 |

**结论**：个人作品集**完全不必为 SSL 花一分钱**。

### 5.9 三档典型预算方案

#### 方案 1：零成本（个人 Demo / 短期展示）

| 项目 | 成本 |
|------|------|
| 托管：Cloudflare Pages | 0 元 |
| 域名：使用 `xxx.pages.dev` 默认域名 | 0 元 |
| SSL：自带 | 0 元 |
| **总计** | **0 元/年** |

适合：简历投递、给朋友看、短期分享。

#### 方案 2：基础自有域名（个人作品集，年均访问 < 5000）

| 项目 | 成本 |
|------|------|
| 托管：Cloudflare Pages / Vercel | 0 元 |
| 域名：`.com` | 75 元/年 |
| SSL：自带 | 0 元 |
| **总计** | **约 75 元/年** |

适合：长期个人品牌站、GitHub 主页挂的链接、海外用户为主。

#### 方案 3：国内访问优化（面向国内用户 + 自有域名）

| 项目 | 成本 |
|------|------|
| 域名：`.cn` 或 `.com`（已备案） | 50–90 元/年 |
| VPS：2 核 2G（国内云） | 200–400 元/年 |
| CDN：按量 | 100–300 元/年（视流量） |
| 对象存储（可选）：OSS 1 GB + 100 GB 流量 | 50 元/年 |
| SSL：Let's Encrypt | 0 元 |
| 备案 | 0 元（时间成本 1–3 周） |
| **总计** | **约 350–840 元/年** |

适合：简历、商业展示、希望国内访问体验良好的长期站点。

### 5.10 隐性 / 易忽略成本清单

- **构建分钟数**：Vercel / Netlify 免费档构建有限额，频繁提交 CI 会超。
- **回源流量**：CDN → 源站的回源流量也会计费，缓存命中率低时翻倍。
- **跨可用区流量**：云厂商内某些区域间传输要钱。
- **域名隐私保护**：部分注册商收费（Cloudflare、Porkbun 免费）。
- **续费陷阱**：首年 9 元续费 90 元的域名，按 3 年 / 5 年总成本对比。
- **国内 CDN 流量包过期**：买的包通常 1 年有效，未用完作废。

### 5.11 成本决策建议

- **先免费、后付费**：先用 Cloudflare Pages 零成本跑起来，等流量真实增长再投 CDN。
- **资源往 R2 / OSS 拆**：大模型放对象存储后，每次部署只推代码差异，构建和回源都更便宜。
- **海外用户优先 Cloudflare，国内用户优先阿里 / 腾讯 CDN**：别用 Cloudflare 兜国内用户。
- **警惕"永久免费"陷阱**：平台政策可能调整，关键站点要有随时迁移到自有服务器的能力（这正是本项目选纯静态的最大好处）。

---

## 六、方案 A：Vercel 部署（推荐）

### 6.1 通过 Dashboard

1. 注册 / 登录 [vercel.com](https://vercel.com)（用 GitHub 账号最快）。
2. 点 `Add New Project` → 导入你在 [2.3](#23-准备-git-仓库所有方案都需要) 推送的仓库。
3. Framework Preset 选 **Vite**。
4. **Build & Output Settings**（关键，必须改）：
   - Build Command: `npm run build`
   - Output Directory: `dist`（**不是默认的 `dist`，需确认**）
   - Install Command: `npm install`
5. 点 `Deploy`，等待 1-2 分钟。
6. 部署完成后获得 `xxx.vercel.app` 临时域名，可直接访问。

### 6.2 通过 Vercel CLI（无需 GitHub）

```bash
npm i -g vercel
vercel login
vercel              # 首次部署，按提示选项目
vercel --prod       # 后续正式发布
```

### 6.3 注意事项

- Vite 的 `root` 是 `src/`，但 Output Directory 仍是项目根的 `dist/`（见 [vite.config.js:14](vite.config.js#L14)），配置时别填错。
- 每次推送到主分支会自动触发部署。

---

## 七、方案 B：Cloudflare Pages 部署

1. 登录 [dash.cloudflare.com](https://dash.cloudflare.com) → Workers & Pages → Create。
2. 选 `Connect to Git` → 授权并选仓库。
3. 构建配置：
   - Framework preset: **Vite**
   - Build command: `npm run build`
   - Build output directory: `dist`
   - Root directory: 留空（因为 vite.config.js 在根目录）
4. 点 `Save and Deploy`，首次约 2-3 分钟。
5. 后续 push 自动部署。

> Cloudflare Pages 对大文件（>25MB 单文件）有限制，若 3D 模型超过请拆分或使用外部对象存储。

---

## 八、方案 C：Netlify 部署

1. 登录 [app.netlify.com](https://app.netlify.com) → Add new site → Import from Git。
2. Build settings：
   - Build command: `npm run build`
   - Publish directory: `dist`
3. Deploy site。

可选：项目根目录新增 `netlify.toml`：

```toml
[build]
  command = "npm run build"
  publish = "dist"
```

---

## 九、方案 D：GitHub Pages 部署

> ⚠️ 项目源码在 `src/`，默认 GH Pages 不会识别，需借助 GitHub Actions。

### 9.1 新增 workflow 文件

在项目根创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [master]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### 9.2 仓库配置

1. 进入仓库 `Settings` → `Pages` → `Source` 选 **GitHub Actions**。
2. push 到 master，等 Actions 跑完即可在 `https://<username>.github.io/<repo>/` 访问。

### 9.3 路径注意

若部署在 `https://<username>.github.io/<repo>/` 这种子路径下，资源可能 404。需要在 [vite.config.js](vite.config.js) 添加 `base`：

```js
export default {
    base: '/<repo>/',
    // 其余保持不变
}
```

---

## 十、方案 E：自有 VPS + Nginx（完全自主）

### 10.1 服务器准备

- 一台 Linux VPS（Ubuntu 22.04 LTS 为例）。
- 安装 Node 20、Git、Nginx。

### 10.2 上传构建产物

本地构建后上传 `dist/`：

```bash
npm run build
scp -r dist/* user@server-ip:/var/www/folio/
```

或在服务器上 clone + build：

```bash
ssh user@server-ip
git clone <your-repo> /srv/folio
cd /srv/folio
npm install
npm run build
```

### 10.3 Nginx 配置

新建 `/etc/nginx/conf.d/folio.conf`：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/folio;     # 或 /srv/folio/dist
    index index.html;

    # gzip 压缩（3D 模型除外，模型本身已压缩）
    gzip on;
    gzip_types text/plain text/css application/javascript application/json image/svg+xml;
    gzip_min_length 1024;

    # 静态资源强缓存（hash 文件名可长期缓存）
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 模型、音频中等缓存
    location ~* \.(glb|gltf|mp3|wav|png|jpg|svg|webp)$ {
        expires 7d;
        add_header Cache-Control "public";
    }

    # SPA fallback
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

重载：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### 10.4 申请 SSL（免费）

```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

自动续期已内置。

---

## 十一、绑定自定义域名

无论选 A/B/C/E 方案，流程一致：

1. 在域名服务商（阿里云 / 腾讯云 / Cloudflare / Namecheap）添加解析记录：
   - **Vercel / Netlify**：加一条 `CNAME` 记录，值为平台给出的 `xxx.vercel.app` / `xxx.netlify.app`。
   - **Cloudflare Pages**：加 `CNAME` 到 `xxx.pages.dev`。
   - **自有服务器**：加 `A` 记录，值为服务器公网 IP。
2. 回到平台后台 → Domains → Add domain，输入你的域名。
3. 等 SSL 证书自动签发（通常几分钟），访问 `https://你的域名` 验证。

> **中国大陆服务器** 必须先完成 **ICP 备案** 才能用自有域名访问 80/443 端口，备案期间可用 IP 临时访问。

---

## 十二、上线后检查清单

部署完成、域名能打开后，依次核对：

- [ ] HTTPS 正常（浏览器地址栏小锁）
- [ ] 移动端可访问（3D 场景性能可能差，这是预期）
- [ ] 控制台无 404 / CORS 报错
- [ ] favicon 正常显示
- [ ] 分享链接到微信/Twitter/Telegram 的预览图正确（OG 标签生效）
- [ ] Lighthouse 跑一次（关注 Performance / Accessibility 分数）
- [ ] 页面标题、描述已替换为你自己的
- [ ] GA / 数据分析已替换或移除
- [ ] 3D 模型加载速度可接受（首屏 < 5MB 为佳，否则考虑 CDN）

---

## 十三、常见问题（FAQ）

**Q1：线上 3D 模型加载失败，控制台报 Draco 相关错误？**
A：确认 [static/draco/](static/draco/) 目录已正确拷贝到 `dist/draco/`。检查 Loader 中 `DRACOLoader.setDecoderPath('/draco/')` 是否指向正确路径。

**Q2：部署后资源 404？**
A：99% 是 Output Directory 填错。Vite 配置中 outDir 是 `../dist`（相对 `src/`），最终产物在**项目根**的 `dist/`，不是 `src/dist/`。

**Q3：GitHub Pages 下子路径资源 404？**
A：见 [9.3](#93-路径注意)，需要在 vite.config.js 配 `base`。

**Q4：构建报错 `glsl` 相关？**
A：确认 `npm install` 完整执行，检查 [vite.config.js:20](vite.config.js#L20) 的 `glsl()` 插件未被注释。

**Q5：国内访问慢？**
A：3D 场景首屏资源大，建议：
1. 域名走国内 CDN（阿里云 / 腾讯云 CDN）；
2. 或把大模型放对象存储（OSS / COS / S3）+ CDN，代码中以 URL 引用；
3. 或部署在国内 VPS 并完成备案。

**Q6：如何更新线上内容？**
A：本地改完 → `git push` → Vercel/Cloudflare/Netlify 自动重新部署 → 等 1-2 分钟生效。GitHub Actions 同理。自有服务器需手动 pull+build 或配 CI。

**Q7：免费额度会突然用完吗？**
A：极低概率。个人作品集日均访问通常 < 100，免费档（100 GB / 月）足够。若被社交平台引爆，平台会邮件提醒，不会自动扣费。

---

## 十四、推荐路径总结

如果**第一次部署、想最快上线**：

> Git 仓库 → Vercel → 用默认 `xxx.vercel.app` 域名访问 → 验证无误后再绑自有域名。

如果**面向国内用户**：

> 备案域名 → 国内云厂商 CDN → 源站用 Cloudflare Pages 或国内 VPS。

如果**预算几乎为零**：

> Cloudflare Pages + Cloudflare R2 存大模型 + `xxx.pages.dev` 默认域名 → **零元长期跑**。

---

文档至此结束，按章节顺序执行即可完成部署。遇特殊问题优先看 **十三、FAQ**，成本问题看 **五、成本估算**。
