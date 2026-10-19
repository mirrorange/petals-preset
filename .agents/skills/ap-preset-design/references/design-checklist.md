# 设计访谈清单

按顺序推进。每一节先问“必须问”的问题；“推荐”是用户没有偏好时的默认答案。每节结束时把结论写进 `DESIGN.md` 对应的段落。

## 1. 核心体验

必须问：

- 模型扮演谁？（一位叙述者 / 一个角色 / 作者 + 编辑 / 游戏主持人……）
- 和用户是什么关系？用户以什么身份参与？
- 每轮回复包含什么？（只有正文 / 正文 + 选项 / 正文 + 状态栏……）
- 主要的使用场景：角色扮演、长篇小说、跑团、文字冒险、陪伴聊天？

推荐：

- 一位有个性的叙述者（narrator）+ 用户（user）两个角色就够用；需要“自我批评 / 讨论”时再加一位编辑类角色（花瓣：Freesia + Petals）。
- 角色要有名字和简短人设，元指令时以这个身份回复。

写进设计稿：角色表（id、名字、前缀、人设一句话）、核心体验一段话。

## 2. 模型与传输方式

必须问：

- 主要用哪些模型 / API？（DeepSeek、Claude、Gemini、GPT、本地模型、各种中转……）
- 是否需要支持没有工具调用 / 结构化输出能力的接口？

推荐（可以按用户的模型调整）：

| 模型 | 传输方式 | 说明 |
| --- | --- | --- |
| Gemini | `tool-single`（`sendThoughtSignature: true`） | 配合示范轮次 |
| DeepSeek 官方 API | `prefill` 或 `tool-single` | prefill 只在支持续写 assistant 的 API 上可用 |
| Claude（较新） | `plaintext` 或 `tool-single` | 旧版 Claude 用 `prefill` |
| 其他 OpenAI 兼容接口 | `structured`（默认）或 `tool-single` | |

决定之后，**写法**随之确定：`prefill` / `plaintext` 默认对话式，其余默认指令式（`styles` 可改）。如果同时支持两类，关键提示词（身份、工作流、结尾）都要写两份。只支持一类时，可以只写一种写法，并在 `styles` 里把所有传输方式映射到它。

## 3. 区域

一般不用问用户，直接按位置给出方案，请用户确认：

| 区域 id | 角色 role | 合并 merge | 锚点 | 内容 |
| --- | --- | --- | --- | --- |
| `top` | system | join | — | 身份、核心规则 |
| `guidelines` | system | `{ wrap: guidelines }` | — | 写作指南（包在 `<guidelines>` 中） |
| `settings` | — | — | `native: true` | ST 的角色卡、世界书、人设 |
| `history` | system | — | `historyStart` | 总结、背景，紧挨聊天记录之前 |
| `memory` | system | — | `historyEnd` | 记忆，紧挨聊天记录之后（可选） |
| `task` | system | — | `historyEnd` | 工作流说明 / 元指令任务（`workflowRegion`） |
| `tail` | system | — | `tail` | 最后一句话：引出输出（预填充时是 assistant 前缀） |

需要“对话式开场”（用户与叙述者的几句寒暄）时，加一个区域放 `@as user` / `@as assistant` 对话。需要被引用的内容（文风、角色设定）用 `wrap` 包成标签。

## 4. 模组

必须问：

- 有哪些规则 / 功能？逐条列出（让用户随便说，你来归类）。
- 哪些必须总是生效？哪些要能开关？默认开还是关？
- 哪些互相排斥（只能选一个）？能不能一个都不选？
- 有没有只适用于特定模型的内容（如“跳过思考 (Gemini)”）？

归类规则：

| 类型 | 写法 | 例子 |
| --- | --- | --- |
| 框架 | `toggle: always` | 身份、输出格式、上下文、元指令处理 |
| 开关 | `toggle: user` + `group` + `default` | 防回避、节奏控制、事件摘要、选项 |
| 单选集 | `exclusive: <分组>`（分组可设 `allowNone`） | 续写 / 扩写；几种文风；防抢话 / 允许抢话 |
| 任务 | `toggle: always` + `when` | `when: meta.pure`、`when: 'meta.task == "summary"'` |
| 模型专用 | 开关，默认关，由推荐配置打开 | 跳过思考 (Gemini) |

每个模组写清：id、名字、类型、分组、默认值、区域、条件（`when`）、一句话内容。故事专用的模组加 `when: "!meta.pure"`。

## 5. 参数

必须问：

- 用户要能设置什么？（回复语言、对话语言、字数、视角、人称、文风强度……）
- 每个参数没填时怎么办？（沿用故事语言 / 不提字数要求……）

推荐：回复语言（`text`，带常用选项，`unset: the language used in the story so far`；id 用 `language`，这样 `{{ap::lang}}` 会自动读取）、回复长度（`number`，`unit: words`，`0` 表示不限，并用 `@if params.length > 0` 包住）。参数放在 `tab: writing` 的分组里。

## 6. 工作流

必须问：

- 每轮回复前希望模型先做什么？（规划要点、回答问题清单、写草稿、自检……）
- 需要几档？（快速 / 标准 / 精修）
- 过程文字给用户看吗？（阶段卡片显示在消息上方，正文保持干净）

推荐：

- `linear`：`plan`（text）→ `final`（content）；
- 需要质量时加 `iterative`：`questions` → `draft`（content）→ `review`（text）→ `final`（content）；或 `edit`：写完后 `patch` 阶段精确替换；
- `task`（`kind: task`）：`thinking`（text，`tag: thinking`）→ `content`（content，`optional: true`）——处理纯元指令与代写；
- 阶段 id 用有意义的英文：它们会变成 JSON 字段名和工具名（`ask_questions`、`write_story`）；
- 需要“清单式提问”时，用构建变量（`dict`）收集各模组贡献的问题，工作流里用 `{{ap::var::<id>::values}}` 列出。

## 7. 内置功能

逐项问要不要（给出推荐）：

| 功能 | 推荐 | 需要的东西 |
| --- | --- | --- |
| 元指令 `@叙述者` | 要 | `meta.trigger`、处理元指令的任务模组、`reply` 结构、任务工作流 |
| 记忆 | 可选 | `memory` 结构 + 记忆区域 + 说明“只在用户要求时更新” |
| 总结 | 可选 | `recap` 结构 + 正则标记（总结 / summary）+ `meta.tasks.summary` |
| 代写 | 可选 | `impersonate` 结构 + `meta.impersonatePrompt` / 正则标记 |
| 事件摘要 + 省 token | 长对话推荐 | `synopsis` 结构 + `tokenSaving` |
| 选项 | 推荐 | `options` 结构（`count`） |
| 状态栏等 | 按需 | `custom` 结构（`display: card` / `hidden`），用 `::last` 逐轮延续 |

## 8. 推荐配置

为第 2 节确定的每类模型准备一套：`connection`（传输方式、工作流、示范轮次）、`sampling`（温度、最大输出、`reasoning_effort` 等 ST 预设键）、`modules`（模型专用模组）。选一套作为 `defaultConfiguration`（通常是最通用的 structured）。

## 9. 扩展与界面

- 插入点：3–6 个，覆盖“身份之后、写作指南内、任务之前”等常用位置。id 稳定。
- 面板语言：模组 / 参数 / 分组名用英文写在源码里，再用 `locales/zh-cn.yaml` 翻译；或直接用中文写名字。
- 主题色（`theme`，可选）。

## 10. 发布

- 预设 id（kebab-case，发布后不改）、显示名、作者、版本（从 `0.1.0` 或 `1.0.0` 开始）。
- 托管：GitHub 仓库，提交 `dist/`，用户通过“导入链接”安装（见 `ap-preset-testing` 技能）。

## 迁移旧 ST 预设

用户提供 ST Chat Completion 预设 JSON 时：

1. 读 `prompts` 与 `prompt_order`：每个启用的条目记下名称、角色、位置（相对聊天记录）、是否默认启用、内容主题。
2. 归类：
   - 固定的身份 / 规则 → 框架模组（`toggle: always`）；
   - 可开关的条目 → 开关模组；内容互斥的几条 → 单选集；
   - “思维链 / CoT / 思考格式”条目 → 工作流阶段（不要照搬成模组）；
   - 用正则提取的选项、状态栏、摘要 → 结构（`options` / `custom` / `synopsis`）；
   - 角色卡、世界书、人设等 marker → 原生区域 `settings`（交给 Carrier 的 `order`）；
   - 位置在聊天记录之后的条目 → `task` / `tail` 区域。
3. 采样参数（`temperature`、`openai_max_tokens` 等）→ `configurationDefaults.sampling`。
4. 只适用于某个模型的“破限 / 越狱”写法 → 模型专用开关，由对应推荐配置打开。
5. 把归类表写进 `DESIGN.md` 的“迁移对照”一节，请用户确认哪些不要了。
