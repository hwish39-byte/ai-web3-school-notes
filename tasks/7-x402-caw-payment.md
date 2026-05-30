# Week 2 Module B - x402 Paywall + CAW Agent 自主支付闭环

---

## 场景

用户（项目方）通过 Agent 调用一个受 x402 保护的 AI 合约审计 API。Agent 自动识别付款要求、在 CAW Pact 预算范围内完成支付、获取审计结果。

---

## 整体架构

```
                   消费端 (Agent)                         服务端 (Paywall + API)

  Agent Workflow        Cobo CAW               x402 Paywall       AI Audit Engine
  (Claude Code)     Pact 预算控制                 
       |                  |                        |                    |
  1. POST /api/audit      |                        |                    |
  ------------------------>                       |                    |
       |                  |                  2. 402 Payment Required   |
       |                  |                   <------------------------
       |                  |                        |                    |
  3. 解析 402 响应        |                        |                    |
  4. 请求付款             |                        |                    |
  ------------------------>                        |                    |
       |            5. Pact 检查                   |                    |
       |              - 限额 ✅                     |                    |
       |              - 白名单 ✅                   |                    |
       |              - 时间窗口 ✅                  |                    |
       |            6. 签名交易                     |                    |
       |             ---------------------------->  |                    |
       |                  |                   7. 验证付款 ✅            |
       |                  |                        |  8. 执行审计       |
       |                  |                        | ------------------> |
       |                  |                        |                    |
       |  9. 返回审计报告  |                        |                    |
       |  <----------------------------------------                     |
       |                  |                        |                    |
 10. 记录日志            |                        |                    |
```

---

## 完整交互流程（11 步）

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1 | Agent -> Paywall POST /api/audit | 提交合约地址和链 |
| 2 | Paywall -> Agent 402 Payment Required | 返回付款要求（金额、代币、接收地址、有效期） |
| 3 | Agent 解析 402 响应，检查 Pact 预算 | 验证金额不超过单笔限额、链在白名单中 |
| 4 | Agent -> CAW 请求签名 | 构造 transfer 5 USDC 交易 |
| 5 | CAW Pact 检查 | 自动检查限额、白名单、时间窗口，不弹窗 |
| 6 | CAW -> Chain 发送交易 | transfer 5 USDC to 0xSERVICE |
| 7 | Agent -> Paywall（带付款证明重试） | POST /api/audit + payment_tx |
| 8 | Paywall 验证链上付款 | 验证 tx 状态、收款地址、金额 |
| 9 | AI Engine 执行审计并返回结果 | 返回结构化风险报告 |
| 10 | Agent 记录日志 | 付款哈希、交付哈希、验收结果 |

---

## 关键接口

### 1. 请求（402 触发）

```
POST /api/audit
Content-Type: application/json

{
  "contract": "0xEcfd4155176a93D478B0eCC9E15F126b0121652F",
  "chain": "sepolia"
}
```

### 2. 402 响应（服务端要求付款）

```
HTTP/1.1 402 Payment Required
Content-Type: application/json

{
  "version": "x402-v1",
  "payment": {
    "chain_id": 84532,
    "token": "0x...USDC",
    "amount": "5000000",
    "decimals": 6,
    "receiver": "0xSERVICE_WALLET",
    "deadline": 600,
    "description": "AI contract audit - 1 contract"
  }
}
```

### 3. 带付款重试

```
POST /api/audit
Content-Type: application/json

{
  "contract": "0xEcfd4155176a93D478B0eCC9E15F126b0121652F",
  "chain": "sepolia",
  "payment_tx": "0xPAY_TRANSACTION_HASH"
}
```

### 4. 成功响应

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "report": { "summary": "...", "risks": [], "sources": [] },
  "receipt": {
    "payment_tx": "0xPAY...",
    "delivery_tx": "0xRESULT...",
    "verified_at": 1717000000
  }
}
```

---

## CAW Pact 配置

```json
{
  "pact": {
    "name": "AI Audit Agent",
    "budget": {
      "per_tx": "10 USDC",
      "daily": "50 USDC",
      "weekly": "200 USDC"
    },
    "allowed_chains": ["base-sepolia", "sepolia"],
    "allowed_contracts": ["0xSERVICE_WALLET"],
    "allowed_tokens": ["0x...USDC"],
    "expires_at": "2026-06-01T00:00:00Z",
    "require_confirmation": false,
    "audit_log": true
  }
}
```

---

## Agent 端伪代码

```python
def call_x402_api(contract_addr, chain):
    # 1. 初次请求
    resp = http_post("/api/audit", {"contract": contract_addr, "chain": chain})
    if resp.status == 200:
        return resp.body
    if resp.status != 402:
        raise Error()

    payment = resp.body["payment"]

    # 2. 检查预算
    caw = CoboCAW(session_key="sk_...")
    caw.assert_budget(payment["chain_id"], payment["token"],
                      payment["amount"], payment["receiver"])

    # 3. 付款
    tx_hash = caw.transfer(payment["receiver"], payment["token"],
                           payment["amount"], payment["chain_id"])

    # 4. 带证明重试
    resp2 = http_post("/api/audit", {"contract": contract_addr,
                     "chain": chain, "payment_tx": tx_hash})
    if resp2.status == 200:
        record_log({"contract": contract_addr, "payment_tx": tx_hash,
                    "report": resp2.body["report"]})
        return resp2.body

    raise Error(f"Payment accepted but delivery failed")
```

---

## 风险边界

| 风险 | 说明 | 缓解 |
|------|------|------|
| 重复付款 | 重试时再次触发付款 | payment_tx 做幂等键，服务端去重 |
| 超时 | 402 报价过期后 Agent 才付款 | CAW 检查 deadline，过期拒绝 |
| 预算耗尽 | 花光日限额后继续请求 | CAW 签名前检查剩余额度 |
| 钓鱼 paywall | 恶意服务诱导 Agent 付款 | Agent 校验 receiver 白名单 |
| 交付失败 | 付款后未返回结果 | 超时重试，上报用户 |
| 日志丢失 | 本地日志丢失无法追溯 | 关键日志上链 |

---

## 交付清单

- [x] 架构图
- [x] 11 步交互流程
- [x] 4 个关键接口说明
- [x] CAW Pact 配置
- [x] Agent 端伪代码
- [x] 6 项风险边界
