# Denco Fork Management

## 概述

`middleware/denco/` 是 [naoina/denco](https://github.com/naoina/denco) 的 fork。

denco 是一个基于 Double-Array 实现的快速 URL 路由器，上游项目由 [naoina](https://github.com/naoina) 维护。

## Fork 原因

上游 `naoina/denco` 是一个通用 URL 路由器，无法满足 go-openapi/swagger 的特殊路由需求。
于 2016 年（commit `6d6e02b`）被 fork 到本仓库，直接内嵌代码而非使用 go module 依赖，以便自由修改。

## 与上游的差异

| 提交 | 说明 |
|------|------|
| `6d6e02b` | **初始 fork**：修复 go-swagger/go-swagger#522，支持转义路径路由。这是 fork 的根本原因——`net/http` 的 `r.URL.Path` 会自动解码转义字符，必须使用 `r.URL.EscapedPath()` |
| `4affd1e` | 修复路径参数中包含横线（dash）的问题 |
| `0aa3c49` | 修正 `:` 只有在路径段**开头**出现时才算参数标记，避免误匹配段中间的冒号 |
| `654e393` | 强制 Accept header 校验 |
| `93443cc` | 增加 RESTCONF 协议（RFC 8040）支持：`PathParamCharacter = '='`，如 `/path/key=:value` |
| `da56347` | golangci-lint 代码清理 |

## 管理原则

1. **尽量不修改**：只在必须满足 OpenAPI/Swagger 路由需求时才修改 denco 代码
2. **记录每一次修改**：每次对 denco 的修改必须在此文档中记录
3. **不要随意合并上游**：上游 `naoina/denco` 已多年未更新（最后更新 ~2017），无需主动追踪上游变更
4. **新功能首先考虑 middleware/router.go**：能在外部包装解决的，不要改 denco 内部。例如 `decodeCompositParams` 就放在了 middleware 层而非 denco 内部

## 升级流程

如果将来需要合并上游变更：

```bash
# 1. 添加上游 remote
git remote add denco-upstream https://github.com/naoina/denco.git
git fetch denco-upstream

# 2. 查看差异
git diff HEAD -- middleware/denco/ \
  <(git show denco-upstream/master:)

# 3. 手动挑选需要的修改，逐个应用到 middleware/denco/

# 4. 运行全部测试
go test ./middleware/denco/... ./middleware/...
```

## 关键文件

- `router.go` — 核心路由逻辑（Double-Array 实现），包含转义路径和 RESTCONF 修改
- `server.go` — HTTP mux 包装器
- `util.go` — `NextSeparator` 辅助函数

## 相关文件（middleware 层，非 denco 内部）

- `middleware/router.go` — 使用 denco 构建 OpenAPI 路由，包含 `decodeCompositParams`（复合参数拆分）
- `middleware/context.go` — 路由上下文，`LookupRoute` 使用 `r.URL.EscapedPath()` 查找路由
