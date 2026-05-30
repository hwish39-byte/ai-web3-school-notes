# x402 Demo - 项目总结

## 项目位置
experiments/x402-demo/

## 文件结构

| 文件 | 说明 |
|------|------|
| paywall-server.mjs | x402 模拟服务端，HTTP 端口 3101，实现受 x402 保护的审计 API |
| caw-wallet.mjs | Cobo CAW 模拟器，实现 Pact 预算控制策略 |
| agent.mjs | Agent 消费端，自动识别 402 -> 付款 -> 拿报告 |
| test.mjs | 3 个场景的自动化测试 |

## 核心逻辑

### paywall-server.mjs
- POST /api/audit 无 payment_tx -> 402 Payment Required（返回 chain_id, token, amount, receiver, deadline）
- POST /api/audit 有 payment_tx -> 验证已支付 -> 200 + 审计报告（summary, risks, sources）
- POST /api/payment -> 记录已完成付款 tx_hash

### caw-wallet.mjs
- Pact 策略：perTxLimit=10, dailyLimit=50, allowedChains=["sepolia"], allowedReceivers=["0xSERVICE"]
- transfer() 检查流程：receiver 白名单 -> chain 白名单 -> 单笔限额 -> 日限额 -> deadline
- 任意检查失败抛 Error，成功后生成模拟 tx_hash 并累计日支出

### agent.mjs
- 命令行参数：node agent.mjs <contract_address>
- 自动完成：请求 audit -> 解析 402 -> CAW 付款 -> 注册 -> 重试拿到报告

## 测试场景

| 场景 | 预期 | 验证点 |
|------|------|--------|
| 1. 正常流程 | 402 -> 付款 -> 200 + 报告 | 完整支付闭环 |
| 2. 超出单笔限额 | CAW 拒绝 | 预算控制有效 |
| 3. Pact 过期 | CAW 拒绝 | 时间窗口控制有效 |

## 关键结论

- 自动付款的前提是 Pact 预设了明确的预算和权限边界
- 被拒绝的交易不计入日支出
- 服务端通过 Set 记录已支付 tx_hash，防止重复使用
