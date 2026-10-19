---
name: ap-preset-authoring
description: Write and edit Advanced Preset (AP) source files for SillyTavern — preset.yaml, modules/*.md, workflows/*.md, locales/*.yaml — following the AP format reference bundled with this skill. Use for implementing a DESIGN.md, adding or changing modules, params, workflows, structures, configurations, insertion points or translations, and fixing `ap check` errors. 编写或修改 AP 预设源码（模组、工作流、参数、结构、推荐配置、翻译）、修复构建报错时使用。
---

# 编写 Advanced Preset 预设

这个技能负责把设计变成正确的源码文件。完整的格式规则在 [references/](references/README.md)（与 AP 官方参考文档相同），本文件是工作方法、速查与易错点。

## 开始之前

1. 读 `DESIGN.md`（如果有）。没有设计稿、而改动涉及新增角色 / 工作流 / 大量模组时，先建议用户用 `ap-preset-design` 技能。
2. 读 `preset.yaml`，`ls -R modules workflows locales`，了解现有 id、区域、分组。
3. 运行 `npx ap check`，记下已有的错误和警告，不要把它们算到你的改动上。
4. 如果用户正在用 `npm run dev` 实时预览，保持它运行；每次保存都会自动重新构建。

## 工作方式

- **小步修改**：每完成一个功能（一个模组、一个阶段、一条配置），运行 `npx ap check`，有错误立刻修。
- **按需查参考**：动手前打开对应的参考文件（见下表），不要凭印象猜字段名。
- **改完看组装结果**：用 `npx ap preview --transport <t>` 看模型实际收到的提示词（详见 `ap-preset-testing` 技能）。
- **提示词措辞是用户的作品**：大段改写已有提示词之前，先说明打算怎么改；照设计稿新写的内容，写完后简要列出。
- **不要手改 `dist/`**：它由 `ap build` / `ap dev` 生成。
- **不要修改 `.agents/skills/ap-*`**：它们会被 `ap skills install` 覆盖。

## 该读哪份参考

| 要做的事 | 参考 |
| --- | --- |
| 改 `preset.yaml` 任一顶层字段（角色、写法、区域、参数、分组、正则、元指令、主题……） | [manifest.md](references/manifest.md) |
| 新建 / 修改模组，开关与单选，排序 | [modules.md](references/modules.md) |
| 新建 / 修改工作流、阶段、字段名与工具名 | [workflows.md](references/workflows.md) |
| `@entry` `@section` `@as` `@style` `@if` | [directives.md](references/directives.md) |
| `when` / `@if` 条件与可用变量 | [expressions.md](references/expressions.md) |
| `{{ap::…}}` 宏 | [macros.md](references/macros.md) |
| 多个模组共同填写的清单 | [build-variables.md](references/build-variables.md) |
| 回复、记忆、总结、代写、事件摘要、选项、自定义结构 | [structures.md](references/structures.md) |
| 传输方式、schema / 工具、`tools` 文案、示范轮次 | [transports.md](references/transports.md) |
| 元指令、任务工作流、生成来源、省 token | [meta-and-tasks.md](references/meta-and-tasks.md) |
| `configurationDefaults`、推荐配置、Carrier、迁移 | [configuration.md](references/configuration.md) |
| 插入点、用户模组 | [user-modules.md](references/user-modules.md) |
| 面板翻译 | [locales.md](references/locales.md) |
| 一次生成的完整组装流程（排查“为什么出现 / 没出现”） | [assembly.md](references/assembly.md) |
| `ap` 命令 | [cli.md](references/cli.md) |
| 产物格式、安装与更新、聊天中保存的数据 | [artifact-and-runtime.md](references/artifact-and-runtime.md) |

常见任务的完整步骤见 [recipes.md](recipes.md)。

## 速查

**源码**：`preset.yaml`（清单）、`modules/**/*.md`（一个文件一个模组，子目录随意）、`workflows/*.md`（只认一层）、`locales/<lang>.yaml`。以 `.` 开头的文件、`node_modules/`、`dist/` 被忽略；其他文件（README、DESIGN.md）不参与编译。

**模组**：

```markdown
---
id: pacing                 # 默认取文件名；全预设唯一，发布后不改
name: Pacing
description: Keep scenes moving.
group: guidelines          # 用户可开关的模组要有分组
default: true              # 默认开启（toggle: user 时）
region: guidelines         # 条目的默认区域（必填，除非每个条目都写了 region）
order: 200                 # 默认 1000，同区域内从小到大
when: "!meta.pure"         # 模组级条件
# toggle: always           # 总是启用，不显示开关
# exclusive: style         # 单选集（groups 的键）
# structures: [options]    # 启用时请求的结构
# vars: { considerations: { pacing: Is the scene moving? } }
---
## Pacing

Keep each scene focused on one change.

<!-- @entry id=reminder region=task order=10 when='transport == "plaintext"' -->
Remember the pacing guideline.
```

**工作流**：

```markdown
---
id: linear
name: Linear
stages:
  - { id: plan, actor: narrator, kind: text, label: Planning }
  - { id: final, actor: narrator, kind: content, label: Writing }
---
<!-- @section intro style=imperative -->
Complete these stages in order:

<!-- @section stage:plan style=dialogue -->
{{ap::prefix::narrator}}First I will outline the next scene.

<!-- @section stage:plan style=imperative -->
Outline the next scene.

<!-- @section stage:final style=dialogue -->
{{ap::prefix::narrator}}Then I will output <START_OF_CONTENT> and write the story.

<!-- @section stage:final style=imperative -->
Write the story content.
```

**指令**：`@entry`（模组）、`@section preamble|intro|outro|stage:<id> [style=…]`（工作流）、`@as system|user|assistant`（新消息）、`@style dialogue|imperative … @end`、`@if <expr> … @else … @end`。其他 HTML 注释原样留给模型。

**常用条件变量**：`transport`、`style`、`workflow`、`gen.origin`（story/task）、`gen.type`、`meta.present` / `meta.pure` / `meta.mixed` / `meta.task`、`modules.<id>`、`params.<id>`、`flags.<id>`、`stages.<id>`、`memory.hasChat`、`recap.present`、`features.tokenSaving`、`chat.group`。

**常用宏**：`{{ap::name::<actor>}}`、`{{ap::prefix::<actor>}}`、`{{ap::param::<id>}}`、`{{ap::lang}}`、`{{ap::structure::<id>}}`、`{{ap::var::<id>}}`（`::keys` / `::values` / `::<key>`）、`{{ap::workflow::summary}}`、`{{ap::input::meta}}`、`{{ap::memory::chat}}`、`{{ap::recap}}`、`{{ap::recent_meta}}`。ST 宏（`{{user}}`、`{{char}}`）在 AP 宏之后展开。

## 易错点

- **YAML 引号**：以 `!`、`@`、`{`、`[`、`*` 开头或含 `: ` 的值要加引号：`when: "!meta.pure"`、`trigger: "@{{ap::name::narrator}}"`。条件里有双引号时外层用单引号：`when: 'meta.task == "summary"'`。
- **`@entry` 属性**里的条件：含空格必须加引号，`when='transport == "prefill"'`。
- **`@as` 不能放在 `@if` / `@style` 里**；要按条件决定一条消息，给整个条目或分节加 `when`。
- **两种写法都要覆盖**：如果预设同时支持对话式和指令式，每个工作流阶段都要有两种写法的分节（或一份共用分节），否则 `ap check` 警告“该写法下这个阶段没有说明”。
- **对话式正文标记**：对话式下写作阶段要让模型先输出 `contentMarker`（默认 `<START_OF_CONTENT>`），AP 据此找到正文开头。
- **结构不会自动出现**：模组必须在 `structures:` 中列出它，并在正文里放 `{{ap::structure::<id>}}` 作为格式说明。
- **故事专用内容**在元指令 / 代写时关掉：`when: "!meta.pure"`，需要时再加 `gen.type != "impersonate"`。
- **单选集**：同一 `exclusive` 集里只给一个成员 `default: true`；分组 `allowNone: true` 才允许都不选。`toggle: always` 的模组不能有 `exclusive`。
- **参数没填就不出现**：用 `unset` 文本，或 `<!-- @if params.length > 0 -->…<!-- @end -->` 包住。
- **阶段 id** 会变成字段名 / 工具名，用字母开头的 snake_case 或 kebab-case；阶段 `when` 里不要引用 `stages.*`。
- **`configurationDefaults.modules`** 只能写用户可开关的模组；推荐配置的 `diff` 是相对默认值的稀疏覆盖。
- **id 稳定**：模组、参数、插入点、构建变量的 id 是用户数据的键。改名必须写 `migrations`（见 [configuration.md](references/configuration.md#迁移)），并升主版本。
- **新增可见名字时同步翻译**：有 `locales/` 时，新模组、分组、参数、工作流、配置、插入点都要加上翻译条目。
- **`minAdvancedPreset`**：用到新版 AP 才有的功能时提高它。

## 完成时

1. `npx ap check --strict` 无错误、无警告（或向用户解释保留的警告）；
2. 按 `ap-preset-testing` 技能至少预览一种对话式和一种指令式传输方式；
3. 向用户简要汇报：改了哪些文件、新增了哪些模组 / 参数 / 配置、需要用户亲自在 ST 里验证的地方。
