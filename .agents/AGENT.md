# .agents/AGENT.md · 协作与技能规范

代理的协作流程、技能调用规则与执行边界。项目形态、目录与数据流约束见 [`../AGENT.md`](../AGENT.md)。

## 1. 本目录

```
.agents/
├── AGENT.md                        # 本文件：协作流程与执行规范
└── skills/
    ├── git-commit-info/SKILL.md    # 提交信息（gitmoji + 约定式提交，中文）
    └── github-flow/SKILL.md        # GitHub Flow 分支工作流
```

规范来源，非代码目录；临时脚本与校验产物放 `.agents/tmp/`（已忽略）；`.pi/` 属本地会话数据，不读不提交。

## 2. 技能

| 技能 | 调用时机 | 产出 | 边界 |
| --- | --- | --- | --- |
| `git-commit-info` | 任何 `git commit` 之前 | 提交信息（标题 + 三段式 body + footer） | 只生成文案，不执行 git |
| `github-flow` | 分支创建、推送、PR、合并、清理 | 分支名、PR 说明、合并方案 | 破坏性操作先获确认 |

改动前读对应 `SKILL.md` 全文，按其模板产出，不凭记忆简化。

## 3. 工作流（GitHub Flow）

`同步 main → 切分支 → 改动自测 → 提交 → 推送 → PR → --no-ff 合并 → 清理`

仓库实情：主分支 `main`、远端 `origin`、无 CI / PR 模板 / `.github` 目录。

- **分支**：`<type>/<描述>`，全小写、`-` 连接，不带人名与日期；`feature` `fix` `docs` `refactor` `chore`。
- **提交**：小步、单一意图、可独立回退；信息按 `git-commit-info` 模板，`type` 仅限 `init` `feat` `fix` `docs` `style` `refactor` `perf` `test` `build` `chore` `revert`。
- **合并**：必须 `git merge --no-ff`，保留分支结构；合并前用 `git diff main <分支>` 核对，避免夹带无关改动；合并提交建议 `🔧 chore(分支合并): 将 <分支> 合并进 main`。
- **清理**：`git branch -d <分支>` + `git push origin --delete <分支>`，远端只留 `main`。

## 4. 执行约束

| 类别 | 约束 |
| --- | --- |
| 语言 | 全程简体中文，仅代码、命令、路径、报错原文保留英文 |
| 写入 | 只改文件用 `edit` / `write`；禁止脚本、`sed`、重定向回写 |
| 校验 | 只读，走 stdin（`node - <<'JS'`），不落地临时文件 |
| 破坏性操作 | `push --force`、合并、变基、删分支、`reset --hard` 前说明命令并获确认 |
| 工作区保护 | 不静默丢弃未提交改动，先询问（stash / 提交 / 放弃） |
| 依赖 | 不新增依赖、构建步骤或配置文件 |

## 5. 交付前检查清单

- [ ] `node --check js/main.js` 通过
- [ ] HTML 闭合与资源引用无缺失（脚本见根 `AGENT.md` 5.4）
- [ ] 视频 `meta` 两段式、`file` 为多吉云永久直链且与 `m3u8_permalinks.json` 一致
- [ ] 改动 `resume.html` 时已重新导出 PDF 并核对 A4 单页
- [ ] 改动 CSS / JS 时已递增 `index.html` 的 `?v=`
- [ ] 无 `YOUR-USERNAME`、旧姓名、旧联系方式残留
- [ ] 提交信息符合模板，body 三段式与 footer 均非空

## 6. 禁止行为

1. 直接向 `main` 提交或推送。
2. fast-forward 合并。
3. 未获确认即合并、变基、强推、删分支、`reset --hard`。
4. 在生成提交信息之外，替用户执行未授权的 git 操作。
5. 跳过 PR 与自测直接合并；用同一分支承载多个不相关改动。
6. 提交 `.pi/`、`.agents/tmp/` 或任何会话数据、临时脚本。

## 7. 上线校验

首页 200、`assets/og-cover.png` 可访问、视频与 PDF 可加载、`canonical` / `og:url` 指向 https://helix.ln.cn/ 。
