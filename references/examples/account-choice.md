# Example: Account Choice And Onboarding

## Example 1: Generic Claim Request

### User Input

- English: `help me claim`
- 中文: `帮我 Claim。`

### Agent Should Choose

- `Account Choice And Onboarding`

### Correct Output Shape

- explain `AA/CA vs EOA`
- explain that `AA` is the preferred user-facing term and `CA` is still accepted as an alias
- explain that `AA/CA` is more account-style and usually uses email / guardian / recovery semantics
- explain that `EOA` is the traditional mnemonic / private key wallet style
- explain that the current campaign reward is `2 AIBOUNTY` for `AA/CA` and `1 AIBOUNTY` for `EOA`
- explain that `AA/CA` has smoother gas experience in the current environment
- explain that `EOA` may fail with `Transaction fee not enough` if there is no `ELF`
- explain that if the user cannot get enough `ELF` for `EOA`, they should switch to `AA/CA`
- recommend `AA/CA`
- ask the user to choose `AA/CA` or `EOA`
- prepare to use `https://github.com/Portkey-Wallet/ca-agent-skills` for `AA/CA` or `https://github.com/Portkey-Wallet/eoa-agent-skills` for `EOA`

## Example 2: Generic Claim Request Without Local Account

### User Input

- English: `help me claim, but I do not have a local account yet`
- 中文: `帮我领取，但我本地还没有账号。`

### Agent Should Choose

- `Account Choice And Onboarding`

### Correct Output Shape

- explain `AA/CA vs EOA`
- explain that `CA` is still accepted as an alias for the `AA/CA` route
- recommend `AA/CA`
- ask the user to choose `AA/CA` or `EOA`
- if the user chooses `AA`, `CA`, or `AA/CA`, guide them to create or recover a local Portkey AA/CA account first with `https://github.com/Portkey-Wallet/ca-agent-skills`
- if the user chooses `EOA`, guide them to create a local mnemonic / private key account first with `https://github.com/Portkey-Wallet/eoa-agent-skills`
- if the user chooses `EOA`, explain they still need enough `ELF` to pay gas before sending `Claim()`
- if the user chooses `EOA`, explain that failing to obtain `ELF` should trigger a switch back to `AA/CA`
- do not jump directly into claim execution
