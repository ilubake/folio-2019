# 本地启动文档

本文档记录在本地运行 `folio-2019`（Bruno Simon 创意作品集）项目的完整流程。

## 环境要求

| 工具 | 版本 |
| --- | --- |
| Node.js | ≥ 18（推荐 v20.11.1） |
| npm | ≥ 9 |

下载地址：<https://nodejs.org/en/download/>

## 技术栈

- **构建工具**：Vite 5 + `vite-plugin-glsl` + `vite-plugin-restart`
- **3D 渲染**：three.js 0.164
- **物理引擎**：cannon 0.6
- **动效**：gsap 3.12
- **音频**：howler 2.2
- **调试**：dat.gui

## 安装依赖

在项目根目录执行：

```bash
npm install
```

## 启动开发服务器

```bash
npm run dev
```

启动成功后会输出：

```
VITE v5.2.11  ready in 919 ms

➜  Local:   http://localhost:5173/
➜  Network: http://<局域网IP>:5173/
```

浏览器会自动打开根目录配置中 `open: true` 的行为；若未自动打开，请手动访问 <http://localhost:5173/>。

> Vite 默认端口为 5173，而非旧 readme 中提到的 1234。原 readme 是 Parcel 时代的遗留说明，实际行为以 [vite.config.js](vite.config.js) 为准。

## 生产构建

```bash
npm run build
```

构建产物会输出到项目根目录的 `dist/`，并开启 sourcemap，清空旧产物。

## 配置要点（[vite.config.js](vite.config.js)）

| 配置 | 说明 |
| --- | --- |
| `root` | `src/`（入口 `index.html` 在 src 下） |
| `publicDir` | `../static/`（静态资源原样服务） |
| `server.host` | `true`（暴露到局域网） |
| `server.open` | 非 CodeSandbox 环境下自动打开浏览器 |
| `build.outDir` | `../dist` |
| `plugins.glsl` | 支持 `.glsl` 着色器文件 import |
| `plugins.restart` | `static/**` 变化时重启 dev server |

## 常见问题

- **端口占用**：若 5173 被占用，Vite 会自动顺延到 5174/5175，以终端输出为准。
- **修改 `static/` 下资源未生效**：该目录由 `vite-plugin-restart` 监听，会触发 server 整体重启，稍等片刻即可。
- **修改 `src/` 代码**：原生 HMR 热更新，无需手动刷新。
- **依赖安装慢**：可切换 npm 镜像：`npm config set registry https://registry.npmmirror.com`。
