# 结构

**结构**是模型在正文之外交回的附加数据：叙述者的回复、记忆更新、剧情总结、代写、事件摘要、选项、顶部状态栏、行内批注，以及预设自定义的块。结构的**机制**（数据形状、解析、保存、卡片 UI）由 AP 内置；预设只为它们**命名**（标签名 / 字段名）并提供**说明文字**。

```yaml
structures:
  options:
    kind: options
    tag: options
    field: options
    count: 4
    description: Four diverse options for {{ap::name::user}}'s next input.
    properties:
      label: Reply type / direction, under 5 words.
      content: The reply itself, written as {{ap::name::user}}'s next input.
    formatPrompt:
      tag: |-
        <options>
        <option title="...">...</option>
        </options>
      field: 'Fill the `{{field}}` field with 4 items, each `{ "label": "...", "content": "..." }`.'
```

## 声明字段

`structures` 是 map，键是结构 id。

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `kind` | 结构 id | 种类：`reply` `memory` `recap` `impersonate` `synopsis` `options` `status` `annotations` `custom`。其他值报错 |
| `tag` | 结构 id | 对话式写法中的 XML 标签名，须匹配 `^[A-Za-z_][\w.-]*$` |
| `field` | 结构 id | 指令式写法中的 JSON 字段名 / 工具参数名，须匹配 `^[A-Za-z_]\w*$`。模型实际看到的名字会加上排序前缀：`tools.orderPrefix`（如 `3_options`）或旧的 `tools.structurePrefix`（默认 `z_`，即 `z_options`），见 [字段名与顺序](#字段名与顺序) |
| `description` | — | 字段整体的说明，进入 JSON Schema 的 `description`（模型可见，支持宏；`{{field}}` 为实际字段名） |
| `properties` | — | 子字段说明，键只能是该种类的固定子字段（见下表），其他键报错 |
| `count` | — | 仅 `options`：要求的选项数，写入 schema 的 `minItems` / `maxItems` |
| `history` | — | 历史回传：最近 N 条助手消息再次带上该结构（任意非行内种类），可指定模板、位置与条件，见 [历史回传](#历史回传)。`annotations` 的 `history` 是数字，含义不同（见下） |
| `historyTemplate` | `{{content}}` + 空行 + `[{{time}} · {{location}}] {{event}}: {{summary}}` | 旧写法，仅 `synopsis` 配合数字 `history`：附带摘要的消息的格式，见 [旧写法](#旧写法synopsis-的数字-history) |
| `title` | 结构 id | 卡片标题（自定义结构用；`reply` 等也可以设，如花瓣的回复卡片显示叙述者名字）。可在 `locales` 中翻译 |
| `formatPrompt.tag` | `''` | 对话式下的格式说明，由 `{{ap::structure::<id>}}` 输出 |
| `formatPrompt.field` | `''` | 指令式下的格式说明，由 `{{ap::structure::<id>}}` 输出；`{{field}}` 为实际字段名 |
| `shape` | `text` | 仅 `custom`：`text`（一段文字）或 `kv`（键值对） |
| `display` | `card` | 仅 `custom`：`card`（消息下方显示可折叠卡片）或 `hidden`（只保存，不显示） |
| `history` | `0` | 仅 `annotations`：把批注以 CriticMarkup 重新注入最近 N 条助手消息的历史（见 [annotations](#annotations)） |
| `order` | 终结正文阶段之后 | 结构在一轮输出中的位置（指令式）：数字排序键，或 `{ before: <阶段> }` / `{ after: <阶段> }`（也可写 `before:<阶段>` / `after:<阶段>`），`<阶段>` 为阶段 id 或 `@content`（终结正文阶段）。工作流可以覆盖，见 [字段名与顺序](#字段名与顺序)。行内结构忽略 |
| `ui` | — | 卡片：`position`（`top` 消息顶部 / `bottom` 消息底部）、`order`（区域内排序键）；`custom` 还可以带自己的组件 `template` / `module` / `style`，见 [卡片](#卡片) |

约束（报错）：

- 除 `custom` 外，**每种 kind 最多一个**结构；
- 结构之间的 `field` 不能重复；结构字段（加前缀后）也不能与工作流阶段字段、携带它的阶段工具的参数同名（见 [workflows.md](workflows.md)）。

使用 `order`、`ui`、`status`、`history` 对象或 `tools.orderPrefix` 的预设应声明 `minAdvancedPreset: 1.10.0`（否则 `ap check` 告警）：更早的 AP 会忽略这些设置，按旧规则把结构放在正文之后。

## 字段名与顺序

部分 API（如 Gemini）不论 schema 中的顺序，都按**字母顺序**生成对象字段。结构字段如果排在阶段字段之前，模型会在开始规划和写正文之前先写出选项、摘要等，结果与正文不一致。反过来，有些结构（如状态栏）本就应该**先于正文**写出。

### 统一排序（推荐）

一轮输出中的阶段与结构按同一个顺序排列（I-134）：

- 阶段按声明顺序，排序键为 `order`（默认 100 × 序号：100、200、300…），显式写的键必须递增；
- 结构的位置依次取：工作流 frontmatter 的 `structures.order.<id>` → 结构的 `order` → 旧的 `structures.after`（工作流）→ 终结正文阶段之后；
- 位置写成数字时与阶段的键比较（相同键时阶段在前）；写成 `{ before: 阶段 }` / `{ after: 阶段 }` 时紧贴该阶段；同一位置的多个结构按声明顺序；
- `@content` 指终结正文阶段；引用的阶段本轮未激活时退回默认位置。

例：思考 → 状态栏 → 正文 → 摘要：

```yaml
structures:
  status: { kind: status, order: { before: "@content" } }
  synopsis: { kind: synopsis }          # 默认：正文之后
```

清单设置 `tools.orderPrefix` 后（I-135），AP 在指令式传输中给每个字段加上**统一排序前缀**：模板中的 `{{n}}` 为该字段在本轮中的位置，按本轮字段总数补零（10 个以上为 `01_`…），于是字母顺序就是工作流顺序：

```text
1_thinking  2_status  3_story  4_synopsis
```

- `orderPrefix: true` 等同 `"{{n}}_"`；自定义模板必须含 `{{n}}`，且只能用字母、数字、`_`；
- 前缀由 AP 在每轮按激活的阶段 / 结构生成，替代旧的阶段序号前缀（`step{{n}}_`，此时 `tools.turn.field` 宜写成 `{{stage}}`）与 `structurePrefix`（被忽略并告警）；
- 结构说明中的 `{{field}}` 在构建时**保留**，运行时替换为本轮的实际字段名；直接写未加前缀的字段名（如 `` `status` ``）会告警；
- 多工具模式下，结构由它**前面最近的阶段**的工具携带（没有则由后面最近的阶段）；携带结构的工具，其自身参数也带前缀（`2_content`、`3_status`），保证参数同样按字母顺序；
- 解析时宽松匹配：模型丢掉前缀、只写 `status` 也能识别。

### 旧的结构前缀

未设置 `orderPrefix` 时，`ap build` 会给每个结构字段加上 `tools.structurePrefix`（默认 `z_`，见 [transports.md](transports.md#tools-文案)），让它排在工作流字段之后：`options` → `z_options`。前缀在构建时写入产物的 `field`，结构化输出、单步工具与携带结构的阶段工具的参数都使用加前缀后的名字；对话式的标签名不受影响。`structurePrefix: ""` 表示不加前缀。

- `formatPrompt.field`、`description` 与 `properties` 中的 `{{field}}` 会替换为实际字段名，说明文字里提到字段名时都用它：`Fill the \`{{field}}\` field …`；
- `ap check` 告警：上述文字直接写了未加前缀的字段名（如 `` `options` ``）；结构字段按字母顺序排在它之前的阶段字段、或携带它的阶段工具的参数之前；显式放在某阶段之前的结构排在该阶段之后；阶段字段本身不按字母顺序排列（如 `step10_…` 排在 `step2_…` 之前、或只用阶段 id 作字段名）。

## 种类一览

| kind | 数据形状 | 固定子字段 | 必填 | 运行期语义 |
| --- | --- | --- | --- | --- |
| `reply` | `string` | — | 是 | 叙述者对用户的回复，显示为回复卡片 |
| `memory` | `MemoryOp[]` | `scope` `operation` `key` `content` | 否 | 记忆的新增 / 替换 / 删除 |
| `recap` | `string \| null` | — | 否 | 完整剧情总结，覆盖之前的聊天记录 |
| `impersonate` | `string` | — | 是 | 代写的用户下一条输入，卡片可“填入输入框” |
| `synopsis` | `{ time, location, event, summary }` | `time` `location` `event` `summary` | 是 | 本段的事件摘要卡片；省 token 时替代旧回复 |
| `options` | `{ label, content }[]` | `label` `content` | 是 | 选项卡片，点击填入输入框 |
| `status` | `{ date, weekday, time, location, weather }` | `date` `weekday` `time` `location` `weather` | 是 | 顶部状态栏：故事日期（或 `第N天`）、星期、时间、地点、天气 |
| `annotations` | 正文内标记 → `Annotation[]` | — | — | **行内**结构：写在正文里，解析后从正文移除，显示为虚线下划线与气泡 |
| `custom` | `string` 或 `Record<string, string>` | — | 是 | 预设自定义的块，可带自己的卡片组件 |

“必填”指生成的 JSON Schema 中该字段是否进入 `required`。`memory` 与 `recap` 是可选的：模型可以不写。

`annotations` 是唯一的**行内结构**：它不是 schema 字段，也没有标签，两种写法下都由模型直接写在正文里（`tag` / `field` / `description` / `properties` 不使用）。

## 何时请求结构

结构**不会自动出现**。只有当某个**当前生效的模组**在 frontmatter 中列出它时，这一轮才会请求它：

```yaml
---
id: options
when: '!meta.pure && gen.type != "impersonate"'
structures: [options]
---
```

请求的结构：

- 指令式：作为字段加入回合 schema（结构化、单工具），或作为携带它的阶段工具的附加参数（多工具），位置见 [workflows.md](workflows.md#结构的位置)；
- 对话式：AP 从模型输出中提取对应标签；
- 行内结构（`annotations`）：两种写法相同，AP 从正文中解析标记；

由脚本或其他扩展发起的任务生成（`gen.type == "script"` 且 `gen.origin == "task"`）不请求任何结构，即使模组列出了它们。

格式说明需要由模组自己放进提示词：在正文里写 `{{ap::structure::<id>}}`，它按当前写法输出 `formatPrompt.tag` 或 `formatPrompt.field`。

## 指令式：JSON Schema

每种 kind 的字段 schema 固定，`description` 与 `properties` 的文字会填进对应位置：

```jsonc
// reply / impersonate
{ "type": "string" }
// recap
{ "type": ["string", "null"] }
// memory
{ "type": "array", "items": { "type": "object",
  "properties": {
    "scope": { "type": "string", "enum": ["memory", "global_memory"] },
    "operation": { "type": "string", "enum": ["replace", "delete"] },
    "key": { "type": "string" },
    "content": { "type": ["string", "null"] } },
  "required": ["scope", "operation", "key", "content"], "additionalProperties": false } }
// synopsis
{ "type": "object", "properties": { "time": {…}, "location": {…}, "event": {…}, "summary": {…} },
  "required": ["time", "location", "event", "summary"], "additionalProperties": false }
// status
{ "type": "object", "properties": { "date": {…}, "weekday": {…}, "time": {…}, "location": {…}, "weather": {…} },
  "required": ["date", "weekday", "time", "location", "weather"], "additionalProperties": false }
// options（count: 4 时加 minItems / maxItems）
{ "type": "array", "items": { "type": "object",
  "properties": { "label": {…}, "content": {…} },
  "required": ["label", "content"], "additionalProperties": false }, "minItems": 4, "maxItems": 4 }
// custom, shape: text
{ "type": "string" }
// custom, shape: kv
{ "type": "object", "additionalProperties": { "type": "string" } }
```

Gemini 不支持自由键的对象：`kv` 形式会改为 JSON 字符串字段，并在说明后追加 `tools.jsonStringHint`（见 [transports.md](transports.md#gemini)）。

## 对话式：标签格式

对话式下，结构写成 XML 块，可以出现在输出的任意位置（通常在正文之后）。提取时这些块会从正文中移除。

| kind | 写法 |
| --- | --- |
| `reply` `recap` `impersonate` `custom(text)` | `<tag>文字</tag>` |
| `memory` | 每个操作一个块：`<tag><scope>memory</scope><operation>replace</operation><key>标题</key><content>内容</content></tag>`，可重复 |
| `synopsis` | `<tag><time>…</time><location>…</location><event>…</event><summary>…</summary></tag>` |
| `status` | `<tag><date>…</date><weekday>…</weekday><time>…</time><location>…</location><weather>…</weather></tag>` |
| `options` | `<tag><option title="类型">内容</option>…</tag>`（也接受 `label="…"`） |
| `custom(kv)` | `<tag><键>值</键>…</tag>`，每个子标签一个键 |

同一结构出现多个块时，除 `memory` 外只取最后一个。标签内容做 XML 反转义（`&lt;` 等）。流式生成时未闭合的块不显示在正文中。

## 宽松解析

无论哪种写法，值都会先规范化：

- `reply` / `impersonate`：转成字符串并去除首尾空白，空串视为没有；
- `recap`：空串或 `null` 视为“无总结”；
- `memory`：单个对象视为一项；`key` 缺失时读 `title`；`scope` 写 `global` 视为 `global_memory`，其他值视为 `memory`；`operation` 非 `delete` 一律视为 `replace`；`delete` 的 `content` 置为 `null`；没有 `key` 的项丢弃；
- `options`：每项接受 `label` / `title` 与 `content` / `text`；字符串项视为 `content`；两者都为空的项丢弃；
- `status`：字符串值先尝试 `JSON.parse`；子字段也接受别名（`day` → `date`，`week` / `dayOfWeek` → `weekday`，`clock` → `time`，`place` → `location`），缺失的子字段为空串；
- `custom(kv)`：字符串值先尝试 `JSON.parse`；所有值转成字符串。

## 历史回传

结构值保存在消息的 Sidecar 中，正文 `mes` 是干净的，历史里默认只发送正文。`history` 让最近几条助手消息**再次带上**结构（I-139）：例如让模型从先前的状态栏 / 事件摘要延续时间线，或在纯文本传输中给出输出格式的示例。

```yaml
structures:
  status:
    kind: status
    order: { before: "@content" }   # 正文之前生成 → 回传时也插在正文之前
    history:
      depth: 1                       # 最近 1 条助手消息
      template: |-                   # 插入的文字，占位符为字段
        <status>
          <date>{{date}}</date>
          <time>{{time}}</time>
          <location>{{location}}</location>
        </status>
      when: 'transport in ["plaintext", "prefill"]'
```

| 键 | 默认 | 说明 |
| --- | --- | --- |
| `depth` | `1` | 最近几条助手消息带上该结构，`0` 为关闭。已被省 token 替换为摘要的消息也计入 N，但只有仍为原文的消息会带上；没有该结构值的消息跳过 |
| `template` | `synopsis`：`[{{time}} · {{location}}] {{event}}: {{summary}}`；其他：`{{value}}` | 插入的文字。占位符：该种类的子字段（如 `synopsis` 的 `{{time}}`、`status` 的 `{{date}}`；`custom` 的 `kv` 形式可用任意键）与 `{{value}}`（整个值的文字形式：`status` 为 `日期 星期 时间 \| 地点 \| 天气`，`options` 为 `- 标题: 内容` 列表，`kv` 为 `键: 值` 行）。其他 `{{…}}` 原样保留；不展开宏 |
| `position` | 结构的排序 | `before` / `after`：紧贴在正文之前 / 之后；数字：排序键，与阶段同一条轴（正文位于终结正文阶段的键，如 `300`）。不写时按结构的 `order`（含工作流的 `structures.order`）相对终结正文阶段的位置；没写 `order` 的结构在正文之后 |
| `when` | 总是 | 条件表达式，在本次请求的条件变量下求值（见下） |

`history: N`（数字）是 `{ depth: N }` 的简写，`synopsis` 除外（数字为旧写法，见下）。

- 只有请求该结构的模组开启、且 `when` 成立时才回传；
- 同一条消息带多个结构时，按（排序键，正文之前 / 之后，声明顺序）排列；结构与正文之间、结构彼此之间以空行分隔；
- 「继续」生成时，被续写的那条消息不带结构；
- 条件在历史改写时（组装之前）求值：`transport`、`style`、`workflow`、`gen.*`、`meta.*`、`modules.*`、`params.*`、`flags.*`、`features.*`、`chat.group`、`memory.*`、`recap.present`、`last.*` 可用；`stages.*` 为空、`input.shaped` 为假；
- 位置按当前配置的故事工作流计算（`gen.origin` 为 `task` 时也是）；
- 开启缓存优化时，每条消息带了哪些结构也属于它的渲染方式，按粘性渲染沿用（见 [meta-and-tasks.md](meta-and-tasks.md#缓存优化粘性渲染)）；
- `annotations` 不使用此机制：它的 `history: N` 把批注以 CriticMarkup 注入正文（见 [annotations](#annotations)）。

常见用法：纯文本 / 预填充传输中，历史里的结构就是格式示例；结构化 / 工具传输中模型不应在正文里写标签，可用条件只在前两种传输下回传，另由模组在别处提供上一次的值（如 `{{ap::structure::status::last}}`，条件 `last.status`）。

### 旧写法：synopsis 的数字 history

1.9 及以前，`synopsis` 用 `history: N` + `historyTemplate`（`{{content}}` 为消息原文）。这种写法仍然可用：CLI 原样写入产物，旧版 AP 照常读取；新版 AP 把 `{{content}}` 之前 / 之后的文字分别插在正文之前 / 之后（不另加空行），输出与以前逐字节一致。它不能写条件与位置，需要时改用对象写法（并声明 `minAdvancedPreset: 1.10.0`）。

## 运行期语义

### reply

回复也会进入 `{{ap::recent_meta}}`（最近的元指令及叙述者的回答）。回复使用受限 HTML：只允许 `<p>`、`<p special>`（强调段落）与 `<blockquote>`。卡片按 HTML 渲染。**纯元指令轮**（没有正文时）消息正文 `mes` 写入回复的纯文本（标签转为段落）。

### memory

- `scope: memory`：本聊天的记忆。每个操作保存为消息 sidecar 中的事件（`memory.replace` / `memory.delete`）。当前记忆由聊天从第 0 条消息开始按顺序折叠得到，因此**切换 swipe、删除消息会自动回滚**对应的记忆修改；
- `scope: global_memory`：全局记忆，按预设保存（所有聊天共享），是只追加的日志，可在面板中撤销。消息 sidecar 保存条目 id（`global.ref`）：删除、重新生成、滑走该消息时条目失效，滑回该版本时重新生效（见 [artifact-and-runtime.md](artifact-and-runtime.md#事件)）；
- `{{ap::memory}}` / `{{ap::memory::global}}` 输出当前记忆的 JSON（见 [macros.md](macros.md)）；条件变量 `memory.hasChat` / `memory.hasGlobal` 见 [expressions.md](expressions.md)。

### recap

总结保存为事件 `recap.set`，`coversUpTo` 为本轮输入所在消息的前一条。之后组装时，如果启用了省 token，`coversUpTo` 及之前的消息不再发送，改由 `{{ap::recap}}` 提供（见 [meta-and-tasks.md](meta-and-tasks.md#省-token)）。纯总结轮的 `mes` 为 `…`。

### impersonate

SillyTavern 的“代写”按钮会发送清单中的 `meta.impersonatePrompt`（输入框中已有的草稿附在后面），走任务工作流。结果显示为代写卡片，可一键填入输入框。纯代写轮的 `mes` 为 `…`。

### synopsis

显示为事件摘要卡片，用户可在卡片上编辑四个字段，保存后直接覆盖该楼层（当前 swipe）Sidecar 中的结构值，不改正文 `mes`，也不保留原值。事件摘要在历史中有两种用法，模板都由预设指定，都读取编辑后的值：

- **替换**：启用省 token 时，较早的助手消息被 `tokenSaving.synopsisTemplate` 渲染的摘要替代（见 [meta-and-tasks.md](meta-and-tasks.md#省-token)）；
- **附带**：声明 [`history`](#历史回传) 时，最近 N 条助手消息保留原文，并附上自己的摘要（与是否启用省 token 无关）。占位符 `{{time}}` `{{location}}` `{{event}}` `{{summary}}`。

故事中没有明确时间时，模型往往只能从先前的事件摘要推断时间；附带最近几条摘要可以避免时间线混乱。只有请求了该结构的模组开启时才附带；已被省 token 替换的消息也计入 N。

```yaml
structures:
  synopsis:
    kind: synopsis
    history:
      depth: 3
      template: |-
        <synopsis>
          <time>{{time}}</time>
          <location>{{location}}</location>
          <event>{{event}}</event>
          <summary>{{summary}}</summary>
        </synopsis>
      # 有状态栏提供时间时，只在纯文本 / 预填充中作为格式示例回传
      when: '!modules.status-bar || transport in ["plaintext", "prefill"]'
```

### options

显示为选项卡片：点击把内容填入输入框；按住 Ctrl / ⌘ / Shift / Alt 点击则直接发送。

### status

顶部状态栏（I-136）：AP 内置的卡片显示在消息**顶部**（思考等阶段卡片之下、正文之上），一行显示 `日期 星期 · 时间 · 地点 · 天气`（天气按关键词配图标），流式生成时随输出实时更新。用户可以点铅笔编辑五个字段，保存后覆盖该楼层的结构值（同 synopsis）。

宜把它放在正文之前生成：`order: { before: "@content" }`。`{{ap::structure::status::last}}` 输出聊天中最近一次的状态（`日期 星期 时间 | 地点 | 天气`），便于让模型延续时间线；条件变量 `last.status` 表示已有先前的状态。也可以用 [`history`](#历史回传) 让最近一条消息在正文之前带上自己的状态：

```yaml
structures:
  status:
    kind: status
    order: { before: "@content" }
    properties:
      date: Story date, or `Day N` when the story has no calendar.
      weather: Current weather, a few words.
    formatPrompt:
      tag: "<status><date>…</date><weekday>…</weekday><time>…</time><location>…</location><weather>…</weather></status>"
      field: "Fill `{{field}}` before the story."
```

### custom

`display: card` 显示可折叠卡片（标题为 `title`）；`hidden` 只保存在 sidecar 中，供脚本或 `{{ap::structure::<id>::last}}` 读取（聊天中最近一次的非空值，`kv` 形式为 JSON）。适合需要逐轮延续的数据：

```yaml
structures:
  status:
    kind: custom
    shape: kv
    title: Status
    tag: status
    field: status
    description: Current status of {{ap::name::char}}.
    formatPrompt:
      tag: "<status><mood>…</mood><location>…</location></status>"
      field: "Fill `{{field}}` with keys `mood` and `location`."
```

```markdown
Previous status: {{ap::structure::status::last}}
```

## 卡片

结构卡片分两个区域：**顶部**（阶段卡片与正文之间）与**底部**（正文之下）。每种内置结构有默认区域与排序键，区域内按键从小到大排列，键相同时按结构声明顺序：

| 区域 | 内置卡片（排序键） |
| --- | --- |
| `top` | `status`（100） |
| `bottom` | `reply`（100）、`impersonate`（200）、`options`（300）、`synopsis`（400）、`memory`（500）、`recap`（600）、`custom`（700） |

`ui.position` / `ui.order` 覆盖默认值，例如把选项放到摘要之后：`options: { kind: options, ui: { order: 450 } }`。顶部区域在流式生成时就显示（实时值），底部区域在生成结束后显示。

### 自定义组件

`custom` 结构可以带自己的前端组件（I-137），替代通用卡片，挂载在自己的 Shadow DOM 中（样式与页面互不影响）：

```yaml
structures:
  mood:
    kind: custom
    shape: kv
    ui:
      position: top
      order: 50
      template: ./ui/mood.ejs        # 或内联源码
      style: ./ui/mood.css
```

- **模板模式** `template`：EJS 模板（语法同 [templates.md](templates.md)），每次值变化时重新渲染为 HTML。可用变量：`value`（结构值）、`structure`（`{ id, kind, title }`）、`structures`（本条消息所有结构值）、`message`（`{ id, streaming, editable }`）、`preset`（`{ id, name, version }`）、`lang`。点击带 `data-ap-fill="文字"` 的元素会把文字填入输入框（按住修饰键直接发送）。适合简单 UI；
- **模块模式** `module`：ES 模块源码，可以打包任意前端框架。导出 `mount(root, context)`（或默认导出函数 / `{ mount }`），`root` 是组件的 Shadow Root；返回清理函数或 `{ update(context), unmount() }`，没有 `update` 时值变化会重新挂载。`context` 含上面的变量，另有 `setValue(value)`（覆盖该楼层的结构值，仅生成结束后可用）与 `fillInput(text, send?)`；
- `style`：放进 Shadow Root 的 CSS（两种模式通用）；
- 三个字段都可以写内联源码，或写 `./ui/…` 单行路径引用预设目录 `ui/` 下的文件（构建时内联进产物）；`template` 与 `module` 二选一；`ap check` 检查模板语法、模块是否有导出；
- **信任**：组件是预设附带的代码。安装 / 更新带组件的预设时，AP 弹窗询问“信任并启用”或“暂不启用”；未启用时组件**不显示**（也不显示通用卡片），之后可在面板 **配置 → 自定义 UI** 中开启。信任绑定组件代码的指纹，更新改动了组件代码时重新询问。内置预设与 `ap dev` 源默认启用；
- 组件出错时卡片位置显示一行错误信息，不影响其他卡片。

### annotations

行内批注：模型在正文中用 CriticMarkup 的子集标出一段文字并附上批注：

```text
她{==轻轻推开门==}{>>这里的“轻轻”呼应上一幕的犹豫。<<}，走了进去。
```

```yaml
structures:
  annotations:
    kind: annotations
    history: 0
    formatPrompt:
      tag: |-
        Inline annotations are written inside the story text as `{==phrase==}{>>note<<}`.
        Mark a short phrase or a single sentence, never across paragraphs, and never nest marks.
```

`formatPrompt` 只需写 `tag` 或 `field` 之一，另一个自动取相同文字（行内写法与输出方式无关）。建议这里只写语法，批注的用途、口吻与数量写在请求它的模组里，这样不同模组（如作者评论、术语注释）可以复用同一份语法说明。

**语法**（严格成对）：

| 写法 | 结果 |
| --- | --- |
| `{==原文==}{>>批注<<}` | 一条批注；正文保留“原文” |
| 单独的 `{==原文==}` | 去掉标记，保留“原文” |
| 单独的 `{>>批注<<}` | 整段移除 |
| 未闭合的 `{==` | 丢弃标记，保留其后文字 |
| 未闭合的 `{>>` | 按原文保留 |

标记不能跨段落（空行），也不能嵌套。被批注的“原文”应是短语或句子。

**保存**：消息的 `mes` 始终是**去掉标记后的干净正文**；批注单独存于 sidecar 的 `annotations`（见 [artifact-and-runtime.md](artifact-and-runtime.md#行内批注)）。每条批注用偏移 + 原文 + 前后各约 32 字的上下文锚定；用户编辑正文后 AP 重新定位，找不到的批注成为**孤立批注**——保留但不显示，可在调试页查看。工作流的补丁作用于带标记的文本，因此补丁前后各有一份批注，随“显示 / 撤销补丁”切换。

**流式**：生成过程中 AP 只把已确定的部分交给 ST，标记不会闪现；下划线在生成结束后出现。

**历史**：默认（`history: 0`）历史中只有干净正文。`history: N` 时，最近 N 条助手消息（未被用户改动过的）在组装时把批注以 CriticMarkup 重新注入，帮助模型保持批注风格；代价是更多 token。

**显示**：被批注的文字显示虚线下划线（CSS Custom Highlight API，不改动消息 DOM；不支持时退化为 `<ap-anno>` 包裹），悬停或轻点弹出气泡，署名为叙述者。同源 iframe 前端（如酒馆助手渲染的界面）自动适配，详见 [artifact-and-runtime.md](artifact-and-runtime.md#行内批注)。用户可在外观设置中关闭显示或更改下划线颜色。

请求方式与其他结构相同：由生效模组的 `structures: [annotations]` 请求，并在正文里放 `{{ap::structure::annotations}}` 输出格式说明。

### 保存条件

记忆与总结只在**拦截器参与的生成**中保存，且生成类型不是 `quiet`（后台生成）。所有结构值都写在消息的 `extra.advanced_preset.structures` 中，行内批注除外（`extra.advanced_preset.annotations`，见 [artifact-and-runtime.md](artifact-and-runtime.md)）。
