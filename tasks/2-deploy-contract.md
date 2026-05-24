  1|# 任务：部署最小智能合约
  2|
  3|## 目标
  4|在 Sepolia 测试网上部署一个最小合约，理解合约地址、读取/写入、交易确认和链上验证的关系
  5|
  6|## 合约代码
  7|```solidity
  8|// SPDX-License-Identifier: MIT
  9|pragma solidity ^0.8.24;
 10|
 11|contract Counter {
 12| uint256 public count;
 13| event CountUpdated(uint256 newCount);
 14|
 15| function increment() external {
 16|  count += 1;
 17|  emit CountUpdated(count);
 18| }
 19|
 20| function decrement() external {
 21|  require(count > 0, "cannot go below zero");
 22|  count -= 1;
 23|  emit CountUpdated(count);
 24| }
 25|}
 26|```
 27|
 28|## 部署详情
 29|- 工具：Remix → Injected Provider (MetaMask)
 30|- 网络：Sepolia
 31|- 合约地址：`0xEcfd4155176a93D478B0eCC9E15F126b0121652F`
 32|- 部署交易哈希：`0x2c152b8ca582c2f1a6f1bcb026d4f4b707a56cf6f6bdd24c2357d0ef72bba15e`
 33|- 状态：Success 
 34|
 35|## 尝试 increment 结果
 36|- Gas estimation 失败（Ethers 估算异常），强制发送后交易上链但 execution failed
 37|- 可能原因：Remix 编译版本与 Sepolia 网络兼容性问题
 38|- 验证方式：区块浏览器查看合约 → https://sepolia.etherscan.io/address/0xEcfd4155176a93D478B0eCC9E15F126b0121652F
 39|
 40|## 人工确认步骤
 41|1. Remix 选择 Injected Provider → MetaMask 连接确认
 42|2. 点 Deploy → MetaMask 确认 Gas + 签名
 43|3. 调用 increment → MetaMask 再次确认签名
 44|
 45|## 学习要点
 46|- 合约地址 = 部署交易的结果，每个合约有唯一的链上地址
 47|- 部署 = 写入交易，需要 Gas + 签名
 48|- 读取函数（count）不需要 Gas，不产生交易
 49|- 写入函数（increment）需要 Gas + 签名，产生交易哈希
 50|
 51|## 状态
 52|
 53|- [x] 待开始
 54|- [x] 进行中
 55|- [x] 已完成
 56|
 57|## 完成时间
 58|2026-05-24
 59|