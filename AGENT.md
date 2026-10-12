# AGENT.md

本文件面向在本仓库中工作的 AI 编码代理（agent），说明项目结构、约定与工作流程。

## 项目简介

**AI MISC** 是一个纯前端的「工具箱」合集：每个工具都是一个独立的、可直接打开的单页应用（SPA），使用原生 HTML + JavaScript 编写，样式依赖 Tailwind CSS CDN。所有处理均在浏览器本地完成，不涉及后端。

- 主页（工具索引）：`docs/index.html`
- 线上地址：Cloudflare Pages（`npm run deploy`）
- 每个工具通常还有一个规范化网址 `https://misc.xplanc.org/<tool-name>`

## 目录结构

```
ai-misc/
├─ docs/                 # 网站根目录（部署目录）
│  ├─ index.html         # 主页，展示并搜索所有工具
│  ├─ <tool-name>.html   # 每个工具一个 HTML 文件
│  ├─ vendor/            # 本地化的第三方库（离线可用，见第 3 节）
│  └─ ...
├─ package.json          # 仅含 wrangler（部署）脚本
├─ README.md             # 面向用户的工具列表
├─ AGENT.md              # 本文件
└─ .gitignore            # 忽略 node_modules、.wrangler
```

## 核心约定

### 1. 每个工具就是一个自包含的 HTML 文件

- 文件放在 `docs/` 下，命名为**小写、短横线分隔**，例如 `five-in-a-row.html`、`pdf-to-png.html`。
- 单个文件内包含全部 HTML / CSS / JS，**不引入构建步骤、不使用打包器**；第三方库不使用 CDN，统一放在 `docs/vendor/` 下并以相对路径引用（见第 3 节）。
- `<html lang="zh-CN">`，界面文案使用中文。
- 应用标题可中英混排（参考 `docs/random.html` 的 `RANDOM TOOL` 风格）。
- 每个页面都必须在 `<head>` 内包含 canonical 链接：`<link rel="canonical" href="https://misc.xplanc.org/<tool-name>"/>`（文件名去掉 `.html`；主页 `index.html` 用根路径 `https://misc.xplanc.org/`）。

### 2. 样式使用 Tailwind CSS（本地文件）

仓库中两种构建方式并存，均以本地文件引入，新工具请优先沿用较新的写法：

```html
<!-- 较新：Tailwind v4 browser build（参考 linux-cmd.html） -->
<script src="vendor/tailwindcss/browser-4.js"></script>

<!-- 旧版 v3（index.html / qrcode.html / random.html 仍在使用） -->
<script src="vendor/tailwindcss/cdn-3.js"></script>
```

- 复杂或需要精细控制的样式可写在 `<style>` 内，但整体布局优先用 Tailwind 的 utility class。
- 图标使用 Font Awesome：`vendor/font-awesome/7.0.1/css/all.min.css`（字体文件同目录下的 `webfonts/`）。

### 3. 第三方库统一放在 `docs/vendor/`

**所有外部依赖都已本地化到 `docs/vendor/`，页面通过相对路径 `vendor/...` 引用，以保证离线可用。不要直接引用 CDN。** 现有依赖：

| 库 | 本地路径 |
| --- | --- |
| Tailwind v4 browser | `vendor/tailwindcss/browser-4.js` |
| Tailwind v3 CDN | `vendor/tailwindcss/cdn-3.js` |
| Font Awesome | `vendor/font-awesome/7.0.1/css/all.min.css`（+ `webfonts/`） |
| crypto-js | `vendor/crypto-js/4.2.0/crypto-js.min.js` |
| JSZip | `vendor/jszip/3.10.1/jszip.min.js` |
| pdf.js 2.x | `vendor/pdf.js/2.16.105/pdf.min.js`（+ `pdf.worker.min.js`） |
| pdf.js 3.x | `vendor/pdf.js/3.11.174/pdf.min.js`（+ `pdf.worker.min.js`） |
| pdf-lib | `vendor/pdf-lib/1.17.1/pdf-lib.min.js` |
| qrcode | `vendor/qrcode/1.5.1/qrcode.min.js` |
| jsQR | `vendor/jsqr/1.4.0/jsQR.min.js` |
| uuid | `vendor/uuid/8.3.2/uuid.min.js` |
| html2canvas | `vendor/html2canvas/1.4.1/html2canvas.min.js` |

新增依赖时：把文件下载到 `docs/vendor/<库>/<版本>/`，保持版本号目录，再在页面里用相对路径引用。

使用 pdf.js 时别忘了设置 worker（同样指向本地文件）：

```js
pdfjsLib.GlobalWorkerOptions.workerSrc = 'vendor/pdf.js/3.11.174/pdf.worker.min.js';
```

注意事项：

- **jsDelivr 主站（`cdn.jsdelivr.net`）在部分网络下会被重置**，下载时可用镜像 `fastly.jsdelivr.net`（路径一致）；能 `cdnjs` / `unpkg` 获取的优先用它们。
- **Blob Worker 里的 `importScripts` 无法解析相对路径**，需先用 `new URL('vendor/...', document.baseURI).href` 算出绝对地址再注入（参考 `pdf-to-png.html` 的 `createWorker`）。
- 字体等带相对 `url()` 的 CSS，要保持目录结构（CSS 与 `webfonts/` 的相对位置不变）。

### 4. 深色模式与国际化（新工具建议支持）

部分较新的工具（`qrcode.html`、`pdf-*.html`、`salary.html`、`budget.html`、`mars-converter.html`、`periodic-table.html`、`jwt.html` 等）已支持深色模式与中英切换。实现约定：

- 深色模式：Tailwind `darkMode: 'class'`，在 `<body>` 及元素上使用 `dark:` 变体。
- 国际化：定义 `const i18n = { zh: {...}, en: {...} }`，通过 `const t = i18n[lang]` 取文案；需翻译的元素加 `t-` 前缀的 `id`（如 `id="t-subtitle"`）。

**默认值必须跟随系统**（页面参数/URL 仅在显式指定时覆盖）：

```js
const params = new URLSearchParams(location.search);

// 语言：?language= 优先，否则按系统语言（zh* → zh，其余 → en）
let lang = params.get('language');
if (lang !== 'zh' && lang !== 'en') {
    lang = (navigator.language || '').toLowerCase().startsWith('zh') ? 'zh' : 'en';
}

// 主题：?theme= 优先，否则按系统配色
let theme = params.get('theme');
if (theme !== 'light' && theme !== 'dark') {
    theme = window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
}
```

- 主题需在 `<head>` 中提前应用，避免闪烁；且当**没有** `?theme=` 参数时，应监听 `prefers-color-scheme` 的变化实时跟随。
- 不要用 `localStorage` 保存主题/语言偏好，以免覆盖"跟随系统"的默认行为。
- **不要添加语言 / 主题切换按钮或控件**：一律按系统默认显示，仅用 `?theme=` / `?language=` 参数覆盖。

新工具可参考这些文件保持一致的交互与视觉风格。

### 5. 新工具必须注册到主页

在 `docs/index.html` 的 `toolsData` 数组中新增一项：

```js
{
    title: "工具名称",
    desc: "一句话描述功能。",
    icon: "fa-calculator",      // Font Awesome solid 图标名
    link: "tool-name.html"
}
```

- `title` / `desc` 会同时用于主页展示与搜索匹配，请写清楚关键词。
- 图标颜色由数组顺序自动分配，无需手动指定颜色。

### 6. 同步更新 README.md

在 `README.md` 的工具列表中补充新工具的链接，保持与主页一致（按字母/拼音大致有序）：

```
* [工具名称](https://hubenchang0515.github.io/ai-misc/tool-name)
```

## 开发与部署

没有构建、测试或 lint 步骤。直接编辑 `docs/` 下的文件，用浏览器打开即可调试。

```bash
npm install          # 安装 wrangler（仅部署需要）
npm run deploy       # npx wrangler pages deploy docs --branch main
```

- 部署目录固定为 `docs/`，分支为 `main`。
- `node_modules/` 与 `.wrangler/` 已在 `.gitignore` 中忽略，不要提交。

## 代码风格

- 使用原生 `function` 声明与简洁的 DOM 操作，保持与现有文件一致的写法。
- 变量命名：数据类型用 `camelCase`，常量与配置对象用 `UPPER_SNAKE_CASE` 或 `camelCase`（参考 `toolsData`、`NAMESPACE_DNS`）。
- 适当添加中文注释说明关键逻辑。
- 保持单文件简洁，避免引入框架或多余抽象。

## 提交约定

- 提交信息简短，使用中文或英文均可（历史提交以中文为主，如「工资计算器」「主页重构」）。
- 一次提交尽量聚焦一个工具或一类改动。

## 注意事项

- **纯前端、保护隐私**：不要在服务端处理用户数据，避免引入需要后端的上传接口。
- **离线可用**：第三方 JS/CSS/字体已本地化到 `docs/vendor/`，页面不依赖 CDN；引入新依赖请按第 3 节处理。
- **仍依赖网络的功能**（属预期）：`linux-cmd` 的 sitemap、`pdf-view` 的示例 PDF、`qrcode` 解析远程图片时的 `corsproxy` 等运行时数据/接口未本地化。
- 新工具即新文件，不要修改无关工具的行为。
- 修改主页 `toolsData` 后，请顺带确认 `README.md` 已同步。
