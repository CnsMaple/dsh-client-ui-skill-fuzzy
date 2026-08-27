# dsh-client-ui-skill-fuzzy

DeepSeek Harness 的 web 端技能引用插件：把聊天框 `/` 菜单里的 **skill 候选**从前缀过滤（`startsWith`）升级为 **fzf 风格模糊匹配**，行为与命令面板（`/`）完全一致——大小写不敏感、子序列匹配、前缀/边界加分排序。

作为 `@deepseek-ai/dsh-client-ui-skill` 的 drop-in 替代品，node half 为空、浏览器 half 复用其实现，仅替换候选过滤逻辑（fuzzyScore / boundaryBonus 提取自命令面板源码）。

## 上游基线版本

| 项目 | 版本 | 说明 |
| --- | --- | --- |
| 官方上游 | `@deepseek-ai/dsh-client-ui-skill@0.1.1-rc.2` | 本插件内置其 `client.js` 基线 |
| 核对日期 | 2026-08-24 | 核对 npm 上该上游的 latest 版本 |
| 本插件版本 | `0.1.0` | 与 `plugins/dsh-client-ui-skill-fuzzy` 仓库 `origin/main` 同步 |

**核对方法**（上游发布新版本后）：

1. 查上游最新版：`npm view @deepseek-ai/dsh-client-ui-skill version`，与上表比对。
2. 若已发布新版本，运行 `npm pack @deepseek-ai/dsh-client-ui-skill@<new-version>` 解包，用其 `lib/client.js` 与本插件的 `lib/client.js` 做规范化对比（忽略 `id` / `tagId` / `dsh-client-ui-skill-fuzzy` 字样差异）。除 fuzzy 过滤逻辑（`boundaryBonus` / `fuzzyScore` / `fuzzyCandidates`，约 50 行）外应无其它差异；若有，把官方新增/修改同步进 `lib/client.js`（`candidates()` 内仍用 `fuzzyCandidates` 替换官方 `startsWith` 过滤）。
3. 更新后同步修订上表并以 `git commit` 记录。

> 注：官方上游即 `@deepseek-ai/dsh-client-ui-skill`（web 端 skill 菜单），而非 `@deepseek-ai/dsh-skill`（后者是 `/` 命令面板的包）。

## 效果示例

| 输入 | 内置（前缀） | 本插件（模糊） |
| --- | --- | --- |
| `/rpa` | 空 | yingdao-rpa → ecommerce-rpa-toolkit → karpathy-guidelines |
| `/ecrpa` | 空 | ecommerce-rpa-toolkit |
| `/commit` | commit-and-push-all | commit-and-push-all → conventional-commit → … |

## 目录结构

```
lib/
  index.js   # node half（无宿主行为）
  client.js  # 浏览器 half：candidates() 用 fuzzyScore 过滤
cordis.patch.yml  # bundle patch：禁用内置 ui-skill 并挂载自身
package.json
```

## 安装

### 方式 A：自动安装（推荐）

用 dsh 的插件命令（转发到 pnpm）安装到 web profile：

```bash
dsh plugin --profile web add "git+https://github.com/CnsMaple/dsh-client-ui-skill-fuzzy.git"
```

安装后确认依赖已登记：

```bash
cd ~/.dsh/profiles/web && pnpm ls dsh-client-ui-skill-fuzzy
```

后续更新版本：

```bash
dsh plugin --profile web up dsh-client-ui-skill-fuzzy
```

> 注意：本插件以 **bundle** 形式交付（`dsh.bundle.patch`），`dsh plugin add` 装包后会把它 reconcile 进 `dsh.profile.bundles` 并自动应用自带 patch，**无需**手动修改 `cordis.patch.yml`。

> 若本机 pnpm 报 lockfile / peer 冲突（如既有 `dsh-mobile` 依赖导致 ERESOLVE），可先提交 lockfile 变更或改用方式 B。

### 方式 B：手动安装

1. 把本包放入 `profiles/web/node_modules/dsh-client-ui-skill-fuzzy/`（或用 `pnpm add` 的等价手动方式）。
2. 在 `profiles/web/package.json` 的 dependencies 中登记：

```json
"dsh-client-ui-skill-fuzzy": "git+https://github.com/CnsMaple/dsh-client-ui-skill-fuzzy.git"
```

### 挂载（自动）

插件自带 `cordis.patch.yml`（禁用内置 `ui-skill` 前缀过滤、挂载自身），装包后由 bundle 层自动应用，**无需**手动修改 `profiles/web/cordis.patch.yml`。

从旧版（手动 insert）升级时清理：

1. 移除 `profiles/web/cordis.patch.yml` 里此前的两段：`- id: ui-skill / disabled: true` 与 `- insert: { id: ui-skill-fuzzy, ... }`，避免重复挂载。
2. 重启 `dsh --profile web`，`__DSH_BOOT__` 中应出现 `dsh-client-ui-skill-fuzzy` 且不再有内置 `@deepseek-ai/dsh-client-ui-skill`；聊天框输入 `/ecrpa` 应命中 ecommerce-rpa-toolkit。

## 注意

- 与命令面板一致：输入含空格（如 `/meegle b`）不匹配；仅匹配技能名称，不匹配描述。
- `exports` 必须暴露 `./package.json` 子路径，否则 `dsh-client-modules` 的 `require.resolve('<pkg>/package.json')` 报 `ERR_PACKAGE_PATH_NOT_EXPORTED` 导致插件不进 boot。

## License

MIT