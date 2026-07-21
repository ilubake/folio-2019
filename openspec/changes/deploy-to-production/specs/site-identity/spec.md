## ADDED Requirements

### Requirement: 站点标题与描述归属正确

[src/index.html](src/index.html) 的 `<title>` 与 `meta[name=description]`、`meta[itemprop=name]`、`meta[itemprop=description]` SHALL 显示站点所有者本人的品牌名与简介，MUST NOT 残留 `Bruno Simon` 或 `bruno-simon.com`。

#### Scenario: 浏览器标签页显示自有品牌

- **WHEN** 用户在浏览器打开线上站点
- **THEN** 浏览器标签页标题显示站点所有者配置的品牌名，而非 `Bruno Simon`

#### Scenario: 搜索引擎抓取到自有描述

- **WHEN** 搜索引擎爬虫读取页面 `meta[name=description]`
- **THEN** 返回站点所有者自定义的简介文本，不包含 `Creative developer living in Paris` 等原作者描述

### Requirement: 社交分享卡片归属正确

Open Graph 与 Twitter Card 的 `url`、`title`、`description`、`image` 字段 SHALL 指向站点所有者的域名与品牌，分享到微信 / Twitter / Telegram 等平台时 MUST 显示自有预览。

#### Scenario: 分享到社交平台显示自有预览

- **WHEN** 用户把线上 URL 粘贴到微信 / Twitter / Telegram
- **THEN** 预览卡片显示站点所有者配置的标题、描述、分享图，且 `og:url` 指向最终域名而非 `https://bruno-simon.com`

#### Scenario: 缺少自定义域名时的降级

- **WHEN** 站点尚未绑定自定义域名，仅使用 `xxx.vercel.app`
- **THEN** `og:url` 临时指向 `xxx.vercel.app`，并在绑定域名后通过一次构建更新为最终域名

### Requirement: 原作者数据分析追踪已清除

[src/index.html](src/index.html) MUST NOT 包含原作者的 Google Analytics ID `UA-3966601-5`，相关的 `googletagmanager.com/gtag/js` 引用与 `gtag('config', ...)` 调用 SHALL 被删除或替换为站点所有者自己的追踪 ID。

#### Scenario: 线上不再向原作者 GA 发送数据

- **WHEN** 用户访问线上站点并触发页面浏览
- **THEN** 浏览器网络请求中不出现 `googletagmanager.com/gtag/js?id=UA-3966601-5`，原作者 GA 后台不会收到此次访问

#### Scenario: 选择不接入数据分析时不报错

- **WHEN** GA 两段 `<script>` 被完整删除
- **THEN** 页面其他功能（3D 场景加载、音频、交互）不受影响，控制台无相关报错

### Requirement: Favicon 与 PWA 元数据归属正确

`<link rel="apple-touch-icon">`、`<link rel="manifest">`、`apple-mobile-web-app-title`、`application-name` 等元数据 SHALL 指向站点所有者的 favicon 资源与品牌名。

#### Scenario: 浏览器收藏夹显示自有图标

- **WHEN** 用户把站点添加到收藏夹或主屏幕
- **THEN** 显示站点所有者配置的 favicon 与应用名，而非 `Bruno Simon`
