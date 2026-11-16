# AGENT.md · 项目说明

面向在本仓库工作的 AI 代理：项目形态、数据流与改动约束。协作流程与技能规范见 [`.agents/AGENT.md`](.agents/AGENT.md)。

## 1. 项目

「王艺纯 · AIGC 影视编辑」求职作品集，纯静态单页；投递简历时附链接，HR 打开即看。

| 项 | 值 |
| --- | --- |
| 仓库 | `helix-byte/WYC-AIGC-Portfolio` |
| 线上 | https://helix.ln.cn/ |
| 主分支 / 部署 | `main`；GitHub Pages（`main` 根目录） |

## 2. 技术形态（改动前必读）

- **零构建零依赖**：无 `package.json`、无 `node_modules`、无打包器；不得引入 npm 依赖、框架、CSS 预处理器或构建步骤。
- **原生三件套**：`index.html` + `css/style.css` + `js/main.js`；`main.js` 是 IIFE，沿用 ES5（`var` + 函数声明）风格。
- **资源本地优先**：字体、图片、剧本等一律仓库内文件，禁止外链。
- **唯一外链**：视频走**多吉云官方播放器 SDK（DogePlayer）**，`index.html` 引入 `https://player.dogecloud.com/js/loader`（官方明确不支持 NPM 安装或本地部署）。`main.js` 只保存 `vcode`，播放地址由 SDK 每次播放时向多吉云实时获取并自动续期 —— **仓库与页面中不存在任何 M3U8 链接**。故**视频播放需联网**，其余区块可离线。
- **双击可用**：`file://` 下须正常渲染（视频受跨域限制，建议本地静态服务器预览），改完按此场景自测。

## 3. 目录

```
├── index.html          # 首页：hero / 关于 / 制作链路 / 作品 / 实践 / 教育 / 联系
├── resume.html         # 简历源文件（A4 单页，打印即 PDF）
├── css/style.css        # 全站样式（hero 背景图约第 95 行）
├── js/main.js          # 交互逻辑 + 作品数据 WORKS
├── assets/
│   ├── posters/        # 视频封面 webp，与视频同名，兼作首页胶片条
│   ├── scripts/        # 分镜剧本 docx
│   ├── images/ai|design/  # AI 概念图 / 视觉设计作品（webp）
│   ├── 王艺纯-简历.pdf   # 由 resume.html 导出
│   ├── og-cover.png    # 分享卡片 1200×630
│   ├── qr-portfolio.png · 证件照.jpg · favicon-*.png
├── README.md           # 面向人的使用与部署说明
└── .agents/            # 代理技能与协作规范
```

## 4. 数据流与硬约束

作品区完全由 `main.js` 顶部 `WORKS` 数组驱动（当前 42 条：11 视频 + 4 剧本 + 27 图片）：`WORKS` → `buildReel()` 生成首页胶片条（仅 `type:"video"`）、`render()`/`cardHTML()` 生成卡片与筛选、`openVideo()`/`openLightboxAt()` 弹窗。

```js
// 视频：vcode 为多吉云 VCode；userId 全局固定（DOGE_USER_ID），不写入条目
{id:"v1", type:"video", cat:"video", code:"REEL 01", title, tag, vcode, poster:"assets/posters/x.webp", meta:"720P · 01:50", desc}
// 剧本
{id:"s1", type:"script", cat:"script", code:"SCRIPT 01", title, tag, file:"assets/scripts/x.docx", sub:"《…》", meta, desc}
// 图片
{id:"a1", type:"image", cat:"asset", code:"CONCEPT 01", title, src:"assets/images/ai/x.webp", meta}
```

硬约束：

1. 视频 `meta` 必须是 `画质 · 时长` **两段**：`reelFrameHTML()` 用 `meta.split(" · ")[1]` 取时长，缺段会显示 `undefined`。
2. 视频**只写 `vcode`**（16 位十六进制，取自多吉云控制台视频列表或视频信息 API）。禁止写入任何 M3U8 直链、`permalink.dogevideo.com` 域名或 `flsign` 签名参数 —— 永久直链无防盗链能力，公开即等于开放下载入口。`userId` 由 `DOGE_USER_ID`（16366）统一提供，不得逐条重复。
3. `id` 全数组唯一（胶片条与弹窗靠它关联）。
4. `cat` ∈ `video` / `script` / `asset` / `design`，与 `#filters` 的 `data-filter` 一一对应；新增分类须同步改 `index.html` 筛选按钮与 `render()`。

## 5. 常见任务

### 5.1 修改个人信息

姓名、求职意向、联系方式在 `index.html` 与 `resume.html` **各写一份**，改前 `grep` 关键词逐个确认。联系方式三处同步：`href="tel:"` / `href="mailto:"` / 按钮 `data-copy`。简历 PDF 文件名在 `index.html` 出现 3 次（导航、hero、联系区）。

### 5.2 新增 / 替换作品

1. 视频：上传多吉云，从控制台视频列表复制 **VCode**，写入 `WORKS` 的 `vcode`（封面另存 `assets/posters/`）。
2. 封面放 `assets/posters/`，与视频同名（webp）。
3. 剧本 / 图片放 `assets/scripts/`、`assets/images/ai|design/`。
4. 改 `WORKS`，路径与文件名逐字一致；图片统一转 WebP。
5. 文件名可中文，但不含空格、`#`、`?`（会破坏 `file://` 与 URL 解析）。

### 5.3 重新导出简历 PDF

改 `resume.html` 后必须重新导出，否则下载的 PDF 与页面不一致：

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu \
  --no-pdf-header-footer \
  --print-to-pdf="$(cygpath -w "$PWD/assets/王艺纯-简历.pdf")" \
  --virtual-time-budget=5000 "file:///$(cygpath -m "$PWD/resume.html")"
```

验收：**A4 单页**（MediaBox ≈ 594.96×841.92 pt）、中文正常、`.print-tip` 不出现（`@media print` 已隐藏）。Chrome 生成的 PDF 用 `pdftotext` 提取中文会丢字（子集字体缺 ToUnicode），属正常现象，需截图核对版面。

### 5.4 改动后的校验基线

仓库根目录执行（只读校验，不得回写）：

```bash
node --check js/main.js

node - <<'JS'
const fs=require('fs');
const V=/^(meta|link|img|br|hr|input|source)$/;
for(const f of ['index.html','resume.html']){
  const st=[];
  fs.readFileSync(f,'utf8').replace(/<(\/?)([a-zA-Z]\w*)\b[^>]*?(\/?)>/g,(_,c,t,sc)=>{
    if(!V.test(t)&&!sc) c?st.pop():st.push(t); return '';});
  console.log(f,'未闭合:',st.join(',')||'无');
}
const idx=fs.readFileSync('index.html','utf8');
const refs=[...new Set([...idx.matchAll(/(?:href|src)="((?!http|#|tel:|mailto:)[^"]+)"/g)]
  .map(m=>m[1].split('?')[0]))];
console.log('index.html 缺失资源:',refs.filter(p=>!fs.existsSync(p)).join(' | ')||'无');
const src=fs.readFileSync('js/main.js','utf8');
const vcodes=[...src.matchAll(/type:"video"[\s\S]*?vcode:"([0-9a-f]{16})"/g)].map(m=>m[1]);
console.log('视频条数:',vcodes.length,'| vcode 重复:',
  vcodes.filter((v,i)=>vcodes.indexOf(v)!==i).join(' | ')||'无');
const leak=[...src.matchAll(/[\w/.:-]*permalink\.dogevideo\.com[^\s"]*|[\w/.:-]*\.m3u8[^\s"]*/g)];
console.log('残留直链/流地址:',leak.map(m=>m[0]).join(' | ')||'无');
JS
```

### 5.5 改完 CSS / JS 要清缓存

`index.html` 中资源带版本号（`css/style.css?v=…`、`js/main.js?v=…`），改动后同步递增 `?v=`，否则线上命中旧缓存。

## 6. 硬性约定

| 项 | 约定 |
| --- | --- |
| 语言 | 全程简体中文（代码、路径、报错原文除外） |
| 写入 / 校验 | 只改文件用 `edit`/`write`；校验脚本走 stdin（`node - <<'JS'`），不落地临时文件 |
| 单文件体积 | ≤ 100MB（GitHub 限制），当前最大约 710KB（`og-cover.png`） |
| 图片 | WebP，单图尽量 < 500KB |
| 外链 | 仅多吉云 DogePlayer SDK（`https://player.dogecloud.com/js/loader`）；页不得出现 m3u8 地址 |
| 兼容性 | 现代浏览器 + `file://` 双击打开 |

## 7. 已知状态

- 视频改为**DogePlayer + `vcode`** 播放：11 条视频各存一个 VCode，播放地址由 SDK 动态获取。仓库内无 MP4、无 M3U8 直链（旧的 `assets/videos/m3u8_permalinks.json` 永久直链清单已删除，对应视频也已重建使旧链接失效）；
- 站点已上线，核对线上用 `curl -I` 访问页面与资源；勿用 GitHub Pages API 作判据（匿名请求可能 404）。
- `assets/og-cover.png` 约 710KB，可压缩至 200KB 以内。
- 远端仅 `main` 一个分支，无长期保留分支。

## 8. 不要做的事

1. 不要引入构建工具、框架、包管理器或第三方依赖。
2. 不要重排 `WORKS` 顺序 —— 数组顺序即展示顺序，`code` 编号须对应。
3. 不要为「精简仓库」删 `assets/` 下在用素材 —— 素材与 `WORKS` 强引用。
4. 不要直接向 `main` 提交，不要用 fast-forward 合并；流程见 [`.agents/AGENT.md`](.agents/AGENT.md)。
5. 不要在未重新导出 PDF 的情况下改 `resume.html`。
