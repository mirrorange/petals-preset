# 宏

AP 宏写作 `{{ap::名称::参数…}}`，参数用 `::` 分隔（两侧空白会被去掉）。它们可以出现在模组条目、工作流分节、用户模组文本，以及 `preset.yaml` 中大多数模型可见的文本字段里（前缀、`prefillGuide`、结构格式说明与描述、`tools` 的描述与结果文本、示范轮次文本等）。

**展开顺序**：先展开 AP 宏，再交给 SillyTavern 展开它自己的宏（`{{user}}`、`{{char}}`、`{{random}}` 等）。AP 宏的结果里可以再含宏，会继续展开（最多 8 层）。未知的 `{{ap::…}}` 原样保留，并被 `ap check` 报错。

## 宏一览

| 宏 | 结果 |
| --- | --- |
| `{{ap::name::<actor>}}` | 角色显示名（配置 `identity.names` 覆盖优先） |
| `{{ap::prefix::<actor>}}` | 角色前缀（配置 `identity.prefixes` → `actors.<id>.prefix` → `名字: `） |
| `{{ap::param::<id>}}` | 参数文本（未设置时走 `fallback` → `unset`；选项取 `text`），见 [manifest.md](manifest.md#参数-params) |
| `{{ap::lang}}` | 语言：有 `language` 参数且有值时为其文本，否则为 `the language used in the story so far` |
| `{{ap::structure::<id>}}` | 结构的格式说明：对话式为 `formatPrompt.tag`，指令式为 `formatPrompt.field` |
| `{{ap::structure::<id>::last}}` | 仅自定义结构：聊天中最近一次非空的值（`kv` 形式为 JSON） |
| `{{ap::workflow::summary}}` | 本轮工作流摘要 `Start -> … -> (End)` |
| `{{ap::workflow::hint}}` | 工作流的 `hint` |
| `{{ap::input::meta}}` | 本次输入中的元指令文本（多段用空行连接） |
| `{{ap::memory::chat}}` | 聊天记忆（JSON 对象，缩进 2） |
| `{{ap::memory::global}}` | 全局记忆（JSON 对象，缩进 2） |
| `{{ap::recap}}` | 当前生效的总结文本 |
| `{{ap::recent_meta}}` | 最近几次元指令及其回复（`用户名: 指令` / `叙述者名: 回复`，数量由 `meta.recentCount` 决定） |
| `{{ap::var::<id>}}` | 构建变量（见 [build-variables.md](build-variables.md)） |
| `{{ap::var::<id>::keys}}` / `::values` | 字典变量的键 / 值列表 |
| `{{ap::var::<id>::<key>}}` | 字典变量某个键的值 |
| `{{ap::var::<id>::<index>}}` | 列表变量的某一项（负数从末尾数） |

## 用法示例

```markdown
{{ap::prefix::user}}{{ap::name::narrator}}, please write in {{ap::lang}}.

<!-- @if memory.hasChat -->
Known facts about this chat:
{{ap::memory::chat}}
<!-- @end -->

{{ap::structure::options}}
```

## 注意

- 宏在**组装时**展开，`@if` 片段先按条件取舍再展开宏。
- 用户的元指令文本（`{{ap::input::meta}}`）里如果出现 `{{ap::…}}`，同样会被展开。
- 运行期数据（记忆、总结、最近元指令、自定义结构的 `last`）在 `ap preview` 中为空。
- `{{ap::name::…}}` 是唯一在“组装之前”也会展开的宏：元指令触发词 `meta.trigger` 与 `meta.impersonatePrompt` 只支持它。

## 诊断

- 未知宏名报错，例如 `{{ap::foo}}`；
- 参数个数不对报错（如 `{{ap::lang::x}}`）；
- 参数值无效报错：未知角色、参数、结构、构建变量；`workflow` 只接受 `summary` / `hint`；`input` 只接受 `meta`；`memory` 只接受 `chat` / `global`；只有自定义结构支持 `::last`；
- 构建变量选择器：字符串变量不接受选择器、列表变量只接受整数（报错）；字典变量读取了没有任何模组设置的键给出警告。
