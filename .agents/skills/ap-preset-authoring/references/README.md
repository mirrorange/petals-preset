# Advanced Preset 参考文档

本参考面向**高级开发者**与 **AI Agent**：逐字段、逐规则地说明 Advanced Preset（AP）预设源码的格式、编译规则与运行期行为。内容以 AP 1.5 为准（源码格式 `formatVersion: 1`，产物格式 `formatVersion: 3`）。

> 想先理解概念？请读[预设开发指南](https://github.com/mirrorange/advanced-preset/blob/main/docs/guide/README.md)。本参考假定你已经知道“模组、区域、工作流、元指令”是什么。

## 目录

| 文件 | 内容 |
| --- | --- |
| [manifest.md](manifest.md) | `preset.yaml` 的每个顶层字段 |
| [modules.md](modules.md) | 模组文件：frontmatter、条目、激活规则、排序 |
| [workflows.md](workflows.md) | 工作流文件：阶段、分节、字段名与工具名 |
| [directives.md](directives.md) | `@entry` `@section` `@as` `@style` `@if` 指令 |
| [expressions.md](expressions.md) | 条件表达式语法与全部条件变量 |
| [macros.md](macros.md) | `{{ap::…}}` 宏 |
| [build-variables.md](build-variables.md) | 构建变量 `vars` |
| [structures.md](structures.md) | 结构（回复、记忆、总结、代写、事件摘要、选项、自定义） |
| [transports.md](transports.md) | 五种传输方式、JSON Schema / 工具生成、`tools` 文案、示范轮次 |
| [meta-and-tasks.md](meta-and-tasks.md) | 元指令、生成来源、任务工作流、历史改写与省 token |
| [configuration.md](configuration.md) | 配置层、推荐配置、Carrier、迁移 |
| [user-modules.md](user-modules.md) | 插入点与用户模组（含导入导出格式） |
| [locales.md](locales.md) | 面板文字翻译 |
| [assembly.md](assembly.md) | 一次生成的完整组装流程 |
| [cli.md](cli.md) | `ap` 命令行 |
| [artifact-and-runtime.md](artifact-and-runtime.md) | 产物 JSON、安装与更新、聊天里保存的数据、全局 API |

## 源码结构

预设仓库里只有四类文件是源码，其余文件（README、`package.json`、`.github/`、`.agents/` 等）都会被编译器忽略：

```
preset.yaml              清单：身份、角色、区域、参数、结构、配置……
modules/**/*.md          模组：每个文件一个模组（可以放在任意深度的子目录里）
workflows/*.md           工作流：每个文件一个工作流（只认 workflows/ 下一层）
locales/<lang>.yaml      面板翻译（可选）
```

编译器读取源码时跳过以 `.` 开头的文件和目录、`node_modules/` 和 `dist/`。`preset.yml` 等同于 `preset.yaml`。

## 构建流程

```
源码树 ──compile──▶ APreset 对象 ──validate──▶ 诊断 ──write──▶ dist/<id>.apreset.json
```

1. **compile**（`packages/cli/src/compile.ts`）：解析 YAML 与 Markdown，填入默认值，把条件表达式编译成 AST，排序模组与条目，生成 Carrier，计算哈希。结构性错误（缺少字段、非法值、重复 id）在这里报告。
2. **validate**（`packages/cli/src/validate.ts`）：交叉引用检查（未知变量、未知宏、未知区域……）与风格警告。
3. 产物写成两空格缩进的 JSON。`hash` 是去掉 `hash` 字段后、键排序的规范化 JSON 的 SHA-256 前 16 位十六进制。相同源码总是得到相同产物。

`ap build` / `ap check` 在有错误时返回非零退出码；加 `--strict` 时警告也算失败（见 [cli.md](cli.md)）。

## 运行期流程（概览）

```
用户发送 ─▶ 拦截器：分类（元指令 / 来源 / 正则标记），改写历史
        ─▶ ST 按 Carrier 组装提示词，每个区域在其中是一个锚点 ⟦ap:region:<id>⟧
        ─▶ AP 在锚点处渲染区域（模组条目 + 工作流）；预填充模式再整形
        ─▶ AP 接管请求：按传输方式加上 JSON Schema / 工具 / 停止词，发送并流式解析
        ─▶ 正文写入消息；阶段、结构、事件写入消息的 sidecar（extra.advanced_preset）
```

详见 [assembly.md](assembly.md)。

## 约定

- **id**：预设 id、插入点 id 必须是小写 kebab-case（`^[a-z0-9][a-z0-9-]*$`）。模组、参数、结构等 id 没有强制格式，但建议同样使用 kebab-case，并在发布后保持稳定（改名需要迁移，见 [configuration.md](configuration.md#迁移)）。
- **模型可见文本**：清单里的前缀、结构格式说明、工具描述等文本会先做 `{{ap::…}}` 宏展开，再做 SillyTavern 宏展开（`{{user}}`、`{{char}}` 等）。工具名、字段名不做宏展开。
- **默认值**：文中“默认”指字段缺省时编译器填入的值。
- **诊断**：“错误”使构建失败；“警告”只在 `--strict` 下导致失败。
