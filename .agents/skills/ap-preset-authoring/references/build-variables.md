# 构建变量

构建变量让多个模组共同“拼出”一段内容：预设在 `preset.yaml` 中声明变量，每个**启用的**模组在 frontmatter 的 `vars` 中往里添加内容，提示词用 `{{ap::var::<id>}}` 读出最终结果。典型用途：问题清单、注意事项清单、输出格式的字段列表。

## 声明

```yaml
vars:
  considerations:
    type: dict
    label: Planning considerations
    description: Points the narrator weighs while planning the next scene.
  styleNotes: { type: string, separator: "\n" }
  tags: list          # 简写：只写类型
```

| 字段 | 说明 |
| --- | --- |
| 键 | 变量 id：字母开头，字母、数字、`_`、`-` |
| `type` | `string` \| `list` \| `dict` |
| `label` / `description` | 面板与用户模组编辑器中显示 |
| `separator` | 仅 `string`：多次追加之间的连接符，默认无 |

## 更新

模组 frontmatter：

```yaml
vars:
  considerations:
    pacing: Is the scene moving at the right speed?
    outline: null          # 删除前面模组设置的键
  styleNotes: Prefer short sentences.
  tags: [pacing, prose]    # 或单个值：tags: pacing
```

| 类型 | 更新写法 | 效果 |
| --- | --- | --- |
| `string` | 标量 | 追加文本（非空时用 `separator` 连接） |
| `list` | 标量或标量列表 | 追加项 |
| `dict` | 键 → 标量或 `null` | 设置键；`null` 删除键；后面的模组覆盖前面的值，但键保持第一次出现的位置 |

值只能是标量（字符串、数字、布尔），会转成字符串；不支持嵌套。更新按**模组顺序**应用（预设模组在前，用户模组在后），未启用的模组不参与。值里可以写宏，读出时展开（变量之间可以相互引用，最多 4 层）。

## 读取

| 写法 | 结果 |
| --- | --- |
| `{{ap::var::<id>}}`（string） | 拼好的文本 |
| `{{ap::var::<id>}}`（list） | YAML 列表 `- 项` |
| `{{ap::var::<id>}}`（dict） | YAML 映射 `键: 值` |
| `{{ap::var::<id>::keys}}` | 字典的键，YAML 列表 |
| `{{ap::var::<id>::values}}` | 字典的值，YAML 列表 |
| `{{ap::var::<id>::<key>}}` | 字典某个键的值，不存在为空 |
| `{{ap::var::<id>::<n>}}` | 列表第 n 项（从 0 开始，负数从末尾数），不存在为空 |

YAML 输出规则：能安全写成普通标量的值原样输出；否则用 JSON 字符串加引号（例如以特殊字符开头、含 `: `、像布尔或数字的值）；多行值写成 `|` 块。

例如两个模组分别设置 `considerations.pacing` 与 `considerations.tone`，`{{ap::var::considerations::values}}` 输出：

```yaml
- Is the scene moving at the right speed?
- Does the tone match the genre?
```

## 用户模组

用户模组也可以更新构建变量（面板中的“变量”部分），在预设模组之后应用。字典类型下，空值表示删除该键，因此用户可以去掉预设内置的某一项。键名保持稳定很重要：用户模组按键名引用。

## 诊断

- 未声明的变量、类型不符的更新、嵌套值报错；
- 声明了但从未被 `{{ap::var::…}}` 读取的变量给出警告；
- 同一个字典键被**可能同时启用**的多个模组设置时给出警告（后面的模组胜出）；同一单选集的成员不算；
- `keys` / `values` 是保留选择器，作为字典键时无法单独读取（警告）。
