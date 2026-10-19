# 元指令、生成来源与历史改写

本文说明 AP 在每次生成开始时如何**分类**这一轮（元指令、正则标记、内置任务、生成来源），如何**选择工作流**，以及发送前如何**改写聊天历史**（剔除元指令、省 token）。清单字段见 [manifest.md](manifest.md) 的 `meta`、`regex`、`tokenSaving` 部分。

## 元指令

元指令是用户写给叙述者本人的话，以触发词（`meta.trigger`，默认 `@{{ap::name::<narrator>}}`，如 `@Freesia`）开头。

**识别规则**：

- 触发词先做 `{{ap::name::…}}` 与 ST 宏展开，然后**大小写不敏感**地查找；
- 触发词后紧跟字母、数字或下划线时不算（`@Freesiax` 不是触发）；
- **段落写法**：从触发词到下一个空行（`\n\s*\n`）或输入结尾。开头的空白、`,` `，` `:` `：` 去掉；
- **花括号写法**（`meta.wrap: true` 时）：`@Freesia{ … }`，花括号可嵌套，可跨段落；
- 一条输入里可以有多段元指令。

去掉所有元指令片段后：

| 剩余内容 | 分类 | 条件变量 |
| --- | --- | --- |
| 没有元指令 | 普通输入 | `meta.present == false` |
| 空 | **纯元指令** | `meta.present && meta.pure` |
| 非空 | **混合** | `meta.present && meta.mixed` |

元指令文字可用 `{{ap::input::meta}}`（各段以空行连接）读取；`{{ap::recent_meta}}` 输出最近 `meta.recentCount` 次元指令及叙述者的回复（见 [macros.md](macros.md)），条件变量 `meta.recent` 表示它非空。

## 正则标记

`regex` 中的每条规则在分类时求值，结果写入 `flags.<id>`：

- `target: input`（整条输入）、`meta`（元指令片段，以空行连接）或 `recent:<N>`（最近 N 条消息，最多取到最近 6 条）；
- 匹配并且 `requires` 条件成立时为真。`requires` 求值时只能使用 `meta.*` 与 `flags.*`（前面规则的结果）。

## 内置任务

**只在纯元指令时**检查 `meta.tasks`，先 `summary` 后 `impersonate`，第一个成立的条件决定 `meta.task`：

```yaml
regex:
  - { id: summary, pattern: "总结|summar", target: meta }
  - { id: impersonate, pattern: "代写|write my", target: meta }
meta:
  tasks: { summary: flags.summary, impersonate: flags.impersonate }
```

任务本身不改变流程，只是给模组一个条件（例如 `when: 'meta.task == "summary"'` 的模组请求 `recap` 结构）。点击 ST 的代写按钮时，AP 把 `meta.impersonatePrompt` 填入输入框并发送（输入框里已有的草稿换行附在后面），所以代写也是一次纯元指令。

## 生成来源

每次生成有一个**来源** `gen.origin`：

- `story`：拦截器参与（ST 原生 `Generate`）、生成类型为 `normal` `regenerate` `swipe` `continue` 之一，并且**不是**纯元指令；
- `task`：其他所有情况——纯元指令、后台生成（`quiet`）、ST 原生代写（`impersonate`）、脚本 / 其他扩展调用（`script`）。

脚本可以强制下一次生成的来源：

```js
AdvancedPreset.nextGeneration({ origin: 'story' }); // 只影响下一次生成
```

### 工作流选择

- `story`：配置 `connection.workflow` 指定的工作流；找不到时用第一个 `kind: story` 工作流；
- `task`：第一个 `kind: task` 工作流（或 id 为 `task` 的工作流）。

**继续**（`continue`）与 ST 原生**代写**（`impersonate`）是“直通”生成：区域照常组装，但不运行工作流、不接管请求，由 ST 原样发送。

## 轮次类型

生成结束后，sidecar 的 `turn` 记录这一轮的类型：

| `turn` | 条件 |
| --- | --- |
| `story` | 故事轮（含纯元指令但模型仍写了正文的情况） |
| `meta-mixed` | 混合元指令的故事轮 |
| `meta-pure` | 任务轮、没有正文、不是总结 / 代写 |
| `summary` | 任务轮、没有正文、`meta.task == "summary"` |
| `impersonate` | 任务轮、没有正文、`meta.task == "impersonate"` |

后三种是**纯轮次**：消息正文 `mes` 为回复的纯文本（没有回复时，代写 / 总结轮为 `…`）。

## 历史改写

拦截器在 ST 组装提示词之前改写**历史副本**（真实聊天不变）：

1. **剔除纯轮次**：`turn` 为 `meta-pure` / `summary` / `impersonate` 的助手消息，连同触发它的用户消息一起不发送（当前输入除外）；
2. **剥离元指令**：较早用户消息中的元指令片段被去掉；剥离后为空的用户消息不发送；当前输入保持原样；
3. **省 token**（启用时，见下）。

## 省 token

配置 `tokenSaving.enabled` 为真时（默认关闭），从末尾开始数助手消息：

- 最近 `keepFullTurns` 条助手消息以及它们前面的用户消息保持完整（用户消息与其后的助手回复算同一轮）；当前输入总是保留；
- 更早的消息中：
  - 已被总结覆盖的（真实索引 ≤ 当前总结的 `coversUpTo`）**不发送**，由模组通过 `{{ap::recap}}` 提供总结；
  - 用户消息超过 `truncateUserAfter` 个字符时截断并加 `…`（0 表示不截断）；
  - 助手消息如果有事件摘要（`synopsis` 结构），替换为 `tokenSaving.synopsisTemplate` 渲染的一行（占位符 `{{time}}` `{{location}}` `{{event}}` `{{summary}}`）；没有则保持原样。

`keepFullTurns`、`truncateUserAfter` 的默认值来自清单 `tokenSaving`，可被配置覆盖。条件变量 `features.tokenSaving` 表示已开启，`recap.present` 表示有生效的总结。

## 脚本路径的回填

脚本 / 其他扩展直接调用生成时拦截器不会运行。AP 在提示词就绪后尝试**回填**：取 `historyStart` 与 `historyEnd` 锚点之间的用户 / 助手消息，与真实聊天从末尾对齐（忽略宏与空白；相等、包含或 Dice 系数 ≥ 0.9 视为匹配，允许跳过少量插入的消息），然后对匹配到的消息执行同样的历史改写。

匹配率低于 90% 或缺少语义锚点时，历史保持原样，并在调试信息中给出警告（`backfill: …`）。脚本发起的任务生成不请求结构（见 [structures.md](structures.md#何时请求结构)）。
