# Example: Portkey AA/CA Claim

## User Input

- English: `I choose CA. My local account is ready. Continue the claim.`
- 中文: `我选 CA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `Portkey AA/CA Claim`

## Must Ask Or Confirm

- whether the user wants to send `ManagerForwardCall(...) -> ClaimByPortkeyToCa(caHash)` after seeing the write summary

## Must Not Ask

- do not re-explain a removed `AA/CA vs EOA` choice after the AA/CA route is already established
- whether the reward should go to the manager address
- irrelevant legacy EOA-only signer questions

## Must-Stop Conditions

- `caHash` cannot be resolved on `tDVV`
- `manager-sync-status` returns `isManagerSynced=false`
- `GetHolderInfo(caHash)` cannot resolve the holder on `tDVV`
- AA/CA path has already claimed
- guardian recovery is still pending and `caHash` cannot be resolved

## Correct Output Shape

- identify the branch as Portkey AA/CA
- explain that user input saying `CA` still maps to the `AA/CA` branch
- tell the user not to fill exchange addresses
- show manager signer, manager sync status, holder lookup status, `caHash`, CA raw contract, reward raw contract, method chain `ManagerForwardCall -> ClaimByPortkeyToCa`, receiver semantics, signer source as local AA/CA context or recovery/login result, and `2 AIBOUNTY` current campaign reward
- if visible `ELF` is low or zero, explain that the current environment may still provide daily gas subsidy and the first AA/CA attempt can still be tried after confirmation
- ask for explicit confirmation before sending
- after sending, return `txId` and `https://aelfscan.io/tDVV/tx/<txid>`

## Suggested Final Reply Templates (Chinese)

### Pre-Send Template

```text
已进入 AA/CA Claim。

前置检查已完成：
- manager sync: `isManagerSynced=true`
- holder lookup: `GetHolderInfo(caHash)` 已在 tDVV 解析成功

写入摘要：
- path: `ManagerForwardCall -> ClaimByPortkeyToCa`
- signer: {manager_signer}
- caHash: {ca_hash}
- reward contract: {reward_contract}
- CA contract: {ca_contract}
- receiver: 奖励会进入该 AA/CA 地址，不会进入 manager signer
- reward: 2 AIBOUNTY

Gas 提示：
当前地址可见 ELF 余额较低或为 0，但当前环境可能存在每日 gas 补贴，所以在前置检查通过后，可以先尝试发送一次；
只有当链上最终返回 `Transaction fee not enough` 时，才将其视为手续费不足问题。

如果确认发送，我就继续。
```

### Preflight Stop Template: New `tDVV` Account Not Ready

```text
当前还不能进入 AA/CA Claim 写入阶段。

原因：
- `manager-sync-status` 或 `GetHolderInfo(caHash)` 还没有在 tDVV 上 ready

判断：
这更像是新注册 tDVV 账号的链上上下文仍在同步，不是手续费问题。

下一步建议：
- 先等待链上上下文继续同步
- 稍后重新尝试 claim

这一步不应提示补 ELF，也不应把它解释成 holder 查询需要手续费。
```

### Preflight Stop Template: Old `AELF` Account Syncing To `tDVV`

```text
当前还不能进入 AA/CA Claim 写入阶段。

原因：
- `manager-sync-status` 返回 `isManagerSynced=false`

判断：
这通常表示老账号从 `AELF` 同步到 `tDVV` sidechain 还没完成。

下一步建议：
- 先等待 manager / holder 同步完成
- 如本地上下文可能过期，可稍后重新登录或恢复后再试

这不是手续费问题，所以这里不应提示补 ELF。
```

### Post-Send Template

```text
交易已发出。
- txId: {tx_id}
- 查询链接: https://aelfscan.io/tDVV/tx/{tx_id}

如果短时间内看到 `NOTEXISTED`，这只表示链上还在确认，不代表失败。
我会继续以最终链上结果为准。
```

### Fee-Error Template

```text
本次 AA/CA 交易最终失败，链上返回：`Transaction fee not enough`。

这表示当前是交易发送阶段的手续费不足问题，不是 claim 逻辑问题，也不是 holder 查询问题。
由于当前环境可能存在 gas 补贴，所以先尝试发送一次是合理的；但这次链上结果已经确认补贴或余额仍不足。

下一步建议：
- 等待下一次可用的 gas 补贴窗口后再试
- 或先给当前 signer 补一点 ELF，再重试

如果这笔交易已经有 txId，可直接查看：
https://aelfscan.io/tDVV/tx/{tx_id}
```

### Holder-Error Template

```text
本次 AA/CA 交易最终失败，链上返回：`Holder is not found`。

这表示当前是 holder / caHash / 目标链上下文尚未 ready 的问题，不是手续费问题。

下一步建议：
- 等待 tDVV 上下文继续同步
- 或重新核对 `caHash` 与目标链是否正确

这里不应提示补 ELF。
```

### Manager-Error Template

```text
本次 AA/CA 交易最终失败，链上返回：`Sender is not a manager of the CA holder.`。

这表示当前 manager 在 tDVV 还没 ready，或者当前本地 manager 上下文不是目标 holder 的有效 manager。

下一步建议：
- 重新登录或恢复本地 AA/CA 上下文
- 或等待 manager 同步完成后再试

这里不应优先把问题解释成手续费不足。
```

## Optimized Claim Through CA Contract

```ts
import { getConfig } from "./lib/config.ts";
import { managerForwardCallWithKey } from "./src/core/contract.ts";

const privateKey = process.env.PRIVATE_KEY!;
const caHash =
  "efe80a13d643af3a55b4e821ef4ef331cf79d20a42b3700e5499c24387b1952f";
const contractAddress = "2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472";

const result = await managerForwardCallWithKey(
  getConfig({ network: "mainnet" }),
  privateKey,
  {
    chainId: "tDVV",
    caHash,
    contractAddress,
    methodName: "ClaimByPortkeyToCa",
    args: {
      value: Buffer.from(caHash, "hex"),
    },
  }
);

console.log(result.TransactionId);
```

## Low-Level Equivalent

```ts
import { getConfig } from "./lib/config.ts";
import { getChainInfoByChainId } from "./src/core/account.ts";
import {
  getWalletByPrivateKey,
  getContractInstance,
  encodeManagerForwardCallParams,
} from "./lib/aelf-client.ts";

const config = getConfig({ network: "mainnet" });
const chainInfo = getChainInfoByChainId(config, "tDVV");
const wallet = getWalletByPrivateKey(process.env.PRIVATE_KEY!);

const caHash =
  "efe80a13d643af3a55b4e821ef4ef331cf79d20a42b3700e5499c24387b1952f";

const payload = await encodeManagerForwardCallParams(chainInfo.endPoint, {
  caHash,
  contractAddress: "2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472",
  methodName: "ClaimByPortkeyToCa",
  args: {
    value: Buffer.from(caHash, "hex"),
  },
});

const caContract = await getContractInstance(
  chainInfo.endPoint,
  chainInfo.caContractAddress,
  wallet
);

const sendResult = await caContract.ManagerForwardCall(payload);
console.log(sendResult.TransactionId);
```

## AA/CA Call Notes

- The standard AA/CA wallet path in this skill is `ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`.
- Before any AA/CA send, first resolve local context and `caHash`, then run `manager-sync-status`, then run `GetHolderInfo(caHash)`.
- `ClaimByPortkeyToCa` is permissionless at the reward method layer, but the reward still goes to the AA/CA address resolved from `caHash`, not to the manager signer.
- `ClaimByPortkeyToCa` does not take a plain string. It expects `.aelf.Hash`, so encode `caHash` as `Hash.value`.
- The observed protobuf bytes for the successful `Hash` payload were:

```hex
0a20efe80a13d643af3a55b4e821ef4ef331cf79d20a42b3700e5499c24387b1952f
```

- A local signer with zero visible `ELF` can still be tried once if the current environment subsidy may cover gas; only treat fee as the blocker after an actual final chain error such as `Transaction fee not enough`.
- `Holder is not found` and `CA holder not found` are context or sync problems, not fee problems.
- SDK calls should use raw addresses when the helper expects raw addresses:
  - reward raw: `2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472`
  - CA raw: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`
- Use the manager signer from the local AA/CA context or recovery/login result, and verify it can act for the target holder before sending.
- Do not route the recommended AA/CA claim through deprecated `ClaimByPortkey(Hash)`.
- Do not pass wrapped addresses such as `ELF_2fc5..._tDVV` into helpers that expect raw addresses.
- When doing descriptor-based encoding, call `root.resolveAll()` before resolving the method or encoding params.
- A successful forwarded AA/CA claim should normally end with a mined transaction and events such as `VirtualTransactionCreated`, `PortkeyClaimedToCa`, and `Transferred`.
