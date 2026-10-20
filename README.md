<div align="center">

# 🌸 花瓣预设 Petals Preset v7

**与小苍兰一起写故事吧！**<br>
Freesia 写故事，Petals 提建议，双角色驱动的工作流。

[![Advanced Preset](https://img.shields.io/badge/Advanced%20Preset-%E2%89%A5%201.4.0-d6409f)](https://github.com/mirrorange/advanced-preset)
[![license](https://img.shields.io/badge/license-CC0%201.0-8b5cf6)](LICENSE)

</div>

花瓣预设是 [Advanced Preset](https://github.com/mirrorange/advanced-preset)（AP）的首个预设，随扩展内置。文学妖精 **Freesia** 负责构思与写作，她的小精灵 **Petals** 负责提问、给建议与审阅——每一轮回复都是两人协作的结果。

## ✨ 特性

| | |
| --- | --- |
| **工作流** | 线性 linear · 迭代 iterative · 编辑 edit · 任务 task |
| **传输** | 预填充 · 纯文本 · 结构化输出 · 单步工具 · 多步工具 |
| **推荐配置** | Gemini · DeepSeek（工具调用 / 预填充）· Claude ≤ 4.5 · Claude 4.6–5 · Claude ≥ 5.5 · 其他（结构化输出 / 工具调用） |
| **写作控制** | 续写 / 扩写模式、写作指南、主文风与 Eros、叙事节奏控制、防抢话、禁词表…… |
| **我的模组** | 开放六个插入位置：身份设定之后 · 角色之后 · 写作指南 · 文风 · 历史之后 · 任务之前 |

## 📦 安装

花瓣预设随 Advanced Preset 扩展内置，安装扩展后在对话补全预设下拉中选择 `AP · Petals Preset` 即可。

想使用最新版本，可以在 **AP 面板 → 预设 → 导入链接** 中填入：

```
https://github.com/mirrorange/petals-preset/blob/main/dist/petals.apreset.json
```

之后可以在预设页一键“从来源更新”，你的配置修改、我的模组与记忆都会保留。

## 🛠️ 开发

需要 Node.js 20+。

```sh
npm install        # 安装 ap 命令行（npm 包 advanced-preset）
npm run dev        # 监视源码并在 http://localhost:4747 提供预设
```

在 SillyTavern 的 **AP 面板 → 调试 → 开发源** 填入 `http://localhost:4747`，保存源码即热更新。提交前运行 `npx ap check --strict` 与 `npm run build`，CI 会校验 `dist/` 是否最新。

用 AI Agent 修改时，[AGENTS.md](AGENTS.md) 与 `.agents/skills/`（`npm run skills` 更新）会告诉 Agent 如何设计、编写、测试 AP 预设。

| 路径 | 内容 |
| --- | --- |
| `preset.yaml` | 清单：actors、区域、参数、结构、正则、Carrier、配置 |
| `modules/**/*.md` | 模组（`<!-- @entry -->` 切分条目） |
| `workflows/*.md` | 工作流（`<!-- @section -->` 按阶段与风格） |
| `locales/zh-cn.yaml` | 面板文案（含预设名称与描述） |
| `dist/petals.apreset.json` | 构建产物（提交进仓库） |

## 📚 延伸阅读

- [案例：花瓣预设是怎样设计出来的](https://github.com/mirrorange/advanced-preset/blob/main/docs/guide/13-petals-case-study.md)
- [AP 预设开发指南](https://github.com/mirrorange/advanced-preset/blob/main/docs/guide/README.md) · [参考文档](https://github.com/mirrorange/advanced-preset/blob/main/docs/reference/README.md)

## 📄 许可

[CC0 1.0](LICENSE)：放弃一切权利，可自由使用、修改与再发布，无需署名。
