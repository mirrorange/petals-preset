# `preset.yaml` 清单

`preset.yaml` 是预设的根文件。清单较长时，除元数据外的字段可以拆到 `manifest/` 下的多个文件中，见[拆分清单](#拆分清单-manifest)。本页按字段列出全部顶层键；每个键的默认值、取值范围和相关诊断都在对应小节里。涉及较多规则的部分（结构、工具文案、配置、插入点等）给出摘要并链接到专门的页面。

## 顶层字段一览

| 字段 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `formatVersion` | `1` | `1` | 源码格式版本。不是 1 时报错 |
| `id` | string | — | **必填**。小写 kebab-case，安装后以此识别预设 |
| `name` | string | `id` | 显示名 |
| `version` | semver | `0.0.0` | 必须形如 `1.2.3` 或 `1.2.3-beta.1` |
| `author` | string | — | |
| `description` | string | — | 面板中显示 |
| `minAdvancedPreset` | semver | — | 安装时检查扩展版本，低于此版本拒绝安装 |
| `actors` | map | — | **至少一个**。见[角色](#角色-actors) |
| `narrator` | actor id | 第一个角色 | 主叙述者 |
| `user` | actor id | `user` | 代表用户的角色，必须在 `actors` 中声明 |
| `styles` | map | 见下 | 传输方式 → 写法 |
| `plaintextFormat` | `prefix` \| `tag` | `prefix` | 纯文本传输下阶段的分隔方式 |
| `contentMarker` | string | `<START_OF_CONTENT>` | 对话式输出中正文开始的标记 |
| `prefillGuide` | string | 空（运行时用 `<START>`） | 预填充模式下助手预填充前的那条用户消息 |
| `historyItemTemplate` | string | `{{prefix}}{{content}}` | 预填充模式下每条历史消息的格式 |
| `stageHeading` | string | `### Step {{n}}: {{label}} ({{actor}})` | 指令式下每个阶段分节前自动加的标题 |
| `stop` | string[] | `[]` | 预填充模式的停止词 |
| `regions` | list | — | **必填**。见[区域](#区域-regions) |
| `anchors` | map | — | 语义锚点的另一种写法 |
| `workflowRegion` | region id | `tail` 锚点的区域 | 工作流提示词放入的区域 |
| `insertionPoints` | list | `[]` | 用户模组插入点，见 [user-modules.md](user-modules.md) |
| `vars` | map | `{}` | 构建变量，见 [build-variables.md](build-variables.md) |
| `params` | list | `[]` | 参数，见[参数](#参数-params) |
| `groups` | map | `{}` | 面板分组与单选集，见[分组](#分组-groups) |
| `structures` | map | `{}` | 结构，见 [structures.md](structures.md) |
| `regex` | list | `[]` | 正则标记，见[正则标记](#正则标记-regex) |
| `meta` | map | 见下 | 元指令与内置任务，见 [meta-and-tasks.md](meta-and-tasks.md) |
| `tokenSaving` | map | 见下 | 省 token 的默认参数 |
| `inputShaping` | map | — | 用户输入整形（包裹或提取本次输入），见[用户输入整形](#用户输入整形-inputshaping) |
| `syntheticTurns` | list | `[]` | 工具模式的示范轮次，见 [transports.md](transports.md#示范轮次-syntheticturns) |
| `tools` | map | 见 transports | 模型可见的工具名、字段名与文案，见 [transports.md](transports.md#tools-文案) |
| `theme` | map | `{}` | 面板与消息 UI 的配色 |
| `carrier` | map | — | **必填 `order`**。见 [configuration.md](configuration.md#carrier) |
| `configurationDefaults` | map | 见 configuration | 配置默认值 |
| `configurations` | list | 一个 `default` | 推荐配置 |
| `defaultConfiguration` | id | 第一个推荐配置 | |
| `migrations` | list | `[]` | 改名迁移，见 [configuration.md](configuration.md#迁移) |
| `moduleOrder` | list | `["*"]` | 模组排序，见 [modules.md](modules.md#排序) |
| `workflowOrder` | list | 文件名顺序 | 工作流在面板中的顺序 |

最小可用的清单：

```yaml
formatVersion: 1
id: my-preset
name: My Preset
version: 0.1.0
actors:
  narrator: { name: Narrator }
  user: { name: "{{user}}" }
regions:
  - { id: top, role: system, merge: join }
  - { id: settings, native: true }
  - { id: history, role: system, anchor: historyStart }
  - { id: task, role: system, anchor: historyEnd }
  - { id: tail, role: system, anchor: tail }
workflowRegion: task
carrier:
  order: [top, { main: false }, worldInfoBefore, charDescription, scenario, worldInfoAfter, dialogueExamples, history, chatHistory, task, tail]
```

## 拆分清单 `manifest/`

`manifest/` 下的每个 `*.yaml` / `*.yml`（可以放在子目录里）都是一份**部分清单**：顶层键与 `preset.yaml` 相同，构建时合并进 `preset.yaml`。可以把一整段原样剪切过去，写法不变。

```
preset.yaml                  元数据、角色、区域、Carrier……
manifest/structures.yaml     structures:
manifest/params.yaml         groups: / params: / vars: / insertionPoints:
manifest/configurations.yaml configurationDefaults: / configurations:
```

规则：

- **元数据只能写在 `preset.yaml`**：`formatVersion`、`id`、`name`、`version`、`author`、`description`、`minAdvancedPreset`。写在片段中时报错（并被忽略）。其他字段写在哪个文件都可以。
- **映射逐层合并**：同一个映射可以分在多个文件里，例如 `structures.synopsis` 与 `structures.options` 各占一个文件，`groups` 的不同分组也可以分开写。
- **标量与列表只能定义一次**：同一个键路径在两个文件中都写了值（字符串、数字、列表等），报错并指出两处位置。列表（`params`、`regions`、`configurations`……）的顺序有意义，所以整个列表必须写在同一个文件里，不会拼接。
- **合并顺序**：先 `preset.yaml`，再按路径排序的片段。映射中键的顺序随之确定，例如 `groups` 未写 `order` 时按出现顺序排。
- 片段必须是映射；空文件忽略。以 `.` 开头的文件和目录被忽略。
- YAML 锚点（`&a` / `*a`）只在同一个文件内有效，引用同一锚点的内容要放在一起。

诊断会指向定义该字段的文件与行，例如 `error manifest/structures.yaml:30 structures.synopsis.history: …`。产物与不拆分时完全相同，拆分不影响已安装的预设。`ap check` 的统计中会显示片段数（`5 manifest fragments`）。

## 角色 `actors`

```yaml
actors:
  narrator: { name: Freesia, prefix: "{{ap::name::narrator}}: ", color: primary }
  editor:   { name: Petals,  prefix: "{{ap::name::editor}}: ",   color: accent }
  user:     { name: "{{user}}" }
narrator: narrator
user: user
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `name` | 角色 id | 显示名，模型也会看到。可含 ST 宏（`{{user}}`） |
| `prefix` | `{{ap::name::<id>}}: ` | 对话式输出里该角色发言的前缀，用于切分阶段 |
| `color` | — | 阶段卡片颜色：`primary`、`accent`（取主题色）或任意 CSS 颜色 |

- 角色 id 用在工作流阶段的 `actor`、宏 `{{ap::name::<id>}}` / `{{ap::prefix::<id>}}` 和配置 `identity` 中。
- 用户可以在配置的“身份”里覆盖名字和前缀（`configurationDefaults.identity.names/prefixes`）；覆盖值优先于这里的声明。
- `user` 角色的前缀出现在模型输出的行首时，AP 认为模型开始替用户说话，在那里截断（见 [transports.md](transports.md#对话式解析)）。

## 写法 `styles`

```yaml
styles: { prefill: dialogue, plaintext: dialogue, structured: imperative, tool-single: imperative, tool-multi: imperative }
```

键是五种传输方式，值是 `dialogue` 或 `imperative`。上面就是默认值。写法决定：条件变量 `style`、`@style` 片段、工作流分节选用哪一份、`{{ap::structure::<id>}}` 输出哪种格式说明、指令式是否加 `stageHeading`。未知传输方式的键给出警告。

## 对话式输出相关字段

| 字段 | 说明 |
| --- | --- |
| `plaintextFormat` | `prefix`：纯文本下按角色前缀切分阶段；`tag`：每个阶段都包在 `<阶段tag或阶段id>…</…>` 里。预填充总是 `prefix` |
| `contentMarker` | 模型输出这个标记后，后面的文字归入终结正文阶段。设为空字符串则不使用标记 |
| `prefillGuide` | 预填充整形时插入的用户消息（可含宏）。空时用 `<START>` |
| `historyItemTemplate` | 预填充整形时每条历史消息的模板。变量：`{{prefix}}`（用户消息为用户前缀，其他为 `名字: `）、`{{name}}`、`{{content}}` |
| `stop` | 预填充请求的停止词（与 ST 自带的合并），并在客户端再截一次 |

## 指令式阶段标题 `stageHeading`

指令式写法下，每个有内容的阶段分节前会自动加上这个标题。可用占位符：

| 占位符 | 值 |
| --- | --- |
| `{{n}}` | 阶段在本轮激活阶段中的序号（从 1 开始） |
| `{{label}}` | 阶段 `label` |
| `{{actor}}` | 阶段角色的显示名 |
| `{{target}}` | 该阶段的字段名（结构化 / 工具）或工具名（多工具） |
| `{{optional}}` | 可选阶段为 ` (optional)`，否则为空 |

设为空字符串可关闭自动标题。

## 区域 `regions`

区域是提示词里按顺序排列的“槽位”。每个模组条目落在某个区域里，工作流提示词落在 `workflowRegion`。

```yaml
regions:
  - { id: top, role: system, merge: join }
  - { id: guidelines, role: system, merge: { wrap: guidelines } }
  - { id: settings, native: true }
  - { id: history, role: system, anchor: historyStart }
  - { id: memory, role: system, merge: join, anchor: historyEnd }
  - { id: task, role: system }
  - { id: bottom, role: system, anchor: tail }
workflowRegion: task
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | — | 必填，唯一 |
| `role` | `system` | 区域消息的默认角色：`system` \| `user` \| `assistant` |
| `merge` | `none` | `none`：每个条目块一条消息；`join`：相邻无 `@as` 的块合并为一条，用 `separator` 连接；`{ wrap: <tag> }`：合并后包在 `<tag>…</tag>` 里 |
| `separator` | `"\n\n"` | `join` / `wrap` 时的连接符 |
| `native` | `false` | 原生区域：只是标记 ST 原生提示词（角色卡、世界书……）在顺序中的位置，AP 不往里放内容，也没有锚点 |
| `anchor` | — | 把语义锚点 `historyStart` / `historyEnd` / `tail` 绑定到这个区域 |

- 三个语义锚点**都必须**绑定到某个区域（可以写在 `regions[].anchor`，也可以写 `anchors: { historyStart: history, … }`）。
  - `historyStart`：聊天记录之前的最后一个区域。预填充整形以它为界（之前的内容合成一条 system 消息）。
  - `historyEnd`：聊天记录之后的第一个区域。
  - `tail`：最末尾的区域，也是 `workflowRegion` 的默认值。
- 在 `merge: join/wrap` 的区域里，`@as` 开头的块不参与合并，单独成为一条消息。
- 区域的实际位置由 `carrier.order` 决定（见 [configuration.md](configuration.md#carrier)）；`regions` 的顺序决定条目的排序与预览顺序，二者应保持一致。
- 诊断：条目指向未知区域或原生区域报错；`workflowRegion` 未知或是原生区域报错；没有任何条目的非原生区域给出警告。

## 参数 `params`

参数是用户在面板（写作页）里填写或选择的值，提示词用 `{{ap::param::<id>}}` 读取，条件用 `params.<id>` 判断。

```yaml
params:
  - id: language
    type: select
    label: Reply language
    group: language
    unset: the language used in the story so far
    options:
      - English
      - { value: zh, label: 简体中文, text: Simplified Chinese }
  - id: length
    type: number
    label: Target length
    unit: words
    min: 0
    max: 5000
    default: 0
    unset: a length that fits the scene
  - id: pov
    type: text
    label: Point of view
    fallback: language
  - id: bannedWords
    type: list
    label: Banned words
    description: Words the narrator must not use.
    default: [闪过]
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | — | 必填 |
| `type` | `text` | `text` \| `number` \| `select` \| `list`（字符串列表，见下） |
| `label` | `id` | 面板标签 |
| `description` | — | 面板说明 |
| `group` | — | 所属分组（`groups` 的键），未声明给出警告 |
| `default` | `''`（number 为 `0`，list 为 `[]`） | 默认值，须与类型相符（list 为字符串数组） |
| `unset` | — | 未设置时 `{{ap::param}}` 输出的文本 |
| `separator` | `", "` | 仅 `list`：`{{ap::param}}` 中项与项之间的文本 |
| `fallback` | — | 未设置时先尝试读取另一个参数（最多 4 层） |
| `unit` / `min` / `max` | — | 数字参数的单位与范围（面板用） |
| `options` | — | `select` 必填；`text` 也可提供，作为输入建议。每项是标量，或 `{ value, label?, text? }`；`list` 不使用 |

**取值规则**（`{{ap::param::<id>}}`）：

1. 值取当前配置的 `params.<id>`，没有则取 `default`；形状与类型不符的值（如列表参数存了字符串）按 `default` 处理。
2. 值为 `''`、`0` 或空列表视为“未设置”：若有 `fallback` 且那个参数有值，用它；否则输出 `unset`（没有则为空）。
3. 已设置时，列表参数输出用 `separator` 连接的各项；其他参数若值匹配某个选项的 `value`，输出该选项的 `text`（没有 `text` 则输出值本身）。

条件里的 `params.<id>` 是原始值（不经过 `unset`/`fallback`/`text`）。诊断：`fallback` 指向不存在的参数报错；`select` 没有 `options` 报错；`default` 与类型不符报错。

### 列表参数 `type: list`

列表参数是一组字符串，适合禁词表、额外关键词这类“用户自己列出若干项”的设置。面板中以标签输入编辑：输入一项后按 Enter 添加（粘贴多行时每行一项），点 × 删除。

- 值总是去掉首尾空白、去掉空项与重复项（保留第一次出现的位置）后的字符串数组；`default`、`configurationDefaults` 与推荐配置中的值同样按此规范化。
- `{{ap::param::<id>}}`：各项用 `separator`（默认 `", "`）连接；空列表视为未设置。
- 条件：空列表为假，因此 `params.bannedWords` 可以直接判断“是否填写了”；`"闪过" in params.bannedWords` 判断是否含某项（[expressions.md](expressions.md)）。
- [模板](templates.md)中 `params.<id>` 是普通数组，可以循环、合并、去重——列表需要逐项排版或与内置项合并时用模板。
- 配置层合并时列表整体替换（不拼接）；用户配置只在与基础配置不同时保存整个列表。
- 用户模组的变量规则对列表参数提供“有内容 / 为空”（`on` / `off`）。
- 诊断：`options` / `unit` / `min` / `max` 对列表参数无效（警告，忽略）；非列表参数写了 `separator` 警告；列表参数与标量参数互为 `fallback` 警告；声明了列表参数而 `minAdvancedPreset` 低于 1.9.0 时警告（旧版本无法编辑列表）。
- 预设更新时，类型变化导致形状不符的已存值会被丢弃（回到新默认值）。

## 分组 `groups`

```yaml
groups:
  guidelines: { label: Guidelines, order: 10 }
  mode: { label: Mode, description: Pick one., allowNone: false }
  language: { label: Language, order: 20, tab: writing }
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `label` | 键 | 面板标题 |
| `description` | — | 面板说明 |
| `order` | 按声明顺序 10、20、30… | 面板排序 |
| `allowNone` | `false` | 仅对单选集有意义：允许一个都不选 |
| `tab` | `modules` | 放在面板的“模组”页还是“写作”页（`modules` \| `writing`） |

分组键同时被两种东西引用：模组的 `group`（面板归类）和模组的 `exclusive`（单选集）；参数的 `group`。单选集的规则见 [modules.md](modules.md#激活规则)。

面板中的单选集：不允许为空时显示为一组单选项；`allowNone: true` 时，单选集还有一个总开关，打开后才列出选项。单选集是它所在分组的唯一内容（单选集 id 等于成员模组的 `group`）时，总开关放在分组标题上；否则作为分组内的一行，以单选集分组的 `label` 命名，避免看起来像在控制分组里的其他模组。

## 正则标记 `regex`

正则标记在每次生成前对文本做一次匹配，结果作为条件变量 `flags.<id>`（布尔）。

```yaml
regex:
  - { id: summary, pattern: "(summary|summarize|recap|总结)", flags: i, target: meta, requires: meta.pure }
  - { id: ooc, pattern: "\\(OOC:", target: input }
  - { id: battle, pattern: "fight|battle", target: "recent:3" }
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | — | 变量名 `flags.<id>` |
| `pattern` | — | JavaScript 正则源码（YAML 里注意转义） |
| `flags` | `i` | 正则标志 |
| `target` | `meta` | `input`：用户本次输入全文；`meta`：本次输入中的元指令文本；`recent:N`：最近 N 条消息 |
| `requires` | — | 条件表达式；匹配且条件成立时标记才为真 |

`requires` 求值时只有 `meta.*` 可用（此时尚未组装）。非法正则或 `target` 报错。

## 元指令 `meta`

```yaml
meta:
  trigger: "@{{ap::name::narrator}}"
  wrap: true
  recentCount: 3
  tasks: { summary: flags.summary, impersonate: flags.impersonate }
  impersonatePrompt: "@{{ap::name::narrator}} Write my next reply for me."
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `trigger` | `@{{ap::name::<narrator>}}` | 元指令触发词（大小写不敏感） |
| `wrap` | `true` | 允许 `@Trigger{ … }` 花括号写法 |
| `recentCount` | `3` | `{{ap::recent_meta}}` 取最近几次元指令（配置 `meta.recentCount` 可覆盖） |
| `tasks.summary` / `tasks.impersonate` | — | 纯元指令时判断是否为“总结”/“代写”任务的条件 |
| `impersonatePrompt` | `<trigger> Write my next reply for me.` | 点击 ST 代写按钮时发送的元指令 |

完整语义见 [meta-and-tasks.md](meta-and-tasks.md)。`tasks` 只接受 `summary` 与 `impersonate`。

## 省 token `tokenSaving`

```yaml
tokenSaving:
  keepFullTurns: 6
  truncateUserAfter: 2000
  userInput: truncate
  synopsisTemplate: "[{{time}} · {{location}}] {{event}}: {{summary}}"
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `keepFullTurns` | `6` | 保留最近几轮完整内容 |
| `truncateUserAfter` | `2000` | `userInput: truncate` 时，更早的用户消息超过这个字符数就截断并加 `…`（0 表示不截断） |
| `userInput` | `truncate` | 更早的用户消息：`keep`（原样）/ `truncate`（截断）/ `remove`（不发送） |
| `synopsisTemplate` | 如上 | 更早的助手消息替换成事件摘要时的格式 |

是否启用由配置 `tokenSaving.enabled` 决定（默认关闭）；`keepFullTurns` / `truncateUserAfter` / `userInput` 是配置的默认值，用户可在面板中修改。最近几条消息附带事件摘要由 `synopsis` 结构的 `history` / `historyTemplate` 声明（见 [structures.md](structures.md#synopsis)）。规则见 [meta-and-tasks.md](meta-and-tasks.md#省-token)。

## 用户输入整形 `inputShaping`

预设级的提示词结构功能：聊天历史以用户消息结尾时（即本次输入），把这条消息**包裹**起来，或从历史中**提取**出来、由宏放到别的位置。用户没有开关；不声明时不整形。

```yaml
# 包裹：历史中的本次输入换成模板，后置指令可以说“<input> 中的内容”
inputShaping:
  mode: wrap
  template: "<input>\n{{content}}\n</input>"
```

```yaml
# 提取：本次输入离开聊天历史，由 {{ap::input::text}} 放到某个区域
inputShaping:
  mode: extract
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `mode` | — | **必填**。`wrap`（原位包裹）或 `extract`（从历史中取出） |
| `template` | `<input>\n{{content}}\n</input>` | 仅 `wrap`：替换输入的文字。`{{content}}` 为输入原文；模板中的 `{{ap::…}}` 与 ST 宏会展开，输入原文不再展开 |

- 只有**本次生成的聊天历史以用户消息结尾**时整形：普通发送、重新生成、滑动。`continue`、`/trigger` 或其他扩展在助手消息之后发起的生成不整形；
- 世界书深度 0 注入、作者注释等 ST 插到这条消息之后的非用户消息**不影响**整形；
- 整形生效时条件变量 `input.shaped` 为真（[expressions.md](expressions.md)），`{{ap::input::text}}` 为输入原文（[macros.md](macros.md)）；未生效时前者为假、后者为空。`extract` 的预设应当在 `input.shaped` 为真时用 `{{ap::input::text}}` 放回输入，否则输入会丢失；
- 整形发生在缓存优化之后，不参与粘性渲染（输入位于历史末尾，下一轮它回到历史中按普通消息渲染，只影响这一条消息的缓存）；
- 需要 Advanced Preset 1.9.0 以上（更早的版本忽略这个字段，`ap check` 在 `minAdvancedPreset` 低于 1.9.0 时警告）。

`ap check` 警告：未知字段；`extract` 写了 `template`（不使用）；`wrap` 的模板没有 `{{content}}`（输入会丢失）；`extract` 却没有提示词使用 `{{ap::input::text}}`；`minAdvancedPreset` 低于 1.9.0。

完整流程见 [meta-and-tasks.md](meta-and-tasks.md#用户输入整形)。

## 主题 `theme`

```yaml
theme:
  light: { primary: "#c026d3", accent: "#9333ea" }
  dark:  { primary: "#e879f9", accent: "#c084fc" }
```

AP 跟随 ST 的明暗模式选择 `light` 或 `dark`。`primary` 与 `accent` 会生成 50–900 的色阶（`--ap-primary-500` 等）；其他键原样成为 CSS 变量 `--ap-<key>`。用户可以在“外观”里用自己的颜色覆盖。

## 其余字段

- `insertionPoints` → [user-modules.md](user-modules.md#插入点-insertionpoints)
- `vars` → [build-variables.md](build-variables.md)
- `structures` → [structures.md](structures.md)
- `syntheticTurns`、`tools` → [transports.md](transports.md)
- `carrier`、`configurationDefaults`、`configurations`、`defaultConfiguration`、`migrations` → [configuration.md](configuration.md)
- `moduleOrder` → [modules.md](modules.md#排序)
- `workflowOrder`：工作流 id 列表；列出的按列出顺序排在前面，其余按文件名。决定面板里工作流的顺序，以及“第一个故事工作流”（默认工作流）是谁。
