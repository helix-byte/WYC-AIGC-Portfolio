# 王艺纯 · AIGC 影视编辑作品集

一个纯静态、离线可用的个人简历作品集网站。v2 设计主题「暗场 · 一点灯」：冷黑基调 + 暖灯单点，首页胶片条缓缓流动，展示全部影片。

## 🌐 线上地址

**https://helix.ln.cn/**

（任何人打开这个链接即可查看；更新内容后 push 到 main 分支，网站会自动更新）

## 快速开始

1. 直接双击打开 `index.html` 即可在浏览器中浏览（无需安装任何软件、无需联网）。
2. 视频、剧本、图片均为本地文件，离线也能播放与查看。
3. 每个视频使用同名「封面」图作为海报。
4. 简历 PDF：`assets/王艺纯-简历.pdf`（源文件 `resume.html`，改完后用浏览器打印成 PDF）。
5. 分享预览图 `assets/og-cover.png`：发微信/QQ 时链接会显示精美卡片。

## 目录结构

```
portfolio/
├── index.html            # 网站首页（所有内容在一个页面）
├── resume.html           # 简历源文件（可在线浏览 / 打印为 PDF）
├── css/style.css         # 样式
├── js/main.js            # 交互逻辑 + 作品数据
└── assets/
    ├── videos/           # AI 视频作品（720P 压缩版）
    ├── scripts/          # 分镜剧本（docx）
    ├── posters/          # 视频封面图（与视频同名，同时用于首页胶片条）
    ├── 王艺纯-简历.pdf     # 简历 PDF（下载用）
    ├── og-cover.png      # 社交平台分享预览图（1200×630）
    ├── favicon-32.png    # 网站图标
    ├── favicon-180.png   # 手机主屏图标
    ├── qr-portfolio.png  # 简历页二维码
    ├── 证件照.jpg         # 简历页头像
    └── images/
        ├── ai/           # AI 概念图 / 分镜资产（WebP 优化版）
        └── design/       # 视觉设计作品（WebP 优化版）
```

## 如何更新内容

所有作品数据都在 `js/main.js` 顶部的 `WORKS` 数组里，按需增删即可：

| 类型 | 说明 |
| --- | --- |
| `video` | AI 视频作品：`file` 指向视频，`poster` 指向封面，`code` 是胶片编号（REEL xx），`meta` 是画质/大小/时长（必须保持三段用 ` · ` 分隔） |
| `script` | 分镜剧本：`file` 指向 docx |
| `image`（`cat:"asset"`） | AI 概念图 / 分镜资产 |
| `image`（`cat:"design"`） | 视觉设计作品 |

新增文件后，把文件放进对应 `assets` 子目录，再在 `WORKS` 里加一条记录即可。首页胶片条会自动收录所有 `video` 类型的作品。

> ⚠️ 当前作品区仍为模板示例数据（原作者的作品），替换方式：把文件名改好后改 `WORKS` 里的 `file` / `poster` / `src` / `title` / `desc`，或整段重写 `WORKS`。

简历正文（个人概述、技能、实践经历、项目、教育等）直接编辑 `index.html` 里对应区块；简历 PDF 的排版源文件是 `resume.html`。

## 部署到 GitHub Pages（免费）

### 第一步：上传
- 最简单：安装 GitHub Desktop（https://desktop.github.com），登录后 Add local repository 选择本文件夹，Publish repository（Public）。
- 或命令行：
```
git add -A
git commit -m "更新内容"
git push
```

### 第二步：开启 Pages
仓库 Settings → Pages → Source 选 "Deploy from a branch"，分支 `main`、目录 `/ (root)`。

访问地址规则：
- 仓库名**恰为** `helix-byte.github.io` → 用户站点，地址为 `https://helix-byte.github.io/`（根路径）；
- 仓库名为其他（如当前的 `WYC-AIGC-Portfolio`）→ 项目站点，地址为 `https://helix-byte.github.io/<仓库名>/`（带仓库名子路径）。
- 当前已绑定自定义根域名，实际访问地址为 `https://helix.ln.cn/`（根路径）；项目仍可通过上面的 github.io 地址访问（会自动跳转到自定义域名）。

若日后改仓库名或改用用户站点，需同步修改 `index.html` 中的 `canonical` / `og:url` / `og:image`。

### 第三步：分享
网址：`https://helix.ln.cn/`

## 部署到腾讯云 COS / 阿里云 OSS（国内访问更快，可选）

把整个文件夹上传到对象存储并开启「静态网站托管」：Bucket 权限「公有读」，地域选离你近的。默认访问域名无需备案即可分享；绑定自有域名才需要 ICP 备案。

## 联系方式

- 电话：17742733852
- 邮箱：17742733852@163.com
- 微信：wyc17742733852
- 期望城市：沈阳
