# DESIGN.md 模板

复制下面的结构到仓库根目录的 `DESIGN.md`，按访谈结果填写。用表格，写到 id 级别；没有决定的写进“待定问题”。

````markdown
# <预设名> 设计稿

## 核心体验

<一段话：模型扮演谁、和用户的关系、每轮回复包含什么、主要使用场景。>

## 基本信息

| 项 | 值 |
| --- | --- |
| id | my-preset |
| 名字 | My Preset |
| 作者 | … |
| 起始版本 | 0.1.0 |
| 提示词语言 | English（面板翻译：zh-cn） |

## 角色

| id | 名字 | 前缀 | 人设 |
| --- | --- | --- | --- |
| narrator | … | `{{ap::name::narrator}}: ` | … |
| user | `{{user}}` | `{{ap::name::user}}: ` | 用户 |

## 传输方式与写法

| 传输方式 | 支持 | 写法 |
| --- | --- | --- |
| prefill | 是 | dialogue |
| plaintext | 是 | dialogue |
| structured | 是（默认） | imperative |
| tool-single | 是 | imperative |
| tool-multi | 否 | imperative |

正文标记：`<START_OF_CONTENT>`；预填充引导：`<START_OF_STORY>`。

## 区域

| id | role | merge | 锚点 | 内容 |
| --- | --- | --- | --- | --- |
| top | system | join | — | 身份与核心规则 |
| … | | | | |

工作流区域：`task`。

## 模组

| id | 名字 | 类型 | 分组 | 默认 | 区域 | 条件 | 内容 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| core | Core | always | — | — | top, tail | — | 身份、结尾引导 |
| pacing | Pacing | toggle | guidelines | 开 | guidelines | `!meta.pure` | 每个场景只推进一件事 |
| style-classic | Classic | exclusive: style | style | 默认 | guidelines | `!meta.pure` | … |
| meta | Meta | always | — | — | task | `meta.present` | 元指令回复 |

模组顺序（`moduleOrder`）：core, …, `*`

## 分组

| id | 名字 | 页 | 单选 / 允许不选 |
| --- | --- | --- | --- |
| guidelines | Guidelines | modules | — |
| style | Style | modules | 单选，allowNone: true |
| language | Language | writing | — |

## 参数

| id | 类型 | 名字 | 默认 | 未设置时 | 用在 |
| --- | --- | --- | --- | --- | --- |
| language | text（选项：English, 简体中文） | Reply language | '' | the language used in the story so far | `{{ap::lang}}` |
| length | number（words） | Target length | 0 | 不出现 | core 中 `@if params.length > 0` |

## 构建变量

| id | 类型 | 用途 | 谁来填 |
| --- | --- | --- | --- |
| considerations | dict | 规划时要考虑的问题 | 各写作指南模组 |

## 工作流

### linear（story，默认）

| 阶段 id | 角色 | 种类 | 名字 | 说明 |
| --- | --- | --- | --- | --- |
| plan | narrator | text | Planning | 列出要点，逐条回答 considerations |
| final | narrator | content | Writing | 写正文 |

### task（task）

| 阶段 id | 角色 | 种类 | 名字 | 说明 |
| --- | --- | --- | --- | --- |
| thinking | narrator | text（tag: thinking） | Thinking | 思考 |
| content | narrator | content（optional） | Output | 需要时输出内容 |

## 结构与内置功能

| 功能 | 是否 | 结构 id / 设置 | 说明 |
| --- | --- | --- | --- |
| 元指令 | 是 | reply；trigger `@{{ap::name::narrator}}` | … |
| 选项 | 是 | options，count 3 | … |
| 总结 | 否 | | |

## 推荐配置

| id | 名字 | diff | 默认 |
| --- | --- | --- | --- |
| default | Default | `{}`（structured） | ✓ |
| prefill | Prefill models | `connection: { transport: prefill }` | |

`configurationDefaults` 要点：temperature 1，max tokens 16384，……

## 插入点

| id | 名字 | 区域 | order |
| --- | --- | --- | --- |
| system-end | End of system prompt | top | 1000 |

## 迁移对照（仅迁移旧预设时）

| 原条目 | 归类 | 新位置 |
| --- | --- | --- |

## 待定问题

- …
````
