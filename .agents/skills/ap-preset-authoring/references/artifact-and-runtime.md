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
| `params` `groups` `modules` `entries` `insertionPoints` `vars` | 参数、分组、模组、已排序的条目（条件已编译为 AST）、插入点、构建变量 |
| `workflows` `structures` `regex` `meta` `tokenSaving` | 工作流、结构、正则标记、元指令、省 token |
| `syntheticTurns` `tools` | 工具传输的文案 |
| `theme` | 主题色 |
| `carrier` | 生成好的 ST 提示词列表与顺序 |
| `configurationDefaults` `configurations` `defaultConfiguration` `migrations` | 配置与迁移 |
| `moduleOrder` `locales` | 面板顺序与翻译 |

产物中的文本是**未展开**的（宏在运行期展开）。除调试外不要手改产物；改源码后重新构建。

## 安装与更新

AP 面板 → **预设**：

| 方式 | 说明 |
| --- | --- |
| 导入文件 | 选择 `.apreset.json` |
| 导入链接 | 填写产物 URL。GitHub 页面链接（`github.com/<owner>/<repo>/blob/<ref>/<path>`）自动换成 `raw.githubusercontent.com`。AP 记住来源，之后可点“从来源更新”重新下载 |
| 开发服务器 | 调试页填写 `ap dev` 的地址，见 [cli.md](cli.md#ap-dev) |
| 内置 | 扩展自带的预设（目前是花瓣），随扩展更新 |

- 产物声明了 `minAdvancedPreset` 而扩展版本更低时，拒绝安装并提示升级扩展；
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
  "events": [ { "type": "memory.replace", "key": "…", "content": "…", "source": "model" } ],
  "metaInput": ["…"],             // 本轮输入中的元指令片段
  "reasoning": "…",               // 模型的推理内容（如果有）
  "diagnostics": { "warnings": [], "requests": 1, "usage": { }, "durationMs": 8123 }
}
```

- 阶段 `status`：`done`（完成）、`missing`（必需阶段没有输出）、`interrupted`（生成被中断）；
- 消息正文 `mes` 只包含终结正文（或纯轮次的回复文字），阶段与结构不写进正文，所以切换预设或关闭 AP 后聊天记录仍然干净；
- 用户在 UI 中切换原稿 / 修订稿时，`mes` 与 `patchesApplied` 随之改变。

### 事件

`events` 记录本轮对聊天状态的修改：

| 事件 | 字段 | 说明 |
| --- | --- | --- |
| `memory.replace` | `key` `content` `source` | 写入聊天记忆 |
| `memory.delete` | `key` `source` | 删除聊天记忆 |
| `recap.set` | `text` `coversUpTo` `source` | 设置总结，覆盖到消息 `coversUpTo` |
| `global.ref` | `index` | 本轮产生的全局记忆日志条目 |

`source` 为 `model`（模型输出）或 `user`（用户在面板中编辑）。当前状态 = 按消息顺序折叠所有事件，所以删除消息、切换 swipe 会自然回滚。用户在聊天还没有 AP 消息时所做的编辑，保存在聊天元数据 `chat_metadata.advanced_preset.seedEvents` 中，排在所有消息之前。

全局记忆按预设保存在扩展设置中，是只追加的日志（每条记录修改前的值，撤销即追加补偿记录）。

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

脚本调用 ST 生成（如 `generateRaw` / `generateQuietPrompt`）时，如果当前选中的是 Carrier，AP 同样会组装区域并接管请求，来源为 `task`；调用方的工具与 JSON Schema 的处理见 [transports.md](transports.md#调用方的工具与-schema)。

## 主题

清单的 `theme` 为面板与卡片提供配色（[manifest.md](manifest.md#主题-theme)）。用户可在“外观”中覆盖。
