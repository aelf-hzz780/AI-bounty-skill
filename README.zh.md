# AI Bounty Claim Skill

[English](./README.md)

## 快速开始

把下面这段直接发给你的 AI。先把邮箱占位符替换成自己的邮箱，不要填写交易所地址或托管地址。

### 直接喂给 AI

```text
1. 请先阅读 https://github.com/aelf-hzz780/AI-bounty-skill/blob/main/SKILL.md，了解这个 skill，并把它安装到本地。
2. 然后按这个 skill 的要求帮我 Claim，使用 AA/CA 方案。
3. 使用 email：<替换成你自己的邮箱> 来 recover/login。
```

这是一个公开 skill 仓库，用于在 `aelf` 的 `tDVV` 主网侧链环境中，通过 `RewardClaimContract` 领取 AI bounty。

对 AA/CA 来说，当前 skill 的标准 wallet 路径是 `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`。

## 仓库结构

仓库采用“单入口 skill + 分支参考文档”的结构，方便较弱的 agent 一次只跟随一条路径。

- [SKILL.md](./SKILL.md)：账户选择、路由规则、硬性停止条件、共享默认值
- [references/flows/](./references/flows)：每条路径的 step-by-step 指令
- [references/examples/](./references/examples)：给较弱模型使用的分支示例

## 支持的方法

当前 skill 只支持以下公开领取路径：

- `EOA`：`Claim()`
- `AA/CA`：`ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`

## 路由分支

- Account choice and onboarding
- Portkey AA/CA claim
- EOA claim
- Diagnostics and stop

## 共享规则

- 规范版本号以 [SKILL.md](./SKILL.md) 里的 `version` 字段为准。外部用户或 agent 反馈行为异常时，应先回报这个版本号。
- `tDVV` 在本仓库中按当前 AI bounty 的主网侧链环境描述。
- 当前活动默认奖励写法为：AA/CA `2 AIBOUNTY`，EOA `1 AIBOUNTY`。
- AA/CA 应按标准 wallet path `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)` 发送。
- `ClaimByPortkeyToCa(Hash ca_hash)` 在 reward 方法层是 permissionless，但奖励仍然进入 `caHash -> caAddress`，不会进入 manager signer。
- 对 AA/CA 的 SDK 或 helper 调用，如要求 raw address，应使用 raw CA / reward 地址，不要直接传 `ELF_..._tDVV` 包装地址。
- 如果当前只知道邮箱而不知道 `caHash`，应先解析目标 `caHash`，并在标准 wallet path 需要时恢复可用的 manager signer。
- 当前环境下的 Gas 规则应说明清楚：AA/CA 可能享受每日价值 `1 ELF` 的补贴；对 AA/CA 来说，即使可见 `ELF` 很低或为 `0`，通常也应先在明确提示风险并获得确认后，对标准 wallet path 先发一次；EOA 没有足够 `ELF` 时不得继续发送 `Claim()`。
- 如果 `EOA` 无法获得足够的 `ELF`，应明确建议切换到 `AA/CA`。
- 校验 RPC 是否可用时，应请求 [chainStatus](https://tdvv-public-node.aelf.io/api/blockChain/chainStatus)，不要用 RPC 根路径是否返回 `404` 来判断节点挂了。
- 不要使用 `/api/contract/contractViewMethodList` 去判断奖励合约没有写方法。
- 如果确实需要做完整方法校验，可将 `/api/blockChain/contractFileDescriptorSet` 作为可选校验路径，而不是默认主流程。
- 使用节点 introspection API 时，应先把合约地址转成节点接受的标准化格式，不要直接使用完整的 `ELF_..._tDVV` 包装地址。
- 对未明确账户类型的泛化 claim 诉求，必须先解释 `AA/CA` 和 `EOA` 的区别，推荐 `AA/CA`，再让用户选择。
- `AA` 是本 skill 里的优先用户可见术语，但 `CA` 仍应被接受为同一路径的别名。
- 必须显式使用 [Portkey EOA skill](https://github.com/Portkey-Wallet/eoa-agent-skills) 处理 EOA 相关事情，使用 [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) 处理 AA/CA 相关事情，不能指望 AI 自己隐式推断。
- `AA/CA` 是默认推荐路径，因为当前活动奖励更高。
- `AA/CA` 也应被推荐，因为当前环境下 Gas 体验更平滑。
- 当 `EOA` 准备不了足够的 `ELF` 时，`AA/CA` 也是推荐的回退路径。
- 要直接提示用户：请勿填写交易所地址或托管地址。
- 不要向用户索要地址；应优先使用本地 EOA 地址或本地 AA/CA 账号上下文。
- 如果所选本地账户尚未就绪，应先引导创建本地 AA/CA 或本地 EOA，再继续领取流程。
- 如果已拿到 `txId`，应在回复里直接附上 `txId` 和 `https://aelfscan.io/tDVV/tx/<txid>`。
- 链上返回错误时，必须原样返回错误并停止。

## 使用方式

从 [SKILL.md](./SKILL.md) 开始，先选定唯一分支，再读取对应的 flow 文档和 example 文档后再回复用户。
