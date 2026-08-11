# win_iso_build

[English Readme](https://github.com/adavak/win_iso_build/blob/main/README.md)

GitHub Actions 自动化构建 — 把官方 Windows ISO 集成最新补丁后发布。

## 三件套关系

| 仓库 | 作用 |
|------|------|
| [Win_ISO_Patching_Scripts](https://github.com/adavak/Win_ISO_Patching_Scripts) | 补丁清单（meta4）+ 集成脚本（W10UI） |
| **win_iso_build** ← 本仓库 | Actions 构建管线：拉原版 ISO → 校验 → 打补丁 → 发布 |
| [win_iso_zip](https://github.com/adavak/win_iso_zip) | 原版 ISO 分卷存放 + 成品 Release 发布 |

```
Win_ISO_Patching_Scripts  ──(dispatch)──→  win_iso_build  ──(download)──→  win_iso_zip
     (meta4 补丁清单)                         (Actions 构建)                   (原版 ISO + 成品)
```

## 触发方式

纯手动：`workflow_dispatch`。通常由 `Win_ISO_Patching_Scripts` 的 Release event 通过 API 触发。

## 配置

构建矩阵由 `.github/version-config.json` 定义：

- `versions` → 产品名称和 ISO 标签
- `locales` → 语言列表（`zh-CN`, `en-US`）
- `releases` → 每种语言/版本对应的 `release_tag`（win_iso_zip 那边的 tag）和 `expected_hash`（原版 ISO 的 SHA256）

## 流程

1. `generate-matrix` — 清理旧 runs，从 `version-config.json` 生成构建矩阵
2. `build`（矩阵并行）：
   - 对比 meta4 hash vs 上次 Release → 无变化则跳过
   - 从 `win_iso_zip` 下载分卷 ISO → 7z 合并
   - SHA256 校验 → 运行 `Start.cmd` 打补丁
   - ISO 重命名 + 分卷（2000MB）→ 创建 Release
