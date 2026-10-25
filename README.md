# scoop-bucket

个人 Scoop 软件库（bucket）——收录官方 bucket 里没有的软件，多机复用。

## 接入

```bash
# 添加 bucket（需 Git）
scoop bucket add mine https://github.com/tinkerJ/scoop-bucket

# 安装库中软件
scoop install mine/zditor
scoop install mine/prompt-optimizer

# 不建 bucket，单次直装
scoop install https://raw.githubusercontent.com/tinkerJ/scoop-bucket/main/bucket/zditor.json
```

## 收录软件

| 软件 | 当前版本 | 说明 | 上游 |
|---|---|---|---|
| [zditor](https://zditor.com) | 1.9.9 | Tauri 所见即所得 Markdown 编辑器，写作+绘图一体 | `zditor-v*` tag |
| [prompt-optimizer](https://github.com/linshenkx/prompt-optimizer) | 2.12.0 | AI 提示词优化器，多模型接入（Electron 便携版） | `v*` tag |

全部启用 `checkver: github` + `autoupdate`，`scoop update *` 即可自动跟进新版本。

## 日常维护

```bash
# 新增/修改软件：编辑 bucket/ 下 manifest，然后推送
git add . && git commit -m "add xxx" && git push

# 各机器同步并更新（自动拉取 bucket 最新 manifest）
scoop update *

# 仅同步 bucket 索引，不更新软件
scoop bucket update mine
```

## 新增 manifest 的要点

- 文件放 `bucket/` 目录，**文件名 = 安装命令里的软件名**
- 必填：`version`、`url`、`hash`（`Get-FileHash <文件>` 计算 SHA256）
- 加 `checkver` / `autoupdate` 才能自动跟新；配合 GitHub Actions（ScoopChecker）可无人值守
- **Electron 应用优先找 `*-win-x64.zip` 便携包**，manifest 最简（纯 url + hash）；zip 根目录即应用本体时无需 `extract_dir`
- NSIS 安装包可加 `#/dl.7z` 后缀让 scoop 当 7z 自动解包、免执行安装器
- 嵌套压缩包（如 electron-builder 的 `app-64.7z`）藏在 `$PLUGINSDIR` 子目录，installer.script 路径需写全并转义：`"$dir\\`$PLUGINSDIR\\app-64.7z"`
- post_install 清理 `$PLUGINSDIR` 等内建变量必须用反引号转义，否则变量展开为空有误删风险
- 写 JSON 时在编辑器挂上本仓库根的 `bucket.json` schema，可获得字段补全与校验
