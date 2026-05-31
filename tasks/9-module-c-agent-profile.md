# Module C - Identity / Reputation / Capability / Interoperability

---

## 任务一：Agent Profile 设计

### 选定 Agent：AI 合约审计助手（Audit Agent）

基于 Module B 的 x402 审计场景，做完整 Profile。

### Agent Identity Card

| 字段 | 内容 |
|------|------|
| Name | Audit Agent v1 |
| Owner | AuditDAO（去中心化审计者组织） |
| Version | 1.0.0 |
| Registry | eth://registry.agent/0x AUDIT_AGENT_ID |
| DID | did:agent:0x AUDIT_AGENT_ID |

### 能力声明

| 能力 | 说明 | 风险等级 | 输入 | 输出 | 价格 |
|------|------|---------|------|------|------|
| Solidity Audit | 智能合约安全审计 | 只读 | contract_address, chain, depth | summary, risks, sources | 5 USDC |
| Tx Interpret | 交易翻译解释 | 只读 | tx_hash, chain | summary, events, risks | 1 USDC |

### 协作对象

| 对象 | 协作方式 | 协议 |
|------|---------|------|
| 用户 | 提交审计请求 -> 获取报告 | x402 / HTTPS |
| CAW Wallet | 检查预算 -> 执行付款 | CAW Pact |
| Contract Explorer | 读取合约源码和 ABI | MCP |
| Reputation Oracle | 验证历史任务记录 | A2A |
| AuditDAO Multisig | 争议仲裁 | 链上治理 |

### 失败处理

| 场景 | 处理方式 |
|------|---------|
| 交付超时（>30 min） | 自动退款，escrow 释放 |
| 报告不完整 | 退回修改（最多 2 次），否则退款 |
| 争议 | AuditDAO 多签仲裁 |
| 恶意审计 | Slashing（罚没 stake）|

### 验证方式

- 审计引擎开源：github.com/auditdao/engine
- 报告用 AuditDAO 密钥签名，链上可验证
- 付款和交付收据在链上

---

## 任务二：协议比较 - MCP vs A2A

| 维度 | MCP | A2A |
|------|-----|-----|
| 谁跟谁 | LLM <-> Tool | Agent <-> Agent |
| 方向 | 消费（Client 调 Server） | 对等（互相委托） |
| 身份 | 无，工具无身份 | Agent Card / DID |
| 能力声明 | Tool schema（JSON Schema） | Agent Card（结构化能力列表） |
| 支付 | 不处理 | 不处理，配合 MPP |
| 失败 | 工具返回错误 | 任务状态同步 + 重试 |
| 适合解决 | 工具调用、文件访问 | Agent 发现、委托、协作 |

MCP 让 LLM 能用工具，A2A 让 Agent 能找 Agent。两者不冲突：Audit Agent 内部用 MCP 调用代码分析工具，对外用 A2A 接收用户的审计委托。

---

## 任务三：Agent Profile 草图

```
Name:       Audit Agent v1
Owner:      AuditDAO (0x...)
Cap:        [Solidity Audit, Tx Interpret]
Endpoint:   https://api.audit-agent.xyz/v1 (x402)
            mcp://audit-agent.xyz/audit (MCP)
Price:      5 USDC / audit, 1 USDC / tx-interpret
Payment:    x402 paywall + escrow
Rep:        94.2% success, 156 tasks, 1000 USDC staked
Fail:       Timeout -> refund | Incomplete -> rework | Dispute -> arbitration
Verify:     Open-source engine, on-chain signed reports
```

---

## 交付清单

- [x] Agent Profile（Audit Agent）
- [x] 协议比较：MCP vs A2A
- [x] Agent Profile 草图
