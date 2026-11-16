# AGENT.md · 项目协作说明

本文件面向在本仓库中工作的 AI 编码代理，说明项目形态、数据流与改动约束。协作流程与技能规范见 [`.agents/AGENT.md`](.agents/AGENT.md)。

## 1. 项目定位

「王艺纯 · AIGC 影视编辑」个人求职作品集网站 —— 纯静态、单页的求职材料（视频走 CDN 流式播放）。

| 项目 | 值 |
| --- | --- |
| 仓库 | `helix-byte/WYC-AIGC-Portfolio` |
| 线上地址 | https://helix.ln.cn/ |
| 主分支 | `main` |
| 部署 | GitHub Pages（`main` 分支 `/ (root)`），已上线可访问 |
| 使用场景 | 投递简历时附带链接；HR 打开即看，无需登录（视频播放需联网） |

## 2. 技术形态（改动前必读）

- **零构建、零依赖**：没有 `package.json`、没有 `node_modules`、没有打包器；不要引入 npm 依赖、框架、CSS 预处理器或构建步骤。
- **原生三件套**：`index.html` + `css/style.css` + `js/main.js`，`js/main.js` 是 IIFE，沿用 `var` / 函数声明式的 ES5 写法，新代码保持同风格。
- **资源本地优先**：页面不得引用外链字体、外部图片或第三方接口；剧本、封面、图片等素材必须是仓库内的本地文件。
- **视频例外（唯一外链）**：视频走多吉云视频云的 HLS 流，`js/main.js` 里是 `https://permalink.dogevideo.com/player/get.m3u8?...` 永久直链，播放器依赖 jsDelivr 上的 HLS.js（`<script src="https://cdn.jsdelivr.net/npm/hls.js@latest">`）。因此**视频播放需要联网**，其余区块仍可离线浏览。
- **双击即用**：`file://` 协议下页面必须能正常渲染（视频因跨域与协议限制，建议用本地静态服务器预览），改动后先按这个场景自测。

## 3. 目录与职责

```
├── index.html          # 首页：hero / 关于 / 制作链路 / 作品 / 实践与项目 / 教育 / 联系
├── resume.html         # 简历排版源文件（A4 单页，打印即 PDF）
├── css/style.css       # 全站样式（含首页 hero 背景图，第 95 行）
├── js/main.js          # 交互逻辑 + 作品数据 WORKS 数组
├── assets/
│   ├── videos/         # 视频永久直链清单（m3u8_permalinks.json；视频本体在 CDN）
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

三种记录结构：

```js
// 视频（file 为多吉云永久直链）
{id:"v1", type:"video", cat:"video", code:"REEL 01", title:"…", tag:"…",
 file:"https://permalink.dogevideo.com/player/get.m3u8?vcode=…&userId=…&flsign=…&ext=.m3u8",
 poster:"assets/posters/x.webp", meta:"720P · 01:50", desc:"…"}

// 剧本
{id:"s1", type:"script", cat:"script", code:"SCRIPT 01", title:"…", tag:"…",
 file:"assets/scripts/x.docx", sub:"《…》", meta:"DOCX · 点击打开", desc:"…"}

// 图片
{id:"a1", type:"image", cat:"asset", code:"CONCEPT 01", title:"…",
 src:"assets/images/ai/x.webp", meta:"分类 · 点击查看大图"}
```

**硬约束**：

1. 视频条目的 `meta` 必须是 `画质 · 时长` **两段**，`reelFrameHTML()` 用 `meta.split(" · ")[1]` 取时长，段数不足胶片条会显示 `undefined`。
2. 视频 `file` 必须是多吉云的**永久直链**（`permalink.dogevideo.com` 域名），不得写 `pl.dogevideo.com` 的带 `tm/sign` 临时地址（会过期）；并需与 `assets/videos/m3u8_permalinks.json` 保持同步。
3. `id` 全数组唯一，胶片条与弹窗都靠它关联。
4. `cat` 只能取 `video` / `script` / `asset` / `design`，与 `#filters` 的 `data-filter` 一一对应，新增分类需同步改 `index.html` 的筛选按钮与 `render()`。

## 5. 常见任务手册

### 5.1 修改个人信息

- 首页与简历页的姓名、求职意向、联系方式**分别写在两处**，`grep` 关键词逐一确认，避免只改一半。
- 联系方式要注意 `href="tel:"` / `href="mailto:"` 与按钮的 `data-copy` 属性三处同步。
- 简历 PDF 文件名在 `index.html` 出现 3 次（导航按钮、hero 按钮、联系区按钮）。

### 5.2 新增/替换作品

1. 视频：上传到多吉云视频云，在控制台复制该视频的**永久直链**（`https://permalink.dogevideo.com/player/get.m3u8?...`），登记到 `assets/videos/m3u8_permalinks.json`，并写入 `WORKS` 对应条目的 `file`。
2. 封面：放入 `assets/posters/`，与视频**同名**（webp）。
3. 剧本/图片：把文件放入 `assets/` 对应子目录（`scripts/`、`images/ai/`、`images/design/`）。
4. 在 `WORKS` 中增删记录，路径与文件名逐字一致。
5. 图片统一转 WebP。
6. 文件名允许中文，但不要含空格、`#`、`?`（会破坏 `file://` 与 URL 解析）。

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

const per=JSON.parse(fs.readFileSync('assets/videos/m3u8_permalinks.json','utf8'));
const vids=[...fs.readFileSync('js/main.js','utf8')
  .matchAll(/type:"video"[\s\S]*?title:"([^"]+)"[\s\S]*?file:"([^"]+)"/g)];
console.log('视频直链与 permalinks 不同步:',
  vids.filter(m=>!Object.values(per).includes(m[2])).map(m=>m[1]).join(' | ')||'无');
JS
```

### 5.5 改完 CSS / JS 要清缓存

`index.html` 尾部引用了带版本号的资源：

```html
<link rel="stylesheet" href="css/style.css?v=20261028b">
<script src="js/main.js?v=20261117"></script>
```

改动后请同步递增 `?v=` 值，否则线上用户会命中旧缓存。

## 6. 硬性约定

| 项目 | 约定 |
| --- | --- |
| 输出语言 | 全程简体中文（代码、路径、报错原文除外） |
| 文件写入 | 只能用 `edit` / `write` 工具，禁止用脚本或命令行重写文件内容 |
| 脚本执行 | 校验脚本走 stdin（`node - <<'JS'`），不落地临时文件 |
| 单文件体积 | ≤ 100MB（GitHub 限制），当前最大约 710KB（`assets/og-cover.png`） |
| 图片格式 | WebP，优先控制单图 < 500KB |
| 页面外链 | 仅视频 CDN（多吉云 HLS）与 HLS.js（jsDelivr）；其余资源禁止外链 |
| 兼容性 | 面向现代浏览器 + `file://` 双击打开 |

## 7. 当前已知状态（需持续更新）

- 视频已全部改为多吉云永久直链（清单见 `assets/videos/m3u8_permalinks.json`），仓库内不再保留 MP4；原 Python 取链脚本（`scripts/fetch_m3u8.py`）因只能取到临时地址已废弃删除。
- 站点已上线，线上地址返回 200；核对线上状态直接用 `curl -I` 访问页面与资源，勿依赖 GitHub Pages API（匿名请求可能返回 404，不能作为未启用判据）。
- `assets/og-cover.png` 约 710KB，社交平台抓取偏慢，可压缩至 200KB 以内。
- 历史分支 `feature/portfolio-page` 已完全合入 `main` 并从远端删除，当前远端仅保留 `main`，无长期保留分支。

## 8. 不要做的事

1. 不要引入构建工具、框架、包管理器或任何第三方依赖。
2. 不要为了「整洁」重排 `WORKS` 数组顺序 —— 数组顺序即页面展示顺序，`code` 编号需与之对应。
3. 不要删除 `assets/` 下仍在使用的素材来「精简仓库」，素材与 `WORKS` 存在强引用关系（视频已迁 CDN，`assets/videos/` 下的 MP4 属历史文件，可清理）。
4. 不要直接向 `main` 提交，流程见 `.agents/AGENT.md`。
5. 不要用 `--ff-only` / fast-forward 方式合并分支，会丢失分支结构痕迹。
6. 不要在没有重新导出 PDF 的情况下修改 `resume.html`。
