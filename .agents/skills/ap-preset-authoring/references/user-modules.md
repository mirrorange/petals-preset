# 插入点与用户模组

**用户模组**是用户在面板里自己写的提示词片段：不需要改预设源码，就能在预设留好的位置加入文字、更新构建变量，并设置触发条件。预设通过**插入点**决定用户能把内容放在哪里。

用户模组保存在 AP 的用户设置中（按预设分开），预设更新后仍然保留。

## 插入点 `insertionPoints`

```yaml
insertionPoints:
  - id: guidelines
    label: Guidelines
    description: Inside the guidelines block, after the built-in guidelines.
    region: guidelines
    order: 2000
  - id: after-history
    label: After history
    region: task
    order: 0
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | — | 必填，小写 kebab-case（`^[a-z0-9][a-z0-9-]*$`），唯一。发布后保持稳定（改名用[迁移](configuration.md#迁移)） |
| `label` | `id` | 用户模组编辑器中的名字（可在 `locales` 翻译） |
| `description` | — | 编辑器中的说明 |
| `region` | — | 必填，目标区域；不能是原生区域（错误） |
| `order` | `1000` | 在区域内的顺序，与模组条目的 `order` 同一尺度 |

没有声明插入点的预设，用户模组只能更新构建变量。

**选择插入点的建议**：在每个用户可能想补充内容的区域留一个插入点，`order` 放在内置内容之后（例如内置条目用 100–1000，插入点用 2000）；在历史前后各留一个，便于用户放“背景设定”与“本轮提醒”。

## 用户模组的结构

```json
{
  "id": "k3x9",
  "name": "Rainy city",
  "enabled": true,
  "entries": [
    { "id": "e1", "point": "guidelines", "text": "It is always raining in this city." },
    { "id": "e2", "point": "after-history", "role": "user", "text": "<!-- @if transport == \"prefill\" -->Remember the rain.<!-- @end -->" }
  ],
  "vars": [
    { "var": "questions", "type": "list", "items": ["Is it still raining?"] }
  ],
  "trigger": {
    "match": "all",
    "rules": [
      { "kind": "var", "path": "modules.options", "op": "on" },
      { "kind": "keyword", "target": "recent", "depth": 3, "words": ["rain", "雨"], "op": "any" }
    ]
  }
}
```

| 字段 | 说明 |
| --- | --- |
| `id` | 用户模组 id。合并进预设时变为 `user-<id>` |
| `name` | 显示名 |
| `enabled` | 是否启用 |
| `entries[]` | 条目：`id`、`point`（插入点 id）、`role`（可选：`system` / `user` / `assistant`）、`text` |
| `vars[]` | 构建变量更新，见下 |
| `trigger` | 触发条件，见下 |

### 条目

- 条目放在插入点的区域与 `order` 上，排在**同区域、同 `order` 的预设条目之后**；多个用户模组按列表顺序排列；
- 没有 `role` 的条目与区域中的其他文字合并；有 `role` 时成为单独的消息（等同于 `@as`）；
- `text` 支持 `{{ap::…}}` 宏、ST 宏，以及 `@if` / `@as` / `@style` 指令（[directives.md](directives.md)），不支持 `@entry` / `@section`；
- 插入点不存在（例如预设更新后被删除）的条目被跳过，并在调试信息中警告；空文字的条目忽略。

### 构建变量更新

| `type` | 字段 | 效果 |
| --- | --- | --- |
| `string` | `text` | 覆盖字符串变量（空文字忽略） |
| `list` | `items: string[]` | 追加列表项（空项忽略） |
| `dict` | `items: [{ key, value }]` | 设置字典键；`value` 为空或 `null` 时**删除**该键 |

只有预设声明过、且类型一致的变量会被更新；用户模组的更新在所有预设模组之后应用（[build-variables.md](build-variables.md#用户模组)）。

### 触发条件

```json
{ "match": "all", "rules": [ … ] }
```

`match`：`all`（全部成立）或 `any`（任一成立）。**没有完整规则时总是生效**。规则有三种：

| `kind` | 字段 | 说明 |
| --- | --- | --- |
| `var` | `path`、`op`、`value` | 比较条件变量（[expressions.md](expressions.md)）。`op`：`on`（为真）、`off`（为假）、`==` `!=` `>` `>=` `<` `<=`（与 `value` 比较）、`in` / `notIn`（`value` 为数组） |
| `keyword` | `target`、`depth`、`words`、`regex`、`op` | 在本次输入（`input`）或最近 `depth` 条消息（`recent`，默认 3，含本次输入）中查找词语。默认不区分大小写的子串匹配；`regex: true` 时每个词是正则（不区分大小写）。`op`：`any`（出现任一）或 `none`（都不出现） |
| `expr` | `source` | 原始条件表达式，语法错误时永不触发 |

“不完整”的规则（没有词、没有值、空表达式）被忽略。关键词规则在组装前根据聊天文本求值（看到的是用户实际写的内容，元指令剥离之前）。

用户模组合并后是 `toggle: always` 的模组，触发条件就是它的 `when`。

## 导入导出格式

面板可以导出 / 导入用户模组文件（建议扩展名 `.ap-modules.json`）：

```json
{
  "format": "advanced-preset.user-modules",
  "version": 1,
  "preset": { "id": "petals", "version": "7.6.0" },
  "modules": [ { "id": "k3x9", "name": "Rainy city", "enabled": true, "entries": [], "vars": [], "trigger": { "match": "all", "rules": [] } } ]
}
```

- 导入时数据被宽松规范化（缺失的条目 id 补为 `e1`、`e2`……，非法规则丢弃），作为**新副本**加入，不覆盖已有模组；
- 文件来自预设的较旧版本时，先按预设的[迁移](configuration.md#迁移)重命名插入点、变量与条件中引用的模组 / 参数。

预设作者可以提供示例用户模组文件，用 `ap preview --user-modules <file>` 检查它们的效果（预览时关键词规则看到的是空文本，见 [cli.md](cli.md)）。
