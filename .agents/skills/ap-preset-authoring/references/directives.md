# 指令

指令写成 HTML 注释：`<!-- @name 参数 -->`。它们在编译时被识别并移除；**其他普通注释会原样保留为提示词文本**（例如花瓣的 `<!-- Workflow Start -->` 就是给模型看的标记）。只写给作者看的说明用[注释](#注释)：`<!--# … -->`。

```
<!-- @entry id=top region=top order=100 when='transport == "prefill"' -->   切分模组文件
<!-- @section stage:final style=imperative -->                              切分工作流文件
<!-- @as assistant -->                                                       开始一条新消息
<!-- @style dialogue --> … <!-- @style imperative --> … <!-- @end -->       按写法选择片段
<!-- @if params.x != "" --> … <!-- @else --> … <!-- @end -->                 条件片段
<!-- @template --> <%- params.x %> <!-- @end -->                            EJS 模板（可执行代码）
<!--# 只给作者看的说明 -->                                                  注释（可跨行）
```

## 通用规则

- 指令名是小写字母开头的单词，后接参数。参数分为**位置参数**（`assistant`、`stage:final`）和**属性**（`key=value`、`key="带空格的值"`、`key='值'`，引号内可用 `\` 转义）。
- 独占一行的指令会连同换行一起移除；行内指令只移除注释本身。
- **围栏代码块**（以 ` ``` ` 或 `~~~` 开头的行之间）里的指令是普通文本，不会被执行。
- 未知指令、不匹配的 `@else` / `@end`、未闭合的 `@if` / `@style` / `@template` 都会报错并给出行号。

## 注释

以 `<!--#` 开头的 HTML 注释是**作者注释**：编译时连同内容一起移除，不执行其中的指令，不展开宏，也不发送给模型。`<!--` 与 `#` 之间可以有空白（`<!-- # 说明 -->` 同样是注释）。

```markdown
<!--# 单行注释：独占一行时连同换行一起移除 -->
Keep each scene focused. <!--# 行内注释：连同一侧的空白一起移除 -->

<!--#
多行注释：从 <!--# 到对应的 --> 为止。
<!-- @if params.length > 0 -->这里的指令不会执行<!-- @end -->
-->
```

- **单行**：`<!--# … -->` 可以独占一行，也可以写在行内。独占一行时整行（含换行）移除；行内注释移除注释本身和一侧的空白（`a <!--# x --> b` → `a b`），注释在行尾时去掉行尾空白。
- **多行**：`<!--#` 之后没有闭合时，注释延续到对应的 `-->`。注释前后的文字接在一起（`Hello <!--# …` ⏎ … ⏎ `--> world` → `Hello world`）；首行注释前与末行注释后都是空白时，这些行全部移除。
- **嵌套**：注释内部的 `<!-- … -->`（指令、普通注释）按对计数，所以把一段带指令的内容整段包进 `<!--# … -->` 就能停用它；要停用单个指令，在 `<!--` 后加 `#` 即可（`<!--# @if x -->`）。注意 Markdown 预览与编辑器的高亮会在第一个 `-->` 处结束注释，这只影响显示。
- **处理顺序**：注释先于指令移除，所以注释里的 `@entry` / `@section` 不会切分文件，围栏标记也不会开始代码块。
- **围栏代码块里的 `<!--#` 是普通文本**（与指令相同）；需要把 `<!--#` 原样发给模型时，放进围栏代码块。
- 注释没有闭合时报错，并给出起始行号。
- frontmatter 与 `preset.yaml` 是 YAML，用 YAML 的 `#` 注释。



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

## `@template` — 模板

```markdown
<!-- @template -->
<% vars.rules.forEach((rule, i) => { -%>
<%- i + 1 %>. <%- rule %>
<% }) -%>
<!-- @end -->
```

- 没有参数，用 `@end` 结束；模板体是 [EJS](templates.md) 模板，每次组装时运行，输出替换模板体，再展开宏。
- 模板体原样保留：里面不能写其他指令（报错），普通 HTML 注释是模板文本；作者注释 `<!--# … -->` 仍会移除。
- 可以放在 `@if` / `@style` 里；不能包含 `@as`。
- 模板是可执行代码，安装时需要用户确认信任。完整说明见 [templates.md](templates.md)。

## 指令可用的位置

| 位置 | `@entry` | `@section` | `@as` | `@style` | `@if` | `@template` |
| --- | --- | --- | --- | --- | --- | --- |
| 模组正文 | ✓ | | ✓ | ✓ | ✓ | ✓ |
| 工作流正文 | | ✓ | ✓ | ✓ | ✓ | ✓ |
| 用户模组条目文本 | | | ✓ | ✓ | ✓ | ✓ |
| `preset.yaml` 中的文本字段 | | | | | | |

注释 `<!--# … -->` 在模组正文、工作流正文与用户模组条目文本中都可用。

`preset.yaml` 里的文本（结构格式说明、工具描述等）只做宏展开，不解析指令；需要按写法区分的地方，AP 已经提供了成对的字段（如结构的 `formatPrompt.tag` / `formatPrompt.field`）。
