# Example: Portkey CA Claim

## User Input

`我选 CA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `Portkey CA Claim`

## Must Ask Or Confirm

- whether the user wants to send `ClaimByPortkeyToCa(ca_hash)` after seeing the write summary

## Must Not Ask

- `CA` 和 `EOA` 的基础区别已经选定后还重复解释一遍
- whether the reward should go to the manager address
- irrelevant EOA-only signer questions

## Must-Stop Conditions

- `ca_hash` cannot be resolved on `tDVV`
- signer is not a current manager
- CA path has already claimed
- guardian recovery is still pending

## Correct Output Shape

- identify the branch as Portkey CA
- tell the user not to fill exchange addresses
- show manager signer, `ca_hash`, contract, method, receiver semantics, signer source as local CA account, and `2 tokens` current campaign reward
- ask for explicit confirmation before sending
