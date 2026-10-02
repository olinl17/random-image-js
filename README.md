# Pic API

一个纯粹的静态随机图 API。不依赖任何客户端 JavaScript，只通过服务端 302 跳转返回随机图片。

## 特性

- **纯 API**：无需在页面中引入任何 JS，直接通过 URL 调用。
- **UA 自适应**：`/random/ua` 根据 User-Agent 自动返回横屏或竖屏图片。
- **横竖屏分离**：`/random/h` 返回横屏图，`/random/v` 返回竖屏图。
- **静态部署**：构建后可直接部署到 EdgeOne Makers、Cloudflare Pages、Vercel、Nginx 等。
- **内置画廊**：构建后生成 `gallery.html`，支持 All / H / V 页内切换、懒加载、骨架屏、点击查看大图。

## 快速开始

### 1. 准备图片

将横屏图片放入 `ri/h/`，竖屏图片放入 `ri/v/`。

### 2. 配置域名

修改 `config.json`：

```json
{
    "domain": "https://pic.olinl.com"
}
```

或使用环境变量（优先级更高）：

```bash
# Linux / Mac
export DOMAIN="https://cdn.example.com"
node build.js

# Windows PowerShell
$env:DOMAIN="https://cdn.example.com"; node build.js
```

### 3. 构建

```bash
node build.js
```

构建产物输出到 `dist/` 目录。

### 4. 部署到 EdgeOne Makers（推荐）

本项目已配置 `edgeone.json`，可直接使用 EdgeOne CLI 部署。

#### 方式一：CLI 自动构建部署（推荐）

```bash
# 1. 安装 EdgeOne CLI
npm install -g edgeone

# 2. 登录（按提示选择 Global / China）
edgeone login

# 3. 一键构建并部署
edgeone makers deploy -n static-randompic
```

CLI 会自动读取 `edgeone.json`：
- 执行 `npm install`
- 执行 `node build.js`
- 将 `dist/` 目录部署到 EdgeOne Makers

#### 方式二：本地构建后手动部署

```bash
# 1. 本地构建
node build.js

# 2. 部署 dist/ 目录
edgeone makers deploy ./dist -n static-randompic
```

#### 方式三：Pages Drop 直接上传

1. 先本地构建：`node build.js`
2. 访问 [EdgeOne Pages Drop](https://pages.edgeone.ai/drop)
3. 将 `dist/` 文件夹拖拽上传
4. 设置域名，点击部署

#### CI/CD 部署（GitHub Actions）

在仓库设置中添加 `EDGEONE_API_TOKEN`，然后创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to EdgeOne
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: node build.js
      - run: npx edgeone makers deploy ./dist -n static-randompic -t ${{ secrets.EDGEONE_API_TOKEN }} -e production
```

> 注意：中国大陆用户需登录控制台获取 API Token；海外用户可使用 `--anonymous` 匿名部署，但预览链接 1 小时内需认领。

## 更新与重新部署

### 更新图片

1. 替换或新增图片到 `ri/h/`（横屏）和 `ri/v/`（竖屏）。
2. 重新构建：

```bash
node build.js
```

3. 重新部署（以 EdgeOne CLI 为例）：

```bash
edgeone makers deploy -n static-randompic
```

> 注意：如果直接拖拽上传，需要重新上传最新的 `dist/` 目录。

### 更新代码

1. 修改源码，例如 `build.js`、`edgeone.json` 等。
2. 重新构建：

```bash
node build.js
```

3. 重新部署：

```bash
edgeone makers deploy -n static-randompic
```

### 本地预览

部署前可以先本地预览：

```bash
node serve.js
```

然后打开 http://localhost:8080/ 查看效果。

## API 端点

| 端点 | 说明 | 示例 |
|------|------|------|
| `/random/ua` | 根据 UA 自动适配横屏/竖屏 | `https://pic.olinl.com/random/ua` |
| `/random/h` | 随机横屏图 | `https://pic.olinl.com/random/h` |
| `/random/v` | 随机竖屏图 | `https://pic.olinl.com/random/v` |

## 跨域（CORS）

`/random/*` 端点已默认允许所有域名跨域（`Access-Control-Allow-Origin: *`），可被任意前端通过 `fetch` 调用：

```javascript
fetch('https://pic.olinl.com/random/h')
  .then(res => console.log(res.headers.get('location'))); // 实际图片地址
```

响应头包含：

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 86400
```

如果需要在 `<canvas>` 中绘制图片，需要给图片资源 `/ri/*` 也开启 CORS：

1. 修改 `edgeone.json` 中 `/ri/*` 的 headers：

```json
{
  "source": "/ri/*",
  "headers": [
    { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" },
    { "key": "Access-Control-Allow-Origin", "value": "*" },
    { "key": "Vary", "value": "Origin" }
  ]
}
```

2. 修改后重新 build 并部署。
3. HTML 中图片标签需要加 `crossorigin` 属性：

```html
<img src="https://pic.olinl.com/random/h" crossorigin="anonymous" alt="random">
```

## 使用示例

直接在 HTML 中使用：

```html
<img src="https://pic.olinl.com/random/ua" alt="random">
```

作为背景图 CSS：

```css
.banner {
    background-image: url('https://pic.olinl.com/random/h');
    background-size: cover;
}
```

## 目录结构

```
.
├── ri/                     # 图片源目录
│   ├── h/                  # 横屏图片
│   └── v/                  # 竖屏图片
├── functions/              # Cloudflare Pages Functions
│   └── random/
│       ├── h.js
│       ├── v.js
│       └── ua.js
├── edge-functions/         # EdgeOne 边缘函数
│   └── random/
│       ├── h.js
│       ├── v.js
│       └── ua.js
├── dist/                   # 部署产物
│   ├── ri/
│   │   ├── h/
│   │   └── v/
│   ├── counts.json
│   ├── index.html          # API 文档页
│   ├── gallery.html        # 画廊页
│   └── edge-functions/     # EdgeOne 边缘函数（构建输出）
├── build.js                # 构建脚本
├── config.json             # 域名配置
├── edgeone.json            # EdgeOne Makers 配置
├── package.json
└── README.md
```

## 版本控制

`dist/` 是构建产物，已加入 `.gitignore`，不需要提交到 Git。每次修改源码或图片后，本地运行 `node build.js` 生成即可。

`.edgeone/` 是 EdgeOne CLI 本地配置目录，也已加入 `.gitignore`。

如果你之前已经把 `dist/` 提交到了 Git，可以用以下命令将其从版本控制中移除（保留本地文件）：

```bash
git rm -r --cached dist
git add .gitignore
git commit -m "chore: ignore dist build output"
```

## 注意事项

- 每次添加新图片后都需要重新运行 `node build.js`。
- 构建会清空 `dist/` 目录，请勿在其中直接修改文件。
- 接口返回 `302` 临时重定向，并附加 `no-cache` 头，避免被浏览器缓存。

## License

ISC
