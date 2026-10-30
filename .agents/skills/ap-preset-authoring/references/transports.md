# 传输方式

**传输方式**（transport）决定 AP 如何把一轮工作流交给模型、又如何从回复里读出阶段与结构。它由配置 `connection.transport` 选择（默认 `structured`，见 [configuration.md](configuration.md)），预设通过 `styles` 为每种传输方式指定**写法**（见 [manifest.md](manifest.md)）。

| 传输方式 | 默认写法 | 请求 | 输出解析 |
| --- | --- | --- | --- |
| `prefill` | 对话式 | 预填充整形：历史合并为一条助手消息，模型接着写；附加停止词 | 角色前缀 / 标签切分 |
| `plaintext` | 对话式 | 普通聊天补全 | 角色前缀或标签切分（`plaintextFormat`） |
| `structured` | 指令式 | `json_schema` 结构化输出 | 一个 JSON 对象 |
| `tool-single` | 指令式 | 一个提交工具，参数即整轮 | 提交工具调用的参数 |
| `tool-multi` | 指令式 | 每个阶段一个工具，循环多步 | 各阶段工具调用的参数 |

写法与传输方式是两回事：写法决定提示词怎么写（`style` 变量、`@style` 片段、分节选用、`{{ap::structure::…}}` 输出哪份格式说明），传输方式决定请求怎么发、回复怎么读。默认组合是“文本传输 + 对话式、JSON 传输 + 指令式”。

所有传输方式都以流式方式解析（如果 ST 开启了流式），阶段边写边显示在消息上方的阶段面板中。

## prefill

请求前做**预填充整形**（[assembly.md](assembly.md#预填充整形)）：`historyStart` 区域之前（含该区域）的消息合并为一条 system 消息；然后是一条用户消息（`prefillGuide`，空时为 `<START>`）；最后是一条助手消息，按 `historyItemTemplate` 把历史逐条拼接起来。模型从助手消息末尾接着写。

- 清单的 `stop` 与 ST 自带的停止词合并后发送；客户端再按 `stop` 截断一次（有些后端不支持停止词）；
- 输出按对话式解析（[对话式解析](#对话式解析)），阶段分隔方式总是 `prefix`。

继续（continue）与代写（impersonate）生成不做整形。

## plaintext

普通聊天补全，不附加任何参数。输出按对话式解析，阶段分隔方式由清单 `plaintextFormat` 决定：

- `prefix`（默认）：按角色前缀切分；
- `tag`：每个阶段包在 `<阶段 tag 或阶段 id>…</…>` 中。

## structured

请求附加 ST 的 `json_schema` 参数：

```json
{ "name": "<tools.turn.name>", "description": "<tools.turn.description>", "value": <回合 schema>, "strict": false }
```

从回复正文读出 JSON（流式时容忍不完整 JSON）；正文为空时改读第一个工具调用的参数（有些后端把结构化输出实现为工具调用）。

## tool-single

工具列表依次为：示范轮次工具（如果有）、**提交工具**、调用方工具。

- 提交工具：名称 `tools.submit.name`，说明 `tools.submit.description`，参数就是回合 schema；
- `tool_choice`：`connection.forceTool` 为真且没有调用方工具时强制调用提交工具，否则 `auto`；
- 读取提交工具调用的参数（只有一个无名调用时也接受它）。

## tool-multi

每个**生效阶段**一个工具（[阶段工具](#回合-schema-与阶段工具)）；结构作为[锚点阶段](workflows.md#结构的位置)工具的附加参数。AP 在一次生成内部循环：

1. 发送请求，读取模型的工具调用；
2. 对每个属于本轮的调用，记录阶段值，并回一条工具结果：还有未完成阶段时为 `tools.results.next`（`{{next}}` = 下一个工具名），否则 `tools.results.done`；不认识的工具回 `tools.results.unknown`（`{{name}}` = 被调用的名字）；
3. 所有必需阶段都已记录 → 结束；否则继续下一步，最多 `connection.maxSteps` 步（默认 6）；
4. 某一步没有调用任何本轮工具时，追加一条用户消息 `tools.results.nudge` 催促一次；再次没有调用则结束。

`tool_choice`：`forceTool` 为假或有调用方工具时为 `auto`；否则 `connection.parallelTools` 为真时为 `required`（允许一次调用多个工具），为假时强制调用下一个未完成阶段的工具。可选阶段（`optional: true`）不计入“必需阶段”。

循环中的中间消息只存在于请求里，不写入聊天。

## 回合 schema 与阶段工具

**回合 schema**（structured 的 `value`、tool-single 的提交工具参数）：

```json
{
  "type": "object",
  "properties": {
    "step1_narrator_think": { "type": "string", "description": "<阶段说明>" },
    "step2_narrator_content": { "type": "string", "description": "…" },
    "z_options": { "…结构字段…" },
    "step3_editor_review": { "type": "string" }
  },
  "required": ["step1_narrator_think", "step2_narrator_content", "z_options", "step3_editor_review"],
  "additionalProperties": false
}
```

- 属性按生效阶段的顺序排列，字段名来自 `stage.field.name` 或模板 `tools.turn.field`（`{{n}}` 是阶段在**本轮生效阶段**中的序号，见 [workflows.md](workflows.md)）；
- 字段说明来自 `stage.field.description` 或模板 `tools.turn.fieldDescription`（默认 `{{label}}`）；
- 结构字段插在锚点阶段之后（没有阶段时放在最后），字段名带 `tools.structurePrefix`（默认 `z_`），schema 见 [structures.md](structures.md#指令式json-schema)；
- 字段名应同时**按字母顺序**排列：部分 API（如 Gemini）忽略 schema 顺序、按字母顺序生成字段。默认的 `step{{n}}_…` 与 `z_` 前缀满足这一点（阶段不超过 9 个时）；`ap check` 会对不满足的命名告警，见 [structures.md](structures.md#字段前缀)；
- `required` 包含所有非可选阶段与必填结构；
- `patch` 阶段的值是 `{ old_string, new_string }` 数组，说明来自 `tools.patch`。

**阶段工具**（tool-multi）：名称来自 `stage.tool.name` 或模板 `tools.stage.name`（默认 `ap_stage_{{stage}}`），说明来自 `stage.tool.description` 或 `tools.stage.description`。参数只有一个：`tools.stage.param.name`（默认 `content`，字符串），`patch` 阶段为 `tools.stage.patchParam.name`（默认 `edits`，数组）；锚点阶段的工具另带结构参数。

解析时字段名宽松匹配：找不到精确字段时，接受键名等于阶段 id 或以 `_<阶段 id>` 结尾的字段（模型丢掉序号前缀时仍能读出）。

## Gemini

Gemini（makersuite / vertexai）只接受 JSON Schema 的 OpenAPI 子集。AP 在发送前自动转换 `json_schema` 与工具参数：

- `type: ["string", "null"]` → `type: "string", nullable: true`；`oneOf` → `anyOf`；`const` → `enum`；
- 去掉 `additionalProperties` 等不支持的关键字，为对象加 `propertyOrdering`（保持字段顺序；部分接口或中转不传递它，此时按字母顺序生成，由字段命名保证顺序）；
- 自由键对象（如 `kv` 自定义结构）改为字符串字段，说明后追加 `tools.jsonStringHint`，模型写 JSON 字符串，AP 再解析。

`connection.streamToolArguments`：`auto`（默认，仅 Vertex AI 请求流式工具参数）、`on`（所有 Gemini 形态的连接都请求）、`off`。

`connection.sendThoughtSignature`：为真且连接是 Gemini 时，示范轮次的工具调用带上 `thoughtSignature`。

## 对话式解析

prefill 与 plaintext 的输出按以下步骤解析（流式时对累计文本反复执行）：

1. 去掉 HTML 注释 `<!-- … -->`；
2. **用户前缀截断**：行首出现用户前缀（如 `{{user}}: `）说明模型开始替用户说话，从那里截断；
3. **提取结构标签**并从文本中移除（[structures.md](structures.md#对话式标签格式)）；
4. **标签阶段**：设置了 `tag` 的阶段（`plaintextFormat: tag` 时所有阶段都以 `tag` 或 id 为标签）按标签提取，多个块以空行连接；`patch` 阶段每个块是一处替换；
5. **前缀阶段**：其余阶段按行首的角色前缀切分（用户以外的所有角色）。每个片段归入“下一个属于该角色的阶段”；开头没有前缀的文字归入第一个阶段；
6. **正文标记**：片段中出现 `contentMarker`（默认 `<START_OF_CONTENT>`）时，标记之后归入终结正文阶段；进入正文之后，只有**排在正文之后的阶段**的角色前缀才会再切分，其他前缀（例如对话中的“Freesia: ”）都留在正文里；
7. **回退**：生成结束时如果终结正文为空——
   - 取最后一个与正文同角色的已有文字阶段作为正文（警告 `content-marker-missing`）；
   - 如果所有阶段都没有文字，整段文本作为正文（警告 `no-stages-found`）。

  设置了 `contentMarker` 但没出现、而正文仍有内容时，也给出 `content-marker-missing` 警告。

如果所有阶段都有标签（例如任务工作流：`<thinking>` + 自由正文），剩余文字归入终结正文。

## 生成之后

对所有传输方式：

- **补丁**：如果有 `patch` 阶段，生成结束后把补丁按顺序应用到终结正文：先精确匹配，再忽略空白差异匹配；匹配 0 次或多次的补丁跳过，并警告 `patches: x/y skipped`。原稿保存在 sidecar 的 `draft`；
- **最终回退**：故事轮的正文为空且 `connection.finalFallback` 为真（默认）时，把模型原始文本（去注释）作为正文，除非它以 `{` 开头（警告 `fallback: plain text used as content`）；
- **流式门控**：只有“故事轮 + 没有调用方 schema + 没有 patch 阶段”时，终结正文才实时流进聊天消息；其他情况在结束时一次写入（过程只显示在阶段面板中）。

## 能力检查

使用 `tool-single` / `tool-multi` 时，如果 ST 当前的连接不支持工具调用，生成会直接报错并提示切换传输方式。

## tools 文案

模型在指令式传输中看到的每个名字与说明都来自清单的 `tools`。缺省值：

```yaml
tools:
  turn:
    name: ap_turn                              # json_schema 的 name
    description: ""                            # json_schema 的 description（可选）
    field: "step{{n}}_{{actor}}_{{stage}}"     # 阶段字段名模板
    fieldDescription: "{{label}}"              # 阶段字段说明模板
  submit:
    name: ap_submit_turn
    description: "Submit the complete workflow for this turn: every stage in order, then the final content and structures."
  stage:
    name: "ap_stage_{{stage}}"                 # 阶段工具名模板
    description: "{{label}}"
    param: { name: content, description: "" }
    patchParam: { name: edits, description: "" }
  patch:                                       # patch 值的说明（可选）
    description: ""
    old: ""                                    # old_string 的说明
    new: ""                                    # new_string 的说明
  envelope:                                    # 调用方 schema 的信封
    thinking: { name: thinking, description: "" }
    result: { name: result, description: "" }
  results:
    next: "Recorded. Continue with `{{next}}`."
    done: "Recorded. The workflow is complete."
    unknown: "Unknown tool {{name}}."
    nudge: "Continue the workflow: call `{{next}}` now."
  structurePrefix: z_                          # 结构字段名前缀（构建时应用，"" 为不加）
  jsonStringHint: "(JSON object encoded as a string)"
  callerSchemaHint: "After your thinking, output the result as one JSON code block (```json … ```) that matches this JSON Schema:\n\n```json\n{{schema}}\n```"
```

- 模板占位符：`{{n}}`（生效阶段序号）、`{{stage}}`（阶段 id）、`{{actor}}`（角色 id）、`{{label}}`（阶段标签）；工具文本中还有 `{{next}}` `{{first}}` `{{submit}}` `{{name}}`；`callerSchemaHint` 中有 `{{schema}}`；
- 说明文字支持 `{{ap::…}}` 宏与 ST 宏；**名字不做宏展开**；
- 单个阶段可以在工作流里覆盖自己的字段名 / 工具名 / 说明（[workflows.md](workflows.md)）；
- `structurePrefix` 在构建时加到每个结构的 `field` 上，结构说明中的 `{{field}}` 是加前缀后的名字（[structures.md](structures.md#字段前缀)）；
- 名字规则（报错）：工具名匹配 `^[A-Za-z0-9_-]{1,64}$`，字段名与参数名匹配 `^[A-Za-z_][A-Za-z0-9_]{0,63}$`；信封的两个字段不能同名；提交工具不能与示范轮次工具同名；
- 旧写法 `stagePrefix`、`stageResult`、`nudge`、`submit: <文字>` 仍能编译但给出警告；未知键给出警告。

## 示范轮次 `syntheticTurns`

工具传输可以在提示词末尾插入一轮**虚构的工具往返**，让模型“看到”自己已经开始了工作流，从而更稳定地调用工具：

```yaml
syntheticTurns:
  - id: init
    label: Workflow init
    user: <!-- Workflow Start -->
    tool:
      name: start_workflow
      description: Open the writing session for this turn.
    thoughtSignature: AY89a19r…        # 可选，Gemini 专用，不要手改
    response:
      single: Workflow initialised. Now call `{{submit}}` exactly once.
      multi: Workflow initialised. Call the stage tools in order, starting with `{{first}}`.
      parallel: …                       # 缺省同 multi
```

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | — | 配置 `connection.syntheticTurn` 用它选择 |
| `label` | `id` | 面板显示名 |
| `user` | `''` | 插入的用户消息 |
| `tool.name` | `ap_workflow_init` | 虚构工具名（须匹配 `^[A-Za-z_][\w-]*$`） |
| `tool.description` | `''` | 虚构工具说明 |
| `thoughtSignature` | — | 仅当 `connection.sendThoughtSignature` 为真且连接是 Gemini 时附在工具调用上 |
| `response.single` / `multi` / `parallel` | `''`（`parallel` 缺省同 `multi`） | 工具结果文字，按 tool-single / tool-multi / tool-multi 并行选用 |

插入的消息依次是：用户消息 `user` → 助手调用虚构工具（参数 `{}`）→ 工具结果 `response[模式]`。虚构工具也会加入工具列表。占位符：`{{submit}}` 提交工具名、`{{first}}` 第一个阶段工具名。

配置 `connection.syntheticTurn` 默认是第一个示范轮次的 id（没有时为 `null`，即不插入）。prefill / plaintext / structured 不使用示范轮次。

## 调用方的工具与 schema

脚本或其他扩展调用生成时（`gen.origin == "task"`）可能自带工具或 JSON Schema，AP 会兼顾：

- **调用方工具**：加入工具列表（预设自己的工具名会被过滤掉）。模型调用了调用方工具时，本轮立即结束，把调用原样交还调用方；
- **调用方 schema**：
  - structured / tool-single：schema 改为信封 `{ <thinking>: string, <result>: <调用方 schema> }`；`thinking` 显示为第一个文字阶段，`result` 交给调用方；
  - prefill / plaintext：在提示词末尾（预填充助手消息之前）追加一条 system 消息 `tools.callerSchemaHint`，然后从回复中提取第一个 ```` ```json ```` 代码块；
  - tool-multi 不提取调用方结果，需要结构化结果的调用方应配合其他传输方式；
- quiet / 脚本生成时，消息正文为调用方结果的 JSON；没有结果时为正文（或去注释的原始文本）。

`ap preview --json` 会输出本轮的 schema 与工具定义（见 [cli.md](cli.md)）。
