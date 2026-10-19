# 花瓣预设 Petals Preset v7

Freesia 与 Petals 的协作写作预设，[Advanced Preset](https://github.com/mirrorange/advanced-preset) 的首个预设，随扩展内置。

- **工作流**：线性 linear · 迭代 iterative（可设草稿语言）· 编辑 edit · 任务 task
- **传输**：预填充 / 纯文本 / 结构化输出 / 单步工具 / 多步工具
- **推荐配置**：Gemini · DeepSeek（工具调用 / 预填充）· Claude ≤ 4.5 · Claude 4.6–5 · Claude ≥ 5.5 · 其他（结构化输出 / 工具调用）
- **我的模组插入位置**：身份设定之后 · 角色之后 · 写作指南 · 文风 · 历史之后 · 任务之前（需 Advanced Preset ≥ 1.2.0）

## 安装

花瓣预设随 Advanced Preset 扩展内置。也可以在 AP 面板 → 预设 → 导入链接中填入：

```
https://github.com/mirrorange/petals-preset/blob/main/dist/petals.apreset.json
```

## 开发

```sh
npm install
npm run dev        # 或在 AP 仓库旁：node ../advanced-preset/packages/cli/dist/ap.js dev .
```

在 SillyTavern 的 Advanced Preset 面板「调试」→「开发源」填入 `http://localhost:4747`，修改源码即热更新。

用 AI Agent 修改时，仓库里的 [AGENTS.md](AGENTS.md) 与 `.agents/skills/`（`npm run skills` 更新）会告诉 Agent 如何设计、编写、测试 AP 预设。花瓣的设计思路见[案例：花瓣预设是怎样设计出来的](https://github.com/mirrorange/advanced-preset/blob/main/docs/guide/13-petals-case-study.md)；格式说明见 [AP 参考文档](https://github.com/mirrorange/advanced-preset/blob/main/docs/reference/README.md)。

| 路径 | 内容 |
| --- | --- |
| `preset.yaml` | 清单：actors、区域、参数、结构、正则、Carrier、配置 |
| `modules/**/*.md` | 模组（`<!-- @entry -->` 切分条目） |
| `workflows/*.md` | 工作流（`<!-- @section -->` 按阶段与风格） |
| `locales/zh-cn.yaml` | 面板文案 |
| `dist/petals.apreset.json` | 构建产物（提交进仓库，CI 校验是否最新） |

## 从 v6 迁移

`{{petals::…}}` → `{{ap::…}}`；actor `freesia/petals` → `narrator/editor`；`cot.transport` → `transport`；`features.alwaysReply` → `modules.always-reply`；`iterative-switch-language` 并入 `iterative`（`draftLanguage` 参数）；Duet 暂不提供。
