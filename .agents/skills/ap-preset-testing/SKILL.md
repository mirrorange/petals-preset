---
name: ap-preset-testing
description: Check, preview, debug and release an Advanced Preset (AP) for SillyTavern — run `ap check --strict`, inspect the assembled prompt for every transport / config / meta mode with `ap preview`, run the `ap dev` live-reload server, diagnose wrong or missing prompt content, and publish a new version. Use after editing AP sources, when output in SillyTavern looks wrong, or before a release. 检查、预览、排查 AP 预设问题，或准备发布新版本时使用。
---

# 测试与发布 Advanced Preset 预设

AP 预设的错误分两类：**格式错误**（`ap check` 能发现）和**提示词问题**（只有看组装结果、在 SillyTavern 里实际试才能发现）。这个技能按“检查 → 预览 → 实测 → 发布”的顺序处理两类问题。

命令都在预设仓库根目录运行；`npx ap` 使用 `package.json` 中记录的 AP 版本。命令细节见 `ap-preset-authoring` 技能的 [cli.md](../ap-preset-authoring/references/cli.md)。

## 1. 检查

```bash
npx ap check --strict
```

- 输出 `✓ <id> <version>: N modules, N entries, N workflows` 表示通过。
- 诊断格式 `<级别> <文件>:<行> <路径>: <消息>`。**错误**必须修；**警告**逐条看——它们常常是笔误（从未被读取的变量、没有分组的模组、某种写法下阶段没有说明）。确实要保留的警告，向用户说明原因。
- 修复方法见 `ap-preset-authoring` 技能的 recipes.md“修复 `ap check` 报错”。

## 2. 预览组装结果

`ap preview` 在本地执行和扩展相同的组装，输出每个区域的消息、工作流阶段，以及 schema / 工具。**读它，就是读模型实际收到的提示词**。

预览矩阵（按预设实际支持的范围裁剪）：

```bash
npx ap preview --transport prefill
npx ap preview --transport plaintext
npx ap preview --transport structured
npx ap preview --transport tool-single
npx ap preview --transport tool-multi
npx ap preview --meta pure                  # 纯元指令：任务工作流
npx ap preview --meta mixed                 # 元指令 + 故事输入
npx ap preview --config <每个推荐配置 id>
npx ap preview --enable <id> --disable <id> # 关键开关、单选集的每个成员
npx ap preview --workflow <每个故事工作流 id>
```

输出较长时写入文件再查：`npx ap preview --transport plaintext --out /tmp/plaintext.txt`。需要程序化检查时用 `--json`（键：`vars`、`buildVars`、`modules`、`structures`、`stages`、`regions`，以及 `schema` 或 `tools`），例如：

```bash
npx ap preview --json --transport tool-multi | node -e "const d=JSON.parse(require('fs').readFileSync(0,'utf8'));console.log(d.modules, d.tools?.map(t=>t.function?.name ?? t.name))"
```

每次预览检查：

- [ ] 开头摘要：传输方式与写法、工作流、生效模组、结构、阶段是否符合预期；
- [ ] 没有重复的段落（两个条目条件重叠），没有该出现却缺失的段落；
- [ ] 对话式：角色前缀正确；写作阶段要求先输出正文标记（默认 `<START_OF_CONTENT>`）；预填充的最后一条是 assistant 前缀；
- [ ] 指令式：每个阶段都有说明；schema 字段 / 工具名称有意义且顺序正确；结构字段出现；
- [ ] 纯元指令：故事专用的写作指南、选项等已经关掉；任务工作流生效；
- [ ] 参数未设置时相关文字不出现，设置后正确出现；
- [ ] 没有残留的 `{{ap::…}}`（未展开的宏）。`{{user}}`、`{{char}}` 原样出现是正常的——ST 在之后展开。

预览与真实运行的差别：ST 宏不展开；记忆、总结、最近元指令为空（`memory.*`、`recap.present`、`meta.recent`、`chat.group` 为假）；元指令文本是占位符 `(meta text)`。

## 3. 在 SillyTavern 里实测

1. 在后台运行开发服务器并保持开启：`npm run dev`（`ap dev`，默认 `http://127.0.0.1:4747`；ST 在其他设备上时用 `npx ap dev --host 0.0.0.0`）。
2. 请用户在 ST 中打开 AP 面板 → **调试** → **开发源**，填入地址。之后每次保存都会自动重建并重新载入。
3. 你无法操作用户的 ST。把需要验证的事项列成清单交给用户（见下），并告诉用户在“调试”页查看**最近组装**（生效模组、各区域内容）和**请求**（原始请求与模型原始输出）。用户贴回原始输出时，对照 [troubleshooting.md](references/troubleshooting.md) 分析。

实测清单（每个推荐配置）：

- [ ] 普通几轮对话；重新生成、滑动（swipe）、继续；
- [ ] 纯元指令（`@叙述者 你好`）、混合元指令；
- [ ] 代写、总结、记忆（如果支持）；
- [ ] 卡片（选项、事件摘要、自定义结构）显示正常；
- [ ] 打开省 token 后的长对话（如果支持事件摘要）；
- [ ] 新用户默认配置的效果。

## 4. 排查

先定位问题出在哪一层，再改：

| 层 | 怎么看 | 典型问题 |
| --- | --- | --- |
| 源码 | `npx ap check --strict` | 格式、引用错误 |
| 组装 | `npx ap preview …` / 调试页“最近组装” | 模组没生效、内容重复、条件写错 |
| 模型输出 | 调试页“请求”中的原始输出 | 不按格式输出、跳过阶段、不调用工具 |
| 解析 | 原始输出 vs 消息正文 / 阶段卡片 | 正文混入思考、卡片缺失 |

常见现象与对策见 [troubleshooting.md](references/troubleshooting.md)。

## 5. 发布

按 [release.md](references/release.md)：升版本号 → `npx ap check --strict` → `npm run build` → **提交 `dist/`** → 推送 →（可选）打 `v*` 标签触发自动 Release。分享给用户的安装链接是仓库里的文件地址 `https://github.com/<用户>/<仓库>/blob/main/dist/<id>.apreset.json`，**不是** Release 附件地址。

## 汇报

测试结束时向用户汇报：跑了哪些检查与预览、发现并修复了什么、还需要用户在 ST 中亲自确认的事项。
