# AI Bounty Claim Skill

[English](./README.md)

这是一个公开 skill 仓库，用于在 `aelf` 的 `tDVV` 主网侧链环境中，通过 `RewardClaimContract` 领取 AI bounty。

## 仓库结构

仓库采用“单入口 skill + 分支参考文档”的结构，方便较弱的 agent 一次只跟随一条路径。

- [SKILL.md](./SKILL.md)：账户选择、路由规则、硬性停止条件、共享默认值
- [references/flows/](./references/flows)：每条路径的 step-by-step 指令
- [references/examples/](./references/examples)：给较弱模型使用的分支示例

## 支持的方法

当前 skill 只支持以下公开领取方法：

- `Claim()`
- `ClaimByPortkeyToCa(Hash ca_hash)`

## 路由分支

- Account choice and onboarding
- Portkey CA claim
- EOA claim
- Diagnostics and stop

## 共享规则

- `tDVV` 在本仓库中按当前 AI bounty 的主网侧链环境描述。
- 当前活动默认奖励写法为：CA `2 tokens`，EOA `1 token`。
- 校验 RPC 是否可用时，应请求 [chainStatus](https://tdvv-public-node.aelf.io/api/blockChain/chainStatus)，不要用 RPC 根路径是否返回 `404` 来判断节点挂了。
- 对未明确账户类型的泛化 claim 诉求，必须先解释 `CA` 和 `EOA` 的区别，推荐 `CA`，再让用户选择。
- 必须显式使用 [Portkey EOA skill](https://github.com/Portkey-Wallet/eoa-agent-skills) 处理 EOA 相关事情，使用 [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) 处理 CA 相关事情，不能指望 AI 自己隐式推断。
- `CA` 是默认推荐路径，因为当前活动奖励更高。
- 要直接提示用户：请勿填写交易所地址或托管地址。
- 不要向用户索要地址；应优先使用本地 EOA 地址或本地 CA 账号上下文。
- 如果所选本地账户尚未就绪，应先引导创建本地 CA 或本地 EOA，再继续领取流程。
- 链上返回错误时，必须原样返回错误并停止。

## 使用方式

从 [SKILL.md](./SKILL.md) 开始，先选定唯一分支，再读取对应的 flow 文档和 example 文档后再回复用户。
