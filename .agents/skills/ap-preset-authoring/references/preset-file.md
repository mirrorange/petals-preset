# 预设文件与安装引导

`ap build` 写出的 `dist/<id>.apreset.json` 是**预设文件**：一个标准的 SillyTavern 对话补全预设（Carrier 的形态），AP 的全部数据（产物，见 [artifact-and-runtime.md](artifact-and-runtime.md#产物)）放在它的额外字段里。因此无论是否安装了 AP，ST 都能把它当作普通预设导入。未安装 AP 时，文件里的一条提示词让模型引导用户安装 AP；装有[酒馆助手](https://github.com/N0VI028/JS-Slash-Runner)时，文件附带的安装脚本还可以直接替用户安装或更新 AP。

需要 AP 1.10.0 或以上版本读取。更早的版本只认裸产物，导入预设文件时会把它交给 ST 当作普通预设（此时安装脚本会提示更新 AP）。AP 仍然可以读取裸产物（1.10.0 以前 `ap build` 的输出、已发布预设的旧文件）。

## 文件结构

```jsonc
{
  // 默认配置的采样参数（temperature、top_p……），ST 选中预设时应用
  "temperature": 1,
  // Carrier 的提示词，外加一条安装提示
  "prompts": [ /* … */, { "identifier": "ap_install_notice", "content": "⟦ap:install-notice⟧\n…" } ],
  // Carrier 的顺序：区域锚点关闭，安装提示在最后
  "prompt_order": [{ "character_id": 100000, "order": [/* … */] }, { "character_id": 100001, "order": [/* … */] }],
  "extensions": {
    // 可选：安装脚本（酒馆助手的预设脚本）
    "tavern_helper": { "scripts": [{ "type": "script", "id": "6f1d6c2e-5a4b-4c3e-9a7d-a9e5c0f1b2d3", "name": "Advanced Preset Installer", /* … */ }], "variables": {} },
    // AP 的数据
    "advanced_preset_file": { "format": 1, "artifact": { /* 产物 */ } }
  }
}
```

- **提示词**：Carrier 的提示词原样保留，但区域锚点（内容为 `⟦ap:region:<id>⟧` 的条目）在顺序中关闭——没有 AP 时它们只会把标记发给模型。ST 自带的条目（角色描述、世界书、聊天记录等）照常启用。
- **安装提示** `ap_install_notice`：放在顺序最后的系统提示词，内容以标记 `⟦ap:install-notice⟧` 开头。它告诉模型：当前预设是 AP 预设，需要 Advanced Preset 插件而用户尚未安装（或未启用）；本轮不要继续故事，而是用用户的语言简要说明，并引导用户在「扩展」面板中用以下任一地址安装 AP、刷新页面，装有酒馆助手时启用预设脚本即可自动安装。
- **安装脚本**：见下文。不附带时 `extensions.tavern_helper` 不出现。
- **`advanced_preset_file`**：`format` 为预设文件格式（目前为 1），`artifact` 为完整产物。

## 镜像

安装提示与安装脚本按顺序使用三个地址：

1. `https://github.com/mirrorange/advanced-preset`
2. `https://gitlab.com/mirrorange/advanced-preset`
3. `https://codeberg.org/mirrorange/advanced-preset`

三者内容相同，ST 都会把 AP 安装到同名文件夹 `advanced-preset`。

## 安装脚本

安装脚本是一个酒馆助手预设脚本（固定 id `6f1d6c2e-5a4b-4c3e-9a7d-a9e5c0f1b2d3`）。用户导入文件后 ST 会选中它，酒馆助手随即询问是否启用预设中的脚本；启用后脚本运行：

| 情况 | 脚本的行为 |
| --- | --- |
| 正在运行的 AP 不低于所需版本 | 什么也不做（AP 会自行接管预设文件，见下文） |
| 正在运行的 AP 版本过低 | 询问是否更新，确认后调用 `AdvancedPreset.update()`（扩展有变化时 AP 自行刷新页面）；已经是能获取的最新版本时提示稍后再试或手动更新 |
| AP 已安装但已停用 | 询问是否启用，确认后启用并刷新页面 |
| AP 已安装但没有运行（其他原因） | 询问是否更新，确认后通过 ST 更新扩展，再询问是否刷新页面 |
| 未安装 | 询问是否安装，确认后依次从三个镜像安装（安装到当前用户的扩展目录），成功后询问是否刷新页面 |

- **所需版本**：预设的 `minAdvancedPreset` 与 1.10.0 中较高者，写在脚本中。
- 所有镜像都失败（或更新失败）时，弹窗请用户检查网络连接后刷新重试，或按步骤手动安装 AP，并列出三个地址。
- 用户选择「稍后」时，在本次浏览器会话中不再询问（刷新页面后仍不询问，关闭标签页后重新询问）。
- 文字跟随 ST 的界面语言（中文 / 英文）。
- 脚本只使用 ST 的扩展接口（`/api/extensions/discover`、`install`、`update`）、ST 的弹窗与 `window.AdvancedPreset`，不调用酒馆助手的函数。
- 是否运行完全由酒馆助手的启用询问决定；未装酒馆助手时脚本不会运行，只有安装提示起作用。

### 是否附带

| 来源 | 默认 | 如何改变 |
| --- | --- | --- |
| `ap build` / `ap dev` 写出的 `dist/` 文件 | 附带 | 命令行 `--no-installer` / `--installer`，或清单 `build.installer`（见 [manifest.md](manifest.md#构建选项-build)）；命令行优先 |
| AP 面板「预设」页导出、选中 Carrier 时 ST 的导出按钮 | 附带 | 导出对话框中的「附带 Advanced Preset 安装脚本」 |

`ap dev` 的 `GET /preset` 端点仍提供裸产物（AP 的开发源只读取产物）。扩展内置的预设以裸产物保存。

## 安装了 AP 时

- **导入**：ST 的预设导入按钮、AP 面板的「导入文件 / 导入链接」都接受预设文件，按产物安装（与[安装与更新](artifact-and-runtime.md#安装与更新)相同的检查与信任确认），选中 Carrier 并打开面板。预设文件不会作为普通 ST 预设导入。
- **接管已在 ST 中的预设文件**：未装 AP 时导入的文件是 ST 预设列表中的一个普通预设（名称为文件名，如 `petals.apreset`）。AP 在启动时（ST 就绪后，与随预设模组的同步一起在后台进行），以及这样的预设被选中时，逐个处理：
  1. 读取其中的产物。格式不受支持，或 `minAdvancedPreset` 高于当前 AP 时跳过（由安装脚本提示更新）；
  2. 尚未安装、或文件中的版本不低于已安装版本且构建不同时，按产物安装（同样会询问模板等的信任）；已安装同一构建或更新的版本时不安装（不降级）；
  3. 确保 Carrier 存在并与产物一致；文件当时被选中时改为选中 Carrier；
  4. 删除该 ST 预设（文件名恰好与 Carrier 同名时，它已被 Carrier 覆盖，不删除），并提示「已根据导入的预设完成设置」。
  用户在信任确认中取消时，该文件保留，AP 记住它携带的构建（按 ST 预设名称），之后不再询问；文件换成其他构建后重新处理。
- **安装提示**：AP 运行时，任何内容含 `⟦ap:install-notice⟧` 的提示词消息都不会发给模型，即使尚未接管的预设文件正被选中。
- **安装脚本**：Carrier 永远不包含安装脚本（重建时按 id 去掉，包括放在文件夹中的副本）。接管后选中的是 Carrier（名称不同），酒馆助手随之改读 Carrier 的脚本。

## 发布

把 `dist/<id>.apreset.json` 提交到仓库或作为 Release 附件即可：已安装 AP 的用户照常用导入文件 / 链接安装；没有 AP 的用户可以直接在 ST 的对话补全预设中导入它，再按提示（或由安装脚本）安装 AP。
