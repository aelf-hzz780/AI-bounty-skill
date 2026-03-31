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

## 宿主快速入口

只有当入口真正对应宿主原生的安装或发现机制时，才单独给它命名。当前这个仓库里，`Gemini CLI` 是唯一真正的一条命令安装；`OpenClaw` 和 `OpenCode` 走原生 workspace skill 发现；`Claude Code` 和 `Codex` 先走 bootstrap 安装文档。

### Gemini CLI

这个仓库现在带有 `gemini-extension.json` 和 `GEMINI.md`，所以 Gemini CLI 可以直接把它当扩展安装：

```bash
gemini extensions install https://github.com/aelf-hzz780/AI-bounty-skill
```

### OpenClaw

这个仓库现在带有 workspace skill：`skills/ai-bounty-claim/SKILL.md`。把这个仓库作为 OpenClaw workspace 打开，并开启一个新 session 即可。

### OpenCode

这个仓库现在带有 project skill：`.opencode/skills/ai-bounty-claim/SKILL.md`。在 OpenCode 里打开这个仓库，并开启一个新 session 即可。

### Claude Code

把下面这句直接发给 Claude Code：

```text
Fetch and follow instructions from https://raw.githubusercontent.com/aelf-hzz780/AI-bounty-skill/main/.claude/INSTALL.md
```

### Codex

把下面这句直接发给 Codex：

```text
Fetch and follow instructions from https://raw.githubusercontent.com/aelf-hzz780/AI-bounty-skill/main/.codex/INSTALL.md
```

### Cursor

暂时还没有专用快速入口。先用上面的通用 Prompt；等这个仓库补上 `.cursor/rules` 或真实的 MCP 安装路径后，再增加 Cursor 专属入口。

这是一个公开 skill 仓库，用于在 `aelf` 的 `tDVV` 主网侧链环境中，通过 `RewardClaimContract` 领取 AI bounty。

对 AA/CA 来说，当前 skill 的标准 wallet 路径是 `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)`。

## 兼容性

- 本仓库 skill 版本：`3.0.0`
- AA/CA 依赖：`@portkey/ca-agent-skills >= 2.3.0`
  - 必需接口：CLI `manager-sync-status`、MCP `portkey_manager_sync_status`
- 发布状态：`@portkey/ca-agent-skills 2.3.0` 已正式发布，因此本仓库的 AI-bounty `3.0.0` 不再把 CA 发布时间当作 blocker

## Change Log

### `3.0.0`

- Breaking change：这个公开 skill 不再支持 legacy EOA claim 路径。
- 已移除：EOA onboarding 引导、EOA reward 文案、EOA gas 建议，以及专用的 EOA flow/example 文件。
- 当前公开领取路径统一为 AA/CA：`ManagerForwardCall -> ClaimByPortkeyToCa`。
- 即使旧版 reward contract 源码快照里还能看到 legacy EOA 方法，本仓库也不再把它们当作支持中的用户路径。

## 仓库结构

仓库采用“单入口 skill + 分支参考文档”的结构，方便较弱的 agent 一次只跟随一条路径。

- [SKILL.md](./SKILL.md)：onboarding、路由规则、硬性停止条件、共享默认值
- [references/flows/](./references/flows)：每条路径的 step-by-step 指令
- [references/examples/](./references/examples)：给较弱模型使用的分支示例

## 支持的方法

当前 skill 只支持以下公开领取路径：

- `AA/CA`：`ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`

## 路由分支

- Account choice and onboarding
- Portkey AA/CA claim
- Diagnostics and stop

## 共享规则

- 规范版本号以 [SKILL.md](./SKILL.md) 里的 `version` 字段为准。外部用户或 agent 反馈行为异常时，应先回报这个版本号。
- `tDVV` 在本仓库中按当前 AI bounty 的主网侧链环境描述。
- 当前活动默认奖励写法为：AA/CA `2 AIBOUNTY`。
- AA/CA 应按标准 wallet path `manager signer -> CA.ManagerForwardCall -> reward.ClaimByPortkeyToCa(Hash ca_hash)` 发送。
- 任意 AA/CA 写入前，必须按固定只读门禁执行：`解析本地上下文和 caHash -> manager-sync-status / portkey_manager_sync_status -> GetHolderInfo(caHash) -> 显式确认后再发送`。
- 只有当 `isManagerSynced=true` 且 `GetHolderInfo(caHash)` 在 `tDVV` 可解析时，AA/CA 才允许继续写入。
- `ClaimByPortkeyToCa(Hash ca_hash)` 在 reward 方法层是 permissionless，但奖励仍然进入 `caHash -> caAddress`，不会进入 manager signer。
- 对 AA/CA 的 SDK 或 helper 调用，如要求 raw address，应使用 raw CA / reward 地址，不要直接传 `ELF_..._tDVV` 包装地址。
- 如果当前只知道邮箱而不知道 `caHash`，应先解析目标 `caHash`，并在标准 wallet path 需要时恢复可用的 manager signer。
- 当前环境下的 Gas 规则应说明清楚：AA/CA 可能享受每日价值 `1 ELF` 的补贴；对 AA/CA 来说，即使可见 `ELF` 很低或为 `0`，只要同步和 holder 门禁已通过，通常也应先在明确提示风险并获得确认后，对标准 wallet path 先发一次。
- `Holder is not found` 与 `CA holder not found` 必须归类为 holder、`caHash` 或目标链上下文尚未 ready，严禁据此推出手续费不足，更不能建议“先转 `0.01 ELF`”。
- `Sender is not a manager of the CA holder.` 必须归类为 manager 在 `tDVV` 尚未 ready，或本地 manager 上下文与目标 holder 不匹配。
- `Transaction fee not enough.` 只能解释为交易发送阶段的手续费问题，不能解释成 holder 查询问题。
- `NOTEXISTED` 只能表示 pending lookup，不能当作最终失败，也不能触发 fee 建议。
- 校验 RPC 是否可用时，应请求 [chainStatus](https://tdvv-public-node.aelf.io/api/blockChain/chainStatus)，不要用 RPC 根路径是否返回 `404` 来判断节点挂了。
- 不要使用 `/api/contract/contractViewMethodList` 去判断奖励合约没有写方法。
- 如果确实需要做完整方法校验，可将 `/api/blockChain/contractFileDescriptorSet` 作为可选校验路径，而不是默认主流程。
- 使用节点 introspection API 时，应先把合约地址转成节点接受的标准化格式，不要直接使用完整的 `ELF_..._tDVV` 包装地址。
- 对未明确账户类型的泛化 claim 诉求，必须先解释当前公开 bounty 路径已经收口为 `AA/CA only`，然后确认用户是已有本地 Portkey AA/CA 上下文，还是需要先用邮箱恢复 / 登录。
- `AA` 是本 skill 里的优先用户可见术语，但 `CA` 仍应被接受为同一路径的别名。
- 这个公开 skill 应把 AA/CA 视为唯一支持路径；即使旧版合约源码快照里还能看到 legacy EOA 方法，也不再对外暴露。
- 必须显式使用 [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) 处理 AA/CA 相关事情，不能指望 AI 自己隐式推断。
- AA/CA 路径应要求 [Portkey CA skill](https://github.com/Portkey-Wallet/ca-agent-skills) `>= 2.3.0`。
- 对新注册的 `tDVV` AA/CA 账号，应强调“链上上下文正在同步，请稍后重试”。
- 对 origin 在 `AELF`、目标写链在 `tDVV` 的老账号，应强调“跨链同步到 sidechain 可能更慢，请先等待 manager/holder 同步完成”。
- 对 `Transaction fee not enough`，应明确这是“交易发送阶段的手续费问题”，不是 holder 查询问题。
- 要直接提示用户：请勿填写交易所地址或托管地址。
- 不要向用户索要地址；应优先使用本地 AA/CA 账号上下文。
- 如果本地 AA/CA 上下文尚未就绪，应先引导创建、恢复或登录本地 AA/CA，再继续领取流程。
- 如果已拿到 `txId`，应在回复里直接附上 `txId` 和 `https://aelfscan.io/tDVV/tx/<txid>`。
- 链上返回错误时，必须原样返回错误并停止。

## 使用方式

从 [SKILL.md](./SKILL.md) 开始，先选定唯一分支，再读取对应的 flow 文档和 example 文档后再回复用户。
