# 任务：Inference 最小实践 — 交易风险推理对比

## 目标
比较规则引擎 vs API 模型的交易风险评估方案，设计混合 fallback 策略

## 参考材料
- Handbook「Inference」章节 - 最小实践

## 输出
- `experiments/inference-mininal-practice/`
- 三方案对比报告 + 基准测试脚本
- 关键发现：规则引擎在 DEX swap 上误报（unlimited approval 实为 DeFi 标准）
- Fallback 设计：规则引擎初筛 → API fallback → 人工兜底

## 状态

- [x] 待开始
- [x] 进行中
- [x] 已完成

## 完成时间
2026-05-23
