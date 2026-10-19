---
name: ap-preset-design
description: Design an Advanced Preset (AP) for SillyTavern before writing files — interview the user, decide actors, regions, modules, toggles/exclusive sets, params, workflows, built-in features, transports and recommended configurations, and record it in DESIGN.md. Use when starting a new AP preset, restructuring one, or migrating a classic SillyTavern preset to AP. 设计 AP 预设、整理设计稿、规划模组与工作流、把旧 ST 预设迁移到 AP 时使用。
---

# 设计 Advanced Preset 预设

这个技能帮你和用户一起把“想让模型怎么写”整理成一份**设计稿** `DESIGN.md`。设计稿确认之后，再用 `ap-preset-authoring` 技能写文件，用 `ap-preset-testing` 技能测试。

**设计阶段只写 `DESIGN.md`，不要改 `preset.yaml`、`modules/`、`workflows/`。** `DESIGN.md` 放在仓库根目录，编译器会忽略它。

## 背景（30 秒）

Advanced Preset（AP）是 SillyTavern 的扩展。预设作者写源码（`preset.yaml` + `modules/*.md` + `workflows/*.md` + `locales/*.yaml`），`ap build` 编译成 `dist/<id>.apreset.json`，用户在 AP 面板里安装。关键概念：

| 概念 | 一句话 |
| --- | --- |
| 角色 actors | 提示词里出现的说话人（叙述者、用户，可再加编辑、助手等），有名字与前缀 |
| 区域 regions | 提示词里的“位置”（开头、写作指南块、角色卡、聊天记录前后、任务、结尾） |
| 模组 modules | 一个文件 = 一个功能 / 一条规则；可以是总是生效、用户开关、单选集成员 |
| 参数 params | 用户填写的值（语言、字数……），没填时相关文字不出现 |
| 工作流 workflows | 每轮回复分几个阶段（提问 → 规划 → 写作 → 自检） |
| 写法 styles | 对话式 dialogue（剧本式对话）/ 指令式 imperative（编号指令）——同一份提示词往往要两种写法 |
| 传输方式 transports | prefill、plaintext（对话式）；structured、tool-single、tool-multi（指令式） |
| 结构 structures | 模型额外输出的数据：回复、记忆、总结、代写、事件摘要、选项、自定义（状态栏等） |
| 元指令 meta | 用户 `@叙述者 …` 直接和叙述者说话；纯元指令走任务工作流 |
| 构建变量 vars | 多个模组共同填写的清单（如“规划时要考虑的问题”），用户模组也能增删 |
| 推荐配置 configurations | 为不同模型准备的一键设置（传输方式、工作流、采样、模组开关） |
| 插入点 insertionPoints | 用户“我的模组”可以插入的位置 |

细节在 `ap-preset-authoring` 技能的 `references/` 中；入门讲解见[预设开发指南](https://github.com/mirrorange/advanced-preset/blob/main/docs/guide/README.md)。

## 流程

1. **了解现状**。
   - 新仓库：看 `preset.yaml` 和 `modules/`（模板自带的小例子），确认 id / 名字 / 作者。
   - 已有预设：读 `preset.yaml`、`ls -R modules workflows`，运行 `npx ap check` 和 `npx ap preview`，先弄清现在的结构，再谈改动。
   - 迁移旧 ST 预设：让用户提供预设 JSON，按 [design-checklist.md](references/design-checklist.md#迁移旧-st-预设) 逐条归类。
2. **访谈**。按 [design-checklist.md](references/design-checklist.md) 的顺序提问。规则：
   - 每次最多问 3–5 个问题，按重要性排序，并给出**你的推荐选项**和理由；用户说“你定”就用推荐值；
   - 用户已经回答过的不要再问；能从材料里推断的先写成假设，请用户确认；
   - 用用户听得懂的话问（“哪些规则要能开关？”），不要堆术语。
3. **写设计稿**。复制 [design-doc-template.md](references/design-doc-template.md) 的结构写成 `DESIGN.md`。要具体到 id：模组 id、所在区域、开关类型、默认值、条件；工作流的阶段 id 与种类；推荐配置的 diff。不确定的地方列在“待定问题”。
4. **确认**。把设计稿要点总结给用户（不要整篇贴出），请用户确认或修改。重大决定（角色、工作流阶段、默认传输方式）必须得到明确确认。
5. **交接**。确认后告诉用户下一步：“使用 `ap-preset-authoring` 技能按 `DESIGN.md` 实现”。如果用户让你直接继续，就切换到那个技能。

## 设计原则

- **先定核心体验**：模型扮演谁、和用户什么关系、每轮回复包含什么。其余都从这里推出。
- **按位置分区**：身份与核心规则在开头；写作指南包在一个标签块里（便于工作流引用 `<guidelines>`）；记忆紧跟聊天记录；本轮任务与提醒靠近结尾。
- **一个模组讲一件事**。核心框架 `toggle: always`；用户能理解、愿意调节的才做开关；互斥的做单选集（`exclusive`），可叠加的做开关；任务类模组“始终生效 + `when` 条件”。
- **参数没填就不出现**：每个参数都要有 `unset` 或条件包裹的文字。
- **工作流分档**：轻（规划 → 写作）、重（提问 → 建议 → 写作 → 自检 / 修订）；每个阶段只做一件事；必须有一个 `content` 阶段；再准备一个 `kind: task` 的任务工作流处理元指令和代写。
- **对话式与指令式各写一份**关键说明。对话式像剧本（角色前缀、`<START_OF_CONTENT>` 标记正文），指令式像清单（字段 / 工具）。先决定要支持哪些传输方式，再决定要不要两种写法都写。
- **元指令、代写、总结时关掉故事专用内容**（`when: "!meta.pure"` 等）。
- **给用户留扩展空间**：少量、稳定的插入点；可扩展的清单用构建变量。
- **id 是用户数据的键**：用 kebab-case，起好就不要改。
- **提示词语言**：模型可见的提示词通常统一用英文（省 token、模型更稳定），面板文字用 `locales/` 翻译；也可以全中文——让用户决定，但要统一。

## 参考

- [design-checklist.md](references/design-checklist.md)：访谈问题清单、推荐默认值、旧 ST 预设迁移方法；
- [design-doc-template.md](references/design-doc-template.md)：`DESIGN.md` 模板；
- [petals-case-study.md](references/petals-case-study.md)：花瓣预设 v7 的完整设计，作为成熟范例。
