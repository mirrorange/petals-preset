# 一次生成的完整流程

本文按时间顺序描述 AP 处理一次生成的每一步。各步骤的细节分别在其他参考文件中，这里给出它们的先后关系与相互影响，便于排查“为什么这段文字没出现 / 出现在这里”。

```
① 拦截器 ─▶ ② ST 组装 ─▶ ③ PROMPT_READY：AP 渲染区域 ─▶ ④ 整形 ─▶ ⑤ 接管请求 ─▶ ⑥ 解析 ─▶ ⑦ 写入
```

## 拦截器

拦截器（`generate_interceptor`）在 ST 原生 `Generate` 开始时运行（脚本直接调用的生成跳过这一步）：

1. 当前选中的 ST 预设不是 Carrier → AP 不参与；
2. 取当前输入（最后一条用户消息；`continue` 时没有输入），**分类**：元指令、正则标记、内置任务、生成来源（[meta-and-tasks.md](meta-and-tasks.md)）；
3. 记录关键词规则要用的文本（改写前的原文）；
4. **改写历史副本**：剔除纯轮次、剥离元指令、省 token（[meta-and-tasks.md](meta-and-tasks.md#历史改写)）；
5. 留下待用的生成上下文。

## ST 组装

ST 按 Carrier 的 `prompt_order` 组装提示词：原生提示词（角色卡、世界书、聊天记录……）照常填入，AP 的每个区域是一条只含锚点 `⟦ap:region:<id>⟧` 的提示词（[configuration.md](configuration.md#carrier)）。

## PROMPT_READY：渲染区域

ST 发出“提示词就绪”事件后：

1. 消息中没有锚点 → 不处理。有锚点但当前 Carrier 对应的预设未安装 → 删掉锚点，不做其他处理；
2. 把锚点拆成独立的占位消息（锚点与其他文字被 ST 合并时也能分离）；
3. 没有经过拦截器的生成（脚本路径）尝试[回填](meta-and-tasks.md#脚本路径的回填)；
4. `continue` 与 ST 原生 `impersonate` 是**直通**：照常渲染区域，但不选工作流、不接管请求；
5. **选择工作流**（[meta-and-tasks.md](meta-and-tasks.md#工作流选择)）；
6. 计算条件变量与宏数据（记忆、总结、最近元指令、自定义结构的上一次值……）；
7. 合并**用户模组**（[user-modules.md](user-modules.md)）；
8. **组装**（下面详述）；
9. 把每个锚点替换为对应区域的消息（逐条做 ST 宏展开，空消息丢弃）。

### 组装

```
条件变量 vars
  │  stages = {}，计算工作流的生效阶段（阶段 when）
  ▼
vars.stages = 生效阶段
  │
  ▼
生效模组：开关状态（单选集决出）+ 模组 when
  │
  ├─▶ 请求的结构：生效模组 structures 的并集
  ├─▶ 构建变量：按模组顺序应用 vars 更新
  ▼
宏展开器（{{ap::…}}）
  │
  ▼
逐条渲染条目（按 regions 顺序、区域内 order 排序）：
  条目 when → @if 求值 → @style 选择 → 宏展开 → 按 @as 切分消息
  │
  ▼
工作流提示词追加到 workflowRegion 末尾：
  preamble + intro[写法] + 每个生效阶段的分节（指令式加 stageHeading）+ outro[写法]
  │
  ▼
每个区域按 merge 规则合并为消息
```

要点：

- **阶段先于模组**：阶段的 `when` 只能看到 `stages = {}`，所以阶段条件不要引用 `stages.*`；模组与条目的 `when` 可以引用 `stages.*`；
- **模组开关与 `when` 是两回事**：`modules.<id>` 只反映开关（[expressions.md](expressions.md)），模组 `when` 为假时它的条目不渲染、结构不请求、变量更新不应用；
- **工作流提示词作为一条消息**：工作流各部分中没有 `@as` 的块合并成一条消息（以空行连接），放在工作流区域所有条目之后；
- **用户模组条目**排在同区域、同 `order` 的预设条目之后。

`ap preview` 在本地执行同样的组装（除了运行时数据：记忆、总结、聊天文本为空）。

## 预填充整形

传输方式为 `prefill` 且不是直通生成时，把整个消息列表重写为三条：

1. **system**：`historyStart` 区域及其之前的所有消息的文字，以空行连接；
2. **user**：`prefillGuide`（空时 `<START>`）；
3. **assistant**：之后的每条消息按顺序渲染、以空行连接——
   - AP 区域消息与 system 消息：原文；
   - 用户消息：`historyItemTemplate`，`{{prefix}}` 为用户前缀；
   - 助手消息：`historyItemTemplate`，`{{prefix}}` 为 `<名字>: `；
   - 模板变量还有 `{{name}}`、`{{content}}`。

模型从助手消息末尾继续写，所以 `tail` 区域通常以第一个阶段的前缀或标签结尾。

## 接管请求

AP 记录本轮的上下文后，拦截 ST 发往后端的请求（`/api/backends/chat-completions/generate`）：

1. 检查能力（工具传输需要连接支持工具调用）；
2. 由传输方式生成请求体：`json_schema`、工具与 `tool_choice`、停止词、示范轮次等（[transports.md](transports.md)）；Gemini 转换 schema；
3. 来自调用方的工具与 schema 在这里处理（[transports.md](transports.md#调用方的工具与-schema)）；
4. 多工具模式在这里循环多次请求。

ST 收到的仍是一个普通的（流式或非流式）补全响应，只是内容换成了 AP 处理后的正文。

## 解析

流式时每收到一段数据就重新解析累计内容：阶段面板实时显示各阶段，满足[流式门控](transports.md#生成之后)时正文实时写入消息。结束后应用补丁、回退（[transports.md](transports.md#生成之后)）。

## 写入

- 消息正文 `mes`：终结正文；纯轮次为回复的纯文本或 `…`；脚本 / 后台生成为调用方结果的 JSON 或正文（[meta-and-tasks.md](meta-and-tasks.md#轮次类型)）；
- sidecar `extra.advanced_preset`：阶段、结构、事件、诊断信息……（[artifact-and-runtime.md](artifact-and-runtime.md#聊天中保存的数据)）；
- 记忆、总结事件只在拦截器参与且不是 `quiet` 时保存（[structures.md](structures.md#保存条件)）。

## 调试

AP 面板的**调试**页显示最近一次组装的快照：来源、传输方式、工作流、生效模组、构建变量、每个区域的消息与字符数、警告（用户模组插入点缺失、回填失败等）；以及每次请求的原始请求 / 响应与用量。开发服务器连接后，调试页还显示构建状态（[cli.md](cli.md#ap-dev)）。
