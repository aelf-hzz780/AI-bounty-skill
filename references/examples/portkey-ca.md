# Example: Portkey AA/CA Claim

## User Input

- English: `I choose CA. My local account is ready. Continue the claim.`
- 中文: `我选 CA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `Portkey AA/CA Claim`

## Must Ask Or Confirm

- whether the user wants to send `ManagerForwardCall(...) -> ClaimByPortkeyToCa(caHash)` after seeing the write summary

## Must Not Ask

- do not re-explain the basic `AA/CA vs EOA` difference after the account type is already chosen
- whether the reward should go to the manager address
- irrelevant EOA-only signer questions

## Must-Stop Conditions

- `caHash` cannot be resolved on `tDVV`
- AA/CA path has already claimed
- guardian recovery is still pending and `caHash` cannot be resolved

## Correct Output Shape

- identify the branch as Portkey AA/CA
- explain that user input saying `CA` still maps to the `AA/CA` branch
- tell the user not to fill exchange addresses
- show manager signer, `caHash`, CA raw contract, reward raw contract, method chain `ManagerForwardCall -> ClaimByPortkeyToCa`, receiver semantics, signer source as local AA/CA context or recovery/login result, and `2 AIBOUNTY` current campaign reward
- ask for explicit confirmation before sending

## Forwarded Claim Through CA Contract

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

console.log(result);
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

## Forwarded Call Notes

- The standard AA/CA wallet path in this skill is `ManagerForwardCall(...) -> ClaimByPortkeyToCa(Hash ca_hash)`.
- `ClaimByPortkeyToCa` is permissionless at the reward method layer, but the reward still goes to the AA/CA address resolved from `caHash`, not to the manager signer.
- `ClaimByPortkeyToCa` does not take a plain string. It expects `.aelf.Hash`, so encode `caHash` as `Hash.value`.
- The observed protobuf bytes for the successful `Hash` payload were:

```hex
0a20efe80a13d643af3a55b4e821ef4ef331cf79d20a42b3700e5499c24387b1952f
```

- SDK calls should use raw addresses when the helper expects raw addresses:
  - reward raw: `2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472`
  - CA raw: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`
- Use the manager signer from the local AA/CA context or recovery/login result, and verify it can act for the target holder before sending.
- Do not route the recommended AA/CA claim through deprecated `ClaimByPortkey(Hash)`.
- Do not pass wrapped addresses such as `ELF_2fc5..._tDVV` into helpers that expect raw addresses.
- When doing descriptor-based encoding, call `root.resolveAll()` before resolving the method or encoding params.
- A successful forwarded AA/CA claim should normally end with a mined transaction and events such as `VirtualTransactionCreated`, `PortkeyClaimedToCa`, and `Transferred`.
