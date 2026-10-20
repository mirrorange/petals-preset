# `ap` 命令行

`ap` 是 AP 的工具链：编译、检查、开发服务器、预览、创建新预设，以及安装 Agent Skills。它随 npm 包 [`advanced-preset`](https://www.npmjs.com/package/advanced-preset) 发布，预设仓库通过 `devDependencies` 安装（`npm install -D advanced-preset`，模板仓库已配置好），在仓库里用 `npx ap …` 或 `npm run <script>` 调用。

```
ap build   [dir] [--out dist] [--strict]
ap check   [dir] [--strict]
ap dev     [dir] [--port 4747] [--host 127.0.0.1] [--out dist]
ap preview [dir] [--transport t] [--workflow w] [--config c] [--meta pure|mixed]
                 [--enable a,b] [--disable c] [--user-modules file] [--json] [--out file]
ap init    <dir> [--id x] [--name "My Preset"] [--author me]
ap skills  list
ap skills  install [dir] [--target .agents/skills]
ap --version
```

`dir` 默认为当前目录。退出码：成功 0，失败 1。

## 诊断格式

`build`、`check`、`dev`、`preview` 都会先编译，并把诊断输出到 stderr：

```
error modules/pacing.md:12 entries.reminder: unknown region `tasks`
warn  preset.yaml groups: module `x` has no group
2 errors, 1 warning
```

格式为 `<级别> <文件>:<行> <路径>: <消息>`（文件、行、路径在已知时出现）。**错误**使构建失败；**警告**只在 `--strict` 时导致失败。建议提交前和 CI 中总是使用 `--strict`。

## ap build

编译并写出产物 `<out>/<id>.apreset.json`（默认 `dist/`）。成功时输出：

```
✓ /path/to/dist/my-preset.apreset.json #3f9c2a1b0d4e5f67
```

`#` 后是产物哈希（[artifact-and-runtime.md](artifact-and-runtime.md)）。相同源码总是得到相同产物。

## ap check

只编译与验证，不写文件。成功时输出统计：

```
✓ my-preset 0.1.0: 12 modules, 18 entries, 2 workflows
```

## ap dev

开发服务器：先构建一次，然后监视源码（`preset.yaml`、`modules/`、`workflows/`、`locales/`），变化后约 120 ms 重新构建并写入 `dist/`。构建失败时继续提供上一次成功的产物。

| 端点 | 内容 |
| --- | --- |
| `GET /` | 状态：`{ id, name, version, hash, builtAt, ok, errors, warnings }`（`errors` / `warnings` 为格式化后的诊断文本） |
| `GET /preset` | 最近一次成功构建的产物 JSON；还没有成功构建时返回 503 |

响应带 CORS 与 `Access-Control-Allow-Private-Network` 头，ST 页面可以直接访问。在 AP 面板 → **调试** → 开发服务器中填入地址（默认 `http://127.0.0.1:4747`）后，AP 每 1.5 秒轮询状态，哈希变化时自动载入新产物并重建 Carrier；面板中显示构建错误与警告。

- `--port`：端口（默认 4747）；
- `--host`：监听地址（默认 `127.0.0.1`）。ST 在其他设备上时用 `--host 0.0.0.0`，并在 AP 中填写电脑的局域网 IP；
- `--out`：产物目录。

## ap preview

在本地执行与扩展相同的组装，输出每个区域的消息、阶段与传输产物（schema / 工具），用于检查“某种配置下模型到底看到什么”。

| 选项 | 说明 |
| --- | --- |
| `--transport` | 传输方式（覆盖配置） |
| `--workflow` | 故事工作流 id（覆盖配置） |
| `--config` | 推荐配置 id（默认 `defaultConfiguration`） |
| `--meta pure` | 模拟纯元指令：来源 `task`，使用任务工作流 |
| `--meta mixed` | 模拟混合元指令：来源 `story` |
| `--enable a,b` / `--disable c` | 打开 / 关闭模组（逗号分隔的 id） |
| `--user-modules <file>` | 合并用户模组文件（[user-modules.md](user-modules.md#导入导出格式)） |
| `--json` | 输出 JSON：`vars`、`buildVars`、`skipped`、`modules`、`structures`、`stages`、`regions`，以及 `schema`（structured）或 `tools`（工具传输） |
| `--out <file>` | 写入文件而不是标准输出 |

文本输出的开头是摘要（预设、传输方式与写法、工作流、配置、生效模组、构建变量、结构、阶段），然后按 `regions` 顺序列出每个区域的消息，最后是 schema / 工具。

预览与真实运行的差别：

- ST 宏（`{{user}}`、`{{char}}`……）不展开；
- 运行时数据为空：记忆、总结、最近元指令、聊天文本；`memory.*`、`recap.present`、`chat.group`、`meta.recent` 为假；用户模组的关键词规则看到空文本；
- 元指令文本为占位符 `(meta text)`；`meta.task` 为 `null`。

常用的预览矩阵：每种传输方式各一次；`--meta pure`；每个推荐配置；关键模组开 / 关。

## ap init

从内置模板创建新的预设仓库：

```bash
npx advanced-preset@latest init my-preset --name "My Preset" --author me
```

- `--id`：预设 id，默认由目录名生成（小写 kebab-case）；
- `--name`：显示名，默认同 id；`--author`：作者；
- 目标目录已有 `preset.yaml` 时拒绝执行；
- 生成的仓库包含示例模组与工作流、`package.json`（`build` / `check` / `dev` / `preview` / `skills` 脚本，以 `^<当前版本>` 依赖 npm 上的 CLI）、GitHub Actions、`LICENSE`（CC0 1.0，可自行替换）、`.agents/skills/` 中的 Agent Skills 与 `AGENTS.md`。

## ap skills

管理随 CLI 发布的 **Agent Skills**——教 AI Agent 设计、编写和测试 AP 预设的说明书与参考资料（包括本参考文档的副本）。

```bash
ap skills list                                # 列出内置技能及说明
ap skills install                             # 安装 / 更新到 ./.agents/skills/
ap skills install ../my-preset                # 安装到其他仓库
ap skills install --target .claude/skills     # 安装到其他目录（相对于 dir）
```

- 内置技能：`ap-preset-design`（设计）、`ap-preset-authoring`（编写，含本参考文档）、`ap-preset-testing`（测试与发布）；
- 每个技能是一个目录（`SKILL.md` + `references/` 等），安装时**整体替换**同名目录，并报告 `installed` / `updated` / `up to date`；
- 只会写入内置技能的同名目录，目录中的其他技能不受影响；
- 升级 CLI 后再运行一次 `ap skills install`，即可让技能与新版本的格式保持一致。不要直接修改安装的技能文件（会被下次安装覆盖）；需要定制时，另建一个不同名字的技能。
