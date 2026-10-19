# 条件表达式与变量

条件表达式用在：模组与条目的 `when`、阶段的 `when`、`@if`、正则标记的 `requires`、`meta.tasks`、用户模组的“表达式”触发规则。表达式在构建时编译成 AST，运行时求值（不使用 `eval`）。

## 语法

```
modules.pacing && params.length > 0
transport in ["structured", "tool-single", "tool-multi"]
!meta.pure && (workflow == "edit" || stages.review)
"小说" in params.genre
```

| 元素 | 写法 |
| --- | --- |
| 字面量 | 字符串 `"…"` 或 `'…'`（支持 `\` 转义），数字，`true` `false` `null`，数组 `[a, b]` |
| 变量 | 点分路径，段内可含 `-`：`modules.mode-preface`、`params.language` |
| 比较 | `==` `!=` `<` `<=` `>` `>=` |
| 包含 | `in` |
| 逻辑 | `&&` `||` `!` |
| 分组 | `( … )` |

**优先级**（从低到高）：`||` → `&&` → `==` `!=` → `<` `<=` `>` `>=` `in` → `!`（一元，只作用于紧随其后的单个操作数）。因此 `!a == b` 是 `(!a) == b`，需要否定比较时写 `!(a == b)` 或 `a != b`。

**求值规则**：

- `==` / `!=` 是宽松相等：数字与字符串按文本比较（`params.n == "3"` 与 `params.n == 3` 等价）；`null` 与“不存在”相等。
- `<` `<=` `>` `>=` 把两边转成数字（不存在当作 0）。
- `a in [x, y]`：`a` 宽松等于数组中任一项；`a in "text"`：`a` 是字符串且是 `"text"` 的子串；其他情况为假。
- `&&` / `||` 结果是布尔值；单独一个变量按真假判断（空字符串、0、`false`、`null`、不存在为假）。
- 未写条件等于恒真。

## 条件变量

| 变量 | 取值 | 说明 |
| --- | --- | --- |
| `transport` | `prefill` `plaintext` `structured` `tool-single` `tool-multi` | 当前传输方式 |
| `style` | `dialogue` `imperative` | 当前写法 |
| `workflow` | 工作流 id | 本轮工作流（继续 / 代写时为空） |
| `gen.origin` | `story` `task` | 生成来源（见 [meta-and-tasks.md](meta-and-tasks.md#生成来源)） |
| `gen.type` | `normal` `continue` `swipe` `regenerate` `quiet` `impersonate` `script` | ST 的生成类型；脚本 / 其他扩展调用为 `script` |
| `gen.callerTools` / `gen.callerSchema` | 布尔 | 预留（组装时总为 `false`） |
| `meta.present` | 布尔 | 本次输入含元指令 |
| `meta.pure` | 布尔 | 输入只有元指令 |
| `meta.mixed` | 布尔 | 元指令与故事输入混合 |
| `meta.task` | `summary` `impersonate` 或 `null` | 识别出的内置任务 |
| `meta.recent` | 布尔 | 最近有元指令记录（`{{ap::recent_meta}}` 非空） |
| `modules.<id>` | 布尔 | 模组开关状态（单选集决出之后，不受模组 `when` 影响） |
| `params.<id>` | 字符串或数字 | 参数原始值 |
| `flags.<id>` | 布尔 | 正则标记结果 |
| `stages.<id>` | 布尔 | 本轮工作流是否运行该阶段（阶段 `when` 求值之后） |
| `features.tokenSaving` | 布尔 | 省 token 已开启 |
| `chat.group` | 布尔 | 群聊 |
| `memory.hasChat` / `memory.hasGlobal` | 布尔 | 聊天记忆 / 全局记忆非空 |
| `recap.present` | 布尔 | 当前有生效的总结 |

注意：

- 阶段的 `when` 在 `stages.*` 计算之前求值，所以阶段条件里不要引用 `stages.*`。
- 正则标记的 `requires` 与 `meta.tasks` 在组装之前求值，只能可靠地使用 `meta.*` 与 `flags.*`。
- `ap preview` 中 `memory.*`、`recap.present`、`chat.group`、`meta.recent` 总为假。

## 诊断

`ap check` 检查表达式中的每个变量：

- 未知根变量（如 `foo.bar`）报错；
- `modules.` / `params.` / `flags.` / `stages.` 后的 id 不存在报错；
- 固定取值的变量与字符串比较时，字面量不在取值范围内报错，例如 `transport == "tool"`、`meta.task == "recap"`、`workflow == "不存在的工作流"`。
