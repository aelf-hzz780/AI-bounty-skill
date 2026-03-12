# Example: Portkey AA/CA Claim

## User Input

- English: `I choose CA. My local account is ready. Continue the claim.`
- 中文: `我选 CA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `Portkey AA/CA Claim`

## Must Ask Or Confirm

- whether the user wants to send `ClaimByPortkeyToCa(caHash)` after seeing the write summary

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
- show gas payer or relayer, `caHash`, reward raw contract, method `ClaimByPortkeyToCa`, receiver semantics, signer source as local gas payer or relayer, and `2 AIBOUNTY` current campaign reward
- ask for explicit confirmation before sending

## Direct Reward Contract Call

```ts
import { getContractInstance, getWalletByPrivateKey } from "./lib/aelf-client.ts";

const privateKey = process.env.PRIVATE_KEY!;
const caHash =
  "efe80a13d643af3a55b4e821ef4ef331cf79d20a42b3700e5499c24387b1952f";

const wallet = getWalletByPrivateKey(privateKey);
const rewardContract = await getContractInstance(
  "https://tdvv-public-node.aelf.io",
  "2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472",
  wallet
);

const result = await rewardContract.ClaimByPortkeyToCa({
  value: Buffer.from(caHash, "hex"),
});

console.log(result.TransactionId);
```

## Direct Call Notes

- `ClaimByPortkeyToCa` is permissionless. Any gas payer or relayer can call it if the `caHash` is valid.
- `ClaimByPortkeyToCa` does not take a plain string. It expects `.aelf.Hash`, so encode `caHash` as `Hash.value`.
- The observed protobuf bytes for the successful `Hash` payload were:

```hex
0a20efe80a13d643af3a55b4e821ef4ef331cf79d20a42b3700e5499c24387b1952f
```

- SDK calls should use the reward raw address when the helper expects raw addresses:
  - reward raw: `2fc5uPpboX9K9e9NTiDHxhCcgP8T9nV28BLyK8rDu8JmDpn472`
- The CA raw address is needed only if you separately query holder info:
  - CA raw: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`
- Do not route the recommended AA/CA claim through deprecated `ClaimByPortkey` or Portkey CA `ManagerForwardCall`.
- Do not pass wrapped addresses such as `ELF_2fc5..._tDVV` into helpers that expect raw addresses.
- When doing descriptor-based encoding, call `root.resolveAll()` before resolving the method or encoding params.
- A successful AA/CA claim should normally end with a mined transaction and events such as `PortkeyClaimedToCa` and `Transferred`.
