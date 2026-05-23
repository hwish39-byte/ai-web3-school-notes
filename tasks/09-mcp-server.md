# 任务：MCP 最小实践 — 只读 MCP Server

## 目标
用 Node.js 创建零依赖只读 MCP Server，暴露 search_docs 和 get_file 两个工具，包含安全防护和审计日志

## 参考材料
- Handbook「MCP」章节 - 最小实践

## 输出
- `experiments/mcp-mininal-practice/mcp-server.mjs`
- 6 项测试全部通过
- 内置写入工具权限升级方案（一次性 token + 撤销 + SHA-256 审计）

## 状态

- [x] 待开始
- [x] 进行中
- [x] 已完成

## 完成时间
2026-05-23
