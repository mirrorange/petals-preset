# 产物、安装与运行期数据

## 产物

`ap build` 输出 `dist/<id>.apreset.json`：一个完整、自包含的 JSON。扩展只读取产物，不读取源码。

- **格式版本** `formatVersion: 3`。扩展只接受当前格式：已安装的旧格式副本会被忽略（如果扩展内置了同 id 的预设，则改用内置版本），请用新版 CLI 重新构建；
- **哈希** `hash`：去掉 `hash` 字段后、键排序的规范化 JSON 的 SHA-256 前 16 位十六进制。扩展用它判断产物是否变化（重建 Carrier、迁移用户配置、开发服务器热更新）；
- 后续版本中不改格式版本新增的字段，扩展读取旧产物时会补上默认值。

顶层字段（按出现顺序）：

| 字段 | 内容 |
| --- | --- |
| `formatVersion` `id` `name` `version` `author` `description` `minAdvancedPreset` `hash` | 身份与元数据 |
| `actors` `narrator` `user` | 角色 |
| `styles` `plaintextFormat` `contentMarker` `prefillGuide` `historyItemTemplate` `stageHeading` `stop` | 写法与输出格式 |
| `regions` `anchors` `workflowRegion` | 区域 |
| `params` `groups` `modules` `entries` `insertionPoints` `userModules` `userModuleTemplates` `vars` | 参数、分组、模组、已排序的条目（条件已编译为 AST）、插入点、随预设安装的用户模组与用户模组模板（条目文字为源码原文，翻译在各项的 `locales` 中，见 [user-modules.md](user-modules.md#预设中的用户模组)）、构建变量 |
| `workflows` `structures` `regex` `meta` `tokenSaving` | 工作流、结构、正则标记、元指令、省 token |
| `syntheticTurns` `tools` | 工具传输的文案 |
| `theme` | 主题色 |
| `carrier` | 生成好的 ST 提示词列表与顺序 |
| `configurationDefaults` `configurations` `defaultConfiguration` `migrations` | 配置与迁移 |
| `moduleOrder` `locales` | 面板顺序与翻译 |

产物中的文本是**未展开**的（宏在运行期展开）。`@template` 片段以模板源码保存，带 `template: { file, line }` 标记（来源位置，用于报错），运行期渲染。除调试外不要手改产物；改源码后重新构建。

## 安装与更新

AP 面板 → **预设**：

| 方式 | 说明 |
| --- | --- |
| 导入文件 | 选择 `.apreset.json` |
| 导入链接 | 填写产物 URL。GitHub 页面链接（`github.com/<owner>/<repo>/blob/<ref>/<path>`）自动换成 `raw.githubusercontent.com`。AP 记住来源，之后可点“从来源更新”重新下载 |
| 开发服务器 | 调试页填写 `ap dev` 的地址，见 [cli.md](cli.md#ap-dev) |
| 内置 | 扩展自带的预设（目前是花瓣），随扩展更新 |

- 产物声明了 `minAdvancedPreset` 而扩展版本更低时，拒绝安装并提示升级扩展；
- 产物包含[模板](templates.md)时，安装前显示全部模板代码并请用户确认信任；更新时模板代码与已安装版本相同则不再询问。取消则不安装。内置与开发服务器的预设不询问；
- 查找顺序：开发服务器 → 已安装 → 内置。已安装副本版本低于内置版本时使用内置版本；
- 安装 / 更新后：按[迁移](configuration.md#迁移)处理用户配置与用户模组，然后确保 Carrier 存在且与产物一致；
- 删除已安装的预设会同时删除它的 Carrier（内置预设则回退到内置版本）。

**发布建议**：把 `dist/<id>.apreset.json` 提交到仓库（或作为 Release 附件），给用户 GitHub 上的文件链接。注意 GitHub Release 附件的下载链接不支持跨域访问，浏览器中的 AP 无法直接从它导入；请使用仓库文件链接（`blob/` 或 `raw.githubusercontent.com`）。

## 聊天中保存的数据

### 消息 sidecar

AP 接管的每条助手消息在 `message.extra.advanced_preset` 中保存一份 sidecar（每个 swipe 各一份，存于 `swipe_info[i].extra`）：

```jsonc
{
  "v": 1,
  "presetId": "petals",
  "presetVersion": "7.6.0",
  "genId": "…",
  "origin": "story",              // story | task
  "transport": "tool-single",
  "workflow": "story",
  "turn": "story",                // story | meta-mixed | meta-pure | summary | impersonate
  "stages": [
    { "id": "think", "actor": "narrator", "kind": "text", "label": "Thinking", "text": "…", "status": "done" },
    { "id": "review", "actor": "editor", "kind": "patch", "label": "Review", "patches": [ { "old_string": "…", "new_string": "…", "status": "applied" } ], "status": "done" }
  ],
  "draft": "…",                   // 有补丁时：修订前的正文
  "patched": "…",                 // 有补丁时：修订后的正文
  "patchesApplied": true,         // 当前显示的是否为修订稿
  "structures": { "options": [ { "label": "…", "content": "…" } ] },
  "annotations": [                // 行内批注（有 annotations 结构时）
    { "id": "a1", "start": 12, "end": 17, "quote": "轻轻推开门", "prefix": "…", "suffix": "…", "comment": "…" }
  ],
  "annotationVersions": { "draft": [ ], "patched": [ ] }, // 有补丁时：两份正文各自的批注
  "events": [ { "type": "memory.replace", "key": "…", "content": "…", "source": "model" } ],
  "metaInput": ["…"],             // 本轮输入中的元指令片段
  "reasoning": "…",               // 模型的推理内容（如果有）
  "diagnostics": { "warnings": [], "requests": 1, "usage": { }, "durationMs": 8123 }
}
```

- 阶段 `status`：`done`（完成）、`missing`（必需阶段没有输出）、`interrupted`（生成被中断）；
- 消息正文 `mes` 只包含终结正文（或纯轮次的回复文字），阶段与结构不写进正文，所以切换预设或关闭 AP 后聊天记录仍然干净；
- 用户在 UI 中切换原稿 / 修订稿时，`mes`、`patchesApplied` 与 `annotations` 随之改变。
- 用户在事件摘要卡片上编辑后，`structures` 中对应的值被直接覆盖（只改当前 swipe，`mes` 不变）。

### 行内批注

`annotations` 中每条批注的 `start` / `end` 是干净正文 `mes` 中的 UTF-16 偏移，`quote` 是被批注的原文，`prefix` / `suffix` 是前后各约 32 字的上下文，`comment` 是批注文字。用户编辑消息后 AP 用 `quote` 与上下文重新定位并更新偏移；找不到时标记 `"orphan": true`，保留但不显示。

**显示面**：AP 在每个显示面中分别定位批注——ST 的 `.mes_text`，以及其中每个**同源** iframe（例如酒馆助手渲染的前端界面）的文档。匹配按可见文字进行（跳过隐藏元素、`script` / `style`，忽略 Markdown 强调符号、零宽字符与引号样式差异），每个显示面里一条批注至多标记一处；有多处时依次用上下文、出现序号消歧。划线使用各文档自己的 `CSS.highlights`（名为 `ap-anno`），不修改前端的 DOM；气泡统一显示在 ST 主页面。跨源 iframe 无法适配。

**前端控制**：在 iframe 的 `<html>` 或 `<body>` 上设置

| 属性 | 效果 |
| --- | --- |
| `data-ap-annotations="off"` | 不在此 iframe 中显示批注 |
| `data-ap-annotations="manual"` | 不自动标记，前端通过 `window.parent.AdvancedPreset.annotations` 自行绘制 |

**TauriTavern**：在受管 ChatSurface 模式下（`isManagedOwnershipRequired()` 为真），AP 通过清单 hook `chatSurface` 注册 participant `advanced-preset/annotations`，在 `didCommitContent` 中挂载批注并返回 disposer；其他模式使用常规 DOM 监听。

### 事件

`events` 记录本轮对聊天状态的修改：

| 事件 | 字段 | 说明 |
| --- | --- | --- |
| `memory.replace` | `key` `content` `source` | 写入聊天记忆 |
| `memory.delete` | `key` `source` | 删除聊天记忆 |
| `recap.set` | `text` `coversUpTo` `source` | 设置总结，覆盖到消息 `coversUpTo` |
| `global.ref` | `id` | 本轮产生的全局记忆日志条目（旧版本写入的引用为日志下标 `index`） |

`source` 为 `model`（模型输出）或 `user`（用户在面板中编辑）。当前状态 = 按消息顺序折叠所有事件，所以删除消息、重新生成、切换 swipe 会自然回滚。滑动生成新 swipe 时，正在重新生成的楼层不再带有上一个 swipe 的 sidecar（上一个 swipe 在 `swipe_info` 中保留自己的副本），因此这一轮写入的聊天记忆在新请求中已经回滚。用户在聊天还没有 AP 消息时所做的编辑，保存在聊天元数据 `chat_metadata.advanced_preset.seedEvents` 中，排在所有消息之前。

全局记忆按预设保存在扩展设置中，是只追加的日志（每条记录有稳定的 `id` 与修改前的值，撤销即追加补偿记录；撤销后再做新的修改会丢弃被撤销的记录）。模型写入的条目在消息到达时才写入日志，并跟随写入它的消息版本：

- 楼层当前选中的版本引用的条目生效；只被其他 swipe 引用的条目失效（`inactive`），折叠时跳过；
- 删除消息、重新生成、滑动生成新 swipe、滑动到其他版本时，原版本的条目失效；滑回该版本时重新生效；
- 引用的条目已不在日志中（撤销后又做了新修改）时忽略；
- 切换聊天不会使任何条目失效，只会按载入聊天中各楼层选中的版本更新它们引用的条目；用户在面板中的编辑不跟随任何消息。

## 返回给 ST 的响应

AP 接管的生成由自己向后端发出一次或多次请求，再按当前来源的格式（OpenAI / Claude / Gemini / Cohere，流式为 SSE，否则为 JSON）合成一份响应交给 ST，其中只有终结正文、推理内容与（任务轮）调用方的工具调用。后端返回的用量（token 数、缓存命中等）原样附在合成响应中：OpenAI 系（含 DeepSeek）为 `[DONE]` 前一个 `choices` 为空、带 `usage` 的块，Claude 写入 `message_delta.usage`（合并 `message_start` 中的输入与缓存 token），Gemini 为最后一个带 `usageMetadata` 的块；非流式响应在顶层带 `usage`（Gemini 为 `usageMetadata`）。多步工具等一轮多次请求时，各数值字段相加。因此通过包装 `fetch` 读取生成响应用量的扩展（费用、缓存命中统计等）无需适配 AP；后端没有返回用量时不附带。

## 缓存状态

缓存优化（[meta-and-tasks.md](meta-and-tasks.md#缓存优化粘性渲染)）的状态不写入聊天，而是保存在浏览器 `localStorage` 的 `advanced-preset.cache.v1` 中，按聊天分开：上一次成功请求的时间、来源与模型、工具指纹、历史前提示词的哈希、历史各消息文本的哈希序列、每条消息的渲染方式（按发送时间、发送方与名字识别），以及连续推迟的请求数。超过 24 小时的记录与最旧的记录（最多保留 30 个聊天）会被清除；清除或丢失只会让下一次请求刷新。

## 全局 API

扩展在页面上暴露 `window.AdvancedPreset`，供脚本与其他扩展使用：

| 成员 | 说明 |
| --- | --- |
| `version` | 扩展版本 |
| `buildId` | 当前运行的构建 id |
| `nextGeneration({ origin })` | 强制下一次生成的来源（`'story'` / `'task'`），传 `null` 取消（[meta-and-tasks.md](meta-and-tasks.md#生成来源)） |
| `listPresets()` | 可用预设：`[{ id, name, version, source }]`，`source` 为 `builtin` / `installed` / `dev` |
| `activePresetId()` | 当前选中的 Carrier 对应的预设 id，未选中时为 `null` |
| `update()` | 检查并应用扩展更新，返回 `'updated'` / `'current'` |
| `togglePanel(open?)` | 打开 / 关闭 AP 面板（不传参数时切换） |
| `annotations.get(mesId)` | 消息当前可见的行内批注（不含孤立批注），偏移对应当前 `mes` |
| `annotations.on('change', (mesId) => …)` | 订阅批注变化（生成、编辑、切换 swipe 或补丁），返回取消订阅函数 |

脚本调用 ST 生成（如 `generateRaw` / `generateQuietPrompt`）时，如果当前选中的是 Carrier，AP 同样会组装区域并接管请求，来源为 `task`；调用方的工具与 JSON Schema 的处理见 [transports.md](transports.md#调用方的工具与-schema)。

## 主题

清单的 `theme` 为面板与卡片提供配色（[manifest.md](manifest.md#主题-theme)）。用户可在“外观”中覆盖。
