# 配置、Carrier 与迁移

**配置**是用户在面板里能调的一切：传输方式、采样参数、角色名字与前缀、模组开关、参数值、省 token……预设通过 `configurationDefaults` 给出默认值，通过 `configurations` 提供若干**推荐配置**（如“Gemini”“Claude”），用户在此基础上做自己的修改。

**Carrier** 是 AP 为每个预设生成的 SillyTavern Chat Completion 预设，负责把 AP 的区域锚点和 ST 原生提示词（角色卡、世界书……）排成顺序。

## 配置层

一个生效的配置由三层深度合并得到：

```
configurationDefaults（清单）  ⊕  推荐配置 / 用户保存的配置的 diff  ⊕  用户当前的 diff
```

- 推荐配置与用户数据都只保存**稀疏 diff**（与下层不同的叶子）。预设更新了默认值，没被用户改过的项会自动跟着变；
- 数组按值整体比较、整体替换；
- 用户可以把当前配置另存为“我的配置”，也可以随时切回某个推荐配置。

## 配置结构

```yaml
configurationDefaults:
  connection:
    transport: structured
    forceTool: true
    parallelTools: false
    sendThoughtSignature: false
    streamToolArguments: auto
    maxSteps: 6
    finalFallback: true
    workflow: story            # 默认：第一个 story 工作流
    syntheticTurn: init        # 默认：第一个示范轮次，没有则 null
  sampling:
    temperature: 1
    openai_max_tokens: 8192
  identity:
    names: { narrator: "" }    # 空字符串 = 使用角色的默认名字
    prefixes: { narrator: "" } # 空字符串 = 使用角色的默认前缀
  modules: { options: true }   # 只能写用户可开关的模组
  params: { length: 600 }
  tokenSaving: { enabled: false, keepFullTurns: 6, truncateUserAfter: 2000 }
  meta: { recentCount: 3 }
```

| 段 | 字段 | 默认 | 说明 |
| --- | --- | --- | --- |
| `connection` | `transport` | `structured` | 传输方式（[transports.md](transports.md)） |
| | `forceTool` | `true` | 工具传输强制调用工具（`tool_choice`） |
| | `parallelTools` | `false` | 多工具模式允许一次调用多个阶段工具 |
| | `sendThoughtSignature` | `false` | Gemini：示范轮次附带 `thoughtSignature` |
| | `streamToolArguments` | `auto` | Gemini 工具参数流式：`auto` / `on` / `off` |
| | `maxSteps` | `6` | 多工具模式最多请求几次 |
| | `finalFallback` | `true` | 解析不出正文时用原始文本作为正文 |
| | `workflow` | 第一个 `story` 工作流 | 故事轮使用的工作流（必须是 `story` 工作流） |
| | `syntheticTurn` | 第一个示范轮次或 `null` | 工具传输插入的示范轮次 |
| `sampling` | 任意键 | `{}` | ST Chat Completion 预设的设置项，见下 |
| `identity` | `names.<actor>` / `prefixes.<actor>` | `''` | 覆盖角色名字 / 前缀；空字符串表示用清单 `actors` 的值 |
| `modules` | `<module id>` | 模组的 `default` | 用户可开关模组的状态 |
| `params` | `<param id>` | 参数的 `default` | 参数值 |
| `tokenSaving` | `enabled` / `keepFullTurns` / `truncateUserAfter` | `false` / 清单 `tokenSaving` 的值 | 省 token（[meta-and-tasks.md](meta-and-tasks.md#省-token)） |
| `meta` | `recentCount` | 清单 `meta.recentCount` | `{{ap::recent_meta}}` 的条数 |

### sampling

`sampling` 是自由的键值表，键使用 **ST Chat Completion 预设文件**中的名字（`temperature`、`top_p`、`openai_max_tokens`、`stream_openai`、`reasoning_effort`、`use_sysprompt`……）。它写入 Carrier，并与 ST 的设置双向同步：用户在 ST 里调温度，AP 配置也跟着变；在 AP 里切换推荐配置，ST 的设置也跟着变。

ST 内部有几项的设置名与预设文件不同，AP 自动换算：`temperature`→`temp_openai`、`frequency_penalty`→`freq_pen_openai`、`presence_penalty`→`pres_pen_openai`、`top_p`→`top_p_openai`、`top_k`→`top_k_openai`、`top_a`→`top_a_openai`、`min_p`→`min_p_openai`、`repetition_penalty`→`repetition_penalty_openai`。请在清单里使用左边的名字。

### 诊断

- `configurationDefaults.modules` 中不存在或不是用户可开关的模组、`params` 中不存在的参数：错误；
- 未知的段（如 `configurationDefaults.foo`）：警告；
- `connection.transport` / `streamToolArguments` 取值非法、`workflow` 不存在或是任务工作流、`syntheticTurn` 不存在：错误（推荐配置的 diff 同样检查）。

## 推荐配置 `configurations`

```yaml
configurations:
  - id: gemini
    name: Gemini
    description: Tool calling with thought signatures.
    diff:
      sampling: { use_sysprompt: false }
      connection: { transport: tool-single, sendThoughtSignature: true }
      modules: { skip-thought: true }
  - id: claude
    name: Claude
    diff:
      connection: { transport: prefill }
defaultConfiguration: gemini
```

| 字段 | 说明 |
| --- | --- |
| `id` | 必填，唯一 |
| `name` | 面板显示名（默认 `id`），可在 `locales` 翻译 |
| `description` | 面板说明 |
| `diff` | 相对 `configurationDefaults` 的稀疏 diff |

- 没有写 `configurations` 时，自动生成 `{ id: default, name: Default, diff: {} }`；
- `defaultConfiguration` 默认是第一个推荐配置，必须存在（错误）；新用户与找不到原推荐配置的用户使用它；
- diff 中不存在于配置结构的键：错误；id 重复：错误；
- 某个推荐配置让同一单选集启用了多个模组：警告。

`ap preview --config <id>` 用指定推荐配置预览组装结果（[cli.md](cli.md)）。

## Carrier

AP 为每个已安装的预设维护一个 ST Chat Completion 预设，名为 `AP · <预设 name>`（按内部元数据 `extensions.advanced_preset.presetId` 查找，不依赖名称）。**用户在 ST 预设下拉框里选中某个 Carrier，就等于启用了对应的 AP 预设。** 选中 Carrier 时，ST 的导出按钮导出 AP 预设产物而非 Carrier，重命名与另存为按钮隐藏；ST 的导入按钮遇到 AP 产物时按 AP 预设安装。

```yaml
carrier:
  label: 🌸 Petals             # 锚点提示词名字的前缀，默认预设 name
  order:
    - top                      # 区域 id → 该区域的锚点
    - role
    - main: false              # ST 原生提示词；{ id: false } 表示放入但默认关闭
    - worldInfoBefore
    - charDescription
    - scenario
    - worldInfoAfter
    - dialogueExamples
    - history
    - chatHistory              # 必须有
    - task
    - bottom
  prompts:                     # 可选：覆盖提示词字段，或声明额外的提示词
    jailbreak: { content: "" }
```

`order` 的每一项是：

- **区域 id**（非原生区域）：生成锚点提示词——`identifier: ap_<区域 id>`、`name: <label> · <区域 id>`、`role` 为区域的角色、内容为锚点 `⟦ap:region:<区域 id>⟧`、`forbid_overrides: true`；
- **ST 原生提示词 id**：`main` `worldInfoBefore` `personaDescription` `charDescription` `charPersonality` `scenario` `enhanceDefinitions` `nsfw` `worldInfoAfter` `dialogueExamples` `chatHistory` `jailbreak`；
- **`carrier.prompts` 中声明的 id**：额外的自定义提示词（system 角色，内容由 `prompts.<id>` 提供）；
- 写成 `{ <id>: false }` 时放入顺序但默认关闭。

`prompts.<id>` 中的字段会覆盖生成的提示词对象（ST 预设的 prompt 字段，如 `content`、`role`、`name`）。

约束（错误）：每个非原生区域都必须出现在 `order` 中；必须包含 `chatHistory`；不能重复；不认识的 id 报错。原生区域（`native: true`）只在区域顺序中标记 ST 内容的位置，不写入 `order`。

**运行期**：

- 组装时 ST 按 Carrier 的顺序拼出提示词，AP 在每个锚点处换入区域内容（[assembly.md](assembly.md)）。锚点永远不会原样发给模型；
- 安装或更新预设后，如果产物哈希变化，Carrier 被**重建**。用户手动改过 Carrier 的提示词时，重建会覆盖并提示；`chat_completion_source`、`reverse_proxy`、`proxy_password` 这些与连接绑定的字段会保留；
- 所以要改提示词内容，请改预设源码，而不是改 Carrier。

## 迁移

字段 `migrations`。模组、参数、插入点、构建变量的 id 是用户数据的键。发布后改名会让用户的设置“失联”，这时用迁移告诉 AP 新旧 id 的对应关系：

```yaml
migrations:
  - from: "<7.0.0"
    rename:
      modules: { old-pacing: pacing }
      params: { wordCount: length }
      insertionPoints: { before-task: task-before }
      vars: { questionList: questions }
```

| 字段 | 说明 |
| --- | --- |
| `from` | 用户之前安装的版本满足这个范围时应用：`<7.0.0`、`>=6 <7`、`6.x`、`=6.2.0` 等。不是合法范围时报错 |
| `rename.modules` | 配置 diff 中的模组开关、用户模组中引用的模组 |
| `rename.params` | 配置 diff 中的参数值、用户模组中引用的参数 |
| `rename.insertionPoints` | 用户模组条目的插入点 |
| `rename.vars` | 用户模组对构建变量的更新 |

**应用时机**：预设升级后（已安装版本低于新版本）第一次加载时，按顺序应用所有 `from` 匹配的迁移——

1. 对用户的当前配置与所有保存的配置做重命名；
2. 对用户模组做重命名（插入点、变量，以及表达式中引用的 `modules.<id>` / `params.<id>`）；
3. 然后**删除新版本中不存在的键**（配置结构中没有的键、已不存在的模组与参数）；
4. 如果启用的用户模组仍引用不存在的插入点，提示用户。

迁移也用于导入旧版本导出的用户模组文件。重命名目标在新版本中不存在时给出警告；未知的 `rename` 段给出警告。
