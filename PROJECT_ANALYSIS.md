# btch-downloader 项目分析文档

## 项目概述

btch-downloader 是一个轻量级的 TypeScript/JavaScript 库，用于从多个社交媒体平台下载视频、图片和音频。

**版本**: 6.0.28  
**许可证**: MIT  
**作者**: Tio (@prm2.0)

## 支持平台

| 平台 | 函数名 | 类型 |
|------|--------|------|
| Instagram | `igdl` | 视频/图片 |
| TikTok | `ttdl` | 视频/音频 |
| Facebook | `fbdown` | 视频 |
| Twitter/X | `twitter` | 视频 |
| YouTube | `youtube` | 视频/音频 |
| MediaFire | `mediafire` | 文件 |
| CapCut | `capcut` | 模板 |
| Google Drive | `gdrive` | 文件 |
| Pinterest | `pinterest` | 图片/视频 |
| Douyin (抖音) | `douyin` | 视频 |
| Xiaohongshu (小红书) | `xiaohongshu` | 图片/视频 |
| SnackVideo | `snackvideo` | 视频 |
| Cocofun | `cocofun` | 视频 |
| Spotify | `spotify` | 音频 |
| YouTube Search | `yts` | 搜索结果 |
| SoundCloud | `soundcloud` | 音频 |
| Threads | `threads` | 视频/图片 |
| Kuaishou (快手) | `kuaishou` | 视频 |

## 项目结构

```
btch-downloader-main/
├── src/
│   ├── index.ts          # 主入口文件，包含所有下载函数
│   ├── Defaults/
│   │   └── site.ts       # 配置文件（API 基础 URL）
│   ├── Types/
│   │   ├── index.ts      # 类型导出
│   │   └── types.ts      # TypeScript 类型定义
│   └── Watermark/
│       └── config.ts     # 开发者水印配置
├── index.html            # 前端 Playground 页面
├── dist/                 # 编译输出目录
├── package.json          # 项目配置
└── tsconfig.json         # TypeScript 配置
```

## 核心架构

### API 调用流程

1. 前端调用平台对应的函数（如 `ttdl(url)`）
2. 函数通过 `btch-http` 库的 `HttpGet` 方法发送请求到后端 API
3. 后端 API 地址: `https://backend1.tioo.eu.org`
4. 返回解析后的媒体数据

### 前端 UI (index.html)

当前 UI 是一个简单的单页面应用：
- 平台选择下拉框
- URL 输入框
- Fetch 按钮
- JSON 结果显示区域

### 数据返回格式

各平台返回的数据结构不同，但通常包含：
- `status`: 请求状态
- `developer`: 开发者信息
- `title`: 内容标题
- `video`/`audio`/`url`: 下载链接
- `thumbnail`: 缩略图

## 当前 UI 问题

1. 界面过于简单，缺乏现代化设计
2. 只显示 JSON 原始数据，用户体验差
3. 没有视频预览功能
4. 没有直接下载按钮
5. 缺少主题切换功能

## 改造需求

1. **现代化 UI 设计**
   - 液态玻璃效果 (Liquid Glass)
   - 扁平化效果 (Flat Design)
   - 两种风格可切换

2. **功能增强**
   - 视频播放框：解析完成后直接播放视频
   - 下载按钮：支持直接下载媒体文件
   - 更友好的结果展示

## 技术栈

- **前端**: 原生 HTML/CSS/JavaScript
- **构建工具**: TypeScript + tsup
- **HTTP 客户端**: btch-http
- **包管理器**: pnpm/yarn/bun
