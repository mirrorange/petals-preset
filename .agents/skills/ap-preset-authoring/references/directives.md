# 指令

指令写成 HTML 注释：`<!-- @name 参数 -->`。它们在编译时被识别并移除；**其他普通注释会原样保留为提示词文本**（例如花瓣的 `<!-- Workflow Start -->` 就是给模型看的标记）。

```
<!-- @entry id=top region=top order=100 when='transport == "prefill"' -->   切分模组文件
<!-- @section stage:final style=imperative -->                              切分工作流文件
<!-- @as assistant -->                                                       开始一条新消息
<!-- @style dialogue --> … <!-- @style imperative --> … <!-- @end -->       按写法选择片段
<!-- @if params.x != "" --> … <!-- @else --> … <!-- @end -->                 条件片段
```

## 通用规则

- 指令名是小写字母开头的单词，后接参数。参数分为**位置参数**（`assistant`、`stage:final`）和**属性**（`key=value`、`key="带空格的值"`、`key='值'`，引号内可用 `\` 转义）。
- 独占一行的指令会连同换行一起移除；行内指令只移除注释本身。
- **围栏代码块**（以 ` ``` ` 或 `~~~` 开头的行之间）里的指令是普通文本，不会被执行。
- 未知指令、不匹配的 `@else` / `@end`、未闭合的 `@if` / `@style` 都会报错并给出行号。

## 切分指令

| 指令 | 文件 | 说明 |
| --- | --- | --- |
| `@entry` | 模组 | 开始一个新条目，属性 `id` `region` `order` `when`（见 [modules.md](modules.md#条目)） |
| `@section` | 工作流 | 开始一个新分节，位置参数 `preamble` \| `intro` \| `outro` \| `stage:<id>`，属性 `style`（见 [workflows.md](workflows.md#分节-section)） |

切分指令只在各自的文件类型中有效；在用户模组文本中不可用。

## `@as` — 开始新消息

```markdown
<!-- @as user -->
{{ap::prefix::user}}Let's begin.
<!-- @as assistant -->
{{ap::prefix::narrator}}Understood.
```

- 参数：`system` \| `user` \| `assistant`。
- 从这里开始一个新的消息块，角色为指定值；之前的内容是另一个块（使用区域的默认角色）。
- 在 `merge: join` / `wrap` 的区域里，`@as` 块不参与合并，单独成为一条消息。
- 不能放在 `@if` 或 `@style` 内部（报错）。如果要按条件决定是否有这条消息，把整个条目或分节加上 `when`，或者让块内容为空（空块会被丢弃）。

## `@style` — 按写法选择片段

```markdown
Suggest four next inputs for {{user}}.
<!-- @style dialogue -->
Write them after the story in the format below.
<!-- @style imperative -->
Put them in the `options` field.
<!-- @end -->
```

- 参数：`dialogue` \| `imperative`。等价于 `@if style == "dialogue"`。
- 紧接着的另一个 `@style` 表示**切换**（不是嵌套）；最后用 `@end`（或 `@style end`）结束。
- 可以与 `@if` 互相嵌套。

## `@if` / `@else` / `@end` — 条件片段

```markdown
<!-- @if params.length > 0 -->
Aim for about {{ap::param::length}} words.
<!-- @else -->
Choose a length that fits the scene.
<!-- @end -->
```

- `@if` 后面直接写条件表达式（见 [expressions.md](expressions.md)），不需要引号。
- 可以嵌套；每层 `@if` 最多一个 `@else`。
- 条件在每次组装时求值；为假的片段被移除。片段移除后若块内只剩空白，整个块被丢弃。

## 指令可用的位置

| 位置 | `@entry` | `@section` | `@as` | `@style` | `@if` |
| --- | --- | --- | --- | --- | --- |
| 模组正文 | ✓ | | ✓ | ✓ | ✓ |
| 工作流正文 | | ✓ | ✓ | ✓ | ✓ |
| 用户模组条目文本 | | | ✓ | ✓ | ✓ |
| `preset.yaml` 中的文本字段 | | | | | |

`preset.yaml` 里的文本（结构格式说明、工具描述等）只做宏展开，不解析指令；需要按写法区分的地方，AP 已经提供了成对的字段（如结构的 `formatPrompt.tag` / `formatPrompt.field`）。
