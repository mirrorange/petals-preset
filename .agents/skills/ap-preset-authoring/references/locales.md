# 面板文字翻译

`locales/<lang>.yaml` 为预设的**面板文字**（预设名称与描述、模组名、参数说明、工作流阶段名……）提供翻译。它只影响 AP 面板与卡片的显示，**不影响发给模型的文字**——提示词的语言请用参数（如 `language`）与条件控制。

```
locales/
  zh-cn.yaml
  ja.yaml
```

文件名（小写）就是语言键。AP 按 SillyTavern 当前的界面语言选择：简体中文为 `zh-cn`，繁体中文（含 `zh-hk`）为 `zh-tw`，找不到时再试 `zh`；其他语言只用基础代码（`ja`、`ko`、`en`……，不区分地区）。都没有时显示清单里的原文。

## 结构

```yaml
name: 我的预设                      # 预设名称（preset.yaml#name）
description: 一句话介绍这个预设       # 预设描述（preset.yaml#description）
groups:
  guidelines: { label: 写作准则, description: 控制叙事风格的规则 }
modules:
  pacing: { name: 节奏控制, description: 让每个场景聚焦一个变化 }
params:
  length:
    label: 正文长度
    description: 每轮大约多少字
  language:
    label: 语言
    options: { zh: 中文, en: 英文 }     # 选项值 → 显示名
workflows:
  story:
    name: 故事
    description: 思考、写作、审阅
    stages: { think: 思考, content: 正文, review: 审阅 }
configurations:
  gemini: { name: Gemini 推荐, description: 工具调用模式 }
structures:
  status: { title: 状态栏 }
insertionPoints:
  guidelines: { label: 写作准则, description: 内置准则之后 }
vars:
  questions: { label: 问题列表, description: 叙述者每轮要回答的问题 }
```

| 段 | 键 | 可翻译字段 |
| --- | --- | --- |
| `name` | — | 预设名称（字符串）：预设列表、面板标题、扩展设置中显示 |
| `description` | — | 预设描述（字符串）：预设列表中显示 |
| `groups` | 分组 id | `label` `description` |
| `modules` | 模组 id | `name` `description` |
| `params` | 参数 id | `label` `description` `options`（`{ 选项值: 显示名 }`） |
| `workflows` | 工作流 id | `name` `description` `stages`（`{ 阶段 id: 显示名 }`） |
| `configurations` | 推荐配置 id | `name` `description` |
| `structures` | 结构 id | `title` |
| `insertionPoints` | 插入点 id | `label` `description` |
| `vars` | 构建变量 id | `label` `description` |

- 任何字段都可以省略，缺失的显示原文；
- `name` 只影响显示：ST 预设列表中的 Carrier 名（`AP · <name>`）始终用清单里的原文，切换界面语言不会产生新的 Carrier；
- 引用不存在的 id 给出警告（`ap check --strict` 会因此失败）；
- 文件必须是 YAML 映射，否则报错。

阶段的显示名也用于阶段面板与 `{{ap::workflow::summary}}` 之外的 UI；模型看到的阶段标签始终是工作流源码中的 `label`。
