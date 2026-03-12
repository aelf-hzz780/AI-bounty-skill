# Example: Account Choice And Onboarding

## Example 1: Generic Claim Request

### User Input

`帮我 Claim。`

### Agent Should Choose

- `Account Choice And Onboarding`

### Correct Output Shape

- explain `CA vs EOA`
- explain that `CA` is more account-style and usually uses email / guardian / recovery semantics
- explain that `EOA` is the traditional mnemonic / private key wallet style
- explain that the current campaign reward is `2 tokens` for `CA` and `1 token` for `EOA`
- recommend `CA`
- ask the user to choose `CA` or `EOA`

## Example 2: Generic Claim Request Without Local Account

### User Input

`帮我领取，但我本地还没有账号。`

### Agent Should Choose

- `Account Choice And Onboarding`

### Correct Output Shape

- explain `CA vs EOA`
- recommend `CA`
- ask the user to choose `CA` or `EOA`
- if the user chooses `CA`, guide them to create or recover a local Portkey CA account first
- if the user chooses `EOA`, guide them to create a local mnemonic / private key account first
- do not jump directly into claim execution
