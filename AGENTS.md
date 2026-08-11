# 仓库贡献指南

## 项目结构与模块组织

这是一个用于 Windows 软件包清单的 Scoop bucket。仓库根目录即当前目录。应用清单位于 `bucket/`，每个应用对应一个 JSON 文件；清单必须直接放在 `bucket/` 根层，不要再创建 `bucket/bucket/` 等嵌套目录。新增清单可从 `bucket/app-name.json.template` 复制开始。`deprecated/` 用于保留废弃清单，`scripts/` 用于放置清单安装或辅助脚本，`bin/` 放置封装 Scoop 官方工具的本地维护脚本。测试入口是 `Scoop-Bucket.Tests.ps1`，它会导入 Scoop 官方 bucket 测试套件。GitHub 工作流、议题模板和 PR 模板位于 `.github/`，编辑器与格式配置位于 `.vscode/`、`.editorconfig` 和 `.markdownlint.json`。

## 构建、测试与开发命令

除非特别说明，以下命令都应在 `scoop-bucket/` 目录下执行。

- `pwsh ./bin/test.ps1`：运行所有清单的 Pester 测试。
- `pwsh ./bin/checkver.ps1`：检查带有 `checkver` 元数据的清单是否有可用版本更新。
- `pwsh ./bin/missing-checkver.ps1`：列出缺少 `checkver` 元数据的清单。
- `pwsh ./bin/checkurls.ps1`：校验清单中的下载地址是否可用。
- `pwsh ./bin/checkhashes.ps1`：校验清单中的哈希值是否正确。
- `pwsh ./bin/formatjson.ps1`：统一格式化 bucket 中的 JSON 文件。
- `scoop install ./bucket/<app>.json`：本地安装并验证某个清单。

这些 `bin/` 脚本会调用本机 Scoop 安装目录中的官方脚本；如果未设置 `SCOOP_HOME`，脚本会通过 `scoop prefix scoop` 自动解析。需要传递官方脚本参数时，可直接追加在本地脚本命令之后。

## 自动更新与上游同步

`.github/workflows/excavator.yml` 每四小时运行一次 Scoop Excavator。只有同时提供有效 `checkver` 和 `autoupdate` 的清单才会自动更新；缺少任一配置的清单仍会被测试，但不会自动跟踪新版本。新增或修改自动更新配置后，先运行 `pwsh ./bin/checkver.ps1 <app>`，再根据变更运行 URL 和哈希检查。

`checkver` 应从软件官方上游解析版本，`autoupdate` 应使用 `$version` 等占位符生成下载地址和哈希。Excavator 负责同步软件发布版本，不会逐字镜像第三方 bucket 清单的任意内容；上游若改变安装脚本、文件布局或持久化规则，仍需人工审查并同步。

## 编码风格与命名规范

遵循 `.editorconfig`：使用 UTF-8、CRLF 换行、四空格缩进、文件末尾保留换行，并移除行尾空格。YAML 文件使用两空格缩进。这里保留 CRLF 不是因为 JSON、YAML 或 PowerShell 无法使用 LF，而是因为仓库导入的 Scoop 官方测试会硬编码检查所有文本文件必须使用 CRLF；同时 `.gitattributes` 会将文本在 Git 索引中规范化为 LF，只在 Windows 工作区检出为 CRLF，因此不会让 GitHub Diff 保存为 CRLF。除非同时调整官方测试、`.editorconfig` 和 `.gitattributes`，否则不要单独把文件改为 LF。

清单文件名应使用应用标识符，优先保持小写或沿用上游既有大小写，例如 `protoc.json` 或 `FlashPad.json`。清单 JSON 应保持 Scoop 常见字段顺序，优先包含 `version`、`description`、`homepage`、`license`、`architecture`、`url`、`hash`、`bin`、`shortcuts`、`checkver` 和 `autoupdate` 等适用字段。手动修改 JSON 后应运行格式化脚本，保持字段顺序和排版稳定。Markdown 遵循 `.markdownlint.json`，当前允许长行，并只允许不同父级下的重复标题。

## 测试指南

通过 `bin/test.ps1` 运行 Pester 测试；环境需要 PowerShell 5.1+、BuildHelpers 2.0.1、Pester 5.2.0，以及可用的 Scoop 安装。提交前应测试所有新增或变更的清单。对版本化应用，重点检查 `version`、`url`、`hash`、`bin`、`shortcuts`，以及适用的 `checkver` 或 `autoupdate` 字段。涉及下载地址或哈希变更时，运行 `checkurls.ps1` 和 `checkhashes.ps1`。优先使用 `scoop install ./bucket/<app>.json` 做本地安装验证，不要只依赖 CI。

## 提交与拉取请求规范

近期提交历史使用简洁的祈使式标题，例如 `Create protoc.json`、`Create alixby.json` 和 `Update FlashPad.json`。每个提交应聚焦于一个清单或一组相关脚本变更。拉取请求应说明应用或更新内容，列出已执行的验证命令，链接上游发布页或项目主页，并注明任何非标准安装行为。对于有界面的应用，只有在有助于确认快捷方式、名称或安装结果时才附带截图。

## 安全与配置提示

不要提交本地 Scoop 缓存、已下载的二进制文件、令牌或机器专属路径。下载地址和校验和应优先来自官方上游来源，不要使用需要登录、会过期或依赖个人会话的链接。`bin/auto-pr.ps1` 当前默认值仍是模板占位符；使用该脚本前应将其改为 `willtse/scoop-bucket:master`，或显式传入 `-Upstream`。
