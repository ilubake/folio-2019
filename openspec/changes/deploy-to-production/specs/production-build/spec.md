## ADDED Requirements

### Requirement: 生产构建不输出 sourcemap

[vite.config.js](vite.config.js) 的生产构建配置 SHALL 关闭 `sourcemap`（设为 `false` 或省略），`dist/` 目录 MUST NOT 包含 `.map` 文件，避免源码在线上被还原。

#### Scenario: 生产构建产物不含 sourcemap

- **WHEN** 执行 `npm run build`
- **THEN** `dist/` 目录及其子目录中不存在任何 `.js.map`、`.css.map` 文件，且浏览器 DevTools 的 Sources 面板无法还原原始 ESM 源码

#### Scenario: 开发模式仍保留 sourcemap

- **WHEN** 执行 `npm run dev`
- **THEN** 开发体验不受影响，DevTools 仍能定位到原始源码行号

### Requirement: 构建产物目录结构符合静态托管预期

`npm run build` SHALL 在项目根目录生成 `dist/` 目录（而非 `src/dist/` 或其他位置），且 `dist/index.html` 作为入口可直接被静态文件服务器托管，无需 rewrite 规则。

#### Scenario: 构建产物在项目根 dist 目录

- **WHEN** 执行 `npm run build`
- **THEN** 项目根出现 `dist/` 目录，其中包含 `index.html`、`assets/`、`draco/`、`models/`、`sounds/`、`favicon/`、`social/` 等子项

#### Scenario: 静态服务器直接托管 dist

- **WHEN** 把 `dist/` 目录作为 web root 启动任意静态服务器（如 `npx serve dist`）
- **THEN** 浏览器访问根路径可正常加载 3D 场景，控制台无 404

### Requirement: 本地预览通过上线前验证清单

`npm run build` 之后 SHALL 使用静态文件服务器本地预览 `dist/`，并逐项通过"3D 场景、模型、音频、Draco、favicon、分享图、控制台无报错"的验证清单，然后才允许推送部署。

#### Scenario: 本地预览通过全部验证项

- **WHEN** 在本地对 `dist/` 运行 `npx vite preview --outDir dist` 并逐项核对验证清单
- **THEN** 3D 场景加载、`.glb` 模型无 404、`.mp3` 音频可播放、控制台无 `draco_decoder` 相关 404、favicon 与分享图无 404、无 CORS 或模块加载错误

#### Scenario: 本地验证未通过时不部署

- **WHEN** 本地预览任一验证项失败（如模型 404、控制台报错）
- **THEN** 阻断推送部署，必须先修复并重新构建、重新验证

### Requirement: 生产构建产物体积可观测

构建过程或产物 SHALL 提供一种方式查看 `dist/` 总体积与单文件大小，便于在资源显著增长时及时介入优化。

#### Scenario: 查看构建产物体积

- **WHEN** 构建完成
- **THEN** 可通过 `du -sh dist` 或等价命令查看总体积，且单文件最大者不超过 5 MB（当前基线为 0.6 MB）
