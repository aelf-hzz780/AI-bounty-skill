# Example: Portkey CA Claim

## User Input

- English: `I choose CA. My local account is ready. Continue the claim.`
- 中文: `我选 CA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `Portkey CA Claim`

## Must Ask Or Confirm

- whether the user wants to send `ClaimByPortkeyToCa(ca_hash)` after seeing the write summary

## Must Not Ask

- do not re-explain the basic `CA vs EOA` difference after the account type is already chosen
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
- show manager signer, `ca_hash`, contract, method, receiver semantics, signer source as local CA account, and `2 AIBOUNTY` current campaign reward
- ask for explicit confirmation before sending
