# gitcode-sync

这个仓库通过 GitHub Actions 定时或手动执行同步任务，把 GitCode 仓库同步到 GitHub 仓库。

当前同步方向是：

- `https://gitcode.com/Ascend/msprof` -> `https://github.com/kali20gakki/msprof`
- `https://gitcode.com/Ascend/msprof-analyze` -> `https://github.com/kali20gakki/msprof-analyze`
- `https://gitcode.com/Ascend/msagent` -> `https://github.com/kali20gakki/msAgent`

## 同步原则

- 以 GitCode 代码为准。
- GitHub 目标仓会被强制覆盖到和 GitCode 保持一致。
- workflow 只同步 `branches` 和 `tags`，并对它们执行强制覆盖与清理。
- 目标仓中仅存在于 GitHub 的分支或标签，可能会被删除。

## 触发方式

### 1. 自动触发

工作流文件是 [sync-matrix.yml](C:/Project/gitcode-sync/.github/workflows/sync-matrix.yml)。

当前已配置定时任务：

- `0 3 * * *`

这表示每天 `UTC 03:00` 自动执行一次。

按北京时间计算，就是每天 `11:00` 自动同步一次。

### 2. 手动触发

GitHub 页面操作步骤：

1. 打开仓库 `Actions` 页面。
2. 选择工作流 `Batch sync GitCode to GitHub`。
3. 点击 `Run workflow`。
4. 选择分支后点击确认运行。

如果你已经把这个仓库推到 GitHub，也可以直接打开对应仓库的 Actions 页面手动运行。

## 必要配置

在这个同步仓库对应的 GitHub 仓库里，进入 `Settings -> Secrets and variables -> Actions`，配置以下 secrets：

- `GITCODE_TOKEN`
  用于拉取 GitCode 私有或受限仓库。如果源仓公开，理论上也建议保留，避免后续权限变化。
- `GH_PAT`
  GitHub Personal Access Token，用于推送到目标 GitHub 仓库，并调用 GitHub API 更新默认分支。

`GH_PAT` 至少需要：

- 对目标仓库的写权限
- 修改仓库设置中默认分支的权限

## 同步配置写法

工作流里的 `REPOS` 支持两种格式：

### 1. 源仓和目标仓同名

```yaml
REPOS: |
  Ascend/msprof
```

这会同步到：

```text
kali20gakki/msprof
```

### 2. 源仓和目标仓不同名

```yaml
REPOS: |
  Ascend/msagent|msAgent
```

这会同步到：

```text
kali20gakki/msAgent
```

格式说明：

- `owner/repo`：同步到 `TARGET_ORG/repo`
- `owner/source|target`：同步到 `TARGET_ORG/target`

## 增加新的同步仓库

编辑 [sync-matrix.yml](C:/Project/gitcode-sync/.github/workflows/sync-matrix.yml) 中的 `REPOS`：

```yaml
REPOS: |
  Ascend/msprof
  Ascend/msprof-analyze
  Ascend/msagent|msAgent
  owner/source-repo|target-repo
```

修改后提交到默认分支，新的仓库就会在下次手动触发或定时任务运行时参与同步。

## 运行结果查看

在 GitHub 仓库的 `Actions` 页面可以看到每次同步的执行记录。

建议重点查看：

- `Sync all repos` 这一步是否成功
- 日志里每个仓库的 `Syncing source -> target` 输出

## 常见注意事项

- 如果 GitHub 目标仓已经有旧代码，但你希望完全以 GitCode 为准，当前配置就是这种模式。
- 如果 GitHub 目标仓还有只保留在 GitHub 的分支或标签，执行同步后可能会被删除。
- GitCode 里的提供方内部引用，例如 `refs/tmp/*`，不会再同步到 GitHub。
- 如果 workflow 触发失败，优先检查 `GITCODE_TOKEN`、`GH_PAT` 是否有效，以及 `GH_PAT` 是否对目标仓有写权限。
