<div align="center">

# 🌸 花瓣预设 Petals Preset v7

**与小苍兰一起写故事吧！**<br>
Freesia 写故事，Petals 提建议，双角色驱动的工作流。

[![Advanced Preset](https://img.shields.io/badge/Advanced%20Preset-%E2%89%A5%201.11.0-d6409f)](https://github.com/mirrorange/advanced-preset)
[![license](https://img.shields.io/badge/license-CC0%201.0-8b5cf6)](LICENSE)

</div>

花瓣预设是 [Advanced Preset](https://github.com/mirrorange/advanced-preset)（AP）的首个预设，随扩展内置。文学妖精 **Freesia** 负责构思与写作，她的小精灵 **Petals** 负责提问、给建议与审阅——每一轮回复都是两人协作的结果。

## ✨ 特性

| | |
| --- | --- |
| **工作流** | 线性 linear · 迭代 iterative · 编辑 edit · 任务 task |
| **传输** | 预填充 · 纯文本 · 结构化输出 · 单步工具 · 多步工具 |
| **推荐配置** | Gemini（Vertex AI / AI Studio / 其他渠道，思维签名按渠道发送）· DeepSeek（工具调用 / 预填充）· Claude ≤ 4.5 · Claude 4.6–5 · Claude ≥ 5.5 · 其他（结构化输出 / 工具调用） |
| **写作控制** | 续写 / 扩写 / 扩写后续写模式、写作指南、主文风与 Eros、控制节奏 / 推进剧情、防抢话、禁词表（写作页可添加自定义禁用词语 / 短语 / 表达）…… |
| **功能** | 用户选项 · 事件摘要 · 故事线（输入末尾写 `#支线名称` 写支线故事，`#if…` 写 if 线，`#` 回到最近的支线，`#?` / `#!?` / `#if?` 开始由 Freesia 起名的新支线 / 新主线 / 新 if 线，`#@支线名称` 让支线对其他故事线不可见；支线默认不更新变量，可在配置中改为仅 if 线或不阻止）· 状态栏（正文上方显示日期 / 星期 / 时间 / 地点 / 天气）· 始终请求元回复 · Freesia 的评论…… |
| **快速回复** | 选中花瓣预设时，ST 的快速回复栏出现五个按钮：元指令（追加 `@Freesia`）· 总结 · 代写回复（发送对应的元指令，输入框中的草稿作为补充）· 故事线（追加 `#`）· 新分支（追加 `#?`）；切换到其他预设时自动移除 |
| **我的模组** | 开放六个插入位置：身份设定之后 · 角色之后 · 写作指南 · 文风 · 历史之后 · 任务之前；新建时可从「自定义文风」「自定义思维链」模板开始 |

## 📦 安装

花瓣预设随 Advanced Preset 扩展内置，安装扩展后在对话补全预设下拉中选择 `AP · Petals Preset` 即可。

想使用最新版本，可以在 **AP 面板 → 预设 → 导入链接** 中填入：

```
https://github.com/mirrorange/petals-preset/blob/main/dist/petals.apreset.json
```

之后可以在预设页一键“从来源更新”，你的配置修改、我的模组与记忆都会保留。

还没有安装 Advanced Preset？`petals.apreset.json` 也是一个标准的 SillyTavern 预设：下载后在 ST 的对话补全预设中直接 **导入**，AI 会告诉你如何安装插件；装有[酒馆助手](https://github.com/N0VI028/JS-Slash-Runner)时，在它询问时启用预设中的脚本，即可自动从 GitHub / GitLab / Codeberg 安装 Advanced Preset。刷新页面后，插件会自动把它换成 `AP · Petals Preset`。

## 🛠️ 开发

需要 Node.js 20+。

```sh
npm install        # 安装 ap 命令行（npm 包 advanced-preset）
npm run dev        # 监视源码并在 http://localhost:4747 提供预设
```

在 SillyTavern 的 **AP 面板 → 调试 → 开发源** 填入 `http://localhost:4747`，保存源码即热更新。提交前运行 `npx ap check --strict` 与 `npm run build`，CI 会校验 `dist/` 是否最新。

分支约定：`dev` 分支依赖 `"advanced-preset": "dev"`（AP `dev` 分支自动发布的预发布 CLI），可直接使用尚未发布的 CLI 功能；用 `npm update advanced-preset` 更新到最新 dev 版本。合并进 `main` 前需改回已发布的版本范围（如 `^1.7.0`），否则 CI 会报错。

用 AI Agent 修改时，[AGENTS.md](AGENTS.md) 与 `.agents/skills/`（`npm run skills` 更新）会告诉 Agent 如何设计、编写、测试 AP 预设。

| 路径 | 内容 |
| --- | --- |
| `preset.yaml` | 清单根文件：元数据、actors、写法、区域、主题、Carrier、迁移 |
| `manifest/*.yaml` | 清单片段，构建时合并：`params`（分组、参数、构建变量、插入点）、`structures`、`tools`（工具文案、示范轮次）、`configurations`（配置默认值、推荐配置）、`behavior`（元指令、输入整形、故事线、省 token、正则、排序） |
| `modules/**/*.md` | 模组（`<!-- @entry -->` 切分条目） |
| `workflows/*.md` | 工作流（`<!-- @section -->` 按阶段与风格） |
| `quick-replies/petals.yaml` | 快速回复集（选中预设时加入 ST 的全局快速回复集列表） |
| `user-module-templates/*.md` | 用户模组模板：新建「我的模组」时可选的示例（`<id>.zh-cn.md` 为中文版） |
| `locales/zh-cn.yaml` | 面板文案（含预设名称与描述） |
| `dist/petals.apreset.json` | 构建产物：预设文件（标准 ST 预设，附带酒馆助手安装脚本；提交进仓库） |

## 📚 延伸阅读

- [案例：花瓣预设是怎样设计出来的](https://github.com/mirrorange/advanced-preset/blob/main/docs/guide/13-petals-case-study.md)
- [AP 预设开发指南](https://github.com/mirrorange/advanced-preset/blob/main/docs/guide/README.md) · [参考文档](https://github.com/mirrorange/advanced-preset/blob/main/docs/reference/README.md)

## 📄 许可

[CC0 1.0](LICENSE)：放弃一切权利，可自由使用、修改与再发布，无需署名。
