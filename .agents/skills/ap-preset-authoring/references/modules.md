# 模组文件

模组是 `modules/` 目录下的 Markdown 文件，一个文件一个模组。文件可以放在任意深度的子目录里（如 `modules/guidelines/pacing.md`），子目录只用于整理，不影响行为。

```markdown
---
id: pacing
name: Pacing
description: Keep scenes moving.
group: guidelines
region: guidelines
order: 200
default: true
---
## Pacing

Keep each scene focused on one change. <!-- @if params.length > 0 -->Aim for about {{ap::param::length}} words.<!-- @end -->

<!-- @entry id=reminder region=task order=10 when='transport == "plaintext"' -->
Remember the pacing guideline while writing.
```

## frontmatter

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | 文件名（去掉 `.md`） | 模组 id，全预设唯一。条件里写作 `modules.<id>` |
| `name` | `id` | 面板显示名 |
| `description` | — | 面板说明 |
| `group` | — | 面板分组（`groups` 的键）。用户可开关的模组没有分组时归入“其他”并给出警告 |
| `toggle` | `user` | `user`：用户可开关；`always`：总是启用（不显示开关）。旧写法 `system: true` 等同 `always` |
| `exclusive` | — | 单选集名（`groups` 的键）。同一单选集里最多启用一个 |
| `default` | `false` | 用户可开关模组的默认状态。只有 `true` 有意义 |
| `region` | — | 条目的默认区域。没有 `region` 的条目会报错 |
| `order` | `1000` | 条目的默认顺序（同一区域内从小到大） |
| `when` | — | 模组级条件，见[激活规则](#激活规则) |
| `structures` | `[]` | 本模组启用时请求的结构 id 列表（见 [structures.md](structures.md)） |
| `vars` | — | 本模组对构建变量的更新（见 [build-variables.md](build-variables.md)） |

已废弃：`questions`（会报错，改用构建变量）。

## 条目

模组正文被 `@entry` 指令切成若干**条目**。每个条目是一段落在某个区域、带顺序和条件的提示词。

- 第一个 `@entry` 之前的文字（如果非空）是名为 `main` 的条目，使用 frontmatter 的 `region` / `order`，没有条目级条件。
- `<!-- @entry … -->` 开始一个新条目，属性：

| 属性 | 默认 | 说明 |
| --- | --- | --- |
| `id` | `entry<N>`（N 为块序号） | 条目本地 id，完整 id 为 `<模组id>/<本地id>` |
| `region` | frontmatter `region` | 区域 id |
| `order` | frontmatter `order` | 数字 |
| `when` | — | 条目级条件（在模组条件之上再判断） |

属性值可以不加引号（不含空格时），也可以用双引号或单引号；引号内用反斜杠转义。条件表达式里有双引号时，外层用单引号最方便：`when='transport == "prefill"'`。

条目内容中可以使用 `@as`、`@style`、`@if` 指令（见 [directives.md](directives.md)）和 `{{ap::…}}` 宏（见 [macros.md](macros.md)）。三个以上连续换行会压成两个，块首尾空白会被去掉，空块被丢弃。

## 激活规则

每次组装时按以下顺序决定哪些模组启用：

1. `toggle: always` 的模组总是候选。
2. 用户可开关的普通模组：配置 `modules.<id>` 为 `true` 时是候选。
3. 单选集（`exclusive: <set>`）：
   - 按模组顺序，第一个在配置中为 `true` 的成员胜出；
   - 没有成员为 `true` 时：若分组 `allowNone: true`，这个集合一个都不启用；否则使用 `default: true` 的成员，没有就用第一个成员。
4. 候选模组再判断模组级 `when`；为假则本轮不启用。
5. 启用模组的每个条目再判断条目级 `when`。

注意：

- 条件变量 `modules.<id>` 反映的是**第 1–3 步的开关状态**，与模组的 `when` 无关。因此条件可以安全地引用一个“开着但本轮因 `when` 未启用”的模组。
- `toggle: always` 的模组不能有 `exclusive`（报错）。
- 单选集中多个成员 `default: true` 给出警告；没有默认且不允许为空给出警告（会用第一个）。
- 推荐配置中同一单选集启用多个成员给出警告。
- 结构与构建变量只取自**启用的**模组。

## 排序

**模组顺序**由 `preset.yaml` 的 `moduleOrder` 决定：

```yaml
moduleOrder: [core, context, role-freesia, "*", output-format]
```

- 列出的 id 按列出顺序；`*` 代表所有未列出的模组（按文件路径字母序）。没写 `*` 时自动追加在末尾；不写 `moduleOrder` 等同 `["*"]`。
- 未知 id 给出警告。
- 模组顺序影响：面板顺序、单选集“第一个”的判定、构建变量的更新顺序（后面的模组覆盖前面的字典键），以及同区域同 `order` 条目的先后。

**条目顺序**：先按区域在 `regions` 中的位置，再按 `order`，再按模组顺序。用户模组的条目插在插入点的 `order` 处，同 `order` 时排在预设条目之后。

## 设计建议

- 一个模组只负责一件事（一个功能开关、一个写作指南），需要在多个区域放内容时用多个 `@entry`。
- 对不同写法差异大的内容，用多个条目加 `when` 分开；差异小的用 `@style` 片段。
- 让用户能理解的才做成 `toggle: user`；框架性的内容（核心规则、输出格式）用 `toggle: always`。
- `id` 发布后不要改；必须改时写迁移（见 [configuration.md](configuration.md#迁移)）。
