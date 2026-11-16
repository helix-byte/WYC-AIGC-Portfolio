# .agents/AGENT.md · 代理协作与技能规范

本文件定义本仓库中 AI 代理的协作流程、技能调用规则与执行边界。项目形态、目录结构与数据流约束见仓库根目录 [`../AGENT.md`](../AGENT.md)。

## 1. 本目录职责

```
.agents/
├── AGENT.md                        # 本文件：协作流程与执行规范
└── skills/
    ├── git-commit-info/SKILL.md    # 提交信息生成（gitmoji + 约定式提交，中文）
    └── github-flow/SKILL.md        # GitHub Flow 分支工作流
```

- 本目录是**规范来源**，不是代码目录；不要在此放置构建产物或临时文件。
- 临时脚本、校验产物放 `.agents/tmp/`（已在 `.gitignore` 中忽略）。
- `.pi/` 为代理本地会话数据，**一律不读取、不提交**。

## 2. 技能索引与调用时机

| 技能 | 何时调用 | 产出 | 注意 |
| --- | --- | --- | --- |
| `git-commit-info` | 任何 `git commit` 之前 | 一条完整提交信息（标题 + body 三段式 + footer） | 只生成文案，**不执行** git 操作 |
| `github-flow` | 涉及分支创建、推送、PR、合并、清理 | 分支名、PR 说明、合并方案 | 破坏性操作必须先获用户确认 |

调用方式：改动前先读对应 `SKILL.md` 全文，按其模板产出，不凭记忆简化。

## 3. 标准工作流（GitHub Flow）

本仓库实情：主分支 `main`、远端 `origin`、**无 CI、无 PR 模板、无 .github 目录**。

```
确认 main 与远端同步 → 切特性分支 → 改动 + 自测 → 提交 → 推送 → PR → --no-ff 合并 → 清理
```

### 3.1 分支命名

`<type>/<描述>`，全小写、`-` 连接，不带人名与日期：

- `feature/xxx` 新功能 · `fix/xxx` 修复 · `docs/xxx` 文档 · `refactor/xxx` 重构 · `chore/xxx` 杂项

示例：`feature/works-data-replace`、`fix/resume-pdf-export`。

### 3.2 提交

- 小步提交，每次提交单一意图、可独立回退。
- 提交信息严格按 `git-commit-info` 技能模板：

```text
<gitmoji> <type>(<scope>): <subject>

- 背景：<一句话，30 字内>
- 改动：
  - <文件/模块>：<改动点，40 字内>
  - <文件/模块>：<改动点，40 字内>
- 影响：<一句话>

需求关联：<关联说明、自测情况>
```

- `type` 仅限：`init` `feat` `fix` `docs` `style` `refactor` `perf` `test` `build` `chore` `revert`。

### 3.3 合并

- **禁止 fast-forward**：必须 `git merge --no-ff`，保留合并提交与分支结构痕迹。
- 合并提交信息同样遵守提交规范，推荐 `🔧 chore(分支合并): 将 <分支> 合并进 <主分支>`。
- 合并前确认 `git diff main <分支>` 与预期一致，避免夹带无关改动。

### 3.4 清理（含当前例外）

默认规则：合并后删除本地与远端特性分支。

**当前例外**：`feature/portfolio-page` 被用户明确要求**长期保留**，不删除、不合并前 `-d` 清理。新增分支按默认规则处理，除非用户另行说明。

## 4. 执行约束

| 类别 | 约束 |
| --- | --- |
| 语言 | 全程简体中文，仅代码、命令、路径、报错原文保留英文 |
| 写入 | 只能使用 `edit` / `write` 工具改文件；禁止脚本、`sed`、重定向回写 |
| 校验 | 只读校验，走 stdin 执行（`node - <<'JS'`），不落地临时文件 |
| 破坏性操作 | `push --force`、合并、变基、删除分支、`reset --hard` 前必须说明命令并获确认 |
| 工作区保护 | 不得静默丢弃未提交改动，先询问（stash / 提交 / 放弃） |
| 依赖 | 不得新增任何依赖、构建步骤或配置文件 |

## 5. 交付前检查清单

- [ ] `node --check js/main.js` 通过
- [ ] HTML 标签闭合、资源引用无缺失（校验脚本见根 `AGENT.md` 5.4）
- [ ] `WORKS` 中视频条目 `meta` 为两段式（`画质 · 时长`），`file` 为多吉云永久直链且与 `assets/videos/m3u8_permalinks.json` 一致
- [ ] 改动涉及 `resume.html` 时，已重新导出 `assets/王艺纯-简历.pdf` 并核对为 A4 单页
- [ ] 改动涉及 CSS / JS 时，已递增 `index.html` 中的 `?v=` 版本号
- [ ] 站内无 `YOUR-USERNAME`、旧姓名、旧联系方式等残留
- [ ] 提交信息符合模板，`body` 三段式与 `footer` 均非空

## 6. 禁止行为

1. 直接向 `main` 提交或推送改动。
2. 以 fast-forward 方式合并，导致历史中缺少合并提交。
3. 未经确认执行合并、变基、强制推送、删除分支、`reset --hard`。
4. 在生成提交信息之外，替用户执行其未授权的 git 操作。
5. 为了「省事」跳过 PR 与自测直接合并。
6. 提交 `.pi/`、`.agents/tmp/` 或任何会话数据、临时脚本。
7. 用同一分支承载多个不相关改动。

## 7. 常见场景手册

### 7.1 修改页面文案 / 联系方式

```bash
git checkout main && git pull
git checkout -b fix/contact-info
# 用 edit 工具改 index.html 与 resume.html（注意两处都要改）
# 校验 → 提交 → 推送 → PR
```

提交前 `grep` 旧值确认清零，再走 5. 检查清单。

### 7.2 替换作品数据

`js/main.js` 的 `WORKS` 是唯一数据源。替换时同步处理：图片 / 剧本文件落地到 `assets/` 对应子目录、路径逐字一致、视频封面同名、图片转 WebP；视频本身不落地仓库，改为把多吉云**永久直链**写入 `file`，并同步 `assets/videos/m3u8_permalinks.json`。改完用本地静态服务器打开 `index.html` 验证胶片条与弹窗播放（`file://` 下视频可能受跳域限制）。

### 7.3 修改简历版式

改 `resume.html` → Chrome headless 导出 PDF → 截图核对单页 A4 → 一并提交两个文件（源文件与 PDF）。

### 7.4 上线校验

线上核对：首页 200、`assets/og-cover.png` 可访问、视频与 PDF 能加载、`canonical` / `og:url` 指向 https://helix.ln.cn/ 。
