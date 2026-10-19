# AGENTS.md

This repository is an **Advanced Preset (AP)** for SillyTavern: 花瓣预设 Petals Preset. 本仓库是一个 AP 预设。

## 先读技能

`.agents/skills/` 中有三个 Agent Skills，开始工作前先读对应的 `SKILL.md`：

| 技能 | 什么时候用 |
| --- | --- |
| [ap-preset-design](.agents/skills/ap-preset-design/SKILL.md) | 从零设计、大改结构、迁移旧 ST 预设：先和用户讨论，写出 `DESIGN.md` |
| [ap-preset-authoring](.agents/skills/ap-preset-authoring/SKILL.md) | 编写或修改源码；`references/` 是完整的 AP 格式参考 |
| [ap-preset-testing](.agents/skills/ap-preset-testing/SKILL.md) | 修改之后检查与预览，排查问题，发布新版本 |

## 仓库结构

```
preset.yaml          清单：角色、区域、参数、结构、推荐配置……
modules/**/*.md      模组（一个文件一个）
workflows/*.md       工作流
locales/*.yaml       面板翻译
dist/                构建产物（由 ap build 生成，发布时提交）
DESIGN.md            设计稿（如果有）
```

## 命令

```bash
npm install            # 安装 ap 命令行
npx ap check --strict  # 检查（每次修改后运行）
npx ap preview --transport <prefill|plaintext|structured|tool-single|tool-multi>
npm run dev            # 开发服务器，在 ST 的 AP 面板 → 调试 → 开发源中填 http://localhost:4747
npm run build          # 生成 dist/
npm run skills         # 更新 .agents/skills 中的 AP 技能
```

## 规则

- 不要手改 `dist/` 和 `.agents/skills/ap-*`（会被重新生成 / 覆盖）。
- 模组、参数、插入点、构建变量的 id 发布后不要改；必须改时写 `migrations`。
- 大段改写已有提示词之前，先向用户说明打算怎么改。
- 提交、推送、打标签之前先征得用户同意。
