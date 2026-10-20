# 发布流程

## 版本号

`preset.yaml` 的 `version` 用语义化版本：

- 措辞、错字：修订号 +1（`1.2.0` → `1.2.1`）；
- 新模组、新功能：次版本 +1（`1.2.1` → `1.3.0`）；
- 改结构、改 id：主版本 +1（`1.3.0` → `2.0.0`），并写好 `migrations`。

`package.json` 的 `version` 可以与之保持一致（可选）。用到了新版 AP 才有的功能时，提高 `minAdvancedPreset`。

## 步骤

1. 确认工作区干净，用户同意发布的版本号。
2. 修改 `preset.yaml` 的 `version`。
3. `npx ap check --strict`，再按 `ap-preset-testing` 跑一遍预览矩阵。
4. `npm run build`，生成 `dist/<id>.apreset.json`。
5. **把 `dist/` 一起提交**（模板默认不忽略 `dist/`）。提交信息写清版本与改动。
6. 推送到 GitHub。模板自带的 `.github/workflows/build.yml` 会检查并构建。
7. （可选）打标签并推送：`git tag v1.3.0 && git push origin v1.3.0`。Actions 会创建 GitHub Release，附上 `.apreset.json`。

推送、打标签前先征得用户同意。

## 用户如何安装与更新

- **导入链接**：AP 面板 → 预设 → 导入链接，填 `https://github.com/<用户>/<仓库>/blob/main/dist/<id>.apreset.json`（AP 自动换成 raw 地址）。这个地址始终指向最新提交。
- **从来源更新**：通过链接安装的预设可以一键更新；用户的配置修改、我的模组、记忆都会保留。
- **导入文件**：从 Release 页面下载 `.apreset.json` 后导入。

> GitHub Release 附件的下载地址不能用于“导入链接”：浏览器的跨域限制会拒绝读取。分享链接请用仓库里的文件地址。

## 更新 AP 工具链与技能

1. `npm install -D advanced-preset@latest`（或把 `package.json` 中 `advanced-preset` 的版本范围改成新版本后 `npm install`）。旧仓库若还写着 GitHub Release 的 tarball 地址，直接用这条命令换成 npm 版本。
2. `npx ap skills install`，把 `.agents/skills/ap-*` 更新到与新 CLI 匹配的版本（另装在 `.claude/skills` 的也要更新：`npx ap skills install --target .claude/skills`）。
3. `npx ap check --strict`：新版本可能新增警告。
