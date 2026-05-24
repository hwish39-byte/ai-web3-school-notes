# AI x Web3 最小交叉流程图

---

## 工作流 A：AI 生成交易 -> 人工确认 -> 链上执行

```mermaid
flowchart LR
 subgraph AI["AI 层"]
 A1[用户提出需求] --> A2[AI 生成交易说明]
 A2 --> A3[AI 模拟交易<br/>检查风险]
 end

 subgraph HUMAN["人工层"]
 A3 --> B1[人工复核交易详情]
 B1 --> B2{是否安全?}
 B2 -- 否 --> B3[退回修改]
 B3 --> A2
 B2 -- 是 --> B4[打开钱包确认]
 end

 subgraph CHAIN["链上层"]
 B4 --> C1[签名交易]
 C1 --> C2[发送到 Mempool]
 C2 --> C3[矿工打包上链]
 C3 --> C4[区块浏览器验证]
 end

 style A1 fill:#6366f1,color:#fff
 style B4 fill:#f59e0b,color:#000
 style C1 fill:#10b981,color:#fff
 style C2 fill:#10b981,color:#fff
 style C3 fill:#10b981,color:#fff
```

| 角色 | 步骤 | 说明 |
|------|------|------|
| 用户发起 | 提出需求 | "帮我看这笔 approve 能不能签" |
| AI 执行 | 生成说明 + 模拟 | 链上查 spender、余额、模拟交易 |
| 人工确认 | **复核 + 决策** | 必须人工判断是否可信 |
| 钱包签名 | **签名** | 必须用户亲手在 MetaMask 签名 |
| 链上执行 | 交易上链 | AI 可后续读取交易收据验证 |

---

## 工作流 B：Agent 自动化 -> 人工兜底

```mermaid
flowchart LR
 subgraph INIT["初始化"]
 U1[用户设定规则] --> U2[部署 Session Key<br/>日限额 100 USDT<br/>白名单: Uniswap]
 end

 subgraph AGENT["Agent 自动运行"]
 U2 --> A1[监听链上条件]
 A1 --> A2{满足策略?}
 A2 -- 否 --> A1
 A2 -- 是 --> A3[构造交易]
 A3 --> A4[检查限额]
 A4 --> A5[用 Session Key 签名]
 end

 subgraph POST["事后"]
 A5 --> C1[交易上链]
 C1 --> C2[记录到日志]
 C2 --> C3{异常?}
 C3 -- 是 --> C4[通知用户]
 C3 -- 否 --> A1
 end

 style U1 fill:#6366f1,color:#fff
 style U2 fill:#f59e0b,color:#000
 style A5 fill:#10b981,color:#fff
 style C4 fill:#ef4444,color:#fff
```

| 角色 | 步骤 | 说明 |
|------|------|------|
| 用户 | 设定规则 | **一次性授权**：部署 Session Key |
| Agent | 全自动执行 | 在限额内自动构造并签名交易 |
| 链上 | 交易上链 | Session Key 权限受限于预设规则 |
| 用户 | 事后监督 | 检查日志，发现异常可撤销 Session Key |

---

## 工作流 C：DAO 治理助手

```mermaid
flowchart LR
 subgraph INPUT["输入"]
 I1[提案文本] --> I2[论坛讨论]
 end

 subgraph AI["AI 分析"]
 I1 --> A1
 I2 --> A1[提取关键信息]
 A1 --> A2[总结支持/反对理由]
 A2 --> A3[标记风险点和缺失信息]
 A3 --> A4[生成投票检查清单]
 end

 subgraph HUMAN["治理"]
 A4 --> B1[人工阅读报告]
 B1 --> B2[讨论 & 决策]
 B2 --> B3{投票?}
 B3 -- 否 --> B4[弃权或反对]
 B3 -- 是 --> B5[连接钱包投票]
 end

 subgraph CHAIN["链上"]
 B5 --> C1[链上投票交易]
 C1 --> C2[提案状态更新]
 C2 --> C3[结果公开可查]
 end

 style I1 fill:#6366f1,color:#fff
 style I2 fill:#6366f1,color:#fff
 style B1 fill:#f59e0b,color:#000
 style B5 fill:#10b981,color:#fff
```

| 角色 | 步骤 | 说明 |
|------|------|------|
| 提案人 | 发起提案 | 提交文本 |
| AI | 分析整理 | 读提案 + 论坛，生成报告 |
| 社区 | **阅读 + 讨论** | AI 辅助但不能替人投票 |
| 投票人 | **钱包签名** | 投票交易需要钱包确认 |
| 链上 | 记录结果 | 提案状态、票数公开可查 |

---

## 三条流程的边界对比

```
                       流程 A                   流程 B                   流程 C
                   (授权检查助手)          (Agent 自动交易)         (DAO 治理助手)

AI 自动化程度      中                       高                       低-中
人工介入点        每笔交易确认             事前设定规则              每次投票决策
签名方式          用户手动签名             Session Key 自动签        用户手动签名
风险场景          用户无视警告签名         Session Key 被盗用        AI 遗漏关键信息
适用场景          高频、高风险决策         低频、规则明确的交易       信息筛选 + 决策辅助
```

## 关键原则

```
AI 不能做的：
  - 接触私钥 / 助记词
  - 替用户签名
  - 自动发送链上交易（除非 Session Key）
  - 替用户做最终决策

AI 可以做的：
  - 读取链上公开数据
  - 生成交易解释和风险分析
  - 模拟交易结果
  - 在预设限额内自动执行（Session Key）
  - 记录和追踪交易状态
```
