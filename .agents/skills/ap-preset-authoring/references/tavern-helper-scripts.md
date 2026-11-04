# 酒馆助手脚本

预设可以携带[酒馆助手](https://github.com/N0VI028/JS-Slash-Runner)（JS-Slash-Runner / Tavern Helper）的**预设脚本**。脚本以文件形式写在预设仓库里，字段名与酒馆助手完全相同；从酒馆助手导出的脚本或文件夹 JSON 可以原样放进来。

```
tavern-helper-scripts/<名称>.yaml|yml|json   一个脚本、一个文件夹，或它们的列表（产物字段 tavernHelperScripts）
tavern-helper-scripts/**                     其他文件：脚本代码，由 content 引用
```

需要 Advanced Preset 1.10.0 以上，更早的版本忽略它们（`ap check` 在 `minAdvancedPreset` 低于 1.10.0 时警告）。用户需要自己安装酒馆助手；没有酒馆助手时脚本只是 Carrier 中的数据，不会运行。

## AP 做什么，不做什么

- AP 把脚本写入 Carrier 的 `extensions.tavern_helper.scripts`——酒馆助手读取预设脚本的位置——之后由酒馆助手自己识别、管理与运行。
- **信任与启用归酒馆助手**：酒馆助手第一次遇到带脚本的预设时会询问是否启用（与导入任何带脚本的预设相同）；用户也可以之后在酒馆助手「脚本库 → 预设脚本」中启用。AP 不会替用户信任或启用脚本，也不会在自己的安装确认中再询问一次。
- AP 只写数据，不调用酒馆助手的代码。

## 文件格式

```yaml
# tavern-helper-scripts/10-dice.yaml
name: Dice
content: ./tavern-helper-scripts/dice/index.js
info: 在输入框上方加一个掷骰按钮。
button:
  buttons: [掷骰]
```

```yaml
# tavern-helper-scripts/20-tools.yaml：一个文件夹
type: folder
name: Tools
icon: fa-solid fa-toolbox
scripts:
  - name: Status panel
    content: ./tavern-helper-scripts/tools/status.js
  - name: Debug
    content: "console.log('loaded');"
    enabled: false
```

一个文件可以是一个脚本、一个文件夹（`type: folder`，或带 `scripts` 列表），也可以是它们的列表。文件夹中不能再放文件夹（与酒馆助手相同）。

### 脚本字段

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `type` | `script` | 可省略 |
| `name` | 文件名（列表 / 文件夹中为 `文件名 序号`） | 脚本名称 |
| `content` | — | **必填**。脚本代码，或 `./tavern-helper-scripts/…` 下的文件路径（文件原样读入） |
| `enabled` | `true` | 脚本自身的开关。注意与酒馆助手不同：酒馆助手新建脚本默认关闭，这里默认打开，因为预设携带的脚本通常就是要运行的；整个预设的脚本是否运行仍由酒馆助手的信任询问决定 |
| `info` | `''` | 作者备注，显示在酒馆助手中 |
| `button` | `{ enabled: true, buttons: [] }` | 脚本按钮：`buttons` 每项为 `名称` 或 `{ name, visible }`（`visible` 默认 `true`） |
| `data` | `{}` | 脚本变量的初始值 |
| `export_with` | `{ data: true, button: true }` | 酒馆助手导出脚本时是否带上变量与按钮 |

### 文件夹字段

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `type` | — | `folder`（有 `scripts` 列表时可省略） |
| `name` | 文件名 | 文件夹名称 |
| `enabled` | `true` | 文件夹开关 |
| `icon` | `fa-solid fa-folder` | Font Awesome 图标类名 |
| `color` | 酒馆助手的默认颜色 | 图标颜色 |
| `scripts` | — | **必填**。脚本列表 |

导出文件中的 `id` 会被忽略：AP 按预设 id 与脚本所在的文件（及列表、文件夹中的位置）生成稳定的 UUID，同一位置的脚本在各版本中 id 不变。重命名或移动文件会改变 id，脚本在用户那里保存的开关与变量随之丢失。

## 运行期

- 选中该预设的 Carrier 时，酒馆助手读取其中的脚本：第一次会询问是否启用；用户启用后，切换到其他预设时脚本停止，切换回来时继续运行。
- 脚本、文件夹与预设变量属于酒馆助手管理的数据。**预设更新或重新安装**（Carrier 重建）时，AP 只替换预设自己的脚本，并保留：
  - 每个脚本与文件夹的开关（按 id 对应）；
  - 脚本的 `data`（用户与脚本写入的值优先，预设新增的键补上）；
  - 预设变量与酒馆助手的其他字段；
  - 用户自己添加到这个预设中的脚本与文件夹（包括放进预设文件夹里的）。
  预设不再携带的脚本会被删除。脚本代码、名称、按钮等其余字段以新版本为准。
- 酒馆助手只在**切换预设**时读取预设脚本，正在使用的 Carrier 被重写（例如在使用中更新预设、`ap dev` 重新构建）时它不会察觉。此时如果脚本有变化，AP 会提示刷新页面。在刷新之前，酒馆助手仍运行旧脚本，并会在保存变量时把旧脚本写回 Carrier；AP 在下一次同步（例如下次打开页面）时发现脚本恰好回到了上一个版本的代码，就重新写入新代码并再次提示刷新。用户在酒馆助手中自己改过的代码不会被覆盖（直到下一次预设更新）。
- 导出 AP 预设（`.apreset.json`）时，脚本按预设源中的内容导出，不含用户的开关与变量（它们在产物中；预设文件本身的 `extensions.tavern_helper` 只放 AP 的安装脚本，见 [preset-file.md](preset-file.md)）。
- 预设文件的安装脚本不属于预设的脚本：Carrier 中不会出现它。

`ap check`：缺少 `content`、`content` 指向的文件不存在或不在 `tavern-helper-scripts/` 下、`type` 未知、文件夹中有文件夹、字段类型不对时报错；未知字段时警告。`ap check` 的统计中会显示脚本数。
