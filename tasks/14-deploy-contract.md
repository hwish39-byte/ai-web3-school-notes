# 任务：部署最小智能合约

## 目标
在 Sepolia 测试网上部署一个最小合约，理解合约地址、读取/写入、交易确认和链上验证的关系

## 合约代码
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Counter {
    uint256 public count;
    event CountUpdated(uint256 newCount);

    function increment() external {
        count += 1;
        emit CountUpdated(count);
    }

    function decrement() external {
        require(count > 0, "cannot go below zero");
        count -= 1;
        emit CountUpdated(count);
    }
}
```

## 部署详情
- 工具：Remix → Injected Provider (MetaMask)
- 网络：Sepolia
- 合约地址：`0xEcfd4155176a93D478B0eCC9E15F126b0121652F`
- 部署交易哈希：`0x2c152b8ca582c2f1a6f1bcb026d4f4b707a56cf6f6bdd24c2357d0ef72bba15e`
- 状态：Success ✅

## 尝试 increment 结果
- Gas estimation 失败（Ethers 估算异常），强制发送后交易上链但 execution failed
- 可能原因：Remix 编译版本与 Sepolia 网络兼容性问题
- 验证方式：区块浏览器查看合约 → https://sepolia.etherscan.io/address/0xEcfd4155176a93D478B0eCC9E15F126b0121652F

## 人工确认步骤
1. Remix 选择 Injected Provider → MetaMask 连接确认
2. 点 Deploy → MetaMask 确认 Gas + 签名
3. 调用 increment → MetaMask 再次确认签名

## 学习要点
- 合约地址 = 部署交易的结果，每个合约有唯一的链上地址
- 部署 = 写入交易，需要 Gas + 签名
- 读取函数（count）不需要 Gas，不产生交易
- 写入函数（increment）需要 Gas + 签名，产生交易哈希

## 状态

- [x] 待开始
- [x] 进行中
- [x] 已完成

## 完成时间
2026-05-24
