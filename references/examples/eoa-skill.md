# Example: EOA Claim

## User Input

- English: `I choose EOA. My local account is ready. Continue the claim.`
- 中文: `我选 EOA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `EOA Claim`

## Must Ask Or Confirm

- whether the user wants to send `Claim()` after seeing the write summary

## Must Not Ask

- do not re-explain the basic `CA vs EOA` difference after the account type is already chosen
- `ca_hash`
- guardian information
- Portkey recovery questions

## Must-Stop Conditions

- signer turns out to be exchange-managed
- signer ownership is unclear
- address has already claimed

## Correct Output Shape

- identify the branch as EOA
- tell the user not to fill exchange addresses
- show signer, contract, method, receiver semantics, signer source as local EOA account, and `1 token` current campaign reward
- ask for explicit confirmation before sending
