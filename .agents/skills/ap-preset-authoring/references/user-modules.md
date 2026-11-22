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
    { "id": "e2", "point": "after-history", "role": "user", "text": "<!-- @if transport == \"prefill\" -->Remember the rain.<!-- @end -->" },
    { "id": "e3", "point": "after-history", "text": "Thunder in the distance.", "enabled": false }
  ],
  "vars": [
    { "var": "questions", "type": "list", "items": ["Is it still raining?"] },
    { "var": "tags", "type": "list", "items": ["storm"], "enabled": false },
    { "var": "considerations", "type": "dict", "items": [{ "key": "weather", "value": "Is the rain getting worse?" }, { "key": "pacing", "value": null, "enabled": false }] }
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
| `entries[]` | 条目：`id`、`point`（插入点 id）、`role`（可选：`system` / `user` / `assistant`）、`text`、`enabled`（可选，`false` 时关闭） |
| `vars[]` | 构建变量更新，见下；更新本身、字典的每个键、列表的每一项都可关闭 |
| `trigger` | 触发条件，见下 |

### 条目

- 条目放在插入点的区域与 `order` 上，排在**同区域、同 `order` 的预设条目之后**；多个用户模组按列表顺序排列；
- 没有 `role` 的条目与区域中的其他文字合并；有 `role` 时成为单独的消息（等同于 `@as`）；
- `text` 支持 `{{ap::…}}` 宏、ST 宏，以及 `@if` / `@as` / `@style` / `@template` 指令与 `<!--# … -->` 注释（[directives.md](directives.md)），不支持 `@entry` / `@section`；
- 插入点不存在（例如预设更新后被删除）的条目被跳过，并在调试信息中警告；空文字的条目忽略。
- `enabled: false` 的条目保留在模组中但不插入（编辑器中每个条目前的开关），省略即开启。

### 条目与变量更新的开关

编辑器中每个插入条目、每条构建变量更新，以及更新里的每个键 / 每一项前都有开关，便于在不删除内容的情况下比较不同组合。开关状态是模组内容的一部分：随模组导出、导入、复制与[随预设导出](#随预设导出)，也计入“用户是否改过随预设模组”的比较。只保存关闭状态（`enabled: false`），重新打开即去掉该字段。模组本身的 `enabled` 则是用户的开关偏好，不计入比较。

### 构建变量更新

| `type` | 字段 | 效果 |
| --- | --- | --- |
| `string` | `text` | 覆盖字符串变量（空文字忽略） |
| `list` | `items: (string \| { text, enabled: false })[]` | 追加列表项（空项忽略） |
| `dict` | `items: [{ key, value, enabled? }]` | 设置字典键；`value` 为空或 `null` 时**删除**该键 |

`enabled: false` 的更新不应用；列表中关闭的项写成 `{ "text": "…", "enabled": false }`（打开的项仍是字符串），字典中关闭的键带 `"enabled": false`，二者都跳过。旧版本 AP 读到时：关闭的列表项被丢弃，关闭的条目、更新与字典键照常生效。只有预设声明过、且类型一致的变量会被更新；用户模组的更新在所有预设模组之后应用（[build-variables.md](build-variables.md#用户模组)）。

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

## 分组

「我的模组」可以把模组放进**组**里（按预设分开保存，与模组一起在用户设置中）：

```json
{ "id": "w7p2", "name": "Writing styles", "mode": "single", "collapsed": true }
```

| 字段 | 说明 |
| --- | --- |
| `id` | 组 id；模组用 `group` 字段引用它（省略 `group` = 未分组） |
| `name` | 显示名 |
| `mode` | `single`：单选（互斥）组；`multi`：多选组 |
| `collapsed` | 可选，面板中折叠 |
| `enabled` | 可选，`false` = 整组关闭（只保存 `false`，打开时去掉该字段） |

- **顺序**：显示顺序就是插入顺序——先是未分组的模组，然后各组按组的顺序依次排列，组内连续。移动模组或组都会改变多个模组在同一插入点的先后。
- **单选组**：同组最多一个模组开启，也可以全部关闭。开启一个即关闭同组其他模组；把开启的模组移入（或新建、复制、导入到）已有开启模组的单选组时，它变为关闭；多选组改为单选时只保留第一个开启的模组。
- **多选组**：每个模组单独开关，与未分组时相同。
- **整组开关**：关闭一个组时，组内所有模组都不生效（不合并进预设，`modules.user-<id>` 条件变量为假），但每个模组自己的开关和单选组的选择保持不变，仍可编辑、切换；重新打开组即恢复原样。组开关不是模组内容，不影响随预设安装的模组的冲突判断。
- 删除组时可以只删组（其中的模组变为未分组，排在未分组模组末尾）或连同模组一起删除。
- 设置中的数据不满足上述规则时（例如手动编辑过），读取时自动修正：引用不存在的组的模组变为未分组，单选组只保留第一个开启的模组。除整组开关外，组只影响面板中的管理与开关；合并进预设时与未分组的模组相同。

面板中：「我的模组」区块的「新建组」按钮新建一个组；组头部可以拖动把手排序组、折叠、改名、整组开关；展开时还可以切换单选 / 多选、在组内新建模组、导出组和删除组（选择保留其中的模组或一起删除）——折叠的组头部只显示把手、折叠箭头、组名、模组数与开关。关闭的组整张卡片变暗。拖动模组的把手可以在组内排序、移到其他组（空组、折叠的组也可以，折叠时放到组末尾），或拖回未分组的模组中（拖动时顶部出现放置区）；模组编辑器底部也可以选择所属组。单选组中的模组以单选按钮代替开关；点亮着的那个可以把它关闭。

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

带分组时，文件多一个 `groups` 字段，模组用 `group` 引用其中的组：

```json
{
  "format": "advanced-preset.user-modules",
  "version": 1,
  "preset": { "id": "petals", "version": "7.6.0" },
  "groups": [ { "id": "w7p2", "name": "Writing styles", "mode": "single" } ],
  "modules": [ { "id": "k3x9", "name": "Rainy city", "enabled": true, "group": "w7p2", "entries": [], "vars": [], "trigger": { "match": "all", "rules": [] } } ]
}
```

- 导出全部、导出所选模组时，文件包含这些模组所在的组（没有模组的组不写，没有组时没有 `groups` 字段）；导出一个组时包含该组与组内全部模组（空组也导出）。组的开关随之导出（关闭的组写 `"enabled": false`），导入后保持。
- `groups` 是可选字段，格式版本仍为 `1`：不支持分组的旧版 AP 忽略 `groups` 与 `group`，模组照常作为未分组模组导入；没有 `groups` 的旧文件在新版本中照常导入。
- 导入时组也作为**新副本**加入（新 id，排在已有组之后，模组的 `group` 随之对应），不会并入同名的已有组；未分组的模组排在已有未分组模组之后。引用文件中不存在的组的模组按未分组导入；导入后同样满足单选组规则（只保留第一个开启的）。
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
| `vars` | 构建变量更新，写法与[模组的 `vars`](build-variables.md) 相同（字典值为 `null` 表示移除该键）；`!off` 标记关闭的部分，见下 |
| `when` | 条件表达式，成为触发条件中的一条“表达式”规则 |
| `trigger` | 完整的触发条件（`{ match, rules }`，格式见[触发条件](#触发条件)，可写关键词规则）；与 `when` 二选一 |
| `enabled` | 仅 `user-modules/`：安装后是否开启，默认 `true`（翻译文件中忽略） |
| `group` | 所在的[组](#预设中的分组-usermodulegroups)（`preset.yaml#userModuleGroups` 中的 id，未声明时报错）；省略为未分组（翻译文件中忽略） |
| `description` | 仅 `user-module-templates/`：新建模组时选择列表中的说明 |

- **id** 取文件名，小写 kebab-case；frontmatter 中不需要写 `id`（写了且不同则报错）。两个目录的 id 互不相关。
- **条目**：正文开头到第一个 `<!-- @entry point=<插入点> role=<角色> -->` 之前是第一个条目（需要 frontmatter 的 `point`）；之后每个 `@entry` 行开始一个新条目。`@entry` 必须独占一行；围栏代码块与注释中的 `@entry` 是普通文本。
- **开关**：`@entry` 上写 `enabled=false` 的条目安装后是关闭的（想关闭第一个条目时，不在 frontmatter 写 `point`，正文以 `@entry` 行开头）。`vars` 中用 YAML 标签 `!off` 关闭整条更新、某个字典键或某个列表项：

  ```yaml
  vars:
    optionalQuestions:
      custom-style: Which details carry the style?
      imagery: !off Which images recur?        # 关闭这个键
    tags: [rain, !off thunder]                 # 关闭一项
    styleNotes: !off Prefer short sentences.   # 关闭整条更新
  ```

  `!off` 只能用在 `vars` 中（其他位置报错）；只关闭条目或部分变量时，模组的 `enabled` 仍控制整个模组。
- **原样保存**：条目文字（含[注释](directives.md#注释) `<!--# … -->`、指令、宏）按原样写进产物，只去掉首尾空行；用户在编辑器里看到的就是源码，注释正好用来写修改说明（不会发送给模型）。编译器仍会检查其中的指令、宏与条件，错误带行号报告。
- **翻译**：`<id>.<lang>.md` 是同一模组的整份翻译（名称、说明、条目、变量与触发条件都以它为准），面板语言为 `<lang>`（或其基础语言，如 `zh` 对应 `zh-cn`）时使用；必须有同 id 的原文件。只想翻译名称（与模板说明）时，在 [locales](locales.md) 的 `userModules` / `userModuleTemplates` 段中写 `name` / `description` 即可。
- **检查**：插入点、角色、变量与类型、触发规则（含条件变量与取值）都与用户模组同样校验；未知的 frontmatter 键、写在不适用目录中的 `enabled` / `description`、翻译文件中的 `group` 警告。
- **代码**：其中的 `@template` 代码（各语言版本都计入）算作预设的模板代码，安装预设时一并请用户确认（[templates.md](templates.md#信任)）。
- 不升产物格式；旧版本 AP 会忽略这两个字段（不安装、不显示模板，不影响其他功能）。分组（`userModuleGroups` 与模组的 `group`）同样不升格式：不支持分组的 AP 照常安装这些模组，都放在未分组中。

### 预设中的分组 `userModuleGroups`

预设可以声明组，让随预设安装的模组（和用模板新建的模组）直接放进组里，表达力与面板中的分组相同：

```yaml
userModuleGroups:
  - { id: writing-styles, name: Writing styles, mode: single }
  - { id: reminders, name: Reminders, collapsed: true }
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | — | 必填，小写 kebab-case，唯一 |
| `name` | `id` | 组名；可在 [locales](locales.md) 的 `userModuleGroups.<id>.name` 翻译（安装时取当时的面板语言） |
| `mode` | `multi` | `single`（单选，互斥）或 `multi`（多选），含义见[分组](#分组) |
| `collapsed` | `false` | 安装后是否折叠 |
| `enabled` | `true` | 安装后整组是否开启；`false` 时组内模组随预设安装，但在用户打开组之前都不生效 |

- 模组文件用 frontmatter `group: <id>` 指定所在的组；组内顺序就是预设中模组的顺序（按文件名）。
- **单选组**中最多一个随预设安装的模组 `enabled` 不为 `false`，否则报错。
- 模板的 `group`：用户从模板新建模组时放进该组——前提是用户的列表中有这个组（例如随预设安装过、没有删除）；否则放在未分组中。在组的头部「在组内新建」时，用的是那个组。
- 未知键警告；`mode`、`collapsed`、`enabled` 取值不对、id 不合法或重复时报错。

安装与更新（与[随预设安装的模组](#预设携带的用户模组-user-modules)规则一致）：

- **第一次**载入带某个组的预设版本时，按预设顺序把组追加到用户的组之后（id 就是源码中的 id），随后安装的新模组放进各自的组。
- 之后组的名称、模式、折叠、开关与顺序**都归用户**：预设的新版本改了组也不会覆盖；用户**删除**的组不再装回，引用它的新模组放到未分组。
- 所属组和顺序一样算“位置”：新模组取预设的组，已安装的模组保留用户放的位置（换成预设的新版本时也保留）；预设把已安装的模组改到别的组，不会移动用户的副本。
- 安装后同样满足单选规则：单选组里已有开启的模组时，新加入的模组是关闭的（用户的选择优先）。

### 预设携带的用户模组 `user-modules/`

适合“预设推荐、但应由用户掌控”的内容：例如一段可按喜好改写的设定补充、默认关闭的可选提醒。与普通模组不同，它们安装后就属于用户，用户可以直接修改提示词。

- **安装**：第一次载入带有该模组的预设版本时（安装、从链接 / 来源更新、从 ST 导入、内置预设随扩展更新、`ap dev`），按预设中的顺序追加到「我的模组」中未分组模组的末尾，开关状态取 `enabled`，文字取当时面板语言的版本，并提示添加了哪些模组。已安装模组的 id 就是文件名 id。
- **预设未改动该模组**：什么也不做——用户的修改、开关与排序都保留；切换面板语言也不会替换已安装的副本。
- **用户删除了它**：之后的版本不会再装回来。
- **预设更新了该模组**：用户的副本与新版本相同时无事发生；不同时弹出一次对话框，列出这些模组，让用户逐个选择保留自己的版本或换成预设的新版本（换成新版本时保留开关状态、位置与所在的组）。用户没改过的副本默认勾选“换成新版本”并标注“未修改过”，改过的默认保留并标注“你修改过”；取消即全部保留。对同一个新版本只询问一次。
- **`ap dev`**：用户没改过的副本直接换成新版本，不询问；改过的仍会询问。
- 预设去掉某个模组时，已安装的副本保留（它已经是用户的模组）。

`ap preview` 默认按全新安装合并这些模组（只含 `enabled` 不为 `false` 的），`--bundled-modules <id,…|all|none>` 指定要合并哪些（见 [cli.md](cli.md#ap-preview)）。合并顺序与全新安装相同（未分组的在前，然后按组）；指定的模组在同一单选组中有多个时，只合并第一个；预设中关闭的组（`enabled: false`）里的模组不合并。`--user-modules <file>` 中关闭的组同样不合并。

### 用户模组模板 `user-module-templates/`

用户在「我的模组」中新建模组时，可以选择**空白模组**或从某个模板开始；选中后模板被**复制**成一个普通的用户模组并打开编辑器（名称已存在时加编号）。之后它完全属于用户：预设更新不会改动已创建的副本。

模板适合向用户展示“在这个预设里可以怎样写自己的模组”：用哪个插入点、怎样更新构建变量、触发条件怎么设。

- **预览**：`ap preview --user-module-templates <id,…|all> [--lang zh-cn]` 把模板当作刚创建的用户模组合并后预览（见 [cli.md](cli.md#ap-preview)）。

**编写建议**（两种都适用）：写一份能直接使用的完整示例，而不是占位符（用户忘了改时，占位符会原样发给模型）；修改说明放在注释里，说清楚每一部分改什么、怎样增删构建变量的项、触发条件的含义；字典变量使用新的键，避免意外替换内置项。

## 随预设导出

在面板「预设」页点导出（或选中 Carrier 时使用 ST 对话补全预设的导出按钮）时，如果当前预设有用户模组，会先列出「我的模组」请用户勾选要一起导出的模组（默认全部勾选，已关闭的模组有标注）。导出的产物中，`userModules` 换成所选模组的当前内容、开关状态与所在的组（未勾选的、已删除的预设模组都不包含）；其中与预设原版完全相同的模组保留原来的源（含翻译）。`userModuleGroups` 换成所选模组所在的组的当前状态（名称、模式、折叠、开关，按用户的顺序；没有时不写这个字段）：用户没改过的预设组保留原来的源，改过名称、模式或折叠的组使用用户的值，并去掉 locales 中该组名称的翻译；开关总是写用户当前的值（只切换开关不影响翻译）。产物其余部分与 `hash` 不变。

对方安装这个文件时，这些组与模组按上面的规则安装进他的「我的模组」；自己重新导入时，同 id、内容相同的模组直接关联，不会重复添加。
