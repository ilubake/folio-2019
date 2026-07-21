## ADDED Requirements

### Requirement: 项目托管在远程 Git 仓库

项目 SHALL 初始化 Git 并推送到一个远程仓库（GitHub / GitLab / Gitee 之一），远程仓库 MUST 排除 `node_modules/`、`dist/`、`.env*`、本地缓存等不应公开的内容。

#### Scenario: 首次提交干净

- **WHEN** 在干净检出（fresh clone）的远程仓库上执行 `npm install && npm run build`
- **THEN** 构建可以成功，且仓库内不包含 `node_modules/`、`dist/` 目录

#### Scenario: 推送触发自动部署

- **WHEN** 本地代码推送到远程仓库默认分支（`master` 或 `main`）
- **THEN** 托管平台（Vercel）自动触发一次新的部署

### Requirement: 通过 Vercel 自动部署

项目 SHALL 通过 Vercel 的 Vite 预设部署：构建命令 `npm run build`、输出目录 `dist`、Node 版本 ≥ 18。部署过程 MUST NOT 需要新增 `vercel.json` 等平台特定配置文件。

#### Scenario: 首次部署成功

- **WHEN** 在 Vvercel Dashboard 导入 Git 仓库并保留默认 Vite 预设
- **THEN** 部署在 3 分钟内成功完成，生成可访问的 `xxx.vercel.app` 默认域名

#### Scenario: 推送变更自动重新部署

- **WHEN** 推送新的 commit 到默认分支
- **THEN** Vercel 自动触发新部署，完成后 `xxx.vercel.app` 反映最新代码

### Requirement: 线上站点通过 HTTPS 访问

默认域名 `xxx.vercel.app` SHALL 通过 HTTPS 提供服务，HTTP 请求 SHALL 自动重定向到 HTTPS，证书由 Vercel 自动签发与续期。

#### Scenario: HTTPS 默认启用

- **WHEN** 浏览器访问 `https://xxx.vercel.app`
- **THEN** 页面正常加载，浏览器地址栏显示安全锁标志，证书有效

#### Scenario: HTTP 自动跳转 HTTPS

- **WHEN** 浏览器访问 `http://xxx.vercel.app`
- **THEN** 自动重定向到对应的 HTTPS URL，不暴露明文流量

### Requirement: 线上版本通过上线后检查清单

首次部署完成后 SHALL 逐项核对线上检查清单：HTTPS、移动端可访问、控制台无 404/CORS、favicon、OG 分享预览、标题与描述已替换、3D 模型加载速度可接受。

#### Scenario: 线上检查清单全部通过

- **WHEN** 在浏览器中访问线上 `xxx.vercel.app`
- **THEN** HTTPS 正常、控制台无 404 或 CORS 错误、favicon 显示正确、页面标题与描述已替换为站点所有者的品牌

#### Scenario: 检查未通过时回滚

- **WHEN** 任一检查项未通过（如控制台报 Draco 404）
- **THEN** 通过 Vercel Dashboard 一键回滚到上一个稳定部署，并在本地修复后再推送

### Requirement: 支持后续绑定自定义域名

部署 SHALL 预留自定义域名接入能力：Vercel 项目可添加自定义域名并自动签发 SSL，无需改动代码或重新构建。

#### Scenario: 绑定自定义域名后可访问

- **WHEN** 在 Vercel Dashboard 添加自定义域名并在域名服务商配置 CNAME 指向 `cname.vercel-dns.com`
- **THEN** SSL 证书在几分钟内自动签发，`https://<自定义域名>` 可访问且与 `xxx.vercel.app` 内容一致

#### Scenario: 绑定域名不影响默认域名

- **WHEN** 自定义域名接入完成
- **THEN** `xxx.vercel.app` 默认域名继续可访问，作为兜底备用入口
