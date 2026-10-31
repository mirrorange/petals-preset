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
- `text` 支持 `{{ap::…}}` 宏、ST 宏，以及 `@if` / `@as` / `@style` / `@template` 指令与 `<!--# … -->` 注释（[directives.md](directives.md)），不支持 `@entry` / `@section`；
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
| `var` | `path`、`op`、`value` | 比较条件变量（[expressions.md](expressions.md)）。`op`：`on`（为真）、`off`（为假）、`==` `!=` `>` `>=` `<` `<=`（与 `value` 比较）、`in` / `notIn`（`value` 为数组）。列表参数用 `on`（有内容）/ `off`（为空） |
| `keyword` | `target`、`depth`、`words`、`regex`、`op` | 在本次输入（`input`）或最近 `depth` 条消息（`recent`，默认 3，含本次输入）中查找词语。默认不区分大小写的子串匹配；`regex: true` 时每个词是正则（不区分大小写）。`op`：`any`（出现任一）或 `none`（都不出现） |
| `expr` | `source` | 原始条件表达式，语法错误时永不触发 |

“不完整”的规则（没有词、没有值、空表达式）被忽略。关键词规则在组装前根据聊天文本求值（看到的是用户实际写的内容，元指令剥离之前）。

面板的规则编辑器按分组列出可选的条件变量；预设声明了 `inputShaping` 时，“生成”分组多出“输入已整形”（`input.shaped`）。

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
- 文件来自预设的较旧版本时，先按预设的[迁移](configuration.md#迁移)重命名插入点、变量与条件中引用的模组 / 参数（[模板](templates.md)代码不会被改写）；
- 文件包含模板时，导入前显示代码并请用户确认信任（见 [templates.md](templates.md#信任)）。

预设作者也可以把用户模组直接写进预设源码：随预设安装的[用户模组](#预设携带的用户模组-user-modules)，或新建时可选的[用户模组模板](#用户模组模板-user-module-templates)。两者用同一种[文件格式](#文件格式)。临时的模组文件可以用 `ap preview --user-modules <file>` 检查效果（预览时关键词规则看到的是空文本，见 [cli.md](cli.md)）。

## 预设中的用户模组

预设源码中有两个目录存放用户模组，写法完全相同：

```
user-modules/<id>.md                   随预设安装的用户模组（产物字段 userModules）
user-module-templates/<id>.md          用户模组模板：新建模组时可选的示例（产物字段 userModuleTemplates）
<目录>/<id>.<lang>.md                  整份翻译：面板语言为 <lang> 时使用
```

无论哪一种，用户最终得到的都是**普通的用户模组**：出现在「我的模组」中，可以随意修改、开关、排序、删除，与在面板中新建的模组没有区别。文件格式的表达力与面板编辑器相同——条目（插入点、角色、文字）、构建变量更新与触发条件（变量、关键词、表达式规则）都能写出来。

### 文件格式

```markdown
---
name: Custom writing style
description: Your own style guide, plus questions to plan with.
point: writing-style
when: '!meta.pure && gen.type != "impersonate"'
vars:
  optionalQuestions:
    custom-style: Which details of this scene best carry the custom style?
---
<!--#
How to edit: replace the guide below with your own style …
-->
## Writing Style: Mine

…

<!-- @entry point=after-history role=user -->
Remember to keep the custom style.
```

| frontmatter | 说明 |
| --- | --- |
| `name` | 模组名，默认为 id（模板新建时同名模组已存在则加编号） |
| `point` / `role` | 第一个条目的插入点与角色（`role` 可省略，含义同用户模组条目） |
| `vars` | 构建变量更新，写法与[模组的 `vars`](build-variables.md) 相同（字典值为 `null` 表示移除该键） |
| `when` | 条件表达式，成为触发条件中的一条“表达式”规则 |
| `trigger` | 完整的触发条件（`{ match, rules }`，格式见[触发条件](#触发条件)，可写关键词规则）；与 `when` 二选一 |
| `enabled` | 仅 `user-modules/`：安装后是否开启，默认 `true`（翻译文件中忽略） |
| `description` | 仅 `user-module-templates/`：新建模组时选择列表中的说明 |

- **id** 取文件名，小写 kebab-case；frontmatter 中不需要写 `id`（写了且不同则报错）。两个目录的 id 互不相关。
- **条目**：正文开头到第一个 `<!-- @entry point=<插入点> role=<角色> -->` 之前是第一个条目（需要 frontmatter 的 `point`）；之后每个 `@entry` 行开始一个新条目。`@entry` 必须独占一行；围栏代码块与注释中的 `@entry` 是普通文本。
- **原样保存**：条目文字（含[注释](directives.md#注释) `<!--# … -->`、指令、宏）按原样写进产物，只去掉首尾空行；用户在编辑器里看到的就是源码，注释正好用来写修改说明（不会发送给模型）。编译器仍会检查其中的指令、宏与条件，错误带行号报告。
- **翻译**：`<id>.<lang>.md` 是同一模组的整份翻译（名称、说明、条目、变量与触发条件都以它为准），面板语言为 `<lang>`（或其基础语言，如 `zh` 对应 `zh-cn`）时使用；必须有同 id 的原文件。只想翻译名称（与模板说明）时，在 [locales](locales.md) 的 `userModules` / `userModuleTemplates` 段中写 `name` / `description` 即可。
- **检查**：插入点、角色、变量与类型、触发规则（含条件变量与取值）都与用户模组同样校验；未知的 frontmatter 键、写在不适用目录中的 `enabled` / `description` 警告。
- **代码**：其中的 `@template` 代码（各语言版本都计入）算作预设的模板代码，安装预设时一并请用户确认（[templates.md](templates.md#信任)）。
- 不升产物格式；旧版本 AP 会忽略这两个字段（不安装、不显示模板，不影响其他功能）。

### 预设携带的用户模组 `user-modules/`

适合“预设推荐、但应由用户掌控”的内容：例如一段可按喜好改写的设定补充、默认关闭的可选提醒。与普通模组不同，它们安装后就属于用户，用户可以直接修改提示词。

- **安装**：第一次载入带有该模组的预设版本时（安装、从链接 / 来源更新、从 ST 导入、内置预设随扩展更新、`ap dev`），按预设中的顺序追加到「我的模组」末尾，开关状态取 `enabled`，文字取当时面板语言的版本，并提示添加了哪些模组。已安装模组的 id 就是文件名 id。
- **预设未改动该模组**：什么也不做——用户的修改、开关与排序都保留；切换面板语言也不会替换已安装的副本。
- **用户删除了它**：之后的版本不会再装回来。
- **预设更新了该模组**：用户的副本与新版本相同时无事发生；不同时弹出一次对话框，列出这些模组，让用户逐个选择保留自己的版本或换成预设的新版本（换成新版本时保留开关状态与位置）。用户没改过的副本默认勾选“换成新版本”并标注“未修改过”，改过的默认保留并标注“你修改过”；取消即全部保留。对同一个新版本只询问一次。
- **`ap dev`**：用户没改过的副本直接换成新版本，不询问；改过的仍会询问。
- 预设去掉某个模组时，已安装的副本保留（它已经是用户的模组）。

`ap preview` 默认按全新安装合并这些模组（只含 `enabled` 不为 `false` 的），`--bundled-modules <id,…|all|none>` 指定要合并哪些（见 [cli.md](cli.md#ap-preview)）。

### 用户模组模板 `user-module-templates/`

用户在「我的模组」中新建模组时，可以选择**空白模组**或从某个模板开始；选中后模板被**复制**成一个普通的用户模组并打开编辑器（名称已存在时加编号）。之后它完全属于用户：预设更新不会改动已创建的副本。

模板适合向用户展示“在这个预设里可以怎样写自己的模组”：用哪个插入点、怎样更新构建变量、触发条件怎么设。

- **预览**：`ap preview --user-module-templates <id,…|all> [--lang zh-cn]` 把模板当作刚创建的用户模组合并后预览（见 [cli.md](cli.md#ap-preview)）。

**编写建议**（两种都适用）：写一份能直接使用的完整示例，而不是占位符（用户忘了改时，占位符会原样发给模型）；修改说明放在注释里，说清楚每一部分改什么、怎样增删构建变量的项、触发条件的含义；字典变量使用新的键，避免意外替换内置项。

## 随预设导出

在面板「预设」页点导出（或选中 Carrier 时使用 ST 对话补全预设的导出按钮）时，如果当前预设有用户模组，会先列出「我的模组」请用户勾选要一起导出的模组（默认全部勾选，已关闭的模组有标注）。导出的产物中，`userModules` 换成所选模组的当前内容与开关状态（未勾选的、已删除的预设模组都不包含）；其中与预设原版完全相同的模组保留原来的源（含翻译）。产物其余部分与 `hash` 不变。

对方安装这个文件时，这些模组按上面的规则安装进他的「我的模组」；自己重新导入时，同 id、内容相同的模组直接关联，不会重复添加。
