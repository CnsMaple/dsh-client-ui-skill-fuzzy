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

## 装配到 web profile

1. 将本包放入 `profiles/web/node_modules/`（或 `pnpm add` 本 git 仓库），并在 `profiles/web/package.json` 登记依赖。
2. 在 `profiles/web/cordis.patch.yml`：

```yaml
# ===== 自定义 ui-skill：模糊候选 =====
- id: ui-skill
  name: '@deepseek-ai/dsh-client-ui-skill'
  disabled: true
- insert:
    - id: ui-skill-fuzzy
      name: dsh-client-ui-skill-fuzzy
```

3. 重启 `dsh --profile web` 后，`__DSH_BOOT__` 中应出现 `dsh-client-ui-skill-fuzzy` 且不再有内置 `@deepseek-ai/dsh-client-ui-skill`。

## 注意

- 与命令面板一致：输入含空格（如 `/meegle b`）不匹配；仅匹配技能名称，不匹配描述。
- `exports` 必须暴露 `./package.json` 子路径，否则 `dsh-client-modules` 的 `require.resolve('<pkg>/package.json')` 报 `ERR_PACKAGE_PATH_NOT_EXPORTED` 导致插件不进 boot。

## License

MIT