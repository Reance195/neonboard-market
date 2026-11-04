# neonboard-market

NeonBoard 的**插件市场索引**仓库。NeonBoard 客户端默认市场源指向这里，读取仓库根目录的 [`market.json`](market.json)。

## 客户端怎么用

NeonBoard 的默认源是仓库简写 `Reance195/neonboard-market`，客户端按顺序尝试：

```
https://gitee.com/Reance195/neonboard-market/raw/main/market.json
   ↓ 失败才回退
https://raw.githubusercontent.com/Reance195/neonboard-market/main/market.json
```

索引会缓存在本地（默认 TTL 6 小时，带 ETag / Last-Modified），断网时降级到缓存。

## 索引格式

```json
{
  "version": 2,
  "updatedAt": "2026-11-04",
  "signature": "",
  "plugins": [
    {
      "id": "com.example.timer",
      "name": "计时器",
      "version": "1.0.0",
      "category": "任务自动化",
      "description": "一个示例插件",
      "dependencies": ["无外部依赖"],
      "author": "someone",
      "homepage": "https://example.com",
      "download": "timer.neon",
      "sha256": "<.neon 包的 SHA-256>",
      "size": 469
    }
  ]
}
```

- `download` 可以写完整 URL，也可以只写文件名（客户端按索引所在目录解析）；
- `sha256` 给了就强制校验，包内容不符整单失败、不落盘；
- `category` 为类别（如：任务自动化 / Agent 接入 / 示例），`dependencies` 为依赖说明（数组，逐条列出运行前提）；
  客户端当前版本忽略未知字段，这两个字段现阶段仅作索引与文档用途；
- 顶层 `signature` 字段已预留（minisign 格式），当前版本不校验。

## 当前已上架

| 名称 | 类别 | 依赖说明 |
| --- | --- | --- |
| Agent 任务桥 | 任务自动化 | apiVersion 2 宿主；需事件投递方（post-commit 钩子或 AI Agent 适配器） |
| 计划钩子 | 任务自动化 | 宿主 Git 钩子链路（仓库自带）；apiVersion 2 |
| Claude Code | Agent 接入 | PATH 中可执行 `claude`；开启「任务执行」总开关（默认关闭） |
| Codex CLI | Agent 接入 | PATH 中可执行 `codex`；开启「任务执行」；该 profile 默认未启用 |
| Cursor Agent | Agent 接入 | PATH 中可执行 `cursor-agent`；开启「任务执行」 |
| opencode | Agent 接入 | PATH 中可执行 `opencode`；开启「任务执行」 |
| 多表面示例 | 示例 | 无外部依赖，不申请权限 |
| 今日概览 | 示例 | 无外部依赖，仅 `tasks.read` 权限（apiVersion 1 即可） |

> 官方插件已随安装包分发（`dist/plugins/`），市场提供重装 / 更新通道；示例插件仅市场分发。

## 怎么发布一个插件

在 NeonBoard 仓库里用发布脚本打包并更新索引：

```powershell
pwsh -File scripts/publish-market.ps1 -PluginPath plugins\plan-hook `
    -OutputDir D:\market -DownloadBase https://raw.githubusercontent.com/Reance195/neonboard-market/main
```

脚本会读 `plugin.json` → 打成单文件 `.neon` 包 → 算 SHA-256 与体积 → 合并进 `market.json`（保留其他条目）。

然后把 `market.json` 和生成的 `.neon` 包一起提交到本仓库根目录即可。

## 相关文档

- 插件 API 与包格式：`NeonBoard/docs/plugin-api.md`
- 市场源解析、缓存与四道校验：同上「市场源（多源并列）」一节
