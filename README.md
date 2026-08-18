# dsh-client-ui-skill-fuzzy

DeepSeek Harness 的 web 端技能引用插件：把聊天框 `/` 菜单里的 **skill 候选**从前缀过滤（`startsWith`）升级为 **fzf 风格模糊匹配**，行为与命令面板（`/`）完全一致——大小写不敏感、子序列匹配、前缀/边界加分排序。

作为 `@deepseek-ai/dsh-client-ui-skill` 的 drop-in 替代品，node half 为空、浏览器 half 复用其实现，仅替换候选过滤逻辑（fuzzyScore / boundaryBonus 提取自命令面板源码）。

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

> 注意：`dsh plugin add` 只负责把包装进 `profiles/web/node_modules` 并在 `package.json` 登记依赖，**不会自动修改 cordis.patch.yml**，仍需完成下面的「公共步骤」。

> 若本机 pnpm 报 lockfile / peer 冲突（如既有 `dsh-mobile` 依赖导致 ERESOLVE），可先提交 lockfile 变更或改用方式 B。

### 方式 B：手动安装

1. 把本包放入 `profiles/web/node_modules/dsh-client-ui-skill-fuzzy/`（或用 `pnpm add` 的等价手动方式）。
2. 在 `profiles/web/package.json` 的 dependencies 中登记：

```json
"dsh-client-ui-skill-fuzzy": "git+https://github.com/CnsMaple/dsh-client-ui-skill-fuzzy.git"
```

### 公共步骤（两种方式都要）

1. 在 `profiles/web/cordis.patch.yml`：

```yaml
# ===== 自定义 ui-skill：模糊候选 =====
- id: ui-skill
  name: '@deepseek-ai/dsh-client-ui-skill'
  disabled: true
- insert:
    - id: ui-skill-fuzzy
      name: dsh-client-ui-skill-fuzzy
```

2. 重启 `dsh --profile web`，`__DSH_BOOT__` 中应出现 `dsh-client-ui-skill-fuzzy` 且不再有内置 `@deepseek-ai/dsh-client-ui-skill`；聊天框输入 `/ecrpa` 应命中 ecommerce-rpa-toolkit。

## 注意

- 与命令面板一致：输入含空格（如 `/meegle b`）不匹配；仅匹配技能名称，不匹配描述。
- `exports` 必须暴露 `./package.json` 子路径，否则 `dsh-client-modules` 的 `require.resolve('<pkg>/package.json')` 报 `ERR_PACKAGE_PATH_NOT_EXPORTED` 导致插件不进 boot。

## License

MIT