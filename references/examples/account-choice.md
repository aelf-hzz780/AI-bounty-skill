# Example: Account Choice And Onboarding

## Example 1: Generic Claim Request

### User Input

- English: `help me claim`
- 中文: `帮我 Claim。`

### Agent Should Choose

- `Account Choice And Onboarding`

### Correct Output Shape

- explain that the current public bounty path is `AA/CA only`
- explain that `AA` is the preferred user-facing term and `CA` is still accepted as an alias
- explain that `AA/CA` is more account-style and usually uses email / guardian / recovery semantics
- explain that `AA/CA` claim now depends on `@portkey/ca-agent-skills >= 2.3.0`
- explain that the current campaign reward is `2 AIBOUNTY` for `AA/CA`
- explain that `AA/CA` has smoother gas experience in the current environment and can usually try one confirmed AA/CA claim before fee is treated as the blocker
- explain that `AA/CA` still needs manager sync and holder lookup to pass on `tDVV` before any write
- explain that the legacy EOA route is no longer exposed by this public skill
- ask the user whether local Portkey AA/CA is already ready on `tDVV`, or whether recovery/login with `email` is needed
- prepare to use `https://github.com/Portkey-Wallet/ca-agent-skills` for AA/CA onboarding and claim

## Example 2: Generic Claim Request Without Local Account

### User Input

- English: `help me claim, but I do not have a local account yet`
- 中文: `帮我领取，但我本地还没有账号。`

### Agent Should Choose

- `Account Choice And Onboarding`

### Correct Output Shape

- explain that the current public bounty path is `AA/CA only`
- explain that `CA` is still accepted as an alias for the `AA/CA` route
- ask whether local Portkey AA/CA is already ready, or whether the user wants to recover/login first with `email`
- guide them to create or recover a local Portkey AA/CA account first with `https://github.com/Portkey-Wallet/ca-agent-skills` at version `>= 2.3.0`
- explain that claim will later check manager sync and holder readiness on `tDVV` before any write
- do not jump directly into claim execution
