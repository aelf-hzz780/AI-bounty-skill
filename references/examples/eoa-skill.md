# Example: EOA Claim

## User Input

`我选 EOA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `EOA Claim`

## Must Ask Or Confirm

- whether the user wants to send `Claim()` after seeing the write summary

## Must Not Ask

- `CA` 和 `EOA` 的基础区别已经选定后还重复解释一遍
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
