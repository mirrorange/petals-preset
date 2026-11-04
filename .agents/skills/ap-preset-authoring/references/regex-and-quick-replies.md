# 正则脚本与快速回复

预设可以携带 SillyTavern 自己的两种资源：**正则脚本**（ST 正则扩展的脚本）与**快速回复集**（ST Quick Reply 扩展的集合）。它们以文件形式写在预设仓库里，字段名与 ST 完全相同，表达力与在酒馆界面中创建的一致；从 ST 导出的 JSON 文件可以原样放进来。

```
regex-scripts/<名称>.yaml|yml|json   正则脚本：一个脚本，或一个脚本列表（产物字段 regexScripts）
quick-replies/<id>.yaml|yml|json     快速回复集：每个文件一个集合（产物字段 quickReplySets）
```

两个目录都只认下一层；JSON 文件按 YAML 解析（JSON 是 YAML 的子集）。都需要 Advanced Preset 1.10.0 以上，更早的版本忽略它们（`ap check` 在 `minAdvancedPreset` 低于 1.10.0 时警告）。

> 不要和清单里的 [`regex`](manifest.md#正则标记-regex) 混淆：那是 AP 的**正则标记**，只在生成前匹配文本、得到条件变量 `flags.<id>`，不改写任何文字。这里的正则脚本是 ST 的正则扩展执行的替换规则。

## 正则脚本 `regex-scripts/`

```yaml
# regex-scripts/10-hide-thinking.yaml
scriptName: Hide thinking
findRegex: /<thinking>[\s\S]*?<\/thinking>/gi
replaceString: ""
placement: [ai-output]
markdownOnly: true
```

一个文件可以是一个脚本（映射），也可以是一个脚本列表（ST 批量导出的形式）。

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `scriptName` | 文件名（列表中为 `文件名 序号`） | 脚本名称 |
| `findRegex` | — | **必填**。`/pattern/flags` 或不带斜杠的正则，与 ST 编辑器中的写法相同 |
| `replaceString` | `''` | 替换文本：`$1`、`{{match}}` 与 ST 宏照常使用 |
| `trimStrings` | `[]` | 从匹配中去掉的字符串（字符串列表；也可以写成多行字符串，每行一项） |
| `placement` | — | 作用位置列表：`user-input`、`ai-output`、`slash-command`、`world-info`、`reasoning`，或 ST 的数字 `1` `2` `3` `5` `6` |
| `disabled` | `false` | 停用 |
| `markdownOnly` | `false` | 只改变聊天显示（ST 的“仅格式显示”） |
| `promptOnly` | `false` | 只改变发出的提示词（ST 的“仅格式提示词”） |
| `runOnEdit` | `true` | 编辑消息时运行 |
| `substituteRegex` | `none` | `findRegex` 中的宏替换：`none` / `raw` / `escaped`（或 `0` / `1` / `2`） |
| `minDepth` / `maxDepth` | 空 | 深度范围（整数，`-1` 起；留空为不限） |

ST 导出文件中的 `id` 会被忽略：AP 按预设 id 与脚本所在的文件（及列表中的位置）生成稳定的 UUID，同一位置的脚本在各版本中 id 不变，ST 的正则预设能持续识别它。

**运行期**：脚本写入 Carrier 的 `extensions.regex_scripts`，成为 ST 的**预设正则**，只在选中该 AP 预设的 Carrier 时生效。ST 第一次遇到带正则的预设时会询问是否允许（与导入任何带正则的预设相同）；用户可以在 ST 正则扩展的“预设正则”中查看、开关脚本。这些修改与其他 Carrier 修改一样，保留到预设更新或重新安装为止。

- 脚本按文件路径排序后依次执行，列表内按列表顺序；需要固定顺序时给文件名加数字前缀（`10-…`、`20-…`）。
- `{{ap::…}}` 宏**不会**在正则脚本中展开（脚本由 ST 执行）；需要角色名等请用 ST 宏并设置 `substituteRegex`。
- 正则只作用于 ST 自己处理的文字：AI 输出是 AP 交还给 ST 的终结正文；`promptOnly` 作用于 ST 组装的聊天记录。

`ap check`：缺少 `findRegex`、正则无法编译（`substituteRegex: none` 时）、未知 `placement`、字段类型不对时报错；未知字段、没有 `placement`（脚本永远不会运行）、使用了 `{{ap::…}}`、多个脚本同名时警告。

## 快速回复集 `quick-replies/`

```yaml
# quick-replies/petals.yaml
qrList:
  - id: meta
    label: Meta
    icon: fa-at
    title: Talk to {{ap::name::narrator}}
    message: "/setinput {{input}} @{{ap::name::narrator}}"
  - id: summary
    label: Summary
    icon: fa-scroll
    message: |-
      @{{ap::name::narrator}} Please summarize the story so far.
      {{input}}
```

文件名（小写 kebab-case）是集合 id，用于翻译与上下文菜单引用。

### 集合字段

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `name` | `AP · <预设名>` | ST 中的集合名（集合的身份，不翻译）。一个预设有多个集合时，至多一个可以用默认名 |
| `global` | `true` | 预设启用时加入 ST 的全局快速回复集列表。`false`：只写入集合（例如只作为上下文菜单使用的子集合） |
| `visible` | `true` | 加入全局列表时是否显示按钮（列表项的“显示按钮”） |
| `disableSend` | `false` | 不发送：把消息放进输入框而不是发送 |
| `placeBeforeInput` | `false` | 放在输入内容之前 |
| `injectInput` | `false` | 自动拼接输入框中已有的内容 |
| `color` | `transparent` | 按钮颜色 |
| `onlyBorderColor` | `false` | 只给边框上色 |
| `qrList` | — | **必填**。快速回复列表，见下 |

ST 导出文件中的 `version`、`idIndex` 被忽略。

### 快速回复字段

| 字段 | 默认 | 说明 |
| --- | --- | --- |
| `id` | 序号（`1`、`2`……） | 在集合内唯一，用于翻译。ST 导出文件中的数字 id 照常可用 |
| `label` | `''` | 按钮文字 |
| `icon` | `''` | Font Awesome 图标类名，例如 `fa-at`（ST 自己加 `fa-solid`，不要写两个类名） |
| `showLabel` | `false` | 有图标时仍显示文字 |
| `title` | `''` | 悬停提示 |
| `message` | `''` | 发送的文字；以 `/` 开头时作为 STscript 执行 |
| `contextList` | `[]` | 右键菜单：集合列表，每项为 `集合` 或 `{ set: 集合, isChained: false }`。`集合` 是本预设的集合 id（编译为它的 ST 名称），或用户 ST 中的集合名 |
| `isHidden` | `false` | 隐藏按钮（仍可由命令或上下文菜单调用） |
| `executeOnStartup` `executeOnUser` `executeOnAi` `executeOnChatChange` `executeOnGroupMemberDraft` `executeOnNewChat` `executeBeforeGeneration` | `false` | 自动执行的时机 |
| `automationId` | `''` | 世界书条目以此 id 触发执行 |
| `preventAutoExecute` | `true` | 被自动执行的命令触发时不再自动执行（防止循环） |

`label`、`title`、`message` 中可以使用 `{{ap::name::<角色>}}`（AP 写入集合时按当前配置的身份名展开，与[元指令](meta-and-tasks.md)触发词相同）；其他 `{{ap::…}}` 宏不可用，ST 宏（`{{user}}`、`{{input}}`……）与 STscript 留给 ST。

### 运行期

- 选中该预设的 Carrier（且 AP 已启用）时，AP 把预设的集合写入 ST 的 Quick Reply 扩展，并把 `global` 的集合加入全局快速回复集列表；切换到其他预设（或停用 AP）时从全局列表移除。集合本身保留，因此用户放在聊天、角色列表或上下文菜单中的引用继续有效；
- 集合由 AP 管理：只有 AP 将要写入的内容变化时（预设更新、身份名、界面语言变化）才重写，因此在 ST 编辑器中对集合的修改保留到那时为止。想长期自定义，请复制一份；
- 用户自己已有同名集合时，AP **不会覆盖**，提示一次并跳过该集合；
- 预设不再声明某个集合（或预设被删除）时，AP 删除它创建的那个集合；
- 无需点击即会执行的快速回复（任一 `executeOn…` 开关或 `automationId`）安装时需要用户确认，与[模板](templates.md)的信任相同；内容不变的更新不再询问。

### 翻译

`locales/<lang>.yaml` 的 `quickReplySets` 段按集合 id、快速回复 id 翻译 `label`、`title`、`message`（见 [locales.md](locales.md)）。快速回复是用户发送的文字，因此 `message` 也可以翻译——这是“翻译不影响发给模型的文字”的唯一例外：

```yaml
quickReplySets:
  petals:
    qrList:
      summary: { label: 总结, message: "@{{ap::name::narrator}} 请总结目前的故事。\n{{input}}" }
```

### 常见写法

| 目的 | 写法 |
| --- | --- |
| 发送一条消息，保留输入框中的草稿作为补充 | `message: "…\n{{input}}"`（集合 `injectInput: false`：`{{input}}` 由 ST 换成当前输入框内容） |
| 在输入框末尾追加文字、不发送 | `message: "/setinput {{input}} 追加的文字"` |
| 整个集合都只填入输入框 | 集合 `disableSend: true`，配合 `injectInput` / `placeBeforeInput` |

`ap check`：文件名不是 kebab-case、`qrList` 不是列表、快速回复 id 重复、`icon` 含空格、字段类型不对、使用了 `{{ap::name::…}}` 以外的 AP 宏或未知角色、两个集合的 ST 名称相同时报错；未知字段、没有 `label` 也没有 `icon`、上下文菜单引用了预设之外的集合时警告。`ap check` 的统计中会显示脚本数与集合数。
