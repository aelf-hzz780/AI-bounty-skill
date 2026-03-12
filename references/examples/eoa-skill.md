# Example: EOA Claim

## User Input

- English: `I choose EOA. My local account is ready. Continue the claim.`
- 中文: `我选 EOA，本地已经创建好了，继续帮我领取。`

## Agent Should Choose

- `EOA Claim`

## Must Ask Or Confirm

- whether the user wants to send `Claim()` after seeing the write summary

## Must Not Ask

- do not re-explain the basic `AA/CA vs EOA` difference after the account type is already chosen
- `ca_hash`
- guardian information
- Portkey recovery questions

## Must-Stop Conditions

- signer turns out to be exchange-managed
- signer ownership is unclear
- address has already claimed
- the local EOA does not have enough `ELF` to pay gas

## Correct Output Shape

- identify the branch as EOA
- tell the user not to fill exchange addresses
- show signer, contract, method, receiver semantics, signer source as local EOA account, and `1 AIBOUNTY` current campaign reward
- show the gas prerequisite clearly before sending
- ask for explicit confirmation before sending

## Example 2: EOA Has No ELF For Gas

### User Input

- English: `I chose EOA and sent the claim, but the new wallet has no ELF for gas`
- 中文: `我选了 EOA 去领取，但新钱包里没有 ELF 支付手续费`

### Agent Should Choose

- `Diagnostics And Stop`

### Correct Output Shape

- explain that the problem is insufficient transaction fee, not claim logic
- explain that `EOA` needs enough `ELF` before sending `Claim()`
- tell the user to get `ELF` transferred in before retrying
- if the user cannot get enough `ELF`, recommend switching to `AA/CA` because `AA/CA` gets `2 AIBOUNTY` in the current campaign and has a smoother gas experience
