# AGENT.md · 项目协作说明

本文件面向在本仓库中工作的 AI 编码代理，说明项目形态、数据流与改动约束。协作流程与技能规范见 [`.agents/AGENT.md`](.agents/AGENT.md)。

## 1. 项目定位

「王艺纯 · AIGC 影视编辑」个人求职作品集网站 —— 纯静态、单页、离线可用的求职材料。

| 项目 | 值 |
| --- | --- |
| 仓库 | `helix-byte/WYC-AIGC-Portfolio` |
| 线上地址 | https://helix-byte.github.io/WYC-AIGC-Portfolio/ |
| 主分支 | `main` |
| 部署 | GitHub Pages（`main` 分支 `/ (root)`），已上线可访问 |
| 使用场景 | 投递简历时附带链接；HR 打开即看，无需登录、无需联网 |

## 2. 技术形态（改动前必读）

- **零构建、零依赖**：没有 `package.json`、没有 `node_modules`、没有打包器；不要引入 npm 依赖、框架、CSS 预处理器或构建步骤。
- **原生三件套**：`index.html` + `css/style.css` + `js/main.js`，`js/main.js` 是 IIFE，沿用 `var` / 函数声明式的 ES5 写法，新代码保持同风格。
- **离线可用**：页面不得引用任何 CDN、外链字体、外部图片或第三方接口；所有资源必须是仓库内的本地文件。
- **双击即用**：`file://` 协议下必须能正常渲染，改动后先按这个场景自测。

## 3. 目录与职责

```
├── index.html          # 首页：hero / 关于 / 制作链路 / 作品 / 实践与项目 / 教育 / 联系
├── resume.html         # 简历排版源文件（A4 单页，打印即 PDF）
├── css/style.css       # 全站样式（含首页 hero 背景图，第 95 行）
├── js/main.js          # 交互逻辑 + 作品数据 WORKS 数组
├── assets/
│   ├── videos/         # 视频（720P 压缩版）
│   ├── posters/        # 视频封面（与视频同名 webp），同时用于首页胶片条
│   ├── scripts/        # 分镜剧本 docx
│   ├── images/ai/      # AI 概念图 / 分镜资产（webp）
│   ├── images/design/  # 视觉设计作品（webp）
│   ├── 王艺纯-简历.pdf   # 下载用简历（由 resume.html 导出）
│   ├── og-cover.png    # 社交分享卡片 1200×630
│   ├── qr-portfolio.png# 简历页二维码
│   ├── 证件照.jpg       # 简历页头像
│   └── favicon-*.png
├── README.md           # 面向人的使用与部署说明
└── .agents/            # 代理技能与协作规范
```

## 4. 数据流与关键约束

作品区完全由 `js/main.js` 顶部 `WORKS` 数组驱动（当前 42 条：11 视频 + 4 剧本 + 27 图片）：

```
WORKS ──┬─→ VIDEOS      ─→ buildReel()  首页胶片条（只收 type:"video"）
        └─→ render()    ─→ cardHTML()   作品卡片 + 筛选（cat: video/script/asset/design）
                        ─→ openVideo() / openLightboxAt()  弹窗
```

四种记录结构：

```js
// 视频
{id:"v1", type:"video", cat:"video", code:"REEL 01", title:"…", tag:"…",
 file:"assets/videos/x.mp4", poster:"assets/posters/x.webp",
 meta:"720P · 约30MB · 01:50", desc:"…"}

// 剧本
{id:"s1", type:"script", cat:"script", code:"SCRIPT 01", title:"…", tag:"…",
 file:"assets/scripts/x.docx", sub:"《…》", meta:"DOCX · 点击打开", desc:"…"}

// 图片
{id:"a1", type:"image", cat:"asset", code:"CONCEPT 01", title:"…",
 src:"assets/images/ai/x.webp", meta:"分类 · 点击查看大图"}
```

**硬约束**：

1. 视频条目的 `meta` 必须是 `画质 · 大小 · 时长` **三段**，`reelFrameHTML()` 用 `meta.split(" · ")[2]` 取时长，段数不足胶片条会显示 `undefined`。
2. `id` 全数组唯一，胶片条与弹窗都靠它关联。
3. `cat` 只能取 `video` / `script` / `asset` / `design`，与 `#filters` 的 `data-filter` 一一对应，新增分类需同步改 `index.html` 的筛选按钮与 `render()`。

## 5. 常见任务手册

### 5.1 修改个人信息

- 首页与简历页的姓名、求职意向、联系方式**分别写在两处**，`grep` 关键词逐一确认，避免只改一半。
- 联系方式要注意 `href="tel:"` / `href="mailto:"` 与按钮的 `data-copy` 属性三处同步。
- 简历 PDF 文件名在 `index.html` 出现 3 次（导航按钮、hero 按钮、联系区按钮）。

### 5.2 新增/替换作品

1. 把文件放入 `assets/` 对应子目录（视频放 `videos/`，封面放 `posters/` 且**同名**）。
2. 在 `WORKS` 中增删记录，路径与文件名逐字一致。
3. 图片统一转 WebP；视频按 720P 压缩。
4. 文件名允许中文，但不要含空格、`#`、`?`（会破坏 `file://` 与 URL 解析）。

### 5.3 重新导出简历 PDF

修改 `resume.html` 后必须重新导出，否则下载的 PDF 与页面不一致：

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu \
  --no-pdf-header-footer \
  --print-to-pdf="$(cygpath -w "$PWD/assets/王艺纯-简历.pdf")" \
  --virtual-time-budget=5000 "file:///$(cygpath -m "$PWD/resume.html")"
```

验收标准：**A4 单页**（MediaBox ≈ 594.96×841.92 pt）、中文显示正常、`.print-tip` 提示条不出现（已被 `@media print` 隐藏）。

> 注意：Chrome 生成的 PDF 用 `pdftotext` 提取中文会丢字，这是子集字体缺 ToUnicode 映射所致，**不是渲染问题**，需用截图核对版面。

### 5.4 改动后的校验基线

在仓库根目录执行（只读校验，不得回写文件）：

```bash
node --check js/main.js

node - <<'JS'
const fs=require('fs');
for(const f of ['index.html','resume.html']){
  const s=fs.readFileSync(f,'utf8');
  const stack=[],void_=new Set(['meta','link','img','br','hr','input','source']);
  const re=/<(\/?)([a-zA-Z][a-zA-Z0-9]*)\b[^>]*?(\/?)>/g; let m;
  while((m=re.exec(s))){
    if(void_.has(m[2].toLowerCase())||m[3]==='/') continue;
    if(!m[1]) stack.push(m[2].toLowerCase());
    else stack.pop();
  }
  console.log(f,'未闭合:',stack.join(',')||'无');
}
const idx=fs.readFileSync('index.html','utf8');
const refs=[...idx.matchAll(/(?:href|src)="((?!http|#|tel:|mailto:)[^"]+)"/g)].map(m=>m[1].split('?')[0]);
console.log('index.html 缺失资源:',[...new Set(refs)].filter(p=>!fs.existsSync(p)).join(' | ')||'无');
JS
```

### 5.5 改完 CSS / JS 要清缓存

`index.html` 尾部引用了带版本号的资源：

```html
<link rel="stylesheet" href="css/style.css?v=20261028b">
<script src="js/main.js?v=20261028"></script>
```

改动后请同步递增 `?v=` 值，否则线上用户会命中旧缓存。

## 6. 硬性约定

| 项目 | 约定 |
| --- | --- |
| 输出语言 | 全程简体中文（代码、路径、报错原文除外） |
| 文件写入 | 只能用 `edit` / `write` 工具，禁止用脚本或命令行重写文件内容 |
| 脚本执行 | 校验脚本走 stdin（`node - <<'JS'`），不落地临时文件 |
| 单文件体积 | ≤ 100MB（GitHub 限制），当前最大 34MB |
| 图片格式 | WebP，优先控制单图 < 500KB |
| 页面外链 | 禁止（破坏离线可用） |
| 兼容性 | 面向现代浏览器 + `file://` 双击打开 |

## 7. 当前已知状态（需持续更新）

- 站点已上线，线上地址返回 200；核对线上状态直接用 `curl -I` 访问页面与资源，勿依赖 GitHub Pages API（匿名请求可能返回 404，不能作为未启用判据）。
- `assets/og-cover.png` 约 710KB，社交平台抓取偏慢，可压缩至 200KB 以内。
- `feature/portfolio-page` 分支为**用户指定长期保留**，与 `.agents/AGENT.md` 中「合并后删除特性分支」的默认规则冲突时，以保留为准。

## 8. 不要做的事

1. 不要引入构建工具、框架、包管理器或任何第三方依赖。
2. 不要为了「整洁」重排 `WORKS` 数组顺序 —— 数组顺序即页面展示顺序，`code` 编号需与之对应。
3. 不要删除 `assets/` 下的素材来「精简仓库」，素材与 `WORKS` 存在强引用关系。
4. 不要直接向 `main` 提交，流程见 `.agents/AGENT.md`。
5. 不要用 `--ff-only` / fast-forward 方式合并分支，会丢失分支结构痕迹。
6. 不要在没有重新导出 PDF 的情况下修改 `resume.html`。
