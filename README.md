# scoop-bucket

个人 Scoop 软件库（bucket）——收录官方 bucket 里没有的软件，多机复用。

## 接入

```bash
# 添加 bucket（需 Git）
scoop bucket add mine https://github.com/tinkerJ/scoop-bucket

# 安装库中软件
scoop install mine/zditor
scoop install mine/prompt-optimizer
scoop install mine/workbuddy

# 不建 bucket，单次直装
scoop install https://raw.githubusercontent.com/tinkerJ/scoop-bucket/main/bucket/zditor.json
```

## 收录软件

| 软件 | 当前版本 | 说明 | 上游 |
|---|---|---|---|
| [zditor](https://zditor.com) | 1.9.9 | Tauri 所见即所得 Markdown 编辑器，写作+绘图一体 | `zditor-v*` tag |
| [prompt-optimizer](https://github.com/linshenkx/prompt-optimizer) | 2.12.0 | AI 提示词优化器，多模型接入（Electron 便携版） | `v*` tag |
| [workbuddy](https://www.workbuddy.cn) | 5.9.2 | 腾讯 WorkBuddy 中国版，全场景 AI 办公客户端（Electron/NSIS 解包） | 官方历史版本页 |

全部启用 `checkver` + `autoupdate`，`scoop update *` 即可自动跟进新版本。

## workbuddy 补充说明

> 官方 extras / spc bucket 里也有同名 `workbuddy`，但那是**海外版**（`workbuddy.ai`，主程序 `WorkBuddyAI.exe`，5.8.4）。本库这份是**中国版**（`workbuddy.cn`，主程序 `WorkBuddy.exe`，5.9.2），两者安装后互不覆盖，可共存。

- **解包路径**：官方只发 `WorkBuddy-win32-x64-user-<版本>.<构建号>-<hash>.exe`（NSIS-3），本体在内层 `$PLUGINSDIR\app-64.7z`（约 449 MB，解开 1.3 GB），manifest 里 `Expand-7zipArchive` 后就地清理 `$PLUGINSDIR` 与卸载器 `$R0`，全程不跑安装器、不写注册表。
- **自动更新**：`checkver` 从官方[历史版本页](https://www.codebuddy.cn/docs/workbuddy/Download-History)抓最新 Windows 链接（页面按版本倒序，首个匹配即最新），构建号与 hash 用命名组 `$matchBuild` / `$matchHash` 拼进下载地址；官方不在发布页提供校验值，scoop 会在 autoupdate 时自行下载算 hash（单独多下一次 429 MB）。
- **另一条更新源**：官方更新接口 `https://www.workbuddy.cn/v2/update?platform=workbuddy-win32-x64-user` 直接返回 `version` / `url` / `sha256hash`，可用 `jsonpath` 取用免去重算 hash；但实测它滞后于发布页（接口 5.8.3 / 发布页 5.9.2），故本 manifest 以发布页为准。
- **客户端自带「检查更新」**会覆盖自己所在的安装目录，用 scoop 管理时请以 `scoop update workbuddy` 为准。
- **不注册的东西**：`workbuddy://` 协议、Office/文档文件关联、`WorkBuddyShareTarget.msix` 共享目标、开机自启与 RepairApp 服务。用不到可忽略；官方版还装着时无需处理（HKLM 里的关联优先生效）。
- 官方版卸载后想保留协议唤起，把下面片段里的路径换成 scoop 目录执行一次（写入 HKCU，随用户配置）：

```powershell
$exe = Join-Path (scoop prefix workbuddy) 'WorkBuddy.exe'
New-Item 'HKCU:\Software\Classes\workbuddy\shell\open\command' -Force | Out-Null
Set-ItemProperty 'HKCU:\Software\Classes\workbuddy' '(default)' 'URL:WorkBuddy Protocol'
New-ItemProperty 'HKCU:\Software\Classes\workbuddy' 'URL Protocol' '' -PropertyType String -Force | Out-Null
Set-ItemProperty 'HKCU:\Software\Classes\workbuddy\shell\open\command' '(default)' "`"$exe`" `"%1`""
```

- **配置目录**：默认 `%USERPROFILE%\.workbuddy`，可用环境变量 `WORKBUDDY_CONFIG_DIR` 改（见仓库外的迁移脚本思路：目录 junction）。

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
- `checkver` 正则可用命名组（`(?<build>\\d+)`），`autoupdate` 里按**首字母大写**引用：`$matchBuild`。URL 里带构建号/随机 hash 的软件全靠这招
- `autoupdate` 不写 `hash` 时，scoop 会下载目标文件自行算 SHA256 并写回 manifest（官方不发校验值时的兜底，代价是多下一份安装包）
- 写 JSON 时在编辑器挂上本仓库根的 `bucket.json` schema，可获得字段补全与校验
