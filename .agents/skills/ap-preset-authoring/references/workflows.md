# 工作流文件

工作流是 `workflows/` 目录下（只认这一层）的 Markdown 文件，一个文件一个工作流。frontmatter 声明**阶段**，正文用 `@section` 写出每个阶段的说明。

```markdown
---
id: linear
name: Linear
description: Plan, then write.
hint: (plan first, then write the content)
stages:
  - { id: plan, actor: narrator, kind: text, label: Planning }
  - { id: final, actor: narrator, kind: content, label: Writing }
---
<!-- @section intro style=imperative -->
Complete these stages in order:

<!-- @section stage:plan style=dialogue -->
{{ap::prefix::narrator}}First I will outline the next scene in a few bullet points.

<!-- @section stage:plan style=imperative -->
Outline the next scene in a few bullet points.

<!-- @section stage:final style=dialogue -->
{{ap::prefix::narrator}}Then I will output <START_OF_CONTENT> and write the story.

<!-- @section stage:final style=imperative -->
Write the story content.
```

## frontmatter

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | 文件名 | 工作流 id，条件变量 `workflow` 的值 |
| `name` | `id` | 面板显示名 |
| `description` | — | 面板说明 |
| `kind` | `story` | `story`：故事工作流（用户可选）；`task`：任务工作流（元指令、脚本调用时使用） |
| `hint` | — | `{{ap::workflow::hint}}` 的值，通常是一句对工作流的概括 |
| `stages` | — | **必填**，至少一个阶段 |
| `structures.after` | 终结正文阶段 | 结构字段放在哪个阶段之后（见下） |

故事工作流必须至少有一个 `content` 阶段（报错）。预设没有任务工作流时给出警告：元指令回复与脚本调用会退回到主工作流。

## 阶段 `stages`

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | — | **必填**，字母开头（字母、数字、`_`、`-`），工作流内唯一。条件变量 `stages.<id>` |
| `actor` | — | **必填**，`actors` 中的角色 id |
| `kind` | `text` | `text`：思考、提问、规划等过程文字；`content`：正文；`patch`：对正文的精确替换 |
| `label` | `id` | 面板与阶段卡片上的名字，也是 `{{label}}` 占位符的值 |
| `optional` | `false` | 可选阶段：不进入 schema 的 `required`；摘要中标注 `[optional]`；多工具模式不催促 |
| `when` | — | 条件；为假时本轮去掉这个阶段（编号、字段名随之重排） |
| `tag` | — | 对话式输出中把阶段包在 `<tag>…</tag>` 里（如 `<thinking>`） |
| `field` | — | 结构化 / 单工具模式的字段：`{ name, description }`，或只写字符串作为 description |
| `tool` | — | 多工具模式的工具：`{ name, description, param: { name, description } }`，或只写字符串作为 description |
| `executor` | `default` | 预留，只支持 `default` |

### 阶段种类

- **`text`**：显示在阶段卡片里，不进入正文。
- **`content`**：最后一个 `content` 阶段是**终结正文阶段**，它的文字成为消息正文（`mes`）。前面若还有 `content` 阶段（例如“初稿”），当终结阶段为空时会退而使用最后一个非空的 `content` 阶段。
- **`patch`**：模型给出 `{ old_string, new_string }` 列表，生成结束后按顺序应用到正文上（先精确匹配，再忽略空白差异匹配；匹配 0 次或多次的替换被跳过并记录）。原稿保存在 sidecar 的 `draft` 中，用户可以在 UI 中切换原稿与修订稿。对话式输出中，`patch` 阶段需要 `tag`，**每个标签块是一处替换**：

  ```
  <Freesia_Edit>
  <old_string>原文中要替换的确切文字</old_string>
  <new_string>替换后的文字</new_string>
  </Freesia_Edit>
  <Freesia_Edit>
  …
  </Freesia_Edit>
  ```

  指令式输出中，`patch` 阶段的值是数组 `[{ "old_string": "…", "new_string": "…" }]`（也接受 `{ "edits": [...] }` 与 `old` / `new` 的简写）。

### 字段名与工具名

指令式传输中，阶段对应 JSON 字段（结构化、单工具）或独立工具（多工具）。名称由模板生成：

| 名称 | 来源（优先级从高到低） | 默认模板 |
| --- | --- | --- |
| 字段名 | `stage.field.name` → `tools.turn.field` | `step{{n}}_{{actor}}_{{stage}}` |
| 字段说明 | `stage.field.description` → `tools.turn.fieldDescription` | `{{label}}` |
| 工具名 | `stage.tool.name` → `tools.stage.name` | `ap_stage_{{stage}}` |
| 工具说明 | `stage.tool.description` → `tools.stage.description` | `{{label}}` |
| 工具参数名 | `stage.tool.param.name` → `tools.stage.param.name`（`patch` 阶段为 `tools.stage.patchParam.name`） | `content`（`patch`：`edits`） |

模板占位符：`{{n}}`（在本轮**激活**阶段中的序号，从 1 开始）、`{{stage}}`（阶段 id）、`{{actor}}`（角色 id）、`{{label}}`。说明文字还会做宏展开；名称不会。

名称必须合法且不冲突（报错）：字段名、参数名匹配 `^[A-Za-z_][A-Za-z0-9_]{0,63}$`，工具名匹配 `^[A-Za-z0-9_-]{1,64}$`；字段名不能与结构字段或其他阶段重复；工具名不能与其他阶段、提交工具或示范轮次工具重复。

> 让字段名带上 `{{n}}` 有助于模型按顺序输出字段。解析时也会容忍模型丢掉数字前缀：键名等于阶段 id 或以 `_<阶段id>` 结尾都能识别。

## 分节 `@section`

正文必须以 `@section` 开始（之前的文字被忽略并警告）。

| 分节 | 说明 |
| --- | --- |
| `preamble` | 两种写法共用的开头，不能带 `style` |
| `intro` | 阶段说明之前的引导 |
| `stage:<id>` | 某个阶段的说明，`<id>` 必须是已声明的阶段 |
| `outro` | 阶段说明之后的收尾 |

除 `preamble` 外都可以加 `style=dialogue` 或 `style=imperative`；不加则两种写法共用同一份。同一分节在同一写法下只能出现一次。分节内可以使用 `@as`、`@style`、`@if` 和宏。

**渲染顺序**：`preamble` → 当前写法的 `intro` → 每个激活阶段（按顺序）的当前写法分节 → 当前写法的 `outro`。指令式写法下，每个阶段分节前自动加 `stageHeading` 标题（见 [manifest.md](manifest.md#指令式阶段标题-stageheading)）。未激活（`when` 为假）的阶段，其分节不渲染。

工作流内容放进 `workflowRegion`，排在该区域所有条目**之后**；不带 `@as` 的块合并为同一条消息。

某个阶段缺少某种写法的分节时给出警告（该写法下这个阶段没有任何说明）。

## 结构的位置

结构（选项、事件摘要等）在 schema 和多工具中跟在某个阶段之后：默认是终结正文阶段，可以用 `structures: { after: <stage id> }` 改为其他阶段（例如审阅阶段之后）。多工具模式下，结构作为这个阶段工具的附加参数。

## 工作流摘要

`{{ap::workflow::summary}}` 输出本轮激活阶段的流程：

```
Start -> Planning (Narrator) -> Writing (Narrator) [optional] -> (End)
```

## 工作流选择

- 故事生成：使用配置 `connection.workflow` 指定的工作流；找不到时用第一个故事工作流（受 `workflowOrder` 影响）。`connection.workflow` 不能是任务工作流（报错）。
- 任务生成（纯元指令、脚本调用等）：使用第一个 `kind: task` 的工作流；没有则找 id 为 `task` 的工作流。
- ST 的“继续”和“代写”不使用工作流（见 [meta-and-tasks.md](meta-and-tasks.md)）。

## 任务工作流的写法

任务工作流通常是“思考 + 结果”：

```yaml
stages:
  - { id: thinking, actor: narrator, kind: text, tag: thinking, label: Thinking }
  - { id: content, actor: narrator, kind: content, optional: true, label: Output }
```

纯元指令的回复主要通过 `reply` 等结构交付；`content` 设为可选，只有用户明确要求故事内容时才写。当调用方（脚本 / 其他扩展）带了自己的 JSON Schema 时，AP 会改用“思考 + 结果”信封，见 [transports.md](transports.md#调用方的工具与-schema)。
