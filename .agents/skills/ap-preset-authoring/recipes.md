# 常见任务

每个任务结束都运行 `npx ap check`。示例中的英文提示词只是占位，按预设的语言和语气写。

## 加一个开关模组

1. 新建 `modules/<分类>/<id>.md`：`id`、`name`、`description`、`group`（已存在于 `preset.yaml` 的 `groups`）、`default`、`region`、`when`。
2. 正文写规则本身；需要放到其他区域的内容用 `@entry`。
3. 要在规划阶段被考虑时，往构建变量里加一项：`vars: { considerations: { <key>: <问题> } }`。
4. 有 `locales/` 时加翻译：`modules: { <id>: { name: …, description: … } }`。
5. 需要某个推荐配置默认打开时，在该配置的 `diff.modules` 里写 `<id>: true`。

## 加一组单选（互斥）模组

1. `preset.yaml` 的 `groups` 中加分组：`style: { label: Style, allowNone: true }`（不允许都不选就省略 `allowNone`）。
2. 每个成员模组写 `group: style` 与 `exclusive: style`；其中一个写 `default: true`。
3. 想控制面板顺序和“第一个成员”，在 `moduleOrder` 中排好它们。

## 加一个参数

1. `preset.yaml` 的 `params` 中加：

   ```yaml
   - id: length
     type: number
     label: Target length
     group: language        # 放在 tab: writing 的分组里
     unit: words
     min: 0
     default: 0
   ```

2. 在用到的模组里：

   ```markdown
   <!-- @if params.length > 0 -->
   Aim for about {{ap::param::length}} words.
   <!-- @end -->
   ```

3. 选择型参数用 `type: select` + `options`（`{ value, label, text }`，`text` 是给模型看的文字）。
4. 加翻译：`params: { length: { label: 目标字数 } }`。

## 按传输方式 / 写法区分内容

- 差异小：同一条目内用 `@style dialogue … @style imperative … @end`。
- 差异大：拆成两个条目，`when='style == "dialogue"'` / `when='style == "imperative"'`。
- 只针对某种传输方式：`when='transport == "prefill"'`，或 `when='transport in ["tool-single", "tool-multi"]'`。

## 给工作流加一个阶段

1. 在 frontmatter 的 `stages` 中按顺序插入：`{ id: review, actor: narrator, kind: text, label: Review }`。
2. 为每种支持的写法写 `<!-- @section stage:review style=dialogue -->` 与 `style=imperative` 的说明。
3. 需要时设 `when`（如 `when: modules.self-review`），用一个开关模组控制这个阶段。
4. 想定制字段名 / 工具名：`field: { name: review_story, description: … }`、`tool: { name: review_story, description: … }`。
5. 加翻译：`workflows: { <wf>: { stages: { review: 审阅 } } }`。
6. 预览 structured 与 tool-multi，确认 schema 和工具顺序。

## 新增一个工作流

1. 新建 `workflows/<id>.md`，`stages` 至少包含一个 `content` 阶段。
2. 写 `intro` / 每个阶段 / `outro` 分节（两种写法）。
3. 需要固定顺序时在 `preset.yaml` 写 `workflowOrder`。
4. 想让某个推荐配置使用它：`diff: { connection: { workflow: <id> } }`。

## 加一个结构（以状态栏为例）

1. `preset.yaml` 的 `structures` 中加 `kind: custom` 的结构（`shape: kv` 或文本、`display: card` / `hidden`、`tag`、`field`、`formatPrompt.tag` / `formatPrompt.field`），见 [structures.md](references/structures.md)。
2. 新建一个模组（通常是开关）：`structures: [status]`，正文放 `{{ap::structure::status}}`，需要延续时加 `Previous status: {{ap::structure::status::last}}`。
3. 预览：对话式检查标签格式说明，指令式检查 schema 中出现 `status` 字段。

## 打开元指令 / 总结 / 代写

参考 [meta-and-tasks.md](references/meta-and-tasks.md) 与 [structures.md](references/structures.md)：

- 元指令：`meta.trigger`、一个 `when: meta.present` 的任务模组（列出 `reply` 结构）、一个 `kind: task` 的工作流；
- 总结：`regex` 中加关键词标记（`target: meta`、`requires: meta.pure`），`meta.tasks.summary: flags.<id>`，一个 `when: 'meta.task == "summary"'` 的模组列出 `recap` 结构；
- 代写：`regex` 中加关键词标记（如“代写 / write my”），`meta.tasks.impersonate: flags.<id>`，`meta.impersonatePrompt` 写一句会命中该标记的话（ST 代写按钮会把它作为纯元指令发送），再用一个 `when: 'meta.task == "impersonate"'` 的模组列出 `impersonate` 结构。

## 让用户能扩展清单（构建变量）

1. `preset.yaml`：`vars: { considerations: { type: dict, label: …, description: … } }`。
2. 各模组：`vars: { considerations: { <key>: <text> } }`。
3. 工作流里读取：`{{ap::var::considerations::values}}`。
4. 用户在“我的模组”中可以新增或删除（`null`）键。

## 加一个推荐配置

```yaml
configurations:
  - id: gemini
    name: Gemini
    description: Tool calling with thought signatures.
    diff:
      sampling: { use_sysprompt: false }
      connection: { transport: tool-single, sendThoughtSignature: true }
      modules: { skip-thought: true }
```

`sampling` 使用 ST Chat Completion 预设文件里的键名（`temperature`、`openai_max_tokens`、`reasoning_effort`……）。加翻译 `configurations: { gemini: { name: … } }`。用 `npx ap preview --config gemini` 检查。

## 加一个插入点

```yaml
insertionPoints:
  - { id: after-role, label: After the role, description: Right after the character setup., region: role, order: 1000 }
```

id 用 kebab-case，发布后不改；加翻译 `insertionPoints: { after-role: { label: …, description: … } }`。

## 改名（已发布的预设）

1. 改 id，同时更新所有引用（`modules.<old>` 条件、配置 diff、`moduleOrder`、翻译）。
2. 写迁移并升主版本：

   ```yaml
   migrations:
     - from: "<2.0.0"
       rename:
         modules: { old-id: new-id }
   ```

3. `npx ap check` 会报告残留的旧引用。

## 修复 `ap check` 报错

报错格式：`error <文件>:<行> <路径>: <消息>`。按文件和行号定位，常见原因：

| 消息 | 原因 |
| --- | --- |
| `unknown region` | 区域 id 拼错，或没在 `regions` 中声明 |
| `unknown variable` / `unknown module` | 条件引用了不存在的模组 / 参数 / 标记 / 阶段 |
| `unknown macro` / `unknown structure` / `unknown actor` | `{{ap::…}}` 名称或参数错（角色、参数、结构 id 不存在） |
| 未闭合的 `@if` / `@style` | 缺 `@end` |
| YAML 解析错误 | 引号、缩进；以 `!` / `@` 开头的值没加引号 |
| 字段名 / 工具名冲突 | 两个阶段或结构生成了同一个名字 |
