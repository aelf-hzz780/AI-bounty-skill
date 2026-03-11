# AI Bounty Claim Skill

[English](./README.md)

这是一个面向 Agent / Codex 类宿主的公开 skill，用于在 `aelf` 测试链 `tDVV` 上通过 `RewardClaimContract` 领取 AI bounty。

## 仓库内容

当前仓库的核心内容是 [SKILL.md](./SKILL.md)。它定义了 skill 的触发描述、适用边界、链上默认值、依赖 skill 路由方式，以及两条受支持的领取路径。

## 支持的领取路径

本 skill 只覆盖以下两条受支持路径：

- `Claim()`
- `ClaimByPortkeyToCa(Hash ca_hash)`

当用户希望直接领取到普通 EOA 地址时，使用 `Claim()`。

当用户希望领取到 Portkey CA 身份，且已经具备或可以解析出 `ca_hash` 时，使用 `ClaimByPortkeyToCa(Hash ca_hash)`。

## 不在范围内

以下场景不在本仓库的目标范围内：

- Deprecated `ClaimByPortkey`
- Contract deployment or upgrade work
- Admin operations such as changing claim windows or withdrawing balances

## 依赖 Skills

本 skill 依赖 Portkey 生态 skill 做 signer 解析、CA 查询和路由，而不是在当前 skill 内重复实现这些能力。

- Portkey CA skill: [Portkey-Wallet/ca-agent-skills](https://github.com/Portkey-Wallet/ca-agent-skills)
- Portkey EOA skill: [Portkey-Wallet/eoa-agent-skills](https://github.com/Portkey-Wallet/eoa-agent-skills)

EOA 路径通常使用 EOA skill；CA 路径通常使用 CA skill。

## 已知 tDVV 默认值

当用户明确处于当前 AI bounty 环境时，可以使用以下默认值；这些是环境默认值，不应被当作永久常量。

- Reward contract: `ELF_2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472_tDVV`
- Public RPC: `https://tdvv-public-node.aelf.io`
- Portkey CA contract: `ELF_2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9_tDVV`

## 安全规则

任何链上写操作前，都必须先展示将要使用的 signer、目标合约、调用方法、关键输入值和预期接收方语义，并要求用户显式确认；如果没有明确确认，必须停止。

`aelf-command` 示例仅是手动 fallback，不应替代依赖 skill 的主流程，也不应绕过前置校验。

## 典型工作流

1. 判断用户要走 EOA 领取还是 Portkey CA 领取。
2. 校验 claim contract 是否处于可领取状态。
3. 解析正确 signer，并校验是否已经领取过。
4. 如为 CA 路径，校验 `ca_hash` 和 manager membership。
5. 在发送交易前要求显式确认。
6. 发送交易并回报 `txid`，失败时返回原始链上错误。

## 仓库结构

当前仓库保持极简，便于作为公开分发 skill 使用。

```text
.
├── README.md
├── README.zh.md
└── SKILL.md
```

## 使用方式

如果宿主支持基于 `SKILL.md` 的技能加载，直接引用本仓库即可；如果宿主只支持本地 skill 目录，也可以把 `SKILL.md` 放入对应的 skill 包结构中使用。

具体的执行规范、参数语义、失败场景和反模式，请以 [SKILL.md](./SKILL.md) 为准。
