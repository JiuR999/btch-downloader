# BTCH Downloader 项目代码说明文档

## 1. 项目概述

**项目名称**: btch-downloader  
**版本**: 6.0.28  
**作者**: Tio (@prm2.0)  
**许可证**: MIT  
**仓库地址**: https://github.com/hostinger-bot/btch-downloader

btch-downloader 是一个轻量级的 TypeScript/JavaScript 库，用于从多个社交媒体平台下载视频、图片和音频内容。支持 Node.js 环境和浏览器环境。

---

## 2. 项目目录结构

```
btch-downloader-main/
├── src/                          # TypeScript 源码目录
│   ├── index.ts                  # 主入口文件，包含所有下载函数实现
│   ├── Defaults/
│   │   └── site.ts               # 配置文件（API 基础 URL、Issue 链接）
│   ├── Types/
│   │   ├── index.ts              # 类型导出入口
│   │   └── types.ts              # 所有 TypeScript 接口定义
│   └── Watermark/
│       └── config.ts             # 开发者水印配置
├── lib/
│   └── Browser/
│       └── index.ts              # 浏览器版本入口（用于 tsup 打包）
├── dist/                         # 构建输出目录
│   ├── index.js                  # Node.js 版本
│   ├── index.d.ts                # TypeScript 类型声明
│   ├── browser/
│   │   ├── index.js              # 浏览器版本（未压缩）
│   │   ├── index.min.js          # 浏览器版本（压缩）
│   │   └── *.map                 # Source Map 文件
│   ├── Defaults/
│   ├── Types/
│   └── Watermark/
├── index.html                    # 前端 Playground 页面（现代化 UI）
├── package.json                  # 项目配置
├── tsconfig.json                 # TypeScript 配置
├── tsup.config.ts                # 打包工具配置
├── eslint.config.mts             # ESLint 配置
├── README.md                     # 项目说明文档
├── LICENSE                       # MIT 许可证
└── test/                         # 测试目录
```

---

## 3. 核心模块详解

### 3.1 主入口文件 `src/index.ts`

这是项目的核心文件，包含所有平台的下载函数实现。

#### 导入依赖

```typescript
import configData from './Defaults/site';     // 配置数据
import watermark from './Watermark/config';   // 水印配置
import pkg from '../package.json';            // 版本号
import { HttpGet } from "btch-http";          // HTTP 请求库
import type { ... } from './Types';           // 类型定义
```

#### 全局配置

```typescript
const { config, issues } = configData as VersionConfig;
const wm = watermark.dev;        // 开发者标识: "@prm2.0"
const timeout = 60000;           // 请求超时时间: 60秒
const version: string = pkg.version;  // 当前版本号
```

#### 错误响应格式化

```typescript
const formatErrorResponse = (error: unknown): ApiErrorResponse => ({
    developer: wm,
    status: false,
    message: error instanceof Error ? error.message : 'Unknown error',
    note: `Please report issues to ${issues}`
});
```

#### 导出的下载函数

| 函数名 | 平台 | 参数类型 | 返回类型 |
|--------|------|----------|----------|
| `ttdl` | TikTok | `string` (URL) | `TikTokResponse` |
| `igdl` | Instagram | `string` (URL) | `InstagramResponse` |
| `twitter` | Twitter/X | `string` (URL) | `TwitterResponse` |
| `youtube` | YouTube | `string` (URL) | `YouTubeResponse` |
| `fbdown` | Facebook | `string` (URL) | `FacebookResponse` |
| `mediafire` | MediaFire | `string` (URL) | `MediaFireResponse` |
| `capcut` | CapCut | `string` (URL) | `CapCutResponse` |
| `gdrive` | Google Drive | `string` (URL) | `GoogleDriveResponse` |
| `pinterest` | Pinterest | `string` (URL/Query) | `PinterestResponse` |
| `aio` | All-In-One | `string` (URL) | `AioResponse` |
| `xiaohongshu` | 小红书 | `string` (URL) | `XiaohongshuResponse` |
| `douyin` | 抖音 | `string` (URL) | `DouyinResponse` |
| `snackvideo` | SnackVideo | `string` (URL) | `SnackVideoResponse` |
| `cocofun` | Cocofun | `string` (URL) | `CocofunResponse` |
| `spotify` | Spotify | `string` (URL) | `SpotifyResponse` |
| `yts` | YouTube Search | `string` (Query) | `YtsResponse` |
| `soundcloud` | SoundCloud | `string` (URL) | `SoundCloudResponse` |
| `threads` | Threads | `string` (URL) | `ThreadsResponse` |
| `kuaishou` | 快手 | `string` (URL) | `KuaishouResponse` |

#### 函数实现模式

每个下载函数遵循相同的模式：

```typescript
async function platformDownloader(url: string): Promise<ResponseType> {
    try {
        // 1. 调用 HttpGet 发送请求到后端 API
        const data = await HttpGet<ApiResponseType>('endpoint', url, version, timeout, config.baseUrl);
        
        // 2. 处理响应数据
        return {
            developer: wm,
            status: true,
            // ... 平台特定的数据字段
        };
    } catch (error) {
        // 3. 错误处理
        return { ...formatErrorResponse(error), status: false };
    }
}
```

---

### 3.2 类型定义文件 `src/Types/types.ts`

定义了所有 API 响应和函数返回的 TypeScript 接口。

#### 基础响应接口

```typescript
export interface BaseResponse {
    developer: string;           // 开发者标识
    status?: boolean | string;   // 请求状态
    message?: string;            // 错误信息
    note?: string;               // 备注信息
    code?: number;               // 状态码
}

export interface ApiErrorResponse extends BaseResponse {
    status: boolean;
    message: string;
    note: string;
}
```

#### 各平台响应结构

**TikTok**
```typescript
export interface TikTokResponse extends BaseResponse {
    title?: string;              // 视频标题
    title_audio?: string;        // 音频标题
    thumbnail?: string;          // 缩略图 URL
    video?: string[];            // 视频下载链接数组
    audio?: string[];            // 音频下载链接数组
}
```

**Instagram**
```typescript
export interface InstagramApiItem {
    thumbnail: string;           // 缩略图
    url: string;                 // 媒体 URL
}
export interface InstagramResponse extends BaseResponse {
    result?: InstagramApiItem[]; // 媒体项数组
}
```

**Twitter/X**
```typescript
export interface TwitterResponse extends BaseResponse {
    title?: string;              // 推文文本
    url?: string | Array<{hd: string, sd: string}>;  // 视频链接
}
```

**YouTube**
```typescript
export interface YouTubeResponse extends BaseResponse {
    title?: string;              // 视频标题
    thumbnail?: string;          // 缩略图
    author?: string;             // 作者
    mp3?: string;                // 音频下载链接
    mp4?: string;                // 视频下载链接
}
```

**Facebook**
```typescript
export interface FacebookResponse extends BaseResponse {
    Normal_video?: string;       // 普通质量视频
    HD?: string;                 // 高清视频
}
```

**Douyin (抖音)**
```typescript
export interface DouyinResponse extends BaseResponse {
    result?: {
        title?: string;
        thumbnail?: string;
        links?: Array<{quality: string, url: string}>;  // 不同质量的视频链接
    };
}
```

**Kuaishou (快手)**
```typescript
export interface KuaishouResponse extends BaseResponse {
    result?: {
        videoUrl: string;        // 视频 URL
        title: string;           // 标题
        author: string;          // 作者
        // ... 其他字段
    };
}
```

---

### 3.3 配置文件 `src/Defaults/site.ts`

```typescript
interface VersionConfig {
    config: {
        baseUrl: string;    // 后端 API 基础 URL
    };
    issues: string;         // Issue 反馈链接
}

const configData: VersionConfig = {
    config: {
        baseUrl: 'https://backend1.tioo.eu.org',  // 后端 API 地址
    },
    issues: 'https://github.com/hostinger-bot/btch-downloader/issues',
};
```

---

### 3.4 水印配置 `src/Watermark/config.ts`

```typescript
const watermark = {
    dev: "@prm2.0"  // 开发者标识
};
```

---

## 4. 构建系统

### 4.1 TypeScript 配置 `tsconfig.json`

```json
{
    "compilerOptions": {
        "target": "ESNext",
        "module": "NodeNext",
        "moduleResolution": "NodeNext",
        "declaration": true,           // 生成 .d.ts 类型声明文件
        "outDir": "./dist",            // 输出目录
        "rootDir": "./src",            // 源码目录
        "strict": true,                // 严格模式
        "esModuleInterop": true,
        "skipLibCheck": true,
        "sourceMap": true              // 生成 Source Map
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "**/*.test.ts"]
}
```

### 4.2 打包配置 `tsup.config.ts`

使用 tsup 进行打包，生成两个浏览器版本：

```typescript
export default defineConfig([
    // 未压缩版本
    {
        entry: ['lib/Browser/index.ts'],
        format: ['iife'],              // 立即执行函数格式
        target: 'es2019',
        platform: 'browser',
        minify: false,
        globalName: 'btch',            // 全局变量名: window.btch
        outDir: 'dist/browser',
        outExtension() { return { js: '.js' }; },
    },
    // 压缩版本
    {
        // ... 同上配置
        minify: true,
        outExtension() { return { js: '.min.js' }; },
    },
]);
```

### 4.3 构建命令

```bash
npm run build        # 完整构建: tsc && tsup
npm run build:browser # 仅构建浏览器版本: tsup
```

---

## 5. 前端 UI `index.html`

### 5.1 设计风格

采用现代化 UI 设计，支持两种主题切换：

- **Glass (液态玻璃)**: 毛玻璃效果、透明模糊、三色渐变背景
- **Flat (扁平化)**: 纯白卡片、清晰边框、简洁灰白背景

### 5.2 CSS 设计系统

#### 色彩令牌

```css
:root {
    --brand-blue: #2563eb;        /* 品牌蓝 */
    --brand-blue-hover: #1d4ed8;  /* 悬停蓝 */
    --accent-indigo: #4f46e5;     /* 辅助靛蓝 */
    --slate-50: #f8fafc;          /* 最浅灰 */
    --slate-100: #f1f5f9;         /* 浅灰 */
    --slate-200: #e2e8f0;         /* 边框灰 */
    --slate-400: #94a3b8;         /* 占位符灰 */
    --slate-500: #64748b;         /* 次级文本 */
    --slate-900: #0f172a;         /* 主文本 */
    --success: #10b981;           /* 成功绿 */
}
```

#### 圆角嵌套原则

```css
--radius-lg: 2rem;      /* 外层大卡片: 32px */
--radius-md: 1rem;      /* 内部元素: 16px */
--radius-sm: 0.75rem;   /* 小元素: 12px */
--radius-full: 9999px;  /* 胶囊/圆形 */
```

#### Glass 主题样式

```css
.theme-glass .main-card {
    background: rgba(255, 255, 255, 0.7);
    backdrop-filter: blur(24px) saturate(160%);
    border: 1px solid rgba(255, 255, 255, 0.4);
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.05);
}
```

#### Flat 主题样式

```css
.theme-flat .main-card {
    background: #ffffff;
    border: 1px solid var(--slate-200);
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.02);
}
```

### 5.3 HTML 结构

```html
<body class="theme-glass">
    <div class="app-container">
        <header class="app-header">...</header>           <!-- 标题 -->
        <div class="theme-switcher">...</div>              <!-- 主题切换 -->
        <div class="main-card">...</div>                   <!-- 输入区域 -->
        <div id="videoSection" class="video-section">...</div>  <!-- 视频播放 -->
        <div class="main-card">                            <!-- 原始响应 -->
            <details class="result-details">...</details>
        </div>
        <div class="examples-section">...</div>            <!-- 示例链接 -->
        <footer class="app-footer">...</footer>            <!-- 页脚 -->
    </div>
</body>
```

### 5.4 JavaScript 功能模块

#### DOM 元素引用

```javascript
const DOM = {
    body: document.body,
    segmentBtns: document.querySelectorAll('.segment-btn'),
    platformSelect: document.getElementById('platformSelect'),
    urlInput: document.getElementById('urlInput'),
    fetchBtn: document.getElementById('fetchBtn'),
    resultArea: document.getElementById('resultArea'),
    videoSection: document.getElementById('videoSection'),
    videoPlayer: document.getElementById('videoPlayer'),
    // ...
};
```

#### 平台自动检测

```javascript
const regexMap = {
    instagram: /instagram\.com/i,
    tiktok: /tiktok\.com/i,
    twitter: /(twitter|x)\.com/i,
    youtube: /(youtube\.com|youtu\.be)/i,
    // ... 更多平台
};

function detectPlatform(input, selected) {
    if (selected !== 'auto') return selected;
    if (isValidUrl(input)) {
        for (const [name, rx] of Object.entries(regexMap)) {
            if (rx.test(input)) return name;
        }
    }
    return 'yts';  // 默认 YouTube 搜索
}
```

#### 视频 URL 提取

```javascript
function extractVideoUrl(data, platform) {
    switch (platform) {
        case 'tiktok': return data.video?.[0];
        case 'youtube': return data.mp4;
        case 'facebook': return data.HD || data.Normal_video;
        case 'twitter':
            if (Array.isArray(data.url)) return data.url[0].hd || data.url[0].sd;
            return data.url;
        case 'douyin': return data.result?.links?.[0]?.url;
        case 'kuaishou': return data.result?.videoUrl;
        case 'instagram': return data.result?.[0]?.url;
        // ... 更多平台
    }
}
```

#### 媒体项提取

```javascript
function extractMediaItems(data, platform) {
    const items = [];
    switch (platform) {
        case 'instagram':
            data.result?.forEach(i => {
                items.push({ url: i.url, thumb: i.thumbnail, type: 'video' });
            });
            break;
        case 'tiktok':
            data.video?.forEach(u => items.push({ url: u, type: 'video' }));
            data.audio?.forEach(u => items.push({ url: u, type: 'audio' }));
            break;
        // ... 更多平台
    }
    return items;
}
```

---

## 6. API 数据流

```
用户输入 URL
    ↓
前端 detectPlatform() 识别平台
    ↓
调用对应的 btch[fnName](url)
    ↓
HttpGet 请求发送到 https://backend1.tioo.eu.org/{endpoint}
    ↓
后端解析并返回媒体数据
    ↓
前端 extractVideoUrl() 提取视频链接
    ↓
显示视频播放器 + 下载按钮
```

---

## 7. 支持平台的数据结构

| 平台 | 响应关键字段 | 视频 URL 位置 |
|------|-------------|--------------|
| TikTok | `video[]`, `audio[]` | `data.video[0]` |
| Instagram | `result[]` | `data.result[0].url` |
| Twitter | `url[]` | `data.url[0].hd` 或 `data.url[0].sd` |
| YouTube | `mp4`, `mp3` | `data.mp4` |
| Facebook | `HD`, `Normal_video` | `data.HD` |
| Douyin | `result.links[]` | `data.result.links[0].url` |
| Kuaishou | `result.videoUrl` | `data.result.videoUrl` |
| Threads | `result.video` | `data.result.video` |
| Xiaohongshu | `result.downloads[]` | `data.result.downloads[0].url` |
| Cocofun | `result.no_watermark` | `data.result.no_watermark` |
| Pinterest | `result.video_url` | `data.result.video_url` |
| CapCut | `originalVideoUrl` | `data.originalVideoUrl` |
| Spotify | `result.formats[]` | `data.result.formats[0].url` |
| SoundCloud | `result.audio` | `data.result.audio` |

---

## 8. 依赖说明

### 生产依赖

| 包名 | 用途 |
|------|------|
| `btch-http` | HTTP 请求库，与后端 API 通信 |
| `typescript` | TypeScript 编译器 |
| `tsup` | 打包工具 |
| `esbuild` | JavaScript/TypeScript 打包器 |

### 开发依赖

| 包名 | 用途 |
|------|------|
| `eslint` | 代码检查工具 |
| `@typescript-eslint/*` | TypeScript ESLint 插件 |

---

## 9. 使用示例

### Node.js 环境

```javascript
// ESM
import { ttdl, youtube, igdl } from 'btch-downloader';

const result = await ttdl('https://www.tiktok.com/@user/video/123');
console.log(result.video[0]);  // 视频下载链接

// CJS
const { ttdl } = require('btch-downloader');
```

### 浏览器环境

```html
<script src="https://cdn.jsdelivr.net/npm/btch-downloader/dist/browser/index.min.js"></script>
<script>
    const result = await window.btch.ttdl('https://tiktok.com/...');
    console.log(result);
</script>
```

---

## 10. 扩展指南

### 添加新平台支持

1. 在 `src/Types/types.ts` 添加新的接口定义
2. 在 `src/index.ts` 添加新的下载函数
3. 在 `src/index.ts` 的 `export` 中导出新函数
4. 在 `index.html` 的 `regexMap` 和 `fnMap` 中添加映射
5. 在 `index.html` 的 `extractVideoUrl` 和 `extractMediaItems` 中添加处理逻辑

---

## 11. 许可证

MIT License - 详见 [LICENSE](./LICENSE) 文件
