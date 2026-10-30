# 模板

`@template` 把一段内容标记为**模板**：每次组装时运行一段 JavaScript，用它的输出替换这段内容。语法与 [EJS](https://ejs.co/) 完全相同，能读取生成参数（`params`、`gen`、`meta`……）与构建变量（`vars`）。条件表达式与宏表达不了的逻辑（循环、计算、字符串处理）可以写成模板。

```markdown
<!-- @template -->
<% const qs = Object.entries(vars.questions); -%>
Before writing, answer these <%- qs.length %> questions:
<% qs.forEach(([key, q], i) => { -%>
<%- i + 1 %>. <%- q %>
<% }) -%>
<!-- @end -->
```

> **模板是可执行代码**。它在 SillyTavern 页面中运行，能做页面上的 JavaScript 能做的任何事。安装含模板的预设、导入含模板的用户模组时，AP 会显示代码并请用户确认信任（见[信任](#信任)）。只在条件、宏与构建变量不够用时才使用模板。

需要 AP 1.9.0 或更高版本：含模板的预设应设置 `minAdvancedPreset: 1.9.0`（未设置时 `ap check` 警告；旧版本会把模板源码当作普通文本发送）。

## 写法

```markdown
<!-- @template -->
…EJS 模板…
<!-- @end -->
```

- `@template` 没有参数，用 `@end` 结束。可以独占一行（连同换行一起移除），也可以写在行内：`第 <!-- @template --><%- gen.type %><!-- @end --> 次`。
- 模板体**原样保留**：其中的普通 HTML 注释是模板文本，不能再写指令（`@if`、`@as`、`@style`、`@template` 等，报错）。作者注释 `<!--# … -->` 仍会被移除。
- 模板可以放在 `@if` / `@style` 里，条件为假时不运行。
- 未闭合的 `@template` 报错。
- 可用位置：模组正文、工作流正文、用户模组条目文本；`preset.yaml` 里的文本字段不支持（见 [directives.md](directives.md#指令可用的位置)）。

## 语法

与 EJS 3 默认选项相同：

| 标签 | 作用 |
| --- | --- |
| `<% code %>` | 运行代码，不输出 |
| `<%_ code %>` | 同上，并去掉标签前同一行的空白 |
| `<%- value %>` | 输出值（原样） |
| `<%= value %>` | 输出值，**HTML 转义**（`<` → `&lt;`、`"` → `&#34;` 等） |
| `<%# comment %>` | 模板注释，不输出 |
| `<%%` / `%%>` | 输出字面的 `<%` / `%>` |
| `%>` | 结束标签 |
| `-%>` | 结束标签，并去掉紧随其后的一个换行 |
| `_%>` | 结束标签，并去掉其后同一行的空白 |

- 提示词不是 HTML，**输出用 `<%-`**；`<%=` 会把 `<tag>`、引号转义成实体，只在确实需要时使用。
- `undefined` / `null` 输出为空。
- 代码以非严格模式运行在 `with (locals) { … }` 中：作用域变量可以直接读（`params.length`），也可以通过 `locals` 读（`locals.params.length`）。
- 不支持 `include()`、自定义分隔符与 EJS 选项。

## 作用域

| 名称 | 内容 |
| --- | --- |
| `transport` `style` `workflow` | 当前传输方式、写法、工作流 id |
| `gen` `meta` `modules` `params` `flags` `features` `stages` `chat` `memory` `recap` | 与[条件变量](expressions.md#条件变量)相同的对象，如 `gen.type`、`params.length`、`modules["mode-preface"]`、`stages.review`；列表参数是字符串数组 |
| `vars` | 构建变量：字符串变量是字符串，列表是数组，字典是对象（键 → 值），见 [build-variables.md](build-variables.md) |
| `sampling` | 当前配置的 [`sampling`](configuration.md#sampling)（`temperature`、`top_p` 等；配置中没有的键不存在） |
| `macro(name, ...args)` | 展开一个 AP 宏，例如 `macro("param", "length")` 等于 `{{ap::param::length}}`；未知宏返回空字符串 |

- 作用域是**只读**的（深度冻结），修改会被忽略。需要可变的数据时先复制：`[...vars.tags]`。
- 同一次组装中所有模板共用同一份作用域快照；不要依赖模板之间传值。
- 构建变量与条件变量都是组装时的值；`ap preview` 中运行期数据（记忆、总结等）为空，与条件相同。

## 输出与展开顺序

1. 组装时先按 `when` / `@if` / `@style` 取舍片段；
2. 运行模板，输出替换模板体；
3. 整段内容展开 `{{ap::…}}` 宏（模板输出里的宏也会展开），然后交给 SillyTavern 展开它的宏。

因此模板可以输出宏，也可以直接调用 `macro()` 拿到展开后的文本。片段与模板都输出空白时，整个块被丢弃（与 `@if` 相同）。

## 出错时

- **编译期**（`ap build` / `ap check`）：未闭合的 `<%` 标签、JavaScript 语法错误报错，并给出行号。语法错误只能定位到模板开头。
- **组装期**：运行时异常（例如读取 `undefined` 的属性）不会中断生成：这段模板输出为空，`ap preview` 显示警告 `template in entry <id> (<file>:<line>): …`，扩展在控制台记录警告并提示一次。
- 模板里的字面文本照常检查宏（未知 `{{ap::…}}` 报错）。

## 诊断

`ap check` 对模板代码做以下检查（静态匹配，动态访问无法检查）：

- `minAdvancedPreset` 未设置或低于 1.9.0 时警告；
- 读取不存在的 `vars.<id>`、`params.<id>`、`modules.<id>`、`flags.<id>`、`stages.<id>` 时警告（未知 id 在运行时只是 `undefined`，所以是警告而不是错误）；
- 模板读取的构建变量算作“已读取”；用动态方式访问 `vars`（如 `vars[name]`、`Object.keys(vars)`）时，所有构建变量都算作已读取。

## 信任

模板在用户的浏览器里运行，所以 AP 只运行用户确认过的代码：

- **安装预设**（导入文件、导入链接、从来源更新、从 ST 预设导入）时，如果产物包含模板（含[随预设安装的用户模组与用户模组模板](user-modules.md#预设中的用户模组)中的模板，各语言版本都计入），AP 弹窗列出全部模板代码，用户选择“信任并安装”才会安装；取消则不安装。
- **更新**时，如果模板代码与已安装的版本完全相同，不再询问。
- **导入用户模组**时，如果文件包含模板，同样需要确认（“信任并导入”）。用户在面板里自己写的模板不需要确认。
- 内置预设与开发服务器（`ap dev`）载入的预设视为可信，不询问。
- 预设页给含模板的预设显示“代码”标记。

## 示例

按构建变量生成编号列表：

```markdown
<!-- @template -->
<% vars.rules.forEach((rule, i) => { -%>
<%- i + 1 %>. <%- rule %>
<% }) -%>
<!-- @end -->
```

按参数计算：

```markdown
<!-- @if params.length > 0 -->
<!-- @template -->
Aim for <%- Math.round(params.length * 0.9) %>–<%- Math.round(params.length * 1.1) %> words.
<!-- @end -->
<!-- @end -->
```

合并内置项（构建变量）与用户填写的[列表参数](manifest.md#列表参数-type-list)，去重后逐项列出；全部为空时不输出任何内容（空块会被丢弃）：

```markdown
<!-- @template -->
<%_ const words = [...new Set([...vars.bannedWords, ...params.bannedWords])]; _%>
<%_ if (words.length) { _%>
Never use these words: <%- words.join(", ") %>
<%_ } _%>
<!-- @end -->
```

根据生成类型与采样参数调整说明：

```markdown
<!-- @template -->
<% if (gen.type === "swipe") { -%>
This is a reroll: take the scene in a noticeably different direction.
<% } -%>
<% if ((sampling.temperature ?? 1) < 0.7) { -%>
Low temperature: vary sentence openings deliberately.
<% } -%>
<!-- @end -->
```
