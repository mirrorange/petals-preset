# 排查手册

先用 `npx ap preview` 复现（选对传输方式、配置、`--meta`、开关），再改源码。能在预览里看到的问题，一定先在预览里修好。

## 组装问题（预览里就能看到）

| 现象 | 可能原因 | 怎么查 / 怎么改 |
| --- | --- | --- |
| 模组没生效 | 开关没开；单选集中选了别的成员；模组 `when` 为假 | 预览摘要的 `modules:`；`--enable <id>`；检查 `when` 中的变量取值（[expressions.md](../../ap-preset-authoring/references/expressions.md)） |
| 一段话出现两次 | 两个条目条件重叠（如对话式和指令式版本都出现） | 预览对应传输方式；把条件改成互斥（`style == "dialogue"` / `style == "imperative"`） |
| 某种写法下阶段没有说明 | 工作流缺少该写法的 `@section stage:<id>` | `ap check` 的警告；补分节或去掉 `style` 共用一份 |
| 结构的格式说明没出现 | 列出结构的模组没生效；正文里没写 `{{ap::structure::<id>}}` | 预览摘要的 `structures:` |
| schema / 工具里没有结构字段 | 同上；或任务生成（脚本调用）不请求结构 | `--json` 看 `schema` / `tools` |
| 参数没填却出现了空句子 | 没有 `unset`，也没有 `@if` 包裹 | 加 `unset` 或 `<!-- @if params.x != "" -->` |
| `{{ap::…}}` 原样出现 | 宏名或参数错 | `ap check` 会报 `unknown macro`；见 macros.md |
| 元指令时还在要求写故事 | 故事专用模组缺 `when: "!meta.pure"` | `npx ap preview --meta pure` |
| 内容位置不对 | 区域或 `order` 不对；锚点区域相对聊天记录的位置 | 预览中的区域顺序；[assembly.md](../../ap-preset-authoring/references/assembly.md) |

## 模型输出问题（需要原始输出）

请用户在 AP 调试页“请求”中复制模型原始输出。

| 现象 | 可能原因 | 对策 |
| --- | --- | --- |
| 正文里混进了思考 / 讨论（对话式） | 模型没输出正文标记（`<START_OF_CONTENT>`）或角色前缀 | 在写作阶段说明里明确要求先输出标记；预填充时检查 `prefillGuide` 与结尾前缀 |
| 预填充模式下开场白出现两次 / 模型重复前缀 | 结尾条目与预填充前缀重复 | 预览 `--transport prefill` 看最后一条 assistant 消息 |
| 纯文本模式下模型直接写正文、跳过工作流 | 结尾引导不够明确 | 要求先输出固定标记（如 `<!-- Workflow Start -->`），再以叙述者前缀开始 |
| 工具模式下模型不调用工具 / 只调一个 | 模型能力不足或说明不清 | 打开 `forceTool`；加示范轮次（`syntheticTurns`）；改进 `tools` 文案；多工具模式检查 `maxSteps` |
| Gemini 思考过长 / 报思维签名错误 | 未配置 Gemini 专用设置 | 推荐配置用 `tool-single` + `sendThoughtSignature: true`，配合示范轮次 |
| 结构化输出报错 | 接口不支持 JSON Schema | 换 `tool-single` 或对话式传输方式 |
| 跳过阶段 / 阶段顺序乱 | 阶段说明不清；字段名没有序号 | 字段名模板带 `{{n}}`；强化 intro 中的顺序说明 |
| 卡片没出现 | 结构未请求；模型没按格式输出 | 先查组装，再看原始输出中有没有对应标签 / 字段 |
| 记忆 / 总结没有保存 | 后台（quiet）生成；结构未请求 | 见 structures.md“保存条件” |

## 构建与加载问题

| 现象 | 对策 |
| --- | --- |
| `ap dev` 显示旧版本 | 构建失败时继续提供上一次成功的产物：看终端或 `GET /` 的 `errors` |
| ST 连不上开发服务器 | 地址与端口；ST 在其他设备时 `--host 0.0.0.0` 并使用局域网 IP |
| 安装时提示需要升级 AP | 预设的 `minAdvancedPreset` 高于用户的 AP 版本 |
| 用户升级后设置“丢了” | 改了模组 / 参数 / 插入点 / 变量 id 却没写 `migrations` |
