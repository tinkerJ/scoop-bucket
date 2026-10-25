# my-scoop-bucket

个人 Scoop 软件库（bucket）——收录官方 bucket 里没有的软件，多机复用。

## 接入方式

**其他机器接入（推荐，需 Git）：**

```bash
scoop bucket add mine https://github.com/tinkerJ/scoop-bucket
scoop install mine/zditor
```

**本机临时使用（直接指向本地目录）：**

```bash
scoop bucket add mine "C:\Users\wda20\WorkBuddy\2026-10-24-15-29-18\my-scoop-bucket"
```

**单次安装（不建 bucket）：**

```bash
scoop install https://raw.githubusercontent.com/tinkerJ/scoop-bucket/main/bucket/zditor.json
```

## 日常维护

```bash
# 新增软件：写好 manifest 后推送
git add bucket/xxx.json && git commit -m "add xxx" && git push

# 各机器更新软件（会自动拉取 bucket 最新 manifest）
scoop update *
```

## 新增 manifest 的要点

- 文件放 `bucket/` 目录，文件名 = 安装命令里的软件名
- 必填：`version`、`url`、`hash`（用 `Get-FileHash <文件>` 计算）
- 推荐加 `checkver` / `autoupdate`，配合 GitHub Actions（如 ScoopInstaller/ScoopChecker）可全自动跟新
- NSIS 安装包可加 `#/dl.7z` 后缀让 scoop 直接解包免执行安装器
- 写 JSON 时在编辑器里挂上本仓库 `bucket.json` 的 schema 可获得字段补全与校验

## 收录软件

| 软件 | 版本策略 | 说明 |
|---|---|---|
| zditor | checkver: github | Tauri 所见即所得 Markdown 编辑器（NSIS 解包安装） |
| prompt-optimizer | checkver: github | AI 提示词优化器（Electron 便携 zip 版） |
