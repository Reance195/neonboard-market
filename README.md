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
  "version": 1,
  "updatedAt": "2026-10-27",
  "plugins": [
    {
      "id": "com.example.timer",
      "name": "计时器",
      "version": "1.0.0",
      "description": "一个示例插件",
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
- 顶层 `signature` 字段已预留（minisign 格式），当前版本不校验。

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
